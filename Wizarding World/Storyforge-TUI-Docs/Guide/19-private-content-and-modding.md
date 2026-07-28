# 19: Isolate private campaigns and support community mods

## Result

By the end of this stage, the engine can load more than one content pack without mixing private campaign material into the public repository. You will also have a repeatable process for turning campaign notes into validated game data.

This stage matters because the engine and its first public demo should be shareable, while your wizarding-world campaign remains a separate, private project.

## Keep the repositories separate

Use two repositories:

```text
storyforge-tui/
  crates/
  campaigns/
    academy-demo/
  docs/

wizarding-world-private/
  campaign.toml
  content/
  source-notes/
  provenance/
```

The public repository contains the engine, development tools, schemas, documentation, and an original academy demo. The private repository contains adapted session notes, player characters, campaign-specific factions, and any material whose license does not allow redistribution.

Do not copy private content into public test fixtures. Write small original fixtures that exercise the same engine behavior.

## Do not use Git history rewriting as the normal release process

Do not commit the private story to the public repository and plan to scrub it later. Deleting the working-tree files does not remove earlier commits. A history rewrite can remove known paths from one repository, but it cannot recall:

- A commit that another person cloned.
- A GitHub cache, pull-request ref, artifact, or fork.
- An npm tarball or native archive that was already published.
- A secret copied into an issue, log, or build service.
- An overlooked file stored under an unexpected name.

Tools such as `git filter-repo` are useful for incident cleanup, not as a privacy boundary. If private material ever reaches a public remote, remove accessible artifacts, rewrite the affected repository, rotate any exposed credentials, and assume that someone could have retained the content.

The safe rule is simpler: private story files and their Git objects never enter the public repository.

## Initialize the two repositories

From the directory that will contain both projects:

```powershell
New-Item -ItemType Directory -Path storyforge-tui
New-Item -ItemType Directory -Path wizarding-world-private
```

Initialize the public engine repository:

```powershell
Set-Location storyforge-tui
git init
git switch -c main
```

Initialize the private campaign separately:

```powershell
Set-Location ..\wizarding-world-private
git init
git switch -c main
```

Create the private remote through the Git hosting provider's private-repository flow. Before the first push, verify its visibility in the provider UI and invite only the people who should read the campaign.

```powershell
$privateRemote = Read-Host 'Paste the private repository URL'
git remote add origin $privateRemote
git remote -v
git push -u origin main
```

Use the exact URL created by the provider. Do not paste a token into the URL or commit it to a shell script.

Keep ordinary story history in the private repository:

```powershell
git add content source-notes provenance campaign.toml
git commit -m "Add the Lantern Row creation session"
git push
```

That history is useful. It shows when an NPC, clue, faction, or ending changed without increasing public-release risk.

## Why the private pack is not a submodule by default

A private Git submodule can work for a closed team, but it still commits `.gitmodules`, the private repository URL, and an exact private commit ID to the public repository. CI may also try to fetch it. That metadata is unnecessary for this project.

Use sibling repositories and an ignored local path. If a future team deliberately chooses a submodule, use a neutral repository name, audit `.gitmodules`, and configure public CI never to fetch private submodules.

## Add a private pack during development

Keep the private campaign next to the engine repository, not inside it:

```text
projects/
  storyforge-tui/
  wizarding-world-private/
```

Run the engine with an explicit pack path:

```powershell
cargo run -p storyforge-tui -- play --pack ..\wizarding-world-private
```

For convenience, create a local configuration file that Git ignores:

```toml
# .storyforge.local.toml

[development]
default_pack = "../wizarding-world-private"
```

Add it to the public repository's `.gitignore`:

```gitignore
.storyforge.local.toml
private-campaigns/
*.private.ron
```

Add one more defense for common local folder names:

```gitignore
wizarding-world-private/
campaigns-private/
```

Run this before the first public commit:

```powershell
git check-ignore -v .storyforge.local.toml
git status --short
```

Expected result: the local configuration is reported by `git check-ignore` and no private path appears in `git status`.

The application should never silently search a home directory for campaign packs. Explicit paths make builds and bug reports reproducible.

## Define pack identity and dependencies

Extend `campaign.toml`:

```toml
schema_version = 1
id = "wizarding-world-private"
name = "Wizarding World Private Campaign"
version = "0.1.0"
engine = ">=0.1.0, <0.2.0"
entry_scene = "arrival.platform"

[dependencies]
storyforge-core = "1"

[content]
characters = "content/characters"
factions = "content/factions"
items = "content/items"
locations = "content/locations"
quests = "content/quests"
scenes = "content/scenes"
spells = "content/spells"
```

Pack IDs are lowercase ASCII with hyphens. Content IDs use a namespace:

```text
wizarding-world-private.character.alex
wizarding-world-private.faction.night-court
storyforge-core.condition.burning
```

Namespaced IDs prevent two packs from accidentally defining the same object.

## Choose a predictable load order

Load content in this order:

1. Built-in engine definitions.
2. Dependencies declared by the selected campaign.
3. The selected campaign pack.
4. User-selected mods, in the order shown by `storyforge mods list`.

Dependencies are loaded before the pack that depends on them. If two independent mods define the same namespaced ID, validation fails. Do not use last-file-wins behavior because it hides mistakes.

Support an intentional override only when the mod declares it:

```toml
[[overrides]]
target = "academy-demo.item.healing-draught"
reason = "Rebalanced for the hard-mode ruleset"
```

An override must name its target and must not change the object's kind. An item may override an item, but it may not replace a scene.

## Create a pack scaffold command

Add this command:

```powershell
storyforge new-pack my-campaign --output ..\my-campaign
```

It should produce:

```text
my-campaign/
  campaign.toml
  README.md
  content/
    characters/
    encounters/
    factions/
    items/
    locations/
    quests/
    scenes/
    spells/
  provenance/
    SOURCES.md
  tests/
    playthroughs/
```

Generate a small valid scene and location so the author can immediately run:

```powershell
storyforge validate --pack ..\my-campaign
storyforge play --pack ..\my-campaign
```

## Turn source notes into content

Do not make the game read PDFs, spreadsheets, or an Obsidian vault at runtime. Those are authoring sources, not stable game formats.

Use this authoring loop:

1. Pick one bounded source, such as a location note or one session recap.
2. Extract facts into a temporary content worksheet.
3. Decide which facts are canon for the game adaptation.
4. Assign stable IDs before writing dialogue.
5. Create the RON records.
6. Add cross-references by ID.
7. Record the source and adaptation decision in the provenance ledger.
8. Run validation.
9. Play the affected scene.
10. Commit the source-to-content change in the private repository.

For a character, extract:

```text
Public name
Role
Age band
Visual cues
Values
Fear
Secret
Voice notes
Faction ties
Starting relationship
Quest hooks
Combat role
Source note
```

For a location, extract:

```text
Name
Region
Arrival description
Ambient details
Connections
Access rules
Time restrictions
People normally present
Interactable objects
Secrets
Encounter hooks
Source note
```

Treat the existing character sheets as references. Do not automatically import every field. The playable adaptation needs a focused role, consistent voice, and a reason to be present in the game's systems.

## Keep a provenance ledger

Create `provenance/SOURCES.md` in the private pack:

```markdown
# Content provenance

| Content ID | Source | Permission or license | Adaptation note |
| --- | --- | --- | --- |
| wizarding-world-private.character.example | Personal session notes, 2025-04-12 | Private use | Combined two minor appearances |
| wizarding-world-private.location.example | Original campaign notes | Owned by campaign author | Description rewritten for interactive play |
```

This is not busywork. It gives you a practical answer when you later ask whether a file can be included in a public release.

## Link saves to a private pack without copying it

The save file stores identity, not source content:

```rust
#[derive(Debug, Clone, serde::Serialize, serde::Deserialize)]
pub struct PackFingerprint {
    pub pack_id: String,
    pub pack_version: String,
    pub schema_version: u32,
    pub content_sha256: String,
}
```

On load:

1. Resolve the pack path from the command line or ignored local config.
2. Load and hash the pack.
3. Compare the four saved fingerprint fields.
4. Load normally when all fields match.
5. Offer a migration when the pack supplies one.
6. Show a recovery screen when the pack is missing or incompatible.

Never embed source notes, PDF text, unrevealed scenes, or the full content pack in a public save fixture.

## Validate paths before reading files

Content packs are untrusted input, including your own pack after a bad merge.

The loader must:

- Canonicalize the pack root.
- Reject resolved paths outside that root.
- Reject absolute paths in the manifest.
- Reject `..` traversal components.
- Apply a maximum file size.
- Apply a maximum total pack size.
- Limit the number of records and dependency depth.
- Reject duplicate IDs.
- Report malformed files without exposing unrelated filesystem contents.

On platforms where symbolic links are available, resolve them and verify that their final path remains inside the pack root.

A path-checking helper can return a typed error:

```rust
#[derive(Debug, thiserror::Error)]
pub enum PackPathError {
    #[error("content path must be relative: {0}")]
    Absolute(std::path::PathBuf),

    #[error("content path leaves the pack root: {0}")]
    OutsideRoot(std::path::PathBuf),

    #[error("content file is too large: {path} ({bytes} bytes)")]
    TooLarge {
        path: std::path::PathBuf,
        bytes: u64,
    },
}
```

Keep filesystem errors as sources where possible so logs retain the underlying cause.

## Add a mod management surface

The first version can use commands rather than an in-game marketplace:

```powershell
storyforge mods list --pack ..\my-campaign
storyforge mods add ..\my-balance-mod --pack ..\my-campaign
storyforge mods remove my-balance-mod --pack ..\my-campaign
storyforge validate --pack ..\my-campaign --with-mods
```

Store the enabled list in the save slot. A save must remember the exact pack and mod versions used to create it.

When a required mod is missing, show a recovery screen with the missing IDs. Do not load the save into a partially compatible world.

## Build the public-release audit

Create a small workspace binary whose only input is the staged public release directory:

```text
tools/release-audit/
├── Cargo.toml
└── src/
    └── main.rs
```

Add `"tools/release-audit"` to the root workspace members. Create `tools/release-audit/Cargo.toml`:

```toml
[package]
name = "release-audit"
version.workspace = true
edition.workspace = true
publish = false

[dependencies]
clap = { version = "4.5", features = ["derive"] }
walkdir = "2.5"
```

Create `tools/release-audit/src/main.rs`:

```rust
use std::{
    error::Error,
    fs,
    path::{Component, Path, PathBuf},
};

use clap::Parser;
use walkdir::WalkDir;

const FORBIDDEN_FRAGMENTS: &[&str] = &[
    "wizarding-world-private",
    ".storyforge.local.toml",
    "source-notes",
    ".obsidian",
    "campaigns-private",
];

const FORBIDDEN_EXTENSIONS: &[&str] = &["pdf", "xlsx", "xls", "docx"];

#[derive(Debug, Parser)]
struct Arguments {
    /// Root of the already-staged public release.
    #[arg(long)]
    root: PathBuf,
}

fn main() -> Result<(), Box<dyn Error>> {
    let arguments = Arguments::parse();
    let failures = audit_release(&arguments.root)?;

    if failures.is_empty() {
        println!("release audit passed: {}", arguments.root.display());
        return Ok(());
    }

    for failure in &failures {
        eprintln!("release audit failed: {failure}");
    }
    Err(format!("{} release audit failure(s)", failures.len()).into())
}

fn audit_release(root: &Path) -> Result<Vec<String>, Box<dyn Error>> {
    let canonical_root = root.canonicalize()?;
    if !canonical_root.is_dir() {
        return Err(format!("release root is not a directory: {}", root.display()).into());
    }

    let mut failures = Vec::new();
    let mut saw_demo = false;
    let mut saw_license = false;

    for entry in WalkDir::new(&canonical_root).follow_links(false) {
        let entry = entry?;
        let path = entry.path();
        let relative = path.strip_prefix(&canonical_root)?;
        let normalized = relative.to_string_lossy().replace('\\', "/");
        let lowercase = normalized.to_ascii_lowercase();

        if relative
            .components()
            .any(|component| matches!(component, Component::ParentDir | Component::RootDir))
        {
            failures.push(format!("unsafe archive path: {normalized}"));
        }

        for fragment in FORBIDDEN_FRAGMENTS {
            if lowercase.contains(fragment) {
                failures.push(format!("forbidden path fragment `{fragment}`: {normalized}"));
            }
        }

        if path
            .extension()
            .and_then(|extension| extension.to_str())
            .is_some_and(|extension| {
                FORBIDDEN_EXTENSIONS.contains(&extension.to_ascii_lowercase().as_str())
            })
        {
            failures.push(format!("forbidden source-document type: {normalized}"));
        }

        if lowercase.contains("campaigns/academy-demo/") {
            saw_demo = true;
        }
        if matches!(
            path.file_name().and_then(|name| name.to_str()),
            Some("LICENSE-MIT" | "LICENSE-APACHE")
        ) {
            saw_license = true;
        }

        if entry.file_type().is_symlink() {
            match path.canonicalize() {
                Ok(target) if !target.starts_with(&canonical_root) => {
                    failures.push(format!(
                        "symbolic link exits release root: {normalized} -> {}",
                        target.display()
                    ));
                }
                Err(error) => {
                    failures.push(format!("unreadable symbolic link `{normalized}`: {error}"));
                }
                Ok(_) => {}
            }
        }

        if entry.file_type().is_file() && entry.metadata()?.len() <= 2_000_000 {
            let bytes = fs::read(path)?;
            if let Ok(text) = std::str::from_utf8(&bytes) {
                let lowercase_text = text.to_ascii_lowercase();
                for fragment in FORBIDDEN_FRAGMENTS {
                    if lowercase_text.contains(fragment) {
                        failures.push(format!(
                            "forbidden content marker `{fragment}` inside {normalized}"
                        ));
                    }
                }
            }
        }
    }

    if !saw_demo {
        failures.push("public academy-demo content is missing".to_owned());
    }
    if !saw_license {
        failures.push("LICENSE-MIT or LICENSE-APACHE is missing".to_owned());
    }

    failures.sort();
    failures.dedup();
    Ok(failures)
}

#[cfg(test)]
mod tests {
    use super::FORBIDDEN_FRAGMENTS;

    #[test]
    fn private_pack_id_is_part_of_the_guard() {
        assert!(FORBIDDEN_FRAGMENTS.contains(&"wizarding-world-private"));
    }
}
```

The release job first copies an allowlisted set of public artifacts into `dist/public-staging`. It never copies the repository root:

```text
dist/public-staging/
├── bin/
│   └── storyforge
├── campaigns/
│   └── academy-demo/
├── LICENSE-APACHE
├── LICENSE-MIT
└── THIRD_PARTY_LICENSES.md
```

Run the guard against that exact directory:

```powershell
cargo run -p release-audit -- --root dist\public-staging
```

The path scan prevents obvious source files from entering the artifact. The small UTF-8 content scan also catches a private pack ID pasted into a public fixture under an innocent filename. Keep the forbidden markers in the public repository generic; do not add private character names or plot secrets to this list.

## Test the boundary

Add tests for:

- Loading a public pack without private files present.
- Loading a private pack from an explicit path.
- Duplicate namespaced IDs.
- A declared override.
- An undeclared collision.
- Missing dependencies.
- Cyclic dependencies.
- Unsupported schema versions.
- Absolute paths.
- Parent-directory traversal.
- A symbolic link that exits the pack root.
- Oversized files and packs.
- A save whose required mod is missing.

Add a CI guard in the public repository that searches release inputs for private pack IDs and source paths. Treat a match as a failed build.

## Verification

Run:

```powershell
cargo test --workspace
cargo run -p storyforge-tui -- validate --pack campaigns\academy-demo
cargo run -p storyforge-tui -- validate --pack ..\wizarding-world-private
cargo run -p storyforge-tui -- play --pack ..\wizarding-world-private
```

Then build a public release archive and inspect its file list. It must contain `academy-demo` and must not contain the private campaign name, local configuration, source notes, or provenance records from the private repository.

## Acceptance check

- The public engine works when the private repository is absent.
- Private content is selected through an explicit path.
- IDs are namespaced and collisions fail clearly.
- Runtime loading is limited to validated game formats.
- Pack paths cannot escape their root.
- Each private adaptation has a provenance entry.
- A public release audit catches private content.

## Suggested commit

```powershell
git add .
git commit -m "feat: support isolated campaign packs and mods"
```
