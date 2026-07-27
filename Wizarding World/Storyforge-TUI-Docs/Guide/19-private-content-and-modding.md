# Stage 19: Private Campaigns and Community Mods

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
