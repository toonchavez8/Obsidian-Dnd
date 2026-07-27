# Stage 20: Testing, Diagnostics, and Performance

## Result

By the end of this stage, the engine has a layered test suite, deterministic playthrough checks, useful logs, a diagnostic command, and measured performance targets. You will be able to change systems without replaying the entire game by hand.

## Build a practical test pyramid

Use several focused test types:

| Test type | Purpose | Typical speed |
| --- | --- | --- |
| Unit | Rules, reducers, calculations, validation | Milliseconds |
| Integration | Multiple crates and content loading | Milliseconds to seconds |
| Snapshot | Stable terminal layouts and reports | Milliseconds |
| Property | Invariants across generated inputs | Seconds |
| Playthrough | A scripted path through a campaign | Seconds |
| Smoke | A packaged binary starts and exits correctly | Seconds |

Most tests should be unit and integration tests. A small number of complete playthroughs protect the seams between systems.

## Keep engine tests deterministic

Pass dependencies into systems instead of reaching for global state:

```rust
pub trait DiceRoller {
    fn roll_die(&mut self, sides: u8) -> u8;
}

pub struct SequenceDice {
    values: std::collections::VecDeque<u8>,
}

impl DiceRoller for SequenceDice {
    fn roll_die(&mut self, sides: u8) -> u8 {
        let value = self.values.pop_front().expect("test dice exhausted");
        assert!((1..=sides).contains(&value));
        value
    }
}
```

Production uses a seeded random-number generator. Tests use known sequences. Save the production seed when starting a game so a defect can be reproduced.

Do not assert against random statistical outcomes in ordinary tests. Test probability distribution separately with a wide tolerance if the random implementation itself needs coverage.

## Test reducers as pure behavior

For each command:

1. Construct the smallest valid state.
2. Submit a command.
3. Assert the returned events.
4. Apply the events.
5. Assert the final state.

Example:

```rust
fn healing_draught_id() -> ItemId {
    ItemId::new("academy-demo.item.healing-draught")
        .expect("the fixture contains a valid literal item ID")
}

#[test]
fn drinking_a_potion_consumes_it_and_restores_health() {
    let mut state = fixture::injured_player_with_healing_draught();

    let events = handle_command(
        &state,
        Command::UseItem {
            item_id: healing_draught_id(),
        },
    )
    .expect("command should be valid");

    assert_eq!(
        events,
        vec![
            Event::ItemConsumed {
                item_id: healing_draught_id(),
            },
            Event::HealthRestored { amount: 8 },
        ]
    );

    apply_events(&mut state, &events);

    assert_eq!(state.player.health.current, 18);
    assert!(!state
        .player
        .inventory
        .contains(&healing_draught_id()));
}
```

Use domain-specific fixture builders. Avoid a single enormous fixture that hides which facts matter to a test.

## Add property tests for invariants

Add `proptest` as a development dependency:

```powershell
cargo add -p storyforge-core --dev proptest
```

Good properties include:

- Health remains between zero and maximum health.
- Normal spell-slot counts remain between zero and their maxima.
- Temporary slots exist only at levels 1 through 5 and disappear at long rest.
- Cantrips never spend a slot.
- Sorcery points remain between zero and their feature maximum.
- Rejected metamagic and flexible-casting commands change no resources.
- A completed quest cannot also be active.
- Inventory weight equals the sum of contained item stacks.
- Applying an event sequence twice is either rejected or explicitly idempotent.
- A valid save round-trips without changing semantic state.
- Sorting initiative always preserves every combatant exactly once.

Property tests are especially useful for dice expressions, resource clamps, and save migration edges.

## Snapshot the terminal

Use Ratatui's `TestBackend` and `insta`:

```rust
#[test]
fn dashboard_at_standard_size() {
    let backend = ratatui::backend::TestBackend::new(100, 30);
    let mut terminal = ratatui::Terminal::new(backend).expect("terminal");
    let app = fixture::dashboard();

    terminal
        .draw(|frame| render(frame, &app))
        .expect("draw dashboard");

    insta::assert_snapshot!(terminal.backend());
}
```

Create snapshots for:

- Minimum supported terminal size.
- Standard 100 by 30 layout.
- Wide layout.
- Character creation.
- Dialogue with choices.
- Skill-check result.
- Combat turn.
- Save recovery warning.
- Validation errors.
- High-contrast mode.

Review snapshot changes. Do not automatically accept every changed snapshot in CI.

## Validate content with small fixtures

Under `crates/storyforge-content/tests/fixtures`, create:

```text
fixtures/
  valid-minimal/
  missing-reference/
  duplicate-id/
  invalid-condition/
  dependency-cycle/
  unsupported-schema/
```

Each invalid fixture should demonstrate one primary failure. Assert the structured diagnostic code and useful context, not an entire unstable debug string.

## Script complete playthroughs

Use a small text format for test actions:

```ron
(
    name: "arrival_to_duel_victory",
    seed: 4172,
    actions: [
        Choose("background.archivist"),
        Choose("trait.curious"),
        ConfirmCharacter,
        Choose("ask_about_the_sealed_door"),
        RollCheck,
        Choose("enter_training_hall"),
        Cast("spell.ember", Cantrip),
        EndTurn,
        Cast("spell.ember", Cantrip),
        Save("mvp-checkpoint"),
    ],
    expect: [
        SceneReached("arrival.after_duel"),
        FlagSet("mvp.duel_won"),
        SpellSlotsRemaining(level: 1, count: 2),
        SaveCreated("mvp-checkpoint"),
    ],
)
```

The test driver sends the same commands as the TUI. It must not mutate state directly. This keeps playthroughs meaningful.

Maintain at least these routes:

- A cooperative path.
- A defiant path.
- A failed-check recovery path.
- Combat victory.
- Combat escape or defeat recovery.
- Cantrip casting with unchanged slots.
- Upcasting with a selected higher-level slot.
- Flexible casting in both directions.
- Each default metamagic option, including the Empowered Spell combination exception.
- Save, quit, and resume.
- One route through each campaign arc once that content exists.

## Test save migrations

Keep one small fixture for every released save schema. For each fixture:

1. Load the old save.
2. Migrate it to the current schema.
3. Validate all references.
4. Save it again.
5. Reload the new file.
6. Assert important story and character facts.

Never generate old fixtures by calling the current serializer. Store the actual old JSON shape so the test can catch incompatible changes.

## Add structured logging

Use `tracing` and `tracing-subscriber`. Log to a rolling file, not into the terminal drawing surface:

```text
logs/
  storyforge.log
  storyforge.log.1
```

Record:

- Application and engine version.
- Operating system and terminal dimensions.
- Selected pack IDs and versions.
- Save schema version.
- Scene transitions.
- Commands and event kinds.
- Validation failures.
- Recoverable content errors.
- Panic information.

Do not log full save data, player-entered names unless diagnostic mode is enabled, or paths outside the selected pack. Redact sensitive local paths in user-facing reports.

Use fields rather than constructed sentences:

```rust
tracing::info!(
    scene_id = %next_scene,
    previous_scene_id = %current_scene,
    "scene changed"
);
```

## Provide a doctor command

Add:

```powershell
storyforge doctor
storyforge doctor --pack campaigns\academy-demo
storyforge doctor --save 1
```

The report should include:

```text
Storyforge version
Operating system and architecture
Terminal size and color support
Data and configuration paths
Selected content pack and schema
Pack validation result
Save metadata and compatibility
Log location
```

It must not print private story contents or tokens. Offer `--output diagnostic.txt` so a player can review the file before sharing it.

## Set budgets before optimizing

Use initial budgets:

| Operation | Target |
| --- | ---: |
| Input-to-frame response | Under 50 ms |
| Normal frame render | Under 16 ms |
| Public demo validation | Under 500 ms |
| Save write | Under 100 ms |
| Save load | Under 250 ms |
| Idle CPU | Near zero |
| Release binary startup | Under 500 ms |

These are development targets, not hard promises for every machine. The application is event-driven and should block while waiting for input. Do not redraw continuously unless an animation is active.

## Benchmark measured hot paths

Add Criterion only when a system has enough work to measure:

```powershell
cargo add -p storyforge-core --dev criterion
```

Useful benchmarks include:

- Parsing and validating a representative campaign pack.
- Looking up content IDs.
- Evaluating a large set of conditions.
- Resolving a combat round with several actors.
- Serializing a late-game save.
- Rendering a dense dashboard buffer.

Run:

```powershell
cargo bench
```

Profile a release build before changing architecture:

```powershell
cargo build --release
```

Use a platform-appropriate profiler and record the command, input pack, build profile, and result in a short performance note. An optimization without a baseline is difficult to evaluate.

## Add continuous integration

The CI pipeline should run:

```powershell
cargo fmt --all --check
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo test --workspace --all-features
cargo doc --workspace --no-deps
cargo run -p storyforge-tui -- validate --pack campaigns\academy-demo
```

Also run:

- Public/private release-content audit.
- Save fixture migrations.
- Scripted MVP playthrough.
- Package smoke tests on every supported operating system.
- `cargo deny` or an equivalent dependency license and advisory check after it is configured.

Pin the Rust toolchain in `rust-toolchain.toml` so local and CI builds agree.

## Handle crashes cleanly

Install a panic hook after terminal setup. It must:

1. Restore the terminal.
2. Write the panic and backtrace to the log.
3. Print a short recovery message.
4. Point to the log and `storyforge doctor`.

Test this path in a development-only command. A panic must never leave the player's terminal in raw mode.

## Verification

Run the complete local gate:

```powershell
cargo fmt --all --check
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo test --workspace --all-features
cargo doc --workspace --no-deps
cargo run -p storyforge-tui -- validate --pack campaigns\academy-demo
cargo run -p storyforge-tui -- playthrough campaigns\academy-demo\tests\playthroughs\mvp.ron
cargo run -p storyforge-tui -- doctor --pack campaigns\academy-demo
```

Resize the terminal while the game runs, trigger the tested panic path, and confirm that the shell returns to normal.

## Acceptance check

- Rules are tested without opening a terminal.
- Random outcomes are reproducible.
- Important layouts have reviewed snapshots.
- Every released save schema has a migration fixture.
- At least one complete MVP playthrough runs automatically.
- Logs preserve diagnostic context without exposing private content.
- The doctor command produces a safe, useful report.
- Performance work starts from measured budgets and profiles.
- CI runs on every public target.

## Suggested commit

```powershell
git add .
git commit -m "test: add full engine and release quality gates"
```
