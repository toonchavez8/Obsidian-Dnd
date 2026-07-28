# 08: Add save, load, and migrations

## Result

The game will save a versioned state snapshot to platform-correct directories, verify the write, keep a backup, list slots, and load a compatible save.

## Add dependencies

```powershell
cargo add -p storyforge-tui serde_json directories time --features time/serde,time/formatting
cargo add -p storyforge-tui sha2
```

Keep filesystem behavior in the executable crate. `storyforge-core` remains serializable but does not choose paths.

## Save directory

Use `directories::ProjectDirs`:

```rust
use std::path::PathBuf;

use directories::ProjectDirs;

pub fn save_root() -> Result<PathBuf, SaveError> {
    ProjectDirs::from("dev", "toonchavez8", "Storyforge")
        .map(|dirs| dirs.data_local_dir().join("saves"))
        .ok_or(SaveError::ProjectDirectoryUnavailable)
}
```

`doctor` prints the resolved path. Tests inject a temporary root and never write into the real user directory.

## Save envelope

```rust
#[derive(Debug, Clone, serde::Serialize, serde::Deserialize)]
pub struct SaveEnvelope {
    pub schema_version: u32,
    pub engine_version: String,
    pub campaign_id: String,
    pub campaign_version: String,
    pub content_fingerprint: String,
    pub saved_at: time::OffsetDateTime,
    pub slot: u8,
    pub rng: SavedRng,
    pub state: storyforge_core::GameState,
}

#[derive(Debug, Clone, serde::Serialize, serde::Deserialize)]
pub struct SavedRng {
    pub seed: u64,
    pub stream: u64,
    pub word_position: u128,
}
```

Restrict manual slot numbers to 1 through 3. Use separate names for autosaves and ironman.

## Spell-resource state

`GameState` serializes the complete spellcasting state:

- Known cantrips.
- Known and prepared leveled spells.
- Remaining and maximum normal slots for levels 1 through 9.
- Temporary slot counts for levels 1 through 5.
- Optional sorcery feature level.
- Current and maximum sorcery points.
- Learned metamagic IDs.
- Active concentration.
- Action, bonus-action, reaction, and per-turn casting markers when saving in combat is allowed.

Do not reconstruct remaining resources from character level while loading. Progression data determines legal maxima, but the save records what the player has spent.

After deserialization, validate:

- Normal remaining slots do not exceed their maxima.
- Temporary slots exist only at levels 1 through 5.
- Sorcery points do not exceed the feature maximum.
- Characters without the sorcery feature have no point pool or metamagic choices.
- Prepared leveled spells are known.
- Known cantrips are level 0.
- Active concentration references a loaded spell and actor.

Before the first public release, keep this model inside save schema 1. If a playtest build already wrote an older spell-resource shape, add a real schema migration and retain one fixture from that build. Do not infer spent slots from incomplete data; mark an unrecoverable save clearly when the old data lacks enough information.

## Atomic save sequence

For slot 1:

```text
slot-1.json
slot-1.json.tmp
slot-1.json.bak
```

Write sequence:

1. Create the save directory.
2. Serialize to formatted JSON in memory.
3. Write `slot-1.json.tmp` with `create_new`.
4. Flush the file.
5. Reopen and deserialize the temporary file.
6. If a primary exists, replace the old backup with it.
7. Rename temporary to primary.
8. Reopen and verify primary.

If startup sees a temporary file:

- Keep a valid primary.
- Restore a valid backup when primary is invalid.
- Promote a valid temporary file only when no valid primary exists.
- Show the recovery action in the load menu and log.

Windows rename behavior differs when the destination exists. Implement platform-aware replacement inside one small function and test it on CI.

## Save errors

Use typed variants:

- Directory unavailable.
- Create directory.
- Serialize.
- Write temporary.
- Flush.
- Parse verification.
- Backup.
- Replace.
- Read.
- Unsupported schema.
- Campaign mismatch.
- Content mismatch.
- Migration failed.

Every I/O variant stores the path and source error.

## Load order

1. Read JSON into `serde_json::Value`.
2. Read `schema_version`.
3. Reject a version newer than the engine supports.
4. Apply one migration at a time.
5. Deserialize current `SaveEnvelope`.
6. Check campaign ID.
7. Compare campaign version and fingerprint.
8. Ask the player before loading a compatible but changed pack.
9. Rebuild runtime-only caches.
10. Validate core invariants.

Do not deserialize directly into the newest struct before migration.

## Migration interface

```rust
pub trait SaveMigration {
    fn from_version(&self) -> u32;
    fn to_version(&self) -> u32;
    fn migrate(&self, value: serde_json::Value) -> Result<serde_json::Value, SaveError>;
}
```

The migration registry must contain a continuous chain. Test startup detection for a missing link.

Migrations cannot:

- Read live campaign content.
- Roll random values.
- Use current time for gameplay fields.
- Delete unknown state without recording a diagnostic.

## Autosave policy

Core events do not write files. The TUI save coordinator requests an autosave after:

- Character confirmation.
- Scene transition marked safe.
- Quest state change.
- Combat completion.
- Travel arrival.
- Arc transition.

Do not autosave during a half-resolved command or animation.

Keep three rotating autosaves:

```text
autosave-0.json
autosave-1.json
autosave-2.json
```

## Load menu

Show:

- Character name.
- Level.
- Current location.
- Arc.
- Play time.
- Saved time.
- Campaign name and version.
- Compatibility status.

Never crash because a single slot is corrupt. Mark that slot and offer backup recovery.

## Tests

Use a temporary directory fixture. Required tests:

- Save then load preserves equivalent state.
- Primary write creates valid JSON.
- Existing primary becomes backup.
- Corrupt primary restores valid backup.
- Newer schema is rejected.
- Missing migration link is rejected.
- Campaign mismatch is blocked.
- Changed compatible fingerprint requires confirmation status.
- Slot zero and slot four are rejected.
- Private campaign path does not enter the save payload.
- Spell slots, temporary slots, sorcery points, and metamagic survive a round trip.
- Invalid spell-resource counts fail invariant validation.
- A long-rest save has no temporary flexible-casting slots.

## Manual test

1. Create a character.
2. Save to slot 1.
3. Exit the process.
4. Relaunch and load slot 1.
5. Confirm character and scene.
6. Save again.
7. Confirm a backup exists.
8. Copy the save directory before deliberately corrupting a test slot.
9. Relaunch and test recovery.

## Verify

```powershell
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features --locked -- -D warnings
cargo test --workspace --all-features --locked
cargo run -p storyforge-tui -- doctor
```

## Common mistakes

- Save paths are hardcoded to Windows.
- The original file is deleted before the new file verifies.
- Runtime terminal state is serialized.
- A content display name is used as save identity.
- Migration jumps from version 1 directly to version 4.
- Ironman disables safety backups.

## Acceptance check

- Manual and autosave files are versioned.
- Save replacement is recoverable.
- Loading validates campaign identity.
- One corrupt slot does not block the menu.
- Tests use isolated temporary paths.

## Suggested commit

```powershell
git add .
git commit -m "Add versioned and recoverable saves"
```
