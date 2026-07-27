# 11: Assemble the first playable MVP

## Result

This chapter does not add a new subsystem. It connects the existing ones into a 20 to 35 minute game with a beginning, choice, check, duel, consequence, save, and resume.

## MVP route

Use this original demo route:

```text
Main menu
  |
  v
Character creation
  |
  v
Rain Gate arrival
  |
  +--> Ask Iren about the seal
  |
  +--> Investigate a side route -- d20 check
  |         |
  |         +--> Quiet success
  |         |
  |         +--> Alarmed success with consequence
  |
  v
Gate Warden duel
  |
  +--> Victory by reducing resolve
  |
  +--> Escape through the side passage
  |
  +--> Defeat and late arrival
  |
  v
Induction scene reflects route, Iren trust, alarm, and duel result
  |
  v
MVP ending and continue prompt
```

The Gate Warden is an enchanted training construct. The duel is nonlethal.

## Write a route contract

Create `campaigns/academy-demo/tests/mvp-route.ron`:

```ron
(
    name: "quiet investigator victory",
    seed: 441993,
    commands: [
        ConfirmCharacter,
        Choose("search_route"),
        Continue,
        Cast("academy.spell.binding_chalk", Slot(1)),
        Move(Toward),
        Cast("academy.spell.ember_thread", Cantrip),
        Cast("academy.spell.ember_thread", Cantrip),
        Continue,
    ],
    expected: (
        scene: "academy.scene.mvp_complete",
        flags: ["academy.flag.side_passage_found"],
        quest_states: [("academy.quest.arrival", Completed)],
        relationship_ranges: [("academy.actor.iren", Trust, 1, 5)],
        remaining_slots: [(1, 1)],
    ),
)
```

The eventual test parser maps route commands into real engine commands. If exact combat rolls change, update the seed or commands deliberately and review the event diff.

## Main menu

Menu choices:

- New game.
- Continue newest compatible save.
- Load game.
- Settings.
- Validate campaign.
- Quit.

Disable Continue when no compatible save exists and display the reason.

## Autosave points

Add explicit safe autosaves after:

1. Character confirmation.
2. Gate-check resolution and scene transition.
3. Duel completion.
4. MVP ending.

Test that the autosave contains the state after the event, not before it.

## Log and history

The final character history should include:

- Character joined the academy.
- Approach to the sealed gate.
- Check result and consequence.
- Relationship change with Iren.
- Duel resolution.
- Arrival outcome.

The short dashboard log keeps recent events. The history tab keeps major events only.

## Error screens

The MVP must handle:

- Campaign validation failure.
- Save directory failure.
- Corrupt selected save.
- Terminal too small.
- Missing referenced content.
- Unexpected top-level error.

An error screen gives:

- Plain description.
- Safe next action.
- Log path.
- Quit key.

The terminal must restore after leaving it.

## Playthrough test harness

Create a headless runner in `storyforge-core` or a dedicated integration test. It loads the campaign through `storyforge-content`, dispatches scripted semantic commands, and returns state plus events.

It does not instantiate Ratatui.

Required routes:

- Quiet check success plus duel victory.
- Check consequence plus escape.
- Duel defeat.
- One 1st-level cast followed by an at-will cantrip.
- One upcast fixture that previews and spends the selected higher-level slot.
- Save after check, reconstruct process objects, load, and finish.

## MVP version

Keep package version `0.1.0`. Add:

- `CHANGELOG.md`.
- `LICENSE-MIT`.
- `LICENSE-APACHE`.
- Root README status.

The changelog entry should list playable behavior, supported platforms for development, known limits, and save schema 1.

## Manual playtest form

Record:

- Terminal and size.
- OS.
- Route chosen.
- Confusing controls.
- Text clipping.
- Unexplained rule.
- Save/resume result.
- Time to finish.
- One favorite moment.
- One boring moment.

Fix control blockers and state-loss bugs before adding world exploration.

## Full MVP gate

Run:

```powershell
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features --locked -- -D warnings
cargo test --workspace --all-features --locked
cargo test --workspace --doc --locked
cargo run -p storyforge-tui -- validate --pack campaigns/academy-demo --strict
cargo build --workspace --release --locked
cargo run -p storyforge-tui --release -- play --pack campaigns/academy-demo
```

After manual completion:

```powershell
git status --short
```

Only expected source, content, snapshot, and documentation files should appear. User saves must be outside the repository.

## Acceptance check

- New game reaches character creation.
- Character confirmation reaches the arrival scene.
- One choice performs a visible check.
- Both check outcomes continue.
- Duel supports victory, escape, and defeat.
- Cantrips spend no slots, and a leveled spell spends the selected legal slot.
- One scripted route verifies an authored upcast effect.
- The final scene reflects at least three prior state values.
- Manual save and all autosaves load.
- Headless routes pass.
- Release build works.
- Terminal restores after every tested exit.

## Suggested commit

```powershell
git add .
git commit -m "Complete the first playable Storyforge MVP"
git tag -a mvp-0.1.0 -m "Character creation, branching scene, duel, and save resume"
```

Do not push the tag until release automation is ready.
