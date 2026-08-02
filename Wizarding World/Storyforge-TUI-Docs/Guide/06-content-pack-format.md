# 06: Load and validate content packs

## Result

The TUI will stop hard-coding the first scene. `storyforge-content` will load a small campaign pack from disk, validate the manifest and scene references, and give the TUI enough data to render the active scene.

This guide does not build the whole content system from the game sheet. It only covers the first useful slice:

```text
campaign.toml -> scene .ron files -> LoadedCampaign -> validation diagnostics -> TUI reads active scene
```

Guide 05 created the engine command/event path. Guide 06 makes the available scene choices come from content instead of a temporary number in `App::visible_choice_count`.

## Flow To Prove

The flow uses these functions and types:

```text
load_campaign
    -> load_scene
    -> validate_campaign
    -> LoadedCampaign::scene
    -> App::visible_choice_count
    -> ui::render_story_panel / render_actions_panel
```

First prove files are read. Then prove parsed data reaches Rust structs. Then validate references. Then connect the TUI.

---

File: `crates/storyforge-content/Cargo.toml`

## Step 1: Add parsing and walking dependencies

### Current state

`storyforge-content` currently depends on:

```toml
[dependencies]
serde.workspace = true
storyforge-core = { path = "../storyforge-core" }
thiserror.workspace = true
```

Its `lib.rs` only exposes `schema_version()`.

### Why this step matters

The content crate owns file formats. It needs TOML for the campaign manifest, RON for authored records, and `walkdir` to find scene files in stable order.

### Before

Use the current dependency block shown above.

### What to change

Add `ron`, `toml`, `sha2`, and `walkdir`.

### Temporary MVP / debug behavior

Do not compute the fingerprint first. Load and validate content before adding hashing. A fingerprint is only useful after the file list is stable.

### After

```toml
[dependencies]
ron = "0.10"
serde.workspace = true
sha2 = "0.10"
storyforge-core = { path = "../storyforge-core" }
thiserror.workspace = true
toml = "0.9"
walkdir = "2"
```

Or run:

```powershell
cargo add -p storyforge-content ron toml sha2 walkdir
```

### Learning checkpoint

You should understand that parsing dependencies belong in `storyforge-content`, not `storyforge-core`. Core should know rules and state, not where files live.

### How to verify

Run:

```powershell
cargo check -p storyforge-content
```

### Next connection

Now define the Rust structs that parsed files should become.

---

File: `crates/storyforge-content/src/model.rs`

## Step 2: Define the first content model

### Current state

There is no `model.rs`. The content crate does not yet have manifest or scene structs.

### Why this step matters

Serde needs concrete Rust types to deserialize into. These types also document what the current content slice supports.

### Before

Create:

```text
crates/storyforge-content/src/model.rs
```

### What to change

Add a campaign manifest, scene definition, and choice definition.

### Temporary MVP / debug behavior

Keep the model intentionally small. A scene only needs `id`, `title`, `body`, `choices`, and `terminal` right now. Conditions, effects, speakers, art, and checks come later.

### After

```rust
use serde::Deserialize;
use storyforge_core::ContentId;

#[derive(Debug, Clone, Deserialize)]
#[serde(rename_all = "kebab-case")]
pub struct CampaignManifest {
    pub id: String,
    pub name: String,
    pub version: String,
    pub schema_version: u32,
    pub rules_profile: String,
    pub default_locale: String,
    pub entry_scene: ContentId,
    pub starting_arc: String,
}

#[derive(Debug, Clone, Deserialize)]
pub struct SceneDefinition {
    pub id: ContentId,
    pub title: String,
    pub body: Vec<String>,
    pub choices: Vec<ChoiceDefinition>,
    pub terminal: bool,
}

#[derive(Debug, Clone, Deserialize)]
pub struct ChoiceDefinition {
    pub id: String,
    pub label: String,
    pub target: Option<ContentId>,
}
```

### Learning checkpoint

You should understand why cross-record references use `ContentId`, while the manifest pack `id` can stay a plain string for now. The immediate risk is broken references between scene records.

### How to verify

Compilation waits until `lib.rs` exports this module.

### Next connection

Create typed errors so file and parse failures point to the correct path.

---

File: `crates/storyforge-content/src/error.rs`

## Step 3: Add content errors and diagnostics

### Current state

There is no typed content error. If loading fails later, errors need path context.

### Why this step matters

Content authoring is a workflow. A useful validator should tell you which file is broken and why, not just say parsing failed.

### Before

Create:

```text
crates/storyforge-content/src/error.rs
```

### What to change

Add `ContentError`, `Diagnostic`, and `Severity`.

### Temporary MVP / debug behavior

Diagnostics can hold path, code, and message. Do not build a rich reporter yet. Stable diagnostic codes are enough for tests.

### After

```rust
use std::{io, path::PathBuf};

#[derive(Debug, thiserror::Error)]
pub enum ContentError {
    #[error("could not read `{path}`: {source}")]
    Read {
        path: PathBuf,
        #[source]
        source: io::Error,
    },
    #[error("invalid campaign manifest `{path}`: {source}")]
    Manifest {
        path: PathBuf,
        #[source]
        source: toml::de::Error,
    },
    #[error("invalid scene `{path}`: {source}")]
    Scene {
        path: PathBuf,
        #[source]
        source: Box<ron::error::SpannedError>,
    },
    #[error("campaign validation failed with {0} error(s)")]
    Validation(usize),
}

#[derive(Debug, Clone, PartialEq, Eq)]
pub struct Diagnostic {
    pub severity: Severity,
    pub code: &'static str,
    pub message: String,
    pub path: Option<PathBuf>,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum Severity {
    Warning,
    Error,
}
```

### Learning checkpoint

You should understand why parse errors are returned as `Result`, while validation can return a list of diagnostics. Parsing may stop at invalid syntax, but validation should report several authored mistakes at once.

### How to verify

Compilation waits until `lib.rs` exports this module.

### Next connection

Now load the manifest and all scene files.

---

File: `crates/storyforge-content/src/loader.rs`

## Step 4: Load a campaign from disk

### Current state

There is no loader. The TUI cannot ask content for a scene.

### Why this step matters

The game sheet says content lives outside the engine. This loader is the first step toward that boundary.

### Before

Create:

```text
crates/storyforge-content/src/loader.rs
```

### What to change

Add `LoadedCampaign`, `load_campaign`, and `load_scene`.

### Temporary MVP / debug behavior

First log or test the number of scenes loaded. Do not connect the TUI until you know file data reaches `LoadedCampaign`.

### After

```rust
use std::{
    fs,
    path::{Path, PathBuf},
};

use crate::{CampaignManifest, ContentError, SceneDefinition};

#[derive(Debug)]
pub struct LoadedCampaign {
    pub root: PathBuf,
    pub manifest: CampaignManifest,
    pub scenes: Vec<SceneDefinition>,
}

impl LoadedCampaign {
    #[must_use]
    pub fn scene(&self, id: &storyforge_core::ContentId) -> Option<&SceneDefinition> {
        self.scenes.iter().find(|scene| &scene.id == id)
    }
}

pub fn load_campaign(root: &Path) -> Result<LoadedCampaign, ContentError> {
    let manifest_path = root.join("campaign.toml");
    let manifest_text = fs::read_to_string(&manifest_path).map_err(|source| {
        ContentError::Read {
            path: manifest_path.clone(),
            source,
        }
    })?;
    let manifest = toml::from_str(&manifest_text).map_err(|source| ContentError::Manifest {
        path: manifest_path,
        source,
    })?;

    let scene_dir = root.join("scenes");
    let mut paths = walkdir::WalkDir::new(&scene_dir)
        .min_depth(1)
        .max_depth(8)
        .into_iter()
        .filter_map(Result::ok)
        .filter(|entry| entry.file_type().is_file())
        .map(walkdir::DirEntry::into_path)
        .filter(|path| path.extension().is_some_and(|extension| extension == "ron"))
        .collect::<Vec<_>>();
    paths.sort();

    let scenes = paths
        .iter()
        .map(|path| load_scene(path))
        .collect::<Result<Vec<_>, _>>()?;

    Ok(LoadedCampaign {
        root: root.to_path_buf(),
        manifest,
        scenes,
    })
}

fn load_scene(path: &Path) -> Result<SceneDefinition, ContentError> {
    let text = fs::read_to_string(path).map_err(|source| ContentError::Read {
        path: path.to_path_buf(),
        source,
    })?;

    ron::from_str(&text).map_err(|source| ContentError::Scene {
        path: path.to_path_buf(),
        source: Box::new(source),
    })
}
```

### Learning checkpoint

You should understand why paths are sorted. Validation output, fingerprints, and tests should be stable across operating systems.

### How to verify

Compilation waits until `lib.rs` exports this module and the campaign files exist.

### Next connection

Create the first campaign files for the loader to read.

---

File: `campaigns/academy-demo/campaign.toml`

## Step 5: Create the demo campaign manifest

### Current state

There is no `campaigns/academy-demo` directory yet.

### Why this step matters

A manifest gives the pack a stable identity and tells the app which scene starts the game.

### Before

Create this directory:

```text
campaigns/academy-demo/
```

### What to change

Create `campaign.toml` with the minimum fields from `CampaignManifest`.

### Temporary MVP / debug behavior

Use an original public demo name. Private campaign names belong in the separate private pack, not this repo.

### After

```toml
id = "academy-demo"
name = "The Ashen Academy"
version = "0.1.0"
schema-version = 1
rules-profile = "storyforge-default"
default-locale = "en"
entry-scene = "academy.scene.arrival"
starting-arc = "reconstruction"
```

### Learning checkpoint

You should understand that `entry-scene` must match a scene file ID. The validator will check that soon.

### How to verify

No command yet. Create the scene files next.

### Next connection

The manifest points at `academy.scene.arrival`, so create that scene.

---

File: `campaigns/academy-demo/scenes/arrival.ron`

## Step 6: Create the first scene record

### Current state

The TUI currently has hard-coded story text. The content pack has no scene files.

### Why this step matters

This is the first authored scene that can be loaded, validated, and eventually rendered.

### Before

Create this directory:

```text
campaigns/academy-demo/scenes/
```

### What to change

Create `arrival.ron` with two choices. For the first validation pass, point both choices at a real terminal scene so missing targets do not block progress.

### Temporary MVP / debug behavior

Do not create a branching story yet. First prove one scene can target another scene.

### After

```ron
(
    id: "academy.scene.arrival",
    title: "The Rain Gate",
    body: [
        "Rain ticks against the glass roof of the arrival hall.",
        "Iren waits beside a sealed gate while the induction bells count down.",
    ],
    choices: [
        (
            id: "ask_gate",
            label: "Ask Iren what sealed the gate.",
            target: Some("academy.scene.debug-end"),
        ),
        (
            id: "search_route",
            label: "Search for another route.",
            target: Some("academy.scene.debug-end"),
        ),
    ],
    terminal: false,
)
```

### Learning checkpoint

You should understand the difference between a choice `id` and a choice `label`. The `id` is for saves/tests. The `label` is what the player reads.

### How to verify

Create the target scene next so validation has something to resolve.

### Next connection

Add a terminal debug scene to complete the tiny route.

---

File: `campaigns/academy-demo/scenes/debug-end.ron`

## Step 7: Add a terminal debug scene

### Current state

`arrival.ron` points to `academy.scene.debug-end`, but that scene does not exist yet.

### Why this step matters

The validator should reject missing scene targets. Creating this temporary end scene lets the first pack validate while keeping scope tiny.

### Before

Create:

```text
campaigns/academy-demo/scenes/debug-end.ron
```

### What to change

Add a terminal scene with no choices.

### Temporary MVP / debug behavior

This scene is a scaffold. Later guides replace it with actual branching scenes and character creation entry points.

### After

```ron
(
    id: "academy.scene.debug-end",
    title: "Debug End",
    body: [
        "This temporary scene proves that scene targets can resolve.",
    ],
    choices: [],
    terminal: true,
)
```

### Learning checkpoint

You should understand that terminal scenes may have no choices. Nonterminal scenes need at least one choice.

### How to verify

After the validator is written, this scene should prevent missing-target errors.

### Next connection

Now implement validation rules.

---

File: `crates/storyforge-content/src/validation.rs`

## Step 8: Validate manifest and scene references

### Current state

The loader can parse files, but it does not yet know whether the campaign is coherent.

### Why this step matters

Content should fail before play starts. A player should not discover a broken scene link by choosing it during a run.

### Before

Create:

```text
crates/storyforge-content/src/validation.rs
```

### What to change

Add a validator that returns a `Vec<Diagnostic>`.

### Temporary MVP / debug behavior

Start with these checks:

1. Schema version is supported.
2. Entry scene exists.
3. Scene IDs are unique.
4. Choice IDs are unique inside a scene.
5. Nonterminal scenes have choices.
6. Nonterminal choices have targets.
7. Every target exists.
8. Terminal scenes do not require outgoing targets.

### After

```rust
use std::collections::{BTreeMap, BTreeSet};

use crate::{Diagnostic, LoadedCampaign, Severity, schema_version};

#[must_use]
pub fn validate_campaign(campaign: &LoadedCampaign) -> Vec<Diagnostic> {
    let mut diagnostics = Vec::new();

    if campaign.manifest.schema_version != schema_version() {
        diagnostics.push(Diagnostic {
            severity: Severity::Error,
            code: "manifest.unsupported-schema",
            message: format!(
                "schema version {} is not supported",
                campaign.manifest.schema_version
            ),
            path: Some(campaign.root.join("campaign.toml")),
        });
    }

    let mut scene_counts: BTreeMap<&str, usize> = BTreeMap::new();
    for scene in &campaign.scenes {
        *scene_counts.entry(scene.id.as_str()).or_default() += 1;
    }

    for (id, count) in &scene_counts {
        if *count > 1 {
            diagnostics.push(Diagnostic {
                severity: Severity::Error,
                code: "scene.duplicate-id",
                message: format!("scene ID `{id}` appears {count} times"),
                path: None,
            });
        }
    }

    let scene_ids = scene_counts.keys().copied().collect::<BTreeSet<_>>();

    if !scene_ids.contains(campaign.manifest.entry_scene.as_str()) {
        diagnostics.push(Diagnostic {
            severity: Severity::Error,
            code: "manifest.missing-entry-scene",
            message: format!(
                "entry scene `{}` does not exist",
                campaign.manifest.entry_scene
            ),
            path: Some(campaign.root.join("campaign.toml")),
        });
    }

    for scene in &campaign.scenes {
        let mut choice_ids = BTreeSet::new();
        for choice in &scene.choices {
            if !choice_ids.insert(choice.id.as_str()) {
                diagnostics.push(Diagnostic {
                    severity: Severity::Error,
                    code: "scene.duplicate-choice-id",
                    message: format!(
                        "scene `{}` repeats choice ID `{}`",
                        scene.id, choice.id
                    ),
                    path: None,
                });
            }

            if !scene.terminal && choice.target.is_none() {
                diagnostics.push(Diagnostic {
                    severity: Severity::Error,
                    code: "scene.choice-missing-target",
                    message: format!(
                        "choice `{}` in scene `{}` has no target",
                        choice.id, scene.id
                    ),
                    path: None,
                });
            }

            if let Some(target) = &choice.target {
                if !scene_ids.contains(target.as_str()) {
                    diagnostics.push(Diagnostic {
                        severity: Severity::Error,
                        code: "scene.missing-target",
                        message: format!(
                            "choice `{}` in scene `{}` targets missing scene `{}`",
                            choice.id, scene.id, target
                        ),
                        path: None,
                    });
                }
            }
        }

        if !scene.terminal && scene.choices.is_empty() {
            diagnostics.push(Diagnostic {
                severity: Severity::Error,
                code: "scene.no-choices",
                message: format!("nonterminal scene `{}` has no choices", scene.id),
                path: None,
            });
        }
    }

    diagnostics.sort_by(|left, right| {
        left.path
            .cmp(&right.path)
            .then(left.code.cmp(right.code))
            .then(left.message.cmp(&right.message))
    });

    diagnostics
}
```

### Learning checkpoint

You should understand why validation collects all diagnostics instead of returning after the first one. Content authors need a repair list.

### How to verify

Compilation waits until `lib.rs` exports the module.

### Next connection

Expose the loader and validator from `storyforge-content`.

---

File: `crates/storyforge-content/src/lib.rs`

## Step 9: Export content loading and validation

### Current state

`lib.rs` currently contains:

```rust
//! Campaign loading and validation for Storyforge.

/// Returns the current content schema version.
#[must_use]
pub const fn schema_version() -> u32 {
    1
}
```

### Why this step matters

Other crates cannot use the loader, model, errors, or validator until they are exported.

### Before

Keep `schema_version()`.

### What to change

Declare the new modules and re-export the public types and functions.

### Temporary MVP / debug behavior

Expose only what the TUI and tests need now.

### After

```rust
//! Campaign loading and validation for Storyforge.

mod error;
mod loader;
mod model;
mod validation;

pub use error::{ContentError, Diagnostic, Severity};
pub use loader::{LoadedCampaign, load_campaign};
pub use model::{CampaignManifest, ChoiceDefinition, SceneDefinition};
pub use validation::validate_campaign;

/// Returns the current content schema version.
#[must_use]
pub const fn schema_version() -> u32 {
    1
}
```

### Learning checkpoint

You should understand that the content crate is now the public API for loading and validating packs. The TUI should not parse RON directly.

### How to verify

Run:

```powershell
cargo check -p storyforge-content
```

### Next connection

Add a CLI command so you can validate the pack without launching the dashboard.

---

File: `crates/storyforge-tui/Cargo.toml`

## Step 10: Add Clap for development commands

### Current state

The TUI launches immediately from `main.rs`. There is no `validate` command yet.

### Why this step matters

Validation should be runnable before play. It also gives you a fast feedback loop while editing content.

### Before

`storyforge-tui` dependencies currently include terminal crates and both local crates.

### What to change

Add Clap with derive support.

### Temporary MVP / debug behavior

Only implement `validate` and the default play behavior for now. `doctor` can remain a later guide.

### After

```toml
clap = { version = "4", features = ["derive"] }
```

Or run:

```powershell
cargo add -p storyforge-tui clap --features derive
```

### Learning checkpoint

You should understand that CLI parsing belongs to the binary crate, not core or content.

### How to verify

Run:

```powershell
cargo check -p storyforge-tui
```

### Next connection

Wire the CLI to `load_campaign` and `validate_campaign`.

---

File: `crates/storyforge-tui/src/main.rs`

## Step 11: Add a `validate` command

### Current state

`main.rs` currently starts the TUI directly:

```rust
fn main() -> Result<()> {
    color_eyre::install()?;
    ratatui::run(|terminal| App::default().run(terminal))?;
    Ok(())
}
```

### Why this step matters

You need a non-interactive way to check content. Validation should run in a normal terminal output mode, not inside the alternate-screen TUI.

### Before

Keep the existing `App::default().run(terminal)` path as the default play path.

### What to change

Add Clap structs and branch in `main`.

### Temporary MVP / debug behavior

Print each diagnostic as plain text. Return an error if any error-severity diagnostic exists.

### After

```rust
use std::path::PathBuf;

use clap::{Parser, Subcommand};
use color_eyre::{Result, eyre::eyre};
use storyforge_content::{Severity, load_campaign, validate_campaign};

use app::App;

#[derive(Debug, Parser)]
#[command(name = "storyforge", version, about)]
struct Cli {
    #[command(subcommand)]
    command: Option<Command>,
}

#[derive(Debug, Subcommand)]
enum Command {
    Validate {
        #[arg(long)]
        pack: PathBuf,
        #[arg(long)]
        strict: bool,
    },
}

fn main() -> Result<()> {
    color_eyre::install()?;
    let cli = Cli::parse();

    match cli.command {
        Some(Command::Validate { pack, strict }) => validate_pack(&pack, strict),
        None => {
            ratatui::run(|terminal| App::default().run(terminal))?;
            Ok(())
        }
    }
}

fn validate_pack(pack: &PathBuf, strict: bool) -> Result<()> {
    let campaign = load_campaign(pack)?;
    let diagnostics = validate_campaign(&campaign);

    for diagnostic in &diagnostics {
        println!("{:?} {}: {}", diagnostic.severity, diagnostic.code, diagnostic.message);
    }

    let error_count = diagnostics
        .iter()
        .filter(|diagnostic| diagnostic.severity == Severity::Error)
        .count();
    let warning_count = diagnostics.len().saturating_sub(error_count);

    println!(
        "{} {}: {} scenes, {} warnings, {} errors",
        campaign.manifest.id,
        campaign.manifest.version,
        campaign.scenes.len(),
        warning_count,
        error_count
    );

    if error_count > 0 || strict && warning_count > 0 {
        return Err(eyre!("campaign validation failed"));
    }

    Ok(())
}
```

### Learning checkpoint

You should understand why `validate_pack` returns `Result<()>` instead of calling `std::process::exit` inside the validator. Returning errors keeps the code testable and lets `color-eyre` format top-level failures.

### How to verify

Run:

```powershell
cargo run -p storyforge-tui -- validate --pack campaigns/academy-demo
```

Expected shape:

```text
academy-demo 0.1.0: 2 scenes, 0 warnings, 0 errors
```

The exact count should match the files you created.

### Next connection

Now replace the TUI's temporary choice count with the active loaded scene's choice count.

---

File: `crates/storyforge-tui/src/app.rs`

## Step 12: Store loaded campaign data in `App`

### Current state

Guide 05 used a temporary `visible_choice_count()` that returned `2`. The renderer still hard-codes story text and choices.

### Why this step matters

The app should ask content for the active scene. That is the first connection between the content pack, the engine's active scene ID, and the renderer.

### Before

`App` already stores `engine: GameEngine` after guide 05.

### What to change

Add an optional `LoadedCampaign` field and a constructor that loads the default pack. Keep `Default` as a fallback for tests if you want, but the play path should use the loading constructor.

### Temporary MVP / debug behavior

If loading fails during early development, return the error from `main` instead of silently falling back to placeholders.

### After

Add import:

```rust
use std::{io, path::Path};

use storyforge_content::{LoadedCampaign, load_campaign, validate_campaign};
```

Add field:

```rust
pub(crate) campaign: Option<LoadedCampaign>,
```

Add constructor:

```rust
impl App {
    /// Loads a campaign pack and starts the engine at its entry scene.
    ///
    /// # Errors
    ///
    /// Returns an error when the campaign cannot be read or has validation errors.
    pub fn load_pack(path: &Path) -> color_eyre::Result<Self> {
        let campaign = load_campaign(path)?;
        let diagnostics = validate_campaign(&campaign);
        let error_count = diagnostics
            .iter()
            .filter(|diagnostic| diagnostic.severity == storyforge_content::Severity::Error)
            .count();

        if error_count > 0 {
            return Err(color_eyre::eyre::eyre!("campaign validation failed"));
        }

        let state = GameState::new(campaign.manifest.entry_scene.clone());
        let mut app = Self::default();
        app.engine = GameEngine::new(state, 42);
        app.campaign = Some(campaign);
        Ok(app)
    }
}
```

Update `Default` to include:

```rust
campaign: None,
```

Change `visible_choice_count`:

```rust
fn active_scene(&self) -> Option<&storyforge_content::SceneDefinition> {
    self.campaign
        .as_ref()
        .and_then(|campaign| campaign.scene(&self.engine.state().active_scene))
}

fn visible_choice_count(&self) -> usize {
    self.active_scene()
        .map_or(0, |scene| scene.choices.len())
}
```

### Learning checkpoint

You should understand that core owns the active scene ID, but content owns the scene definition. The TUI joins those two pieces for rendering.

### How to verify

Run:

```powershell
cargo check -p storyforge-tui
```

### Next connection

Render the loaded scene title, body, and choices.

---

File: `crates/storyforge-tui/src/ui.rs`

## Step 13: Render the active scene from content

### Current state

`render_story_panel` and `render_actions_panel` still contain temporary hard-coded text.

### Why this step matters

This is the end-to-end proof that content data reaches the player:

```text
RON file -> SceneDefinition -> App::active_scene -> ui.rs
```

### Before

`render_story_panel` has text like:

```rust
Screen::Story => "The academy doors stand open. What do you do?",
```

`render_actions_panel` has hard-coded story choices from guide 05.

### What to change

For the Story screen, read `app.active_scene()`. If there is no campaign, keep a placeholder for tests.

### Temporary MVP / debug behavior

If `active_scene()` returns `None`, show `No active scene loaded.` Do not panic. This keeps tests and early startup failures readable.

### After

Make `active_scene` visible to `ui.rs`:

```rust
pub(crate) fn active_scene(&self) -> Option<&storyforge_content::SceneDefinition> {
    self.campaign
        .as_ref()
        .and_then(|campaign| campaign.scene(&self.engine.state().active_scene))
}
```

In `render_story_panel`, use:

```rust
let text = match app.screen {
    Screen::Story => app
        .active_scene()
        .map(|scene| scene.body.join("\n\n"))
        .unwrap_or_else(|| "No active scene loaded.".to_owned()),
    Screen::Character => "Character sheet will appear here.".to_owned(),
    Screen::Journal => "Journal entries will appear here.".to_owned(),
    Screen::Map => "Map will appear here.".to_owned(),
    Screen::Help => "Help text will appear here.".to_owned(),
};
```

In `render_actions_panel`, use:

```rust
let text = if app.screen == Screen::Story {
    if let Some(scene) = app.active_scene() {
        let selected = app.engine.state().selected_choice;
        let mut lines = Vec::new();
        lines.push(String::new());
        for (index, choice) in scene.choices.iter().enumerate() {
            let marker = if index == selected { ">" } else { " " };
            lines.push(format!("{marker} {}", choice.label));
        }
        lines.push(String::new());
        lines.push("[j/k] Choose  [Enter] Confirm".to_owned());
        lines.join("\n")
    } else {
        "\nNo choices loaded.".to_owned()
    }
} else {
    "\n[c] Character\n[l] Journal\n[m] Map\n[?] Help\n[Esc] Back/Quit".to_owned()
};
```

### Learning checkpoint

You should understand that this step displays content but still does not advance scenes. Pressing `Enter` to follow a choice is a later command after scene transitions are represented as events.

### How to verify

Run:

```powershell
cargo run -p storyforge-tui
```

You should see the `arrival.ron` body and choices.

### Next connection

Guide 07 uses the same content and command/event pattern to start character creation.

## Full Verification

Run:

```powershell
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features --locked -- -D warnings
cargo test --workspace --all-features --locked
cargo run -p storyforge-tui -- validate --pack campaigns/academy-demo
cargo run -p storyforge-tui
```

Manual checks:

1. Validation reports zero errors for `campaigns/academy-demo`.
2. The TUI starts at the manifest `entry-scene`.
3. Story text comes from `arrival.ron`.
4. Choices come from `arrival.ron`.
5. `j` and `k` move through the loaded choices.
6. Rendering still does not parse files or mutate state.

## Common Mistakes

- Parsing RON directly from `ui.rs`.
- Letting content use display names as references.
- Returning after the first validation error.
- Forgetting to sort loaded file paths.
- Letting a nonterminal scene have no choices.
- Creating private campaign names or licensed setting text inside the public demo pack.

## Acceptance Check

- `storyforge-content` owns manifest and scene parsing.
- Parse errors include file paths.
- Validation catches missing entry scenes and missing targets.
- The demo pack validates before play.
- The TUI reads the active scene by matching `GameState.active_scene` to loaded content.
- Hard-coded story body and choice labels are removed from the Story screen.
