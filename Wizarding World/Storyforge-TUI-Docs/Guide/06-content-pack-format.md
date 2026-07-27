# 06: Load and validate content packs

## Result

`storyforge validate --pack campaigns/academy-demo` will load a TOML manifest and RON scene records, resolve IDs, and print actionable errors before the game starts.

## Add dependencies

```powershell
cargo add -p storyforge-content ron toml sha2 walkdir
cargo add -p storyforge-content serde --features derive
cargo add -p storyforge-tui clap --features derive
```

## Campaign layout

Create:

```text
campaigns/academy-demo/
├── campaign.toml
├── art/
├── actors/
├── encounters/
├── endings/
├── factions/
├── items/
├── locations/
├── quests/
├── scenes/
│   └── arrival.ron
├── spells/
└── tests/
```

Empty category directories do not need to be committed until they contain data.

## Manifest

Create `campaign.toml`:

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

## Content structures

In `storyforge-content`, create `model.rs`:

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

The manifest may use a simple string ID because it names the pack itself. All cross-record references use `ContentId`.

## First scene

Create `scenes/arrival.ron`:

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
            target: Some("academy.scene.gate_question"),
        ),
        (
            id: "search_route",
            label: "Search for another route.",
            target: Some("academy.scene.side_passage"),
        ),
    ],
    terminal: false,
)
```

Create referenced scene files or change both choices to a temporary terminal scene that exists. The validator must not accept missing targets.

## Pack loader

Create `loader.rs`:

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

pub fn load_campaign(root: &Path) -> Result<LoadedCampaign, ContentError> {
    let manifest_path = root.join("campaign.toml");
    let manifest_text = fs::read_to_string(&manifest_path).map_err(|source| {
        ContentError::Read {
            path: manifest_path.clone(),
            source,
        }
    })?;
    let manifest = toml::from_str(&manifest_text).map_err(|source| {
        ContentError::Manifest {
            path: manifest_path,
            source,
        }
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

Boxing the RON error keeps `ContentError` variants closer in size. Confirm with Clippy instead of guessing about every error type.

## Typed errors

Create `error.rs`:

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
```

## Validation report

Use a report that can hold several errors at once:

```rust
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

Validation must check:

1. Manifest schema version is supported.
2. Entry scene exists.
3. Scene IDs are unique.
4. Choice IDs are unique inside a scene.
5. Nonterminal scenes have choices.
6. Every choice has a target.
7. Every target exists.
8. Terminal scenes have no required outgoing target.

Sort diagnostics by path, code, and message for stable output and tests.

## CLI

Use Clap subcommands:

```rust
#[derive(Debug, clap::Parser)]
#[command(name = "storyforge", version, about)]
struct Cli {
    #[command(subcommand)]
    command: Option<Command>,
}

#[derive(Debug, clap::Subcommand)]
enum Command {
    Play {
        #[arg(long, default_value = "campaigns/academy-demo")]
        pack: PathBuf,
    },
    Validate {
        #[arg(long)]
        pack: PathBuf,
        #[arg(long)]
        strict: bool,
    },
    Doctor,
}
```

No subcommand defaults to `Play` with the bundled demo in chapter 21. During development, use an explicit path.

`validate` prints all diagnostics and returns a nonzero process result when errors exist. Return an error from `main`; do not call `process::exit` from inside the validator.

## Fingerprint

Create a SHA-256 fingerprint from:

- Normalized manifest bytes.
- Relative content path.
- Content bytes in sorted path order.

Do not include modification timestamps or absolute paths. Store the lowercase hex result in saves.

## Tests

Create fixture directories under:

```text
crates/storyforge-content/tests/fixtures/
```

Include:

- `valid-minimal`
- `missing-entry`
- `duplicate-scene`
- `missing-target`

Each fixture has complete files for its intended case.

Name tests by behavior:

```rust
#[test]
fn validate_should_report_missing_choice_target() {
    let campaign = load_fixture("missing-target")
        .expect("fixture should parse before reference validation");
    let report = validate(&campaign);

    assert!(report.iter().any(|diagnostic| diagnostic.code == "scene.missing-target"));
}
```

## Verify

```powershell
cargo run -p storyforge-tui -- validate --pack campaigns/academy-demo
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features --locked -- -D warnings
cargo test --workspace --all-features --locked
```

Expected success summary:

```text
academy-demo 0.1.0: valid
2 scenes, 2 choices, 0 warnings, 0 errors
```

Use actual counts from the files you created.

## Common mistakes

- The loader depends on the current working directory inside library code.
- Diagnostics stop after the first missing target.
- Paths are not sorted, causing unstable fingerprints.
- Display names are used as references.
- Validation warnings silently become errors without `--strict`.

## Acceptance check

- The demo pack validates.
- Invalid fixtures produce the intended diagnostic codes.
- Output order is stable.
- The fingerprint is identical across two runs.
- Parse errors include their file path.

## Suggested commit

```powershell
git add .
git commit -m "Add versioned campaign loading and validation"
```
