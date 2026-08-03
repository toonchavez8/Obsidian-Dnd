# 04: Finish the responsive dashboard

## Guide order

These four guides build one learning path from the current repo state:

1. `04-responsive-dashboard.md` finishes the visible TUI shell and proves keyboard input reaches app state.
2. `05-engine-command-event-model.md` moves gameplay choice selection out of the TUI and into `storyforge-core`.
3. `06-content-pack-format.md` loads scene data from files so the engine and TUI stop depending on hard-coded story text.
4. `07-character-creation.md` starts the playable character-creation prologue using the same command/event path.

The end-goal game sheet describes a much larger RPG. These guides do not try to reach the full game. They build the next stable slice: responsive dashboard, command/event engine, content pack loading, and the first character-creation state machine.

## What already exists

The repo already has this TUI skeleton:

```text
crates/storyforge-tui/src/
â”œâ”€â”€ action.rs
â”œâ”€â”€ app.rs
â”œâ”€â”€ layout.rs
â”œâ”€â”€ main.rs
â”œâ”€â”€ theme.rs
â””â”€â”€ ui.rs
```

The current TUI already:

- Starts through `ratatui::run` in `main.rs`.
- Stores visible app state in `App`.
- Converts crossterm key events into `UiAction`.
- Opens Story, Character, Journal, Map, and Help screens.
- Chooses between compact and standard render functions.
- Shows placeholder story, action, character, quest, and log panes.
- Displays temporary spell-slot and sorcery-point values.

Guide 04 keeps all gameplay logic out of the renderer. The goal is to prove this flow:

```text
key press -> UiAction -> App::update -> App fields -> ui::render
```

Later guides replace placeholder state with engine and content data.

---

File: `crates/storyforge-tui/src/layout.rs`

## Step 1: Correct the dashboard size thresholds

### Current state

`layout.rs` already has a pure function named `mode_for`. It receives the current terminal `Rect` and returns one of three layout modes.

The tests expect these design thresholds:

- Below `80x24`: `TooSmall`.
- At least `80x24`: `Compact`.
- At least `120x36`: `Standard`.

The current constants still say:

```rust
const MIN_WIDTH: u16 = 56;
const MIN_HEIGHT: u16 = 24;
const STANDARD_WIDTH: u16 = 100;
const STANDARD_HEIGHT: u16 = 30;
```

### Why this step matters

Every later screen depends on the same size contract. Character creation, scene rendering, combat, and save menus should all agree on what a supported terminal size means.

### Before

In `layout.rs`, find the constants at the top of the file:

```rust
/// Minimum terminal width for the UI to render anything useful.
const MIN_WIDTH: u16 = 56;
/// Minimum terminal height for the UI to render anything useful.
const MIN_HEIGHT: u16 = 24;
/// Width at which the UI can switch from compact to standard layout.
const STANDARD_WIDTH: u16 = 100;
/// Height at which the UI can switch from compact to standard layout.
const STANDARD_HEIGHT: u16 = 30;
```

### What to change

Change only the numeric constants. Do not change `mode_for` yet.

### Temporary MVP / debug behavior

Start with this pure function because it is easy to test. It does not need the terminal, the event loop, or the renderer.

### After

```rust
/// Minimum terminal width for the UI to render anything useful.
const MIN_WIDTH: u16 = 80;
/// Minimum terminal height for the UI to render anything useful.
const MIN_HEIGHT: u16 = 24;
/// Width at which the UI can switch from compact to standard layout.
const STANDARD_WIDTH: u16 = 120;
/// Height at which the UI can switch from compact to standard layout.
const STANDARD_HEIGHT: u16 = 36;
```

### Learning checkpoint

You should understand that `mode_for` turns raw terminal size into layout intent. That intent is recomputed from the live frame area instead of being stored in `App`, so resizing the terminal cannot leave stale layout state behind.

### How to verify

Run:

```powershell
cargo test -p storyforge-tui layout
```

The existing layout tests should pass.

### Next connection

The warning screen in `ui.rs` must tell the player the same minimum size that `layout.rs` enforces.

---

File: `crates/storyforge-tui/src/ui.rs`

## Step 2: Make the size warning match the real minimum

### Current state

`render_size_warning` already renders a full-screen message when the terminal is too small. It currently says:

```rust
Line::from("Storyforge needs at least 56 columns x 24 rows."),
```

After Step 1, that message is obsolete.

### Why this step matters

The UI should not explain one rule while the layout function enforces another. This is a small habit that matters later when rules, content validation, and save errors become player-facing.

### Before

Find this block inside `render_size_warning`:

```rust
let message = Text::from(vec![
    Line::from("Storyforge needs at least 56 columns x 24 rows."),
    Line::from(format!("Current size: {} x {}", area.width, area.height)),
    Line::from("Resize the terminal or press q to quit."),
]);
```

### What to change

Change only the first line of the message.

### Temporary MVP / debug behavior

Shrink the terminal after this change. The goal is only to prove the warning text is correct before changing any other render behavior.

### After

```rust
let message = Text::from(vec![
    Line::from("Storyforge needs at least 80 columns x 24 rows."),
    Line::from(format!("Current size: {} x {}", area.width, area.height)),
    Line::from("Resize the terminal or press q to quit."),
]);
```

### Learning checkpoint

You should understand that this warning is still part of the normal app loop. The app keeps reading input, so pressing `q` can still quit even when the game dashboard is not being drawn.

### How to verify

Run:

```powershell
cargo run -p storyforge-tui
```

Shrink the terminal below `80x24`. You should see the warning and the current terminal size.

### Next connection

Now that layout behavior is stable, update keyboard mappings so movement keys are ready for choice navigation.

---

File: `crates/storyforge-tui/src/action.rs`

## Step 3: Add `j` and `k` movement while keeping a journal shortcut

### Current state

`UiAction::from` already maps raw keys to semantic UI actions:

```rust
KeyCode::Up => Self::Up,
KeyCode::Down => Self::Down,
KeyCode::Enter => Self::Confirm,
KeyCode::Esc => Self::Back,
KeyCode::Char('c') => Self::OpenCharacter,
KeyCode::Char('j') => Self::OpenJournal,
KeyCode::Char('m') => Self::OpenMap,
KeyCode::Char('?') => Self::OpenHelp,
KeyCode::Char('q') => Self::Quit,
_ => Self::None,
```

The end-goal input model says arrow keys or `j`/`k` should move through menus. Since `j` is currently Journal, this early version should move Journal to `l`, which can also stand for log until the later keymap is configurable.

### Why this step matters

This keeps input translation separate from app behavior:

- `action.rs` decides what a key means.
- `app.rs` decides what the action changes.
- `ui.rs` draws the result.

That separation makes it easier to add mouse input or remappable keys later without touching game rules.

### Before

Use the existing `match key.code` block shown above.

### What to change

Map:

- `KeyCode::Up` and `KeyCode::Char('k')` to `UiAction::Up`.
- `KeyCode::Down` and `KeyCode::Char('j')` to `UiAction::Down`.
- `KeyCode::Char('l')` to `UiAction::OpenJournal`.

### Temporary MVP / debug behavior

Before the engine exists, `Up` and `Down` may only change a compact tab. That is enough for this guide. First prove the key mapping. Then prove the app reacts to the action in Step 4.

### After

```rust
match key.code {
    KeyCode::Up | KeyCode::Char('k') => Self::Up,
    KeyCode::Down | KeyCode::Char('j') => Self::Down,
    KeyCode::Enter => Self::Confirm,
    KeyCode::Esc => Self::Back,
    KeyCode::Char('c') => Self::OpenCharacter,
    KeyCode::Char('l') => Self::OpenJournal,
    KeyCode::Char('m') => Self::OpenMap,
    KeyCode::Char('?') => Self::OpenHelp,
    KeyCode::Char('q') => Self::Quit,
    _ => Self::None,
}
```

Then add this focused test at the bottom of `action.rs`:

```rust
#[cfg(test)]
mod tests {
    use crossterm::event::{KeyCode, KeyEvent, KeyModifiers};

    use super::UiAction;

    #[test]
    fn vim_keys_should_map_to_vertical_movement() {
        let down = KeyEvent::new(KeyCode::Char('j'), KeyModifiers::NONE);
        let up = KeyEvent::new(KeyCode::Char('k'), KeyModifiers::NONE);

        assert_eq!(UiAction::from(down), UiAction::Down);
        assert_eq!(UiAction::from(up), UiAction::Up);
    }

    #[test]
    fn l_should_open_the_journal_screen() {
        let key = KeyEvent::new(KeyCode::Char('l'), KeyModifiers::NONE);

        assert_eq!(UiAction::from(key), UiAction::OpenJournal);
    }
}
```

### Learning checkpoint

You should understand that several physical inputs can produce the same semantic action. The rest of the app should care about `UiAction::Down`, not whether it came from the Down arrow, `j`, or a future mouse wheel event.

### How to verify

Run:

```powershell
cargo test -p storyforge-tui action
```

### Next connection

Step 4 makes `UiAction::Up` and `UiAction::Down` visibly change `App` state.

---

File: `crates/storyforge-tui/src/app.rs`

## Step 4: Make tab movement visibly testable

### Current state

`App` already has this field:

```rust
pub(crate) selected_tab: usize,
```

The compact renderer already reads it through `render_selected_tab`. But `App::update` currently ignores movement:

```rust
UiAction::Up | UiAction::Down | UiAction::Confirm | UiAction::None => {}
```

### Why this step matters

:This gives you a tiny end-to-end test of the app loop before game rules exist:

```text
key -> UiAction -> App::update -> selected_tab -> render_selected_tab
```

This is UI state, not story state. It is safe for `App` to own it.

### Before

Find this final match arm in `App::update`:

```rust
// Up, Down, and Confirm are reserved for future lists and forms.
UiAction::Up | UiAction::Down | UiAction::Confirm | UiAction::None => {}
```

### What to change

Let Down move to the next compact tab and Up move to the previous compact tab. Keep wrapping inside the four visible tabs.

### Temporary MVP / debug behavior

This is intentionally a temporary debug behavior. In guide 05, gameplay choice selection moves into `storyforge-core`. Compact tab movement can remain a TUI concern because it only changes what panel is visible.

### After

```rust
UiAction::Down => {
    self.selected_tab = (self.selected_tab + 1) % 4;
}
UiAction::Up => {
    self.selected_tab = self.selected_tab.checked_sub(1).unwrap_or(3);
}
UiAction::Confirm | UiAction::None => {}
```

Keep the screen-opening, Back, and Quit arms above these movement arms.

Add this test beside the existing `q_should_request_quit` test:

```rust
#[test]
fn movement_should_change_the_selected_compact_tab() {
    let mut app = App::default();

    app.update(crate::action::UiAction::Down);
    assert_eq!(app.selected_tab, 1);

    app.update(crate::action::UiAction::Up);
    assert_eq!(app.selected_tab, 0);

    app.update(crate::action::UiAction::Up);
    assert_eq!(app.selected_tab, 3);
}
```

### Learning checkpoint

You should understand the difference between UI state and game state. `selected_tab` belongs in `App` because it only controls which panel is visible. A selected story choice belongs in the engine because it affects gameplay commands.

### How to verify

Run:

```powershell
cargo test -p storyforge-tui app
cargo run -p storyforge-tui
```

At compact size, press `j` and `k`. The selected bottom panel should change.

### Next connection

Guide 05 uses the same input path, but instead of changing only a TUI tab, `App` dispatches a gameplay command to `storyforge-core`.

---

File: `crates/storyforge-tui/src/ui.rs`

## Step 5: Update visible shortcuts and keep rendering pure

### Current state

The renderer already follows the right architecture. `render` reads `App`, asks `mode_for(frame.area())`, and draws widgets. It does not read files, roll dice, or mutate `App`.

One visible hint still says `c/j/m/?`, and the Actions pane still says `[j] Journal`.

### Why this step matters

Once `j` means Down, the dashboard should not teach the old shortcut. This step also reinforces the renderer rule: display data, do not change data.

### Before

In `render_header`, find:

```rust
Span::styled("c/j/m/?", Style::default().fg(theme.focus)),
Span::styled(" screens", Style::default().fg(theme.muted)),
```

In `render_actions_panel`, find:

```rust
let text = "\n[c] Character\n[j] Journal\n[m] Map\n[?] Help\n[Esc] Back/Quit";
```

### What to change

Change the visible shortcuts from `j` to `l`.

Do not add file loading, random rolls, command dispatch, or state mutation inside `ui.rs`.

### Temporary MVP / debug behavior

Hard-coded placeholder strings are still acceptable in guide 04. Guide 06 replaces story text and choices with loaded content.

### After

```rust
Span::styled("c/l/m/?", Style::default().fg(theme.focus)),
Span::styled(" screens", Style::default().fg(theme.muted)),
```

```rust
let text = "\n[c] Character\n[l] Journal\n[m] Map\n[?] Help\n[Esc] Back/Quit";
```

The top-level render function should still keep this shape:

```rust
pub fn render(frame: &mut Frame, app: &App) {
    match mode_for(frame.area()) {
        LayoutMode::TooSmall => render_size_warning(frame, app.theme),
        LayoutMode::Compact => render_compact(frame, app),
        LayoutMode::Standard => render_standard(frame, app),
    }

    render_overlay(frame, app);
}
```

### Learning checkpoint

You should understand that `ui.rs` answers: "Given this app state and terminal size, what should the frame look like?" It should not answer: "What happens next in the game?"

### How to verify

Run:

```powershell
cargo run -p storyforge-tui
```

Confirm the header and Actions pane show `l` for Journal. Press `l` and confirm the Journal screen opens.

### Next connection

Guide 05 adds an engine field to `App`. The renderer will read engine state, but it should still not mutate engine state.

## Full verification

Run:

```powershell
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features --locked -- -D warnings
cargo test --workspace --all-features --locked
cargo run -p storyforge-tui
```

Manual checks:

1. Shrink below `80x24` and verify the warning.
2. Set `80x24` and verify compact mode.
3. Set at least `120x36` and verify standard mode.
4. Press `c`, `l`, `m`, and `?`; each screen should open.
5. Press `Esc` from a subscreen; it should return to Story.
6. Press `j` and `k` at compact size; the selected bottom panel should change.
7. Press `q` from Story; it should quit.

## Acceptance check

- The three layout thresholds match the design.
- The size warning text matches the code.
- `j` and `k` map to movement.
- `l` opens the journal/log screen for now.
- Compact tab movement proves input changes visible UI state.
- Render functions do not read files, roll dice, dispatch commands, or mutate game state.

