# 05b: Layout focus, directional navigation, and the inventory sidebar

## Guide order

These five guides build one learning path from the current repo state:

1. `04-responsive-dashboard.md` finishes the visible TUI shell and proves keyboard input reaches app state.
2. `05-engine-command-event-model.md` moves gameplay choice selection out of the TUI and into `storyforge-core`.
3. `05b-layout-focus-navigation-and-inventory.md` (this guide) separates **focus** from **movement**, teaches `j`/`k` to move between panels, adds arrow/WASD movement inside the focused panel, and builds a temporary inventory sidebar.
4. `06-content-pack-format.md` loads scene data from files so the engine and TUI stop depending on hard-coded story text.
5. `07-character-creation.md` starts the playable character-creation prologue using the same command/event path.

This guide only covers the focus and inventory slice. It does not try to build the full inventory, equipment, or shop systems from the end-goal game sheet. Those come later, once the command/event and content paths are stable.

## Result

The TUI will distinguish two kinds of keyboard input:

- **Focus movement**: `j` and `k` move the highlighted panel forward and backward around the dashboard.
- **Movement inside the focused panel**: arrow keys and `w`/`a`/`s`/`d` move the cursor inside whatever panel is focused. If the panel has no horizontal layout, left and right fall back to moving focus.

The dashboard also gains a temporary **Inventory** panel that opens with `i` and sits on the right side:

- Standard layout: one third of the width.
- Compact layout: one half of the width.
- Too-small layout: seven eighths of the width (a fallback, since the size warning normally hides this mode).

The inventory panel is a placeholder: it owns a small list of dummy items and shows how full `w`/`a`/`s`/`d` movement works inside a panel. Later guides will replace the dummy list with real character inventory data from `storyforge-core`.

## Flow to prove

The new flow uses these functions and types:

```text
KeyEvent -> UiAction (MoveUp/Down/Left/Right, FocusNext/Previous, ToggleInventory)
    -> App::update
        -> App::focus (which panel is active)
        -> App::engine.dispatch (when Actions is focused on the Story screen)
        -> App::inventory_selected (when Inventory is focused)
    -> ui::render (highlight the focused panel, draw the inventory sidebar)
```

First prove the key mappings. Then prove `focus` changes. Then prove movement inside the focused panel. Then prove the inventory sidebar renders. Then connect everything.

---

File: `crates/storyforge-tui/src/action.rs`

## Step 1: Split movement into focus actions and directional actions

### Current state

`UiAction` currently has two movement variants: `Up` and `Down`. The `From<KeyEvent>` implementation maps both arrow keys and `j`/`k` to them, and `w`/`s` are also mapped to `Up`/`Down`:

```rust
KeyCode::Up | KeyCode::Char('w' | 'k') => Self::Up,
KeyCode::Down | KeyCode::Char('s' | 'j') => Self::Down,
```

That worked while movement only meant "move up or down the current list." Now the dashboard needs two separate ideas: moving between panels and moving inside a panel.

### Why this step matters

`j` and `k` should not mean the same thing as the arrow keys in this layout. Separating `FocusNext`/`FocusPrevious` from `MoveUp`/`MoveDown`/`MoveLeft`/`MoveRight` lets each panel decide what a direction does. The engine still receives commands only when the Actions panel is focused on the Story screen.

### Before

`UiAction` looks like this:

```rust
pub enum UiAction {
    Up,
    Down,
    Confirm,
    Back,
    OpenCharacter,
    OpenJournal,
    OpenMap,
    OpenHelp,
    Quit,
    None,
}
```

And `From<KeyEvent>` uses the combined mapping shown above.

### What to change

Replace the movement variants with directional movement and explicit focus movement. Map:

- Arrow keys and `w`/`a`/`s`/`d` to `MoveUp`, `MoveLeft`, `MoveDown`, `MoveRight`.
- `j` and `k` to `FocusNext` and `FocusPrevious`.
- `i` to `ToggleInventory`.

Keep `l` as `OpenJournal` from the previous guide.

### Temporary MVP / debug behavior

Do not change `App::update` yet. First, only change the action definitions and the tests. This proves that pressing a key produces the right `UiAction` before any behavior depends on it.

### After

```rust
use crossterm::event::{KeyCode, KeyEvent, KeyEventKind};

/// A semantic action the user can ask the UI to perform.
///
/// Converting raw key events into these actions keeps the event loop small and
/// makes the update logic easy to read and test.
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum UiAction {
    /// Move up inside the currently focused panel.
    MoveUp,

    /// Move down inside the currently focused panel.
    MoveDown,

    /// Move left inside the currently focused panel, or fall back to the
    /// previous panel if the panel has no horizontal layout.
    MoveLeft,

    /// Move right inside the currently focused panel, or fall back to the next
    /// panel if the panel has no horizontal layout.
    MoveRight,

    /// Move focus to the next panel in the dashboard layout.
    FocusNext,

    /// Move focus to the previous panel in the dashboard layout.
    FocusPrevious,

    /// Open or close the inventory sidebar.
    ToggleInventory,

    Confirm,
    Back,
    OpenCharacter,
    OpenJournal,
    OpenMap,
    OpenHelp,
    Quit,
    None,
}

/// Converts a crossterm key event into the UI action it represents.
///
/// Release and repeat events map to `None` so the app only acts once per
/// physical key press.
impl From<KeyEvent> for UiAction {
    fn from(key: KeyEvent) -> Self {
        if key.kind != KeyEventKind::Press {
            return Self::None;
        }

        match key.code {
            // Arrow keys and WASD move inside the focused panel.
            KeyCode::Up | KeyCode::Char('w') => Self::MoveUp,
            KeyCode::Down | KeyCode::Char('s') => Self::MoveDown,
            KeyCode::Left | KeyCode::Char('a') => Self::MoveLeft,
            KeyCode::Right | KeyCode::Char('d') => Self::MoveRight,

            // j/k move focus to the next or previous panel.
            KeyCode::Char('j') => Self::FocusNext,
            KeyCode::Char('k') => Self::FocusPrevious,

            // i opens or closes the inventory sidebar.
            KeyCode::Char('i') => Self::ToggleInventory,

            KeyCode::Enter => Self::Confirm,
            KeyCode::Esc => Self::Back,
            KeyCode::Char('c') => Self::OpenCharacter,
            KeyCode::Char('l') => Self::OpenJournal,
            KeyCode::Char('m') => Self::OpenMap,
            KeyCode::Char('?') => Self::OpenHelp,
            KeyCode::Char('q') => Self::Quit,
            _ => Self::None,
        }
    }
}

#[cfg(test)]
mod tests {
    use crossterm::event::{KeyCode, KeyEvent, KeyModifiers};

    use super::UiAction;

    #[test]
    fn arrow_keys_should_map_to_directional_movement() {
        let up = KeyEvent::new(KeyCode::Up, KeyModifiers::NONE);
        let down = KeyEvent::new(KeyCode::Down, KeyModifiers::NONE);
        let left = KeyEvent::new(KeyCode::Left, KeyModifiers::NONE);
        let right = KeyEvent::new(KeyCode::Right, KeyModifiers::NONE);

        assert_eq!(UiAction::from(up), UiAction::MoveUp);
        assert_eq!(UiAction::from(down), UiAction::MoveDown);
        assert_eq!(UiAction::from(left), UiAction::MoveLeft);
        assert_eq!(UiAction::from(right), UiAction::MoveRight);
    }

    #[test]
    fn wasd_keys_should_map_to_directional_movement() {
        let up = KeyEvent::new(KeyCode::Char('w'), KeyModifiers::NONE);
        let down = KeyEvent::new(KeyCode::Char('s'), KeyModifiers::NONE);
        let left = KeyEvent::new(KeyCode::Char('a'), KeyModifiers::NONE);
        let right = KeyEvent::new(KeyCode::Char('d'), KeyModifiers::NONE);

        assert_eq!(UiAction::from(up), UiAction::MoveUp);
        assert_eq!(UiAction::from(down), UiAction::MoveDown);
        assert_eq!(UiAction::from(left), UiAction::MoveLeft);
        assert_eq!(UiAction::from(right), UiAction::MoveRight);
    }

    #[test]
    fn j_and_k_should_move_focus() {
        let next = KeyEvent::new(KeyCode::Char('j'), KeyModifiers::NONE);
        let previous = KeyEvent::new(KeyCode::Char('k'), KeyModifiers::NONE);

        assert_eq!(UiAction::from(next), UiAction::FocusNext);
        assert_eq!(UiAction::from(previous), UiAction::FocusPrevious);
    }

    #[test]
    fn i_should_toggle_inventory() {
        let key = KeyEvent::new(KeyCode::Char('i'), KeyModifiers::NONE);

        assert_eq!(UiAction::from(key), UiAction::ToggleInventory);
    }

    #[test]
    fn l_should_open_journal() {
        let key = KeyEvent::new(KeyCode::Char('l'), KeyModifiers::NONE);

        assert_eq!(UiAction::from(key), UiAction::OpenJournal);
    }
}
```

### Learning checkpoint

You should understand that `UiAction` is intentionally a small set of *intents*, not a copy of the keyboard layout. One physical key can only map to one intent, but several keys can map to the same intent. The rest of the app cares about the intent, not the key.

### How to verify

Run:

```powershell
cargo test -p storyforge-tui action
```

All the action tests should pass.

### Next connection

Now `App` needs a focus model that knows which panel is active, and inventory state for the new sidebar.

---

File: `crates/storyforge-tui/src/app.rs`

## Step 2: Expand the focus model and add inventory state

### Current state

`App` already owns `focus: Focus`, but `Focus` only has `Actions`, an unused `Story` variant, and `Log`. The default focus is `Actions`. There is no inventory state.

### Why this step matters

The dashboard now has six layout panels that can receive focus: Story, Actions, Character, Quest, Log, and Inventory. Inventory is special because it is only visible when the player opens it, so the focus ring must skip it while it is closed.

### Before

`Focus` currently looks like this:

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq, Default)]
pub enum Focus {
    /// Action list or choice list.
    #[default]
    Actions,

    /// Reserved for future story text navigation.
    #[expect(dead_code)]
    Story,

    /// Event or combat log.
    Log,
}
```

And `App` has no inventory fields.

### What to change

1. Expand `Focus` to cover all six panels. Make `Story` the default because the player starts by reading the narrative panel.
2. Add an `inventory_open` flag, an `inventory_selected` index, and a dummy item list to `App`.
3. Add a constant for the inventory grid column count so movement math is easy to read and test.

### Temporary MVP / debug behavior

The inventory starts with a hard-coded list of six placeholder items. The columns are fixed at two for now. Later guides will replace the list with real data from the engine and make the column count depend on the panel width.

### After

```rust
/// Number of columns used for the temporary inventory grid.
///
/// This is a UI constant, not gameplay state. It will eventually be computed
/// from the rendered inventory panel width.
const INVENTORY_COLUMNS: usize = 2;

/// Holds every piece of visible application state.
///
/// The `App` is responsible for application-level concerns such as navigation,
/// keyboard focus, UI state, and owning the gameplay engine.
///
/// Rendering only reads from this structure, making it easy to construct an
/// `App` during tests and render snapshots without creating a real terminal.
#[derive(Debug)]
pub struct App {
    /// Set to `true` once the user has requested to exit the application.
    pub(crate) should_quit: bool,

    /// The currently visible top-level screen.
    pub(crate) screen: Screen,

    /// Which panel currently owns keyboard focus.
    pub(crate) focus: Focus,

    /// Index of the currently selected compact-mode tab.
    ///
    /// The value is wrapped using modulo arithmetic so it always remains within
    /// the valid tab range. The focus system keeps this in sync with the
    /// focused detail panel.
    pub(crate) selected_tab: usize,

    /// Deterministic gameplay engine.
    ///
    /// The TUI never implements game rules directly. Instead it converts user
    /// input into commands and sends them to the engine, which owns the game's
    /// authoritative state.
    pub(crate) engine: GameEngine,

    /// Active color theme used throughout the UI.
    ///
    /// Keeping the theme in the application state allows future settings
    /// screens to modify it without changing the renderer.
    pub(crate) theme: Theme,

    /// Current spell-slot count for spell levels 1 through 9.
    pub(crate) spell_slots_current: [u8; 9],

    /// Maximum spell-slot count for spell levels 1 through 9.
    pub(crate) spell_slots_max: [u8; 9],

    /// Temporary spell-slot count for spell levels 1 through 9.
    pub(crate) spell_slots_temp: [u8; 9],

    /// Current and maximum sorcery points.
    ///
    /// A value of `None` indicates the character does not have the Sorcery
    /// Points feature, so the UI hides that row completely.
    pub(crate) sorcery_points: Option<(u8, u8)>,

    /// Whether the inventory sidebar is open.
    pub(crate) inventory_open: bool,

    /// Index of the currently selected item in the inventory grid.
    pub(crate) inventory_selected: usize,

    /// Placeholder inventory items for the sidebar.
    ///
    /// This will be replaced by the real inventory from `storyforge-core` once
    /// character and item systems exist.
    pub(crate) inventory_items: Vec<String>,
}

/// Top-level screens the player can navigate between.
#[derive(Debug, Clone, Copy, PartialEq, Eq, Default)]
pub enum Screen {
    /// Interactive story view.
    #[default]
    Story,

    /// Character sheet.
    Character,

    /// Quest and event journal.
    Journal,

    /// World map.
    Map,

    /// Help and keyboard shortcuts.
    Help,
}

/// Indicates which panel of the interface currently receives keyboard input.
#[derive(Debug, Clone, Copy, PartialEq, Eq, Default)]
pub enum Focus {
    /// Main narrative panel. Will later support scrolling prose.
    #[default]
    Story,

    /// Action list or choice list.
    Actions,

    /// Character summary panel.
    Character,

    /// Quest tracker panel.
    Quest,

    /// Event or combat log.
    Log,

    /// Inventory sidebar.
    Inventory,
}
```

Then update `Default` for `App` to initialize the new fields:

```rust
impl Default for App {
    fn default() -> Self {
        let active_scene = ContentId::new("academy.scene.arrival")
            .expect("temporary built-in scene ID should be valid");

        Self {
            should_quit: false,
            screen: Screen::default(),
            focus: Focus::default(),
            selected_tab: 0,
            engine: GameEngine::new(GameState::new(active_scene), 42),
            theme: Theme::default(),
            spell_slots_current: [3, 2, 0, 0, 0, 0, 0, 0, 0],
            spell_slots_max: [4, 2, 0, 0, 0, 0, 0, 0, 0],
            spell_slots_temp: [0; 9],
            sorcery_points: Some((3, 3)),

            // Inventory starts closed with a small dummy list so the sidebar
            // can be rendered and navigated before real items exist.
            inventory_open: false,
            inventory_selected: 0,
            inventory_items: vec![
                "Wand".to_owned(),
                "Spellbook".to_owned(),
                "Potion".to_owned(),
                "Robe".to_owned(),
                "Key".to_owned(),
                "Scroll".to_owned(),
            ],
        }
    }
}
```

### Learning checkpoint

You should understand that `Focus` is a plain `Copy` enum. That means `App` can compare `self.focus == Focus::Actions` without borrowing, and the renderer can read it from a shared `&App`. The inventory list is owned by `App` because it is purely presentational placeholder data for now.

### How to verify

Run:

```powershell
cargo check -p storyforge-tui
```

This should compile. The new fields are not used yet, so no behavior changes.

### Next connection

Add helper methods that compute the next and previous focusable panel, taking the inventory-open flag into account.

---

File: `crates/storyforge-tui/src/app.rs`

## Step 3: Add focus navigation helpers

### Current state

`App` has no methods for changing focus. The `update` method will need them, and the renderer will need to know which panel is focused.

### Why this step matters

Focus movement is a ring, not a pair of `+1`/`-1` operations on an enum. Inventory is only in the ring when it is open. Keeping this logic in small helpers keeps `update` readable and makes the rules easy to test without a terminal.

### Before

`App` only has `run`, `handle_event`, `update`, and `visible_choice_count` methods. There is no `next_focus` or `select_focus`.

### What to change

Add three helpers inside `impl App`:

1. `next_focus`: returns the next focusable panel in the ring.
2. `previous_focus`: returns the previous focusable panel in the ring.
3. `select_focus`: sets `self.focus` and, when the new focus is a detail panel, syncs `selected_tab` so compact mode still shows the right panel.

Also add a small free function `detail_panel_index` that maps `Focus` to the compact tab index.

### Temporary MVP / debug behavior

Write unit tests for these helpers before wiring them to keys. This proves the ring order is correct with the inventory both open and closed.

### After

```rust
impl App {
    // ... existing methods ...

    /// Returns the next focusable panel in the dashboard ring.
    ///
    /// Inventory is only included while the sidebar is open. This prevents the
    /// focus ring from stopping on an invisible panel.
    fn next_focus(&self) -> Focus {
        match self.focus {
            Focus::Story => Focus::Actions,
            Focus::Actions => Focus::Character,
            Focus::Character => Focus::Quest,
            Focus::Quest => Focus::Log,
            Focus::Log => {
                if self.inventory_open {
                    Focus::Inventory
                } else {
                    Focus::Story
                }
            }
            Focus::Inventory => Focus::Story,
        }
    }

    /// Returns the previous focusable panel in the dashboard ring.
    fn previous_focus(&self) -> Focus {
        match self.focus {
            Focus::Story => {
                if self.inventory_open {
                    Focus::Inventory
                } else {
                    Focus::Log
                }
            }
            Focus::Actions => Focus::Story,
            Focus::Character => Focus::Actions,
            Focus::Quest => Focus::Character,
            Focus::Log => Focus::Quest,
            Focus::Inventory => Focus::Log,
        }
    }

    /// Changes the active focus and keeps the compact tab in sync.
    ///
    /// Compact mode only shows one detail panel at a time. When the player
    /// moves focus to a detail panel, `selected_tab` should update so the
    /// visible panel matches the focused one.
    fn select_focus(&mut self, focus: Focus) {
        self.focus = focus;
        if let Some(tab) = detail_panel_index(focus) {
            self.selected_tab = tab;
        }
    }

    /// Returns the number of visible choices for the current tab.
    fn visible_choice_count(&self) -> usize {
        2
    }
}

/// Maps detail-panel focus values to the compact tab index.
///
/// Story and Inventory are not part of the compact tab strip, so they return
/// `None`.
fn detail_panel_index(focus: Focus) -> Option<usize> {
    match focus {
        Focus::Actions => Some(0),
        Focus::Character => Some(1),
        Focus::Quest => Some(2),
        Focus::Log => Some(3),
        _ => None,
    }
}
```

### Learning checkpoint

You should understand why `next_focus` and `previous_focus` take `&self` and return a value instead of mutating `self.focus` directly. Rust ownership makes this safe: the caller can inspect the result before deciding to apply it, which is useful for tests and for guarded logic.

### How to verify

Add these tests at the bottom of `app.rs`, inside the existing `#[cfg(test)] mod tests` block:

```rust
#[test]
fn focus_next_should_cycle_through_panels_and_skip_closed_inventory() {
    let mut app = App::default();

    app.select_focus(Focus::Actions);
    assert_eq!(app.focus, Focus::Actions);
    assert_eq!(app.selected_tab, 0);

    app.select_focus(Focus::Character);
    assert_eq!(app.selected_tab, 1);

    app.select_focus(Focus::Quest);
    assert_eq!(app.selected_tab, 2);

    app.select_focus(Focus::Log);
    assert_eq!(app.selected_tab, 3);

    // Inventory is closed, so next from Log wraps back to Story.
    app.select_focus(app.next_focus());
    assert_eq!(app.focus, Focus::Story);
    // Story is not a detail panel, so selected_tab stays where it was.
    assert_eq!(app.selected_tab, 3);
}

#[test]
fn focus_previous_should_include_inventory_when_open() {
    let mut app = App::default();

    app.inventory_open = true;
    app.select_focus(Focus::Story);

    app.select_focus(app.previous_focus());
    assert_eq!(app.focus, Focus::Inventory);
}

#[test]
fn inventory_open_should_appear_in_the_focus_ring() {
    let app = App::default();

    assert_eq!(app.next_focus(), Focus::Actions);

    let mut app_with_inventory = App::default();
    app_with_inventory.inventory_open = true;
    app_with_inventory.focus = Focus::Log;

    assert_eq!(app_with_inventory.next_focus(), Focus::Inventory);
    assert_eq!(app_with_inventory.next_focus(), Focus::Story);
}
```

Run:

```powershell
cargo test -p storyforge-tui app
```

### Next connection

Wire the new actions to the focus helpers and to movement inside the focused panel.

---

File: `crates/storyforge-tui/src/app.rs`

## Step 4: Wire focus, directional movement, and inventory toggle

### Current state

`App::update` still handles the old `UiAction::Up` and `UiAction::Down` values. Those variants no longer exist, so the code will not compile after Step 1. The update method also does not know about `FocusNext`, `FocusPrevious`, `MoveLeft`, `MoveRight`, or `ToggleInventory`.

### Why this step matters

This is the central wiring step. It separates "which panel is active" from "what does the active panel do with a direction." The engine only receives a choice-selection command when the Actions panel is focused on the Story screen. The inventory only moves when the Inventory panel is focused. Left and right fall back to focus movement everywhere else.

### Before

`App::update` currently contains these movement arms:

```rust
UiAction::Down if self.screen == Screen::Story => {
    self.engine.dispatch(GameCommand::SelectNextChoice {
        choice_count: self.visible_choice_count(),
    });
}
UiAction::Up if self.screen == Screen::Story => {
    self.engine.dispatch(GameCommand::SelectPreviousChoice {
        choice_count: self.visible_choice_count(),
    });
}
UiAction::Down => {
    self.selected_tab = (self.selected_tab + 1) % 4;
}
UiAction::Up => {
    self.selected_tab = self.selected_tab.checked_sub(1).unwrap_or(3);
}
UiAction::Confirm | UiAction::None => {}
```

### What to change

Replace the movement arms with handlers for the new actions. The match order matters: the guarded arms for the Actions panel and the Inventory panel must come before the fallback `MoveLeft`/`MoveRight` and the catch-all `MoveUp`/`MoveDown` arms.

### Temporary MVP / debug behavior

Before rendering the inventory sidebar, you can verify this step by adding a temporary debug line to the footer in `ui.rs`. After Step 7, once the borders and the sidebar show the state, you should remove that debug line.

A minimal debug line looks like this:

```rust
spans.push(Span::styled(
    format!(" focus:{:?} ", app.focus),
    Style::default().fg(theme.focus),
));
```

Add it to the end of `render_footer` in `ui.rs`, after the sorcery-point span. It will print the current `Focus` value to the footer while you test.

### After

```rust
fn update(&mut self, action: UiAction) {
    match action {
        UiAction::OpenCharacter => self.screen = Screen::Character,

        UiAction::OpenJournal => self.screen = Screen::Journal,

        UiAction::OpenMap => self.screen = Screen::Map,

        UiAction::OpenHelp => self.screen = Screen::Help,

        UiAction::Back if self.screen != Screen::Story => {
            self.screen = Screen::Story;
        }

        UiAction::Back | UiAction::Quit => {
            self.should_quit = true;
        }

        // j/k move focus around the dashboard ring.
        UiAction::FocusNext => {
            let next = self.next_focus();
            self.select_focus(next);
        }
        UiAction::FocusPrevious => {
            let previous = self.previous_focus();
            self.select_focus(previous);
        }

        // When the Actions panel is focused on the Story screen, up/down
        // dispatch the engine choice-selection commands.
        UiAction::MoveUp
            if self.focus == Focus::Actions && self.screen == Screen::Story =>
        {
            self.engine.dispatch(GameCommand::SelectPreviousChoice {
                choice_count: self.visible_choice_count(),
            });
        }
        UiAction::MoveDown
            if self.focus == Focus::Actions && self.screen == Screen::Story =>
        {
            self.engine.dispatch(GameCommand::SelectNextChoice {
                choice_count: self.visible_choice_count(),
            });
        }

        // When the Inventory panel is focused, WASD moves inside the grid.
        UiAction::MoveUp if self.focus == Focus::Inventory => {
            self.move_inventory_up();
        }
        UiAction::MoveDown if self.focus == Focus::Inventory => {
            self.move_inventory_down();
        }
        UiAction::MoveLeft if self.focus == Focus::Inventory => {
            self.move_inventory_left();
        }
        UiAction::MoveRight if self.focus == Focus::Inventory => {
            self.move_inventory_right();
        }

        // Left/right are the fallback direction. If the focused panel has no
        // horizontal layout, they move focus instead.
        UiAction::MoveLeft => {
            let previous = self.previous_focus();
            self.select_focus(previous);
        }
        UiAction::MoveRight => {
            let next = self.next_focus();
            self.select_focus(next);
        }

        // Any other vertical movement does nothing for now. Story scrolling,
        // character sheet paging, and log scrolling will live here later.
        UiAction::MoveUp | UiAction::MoveDown => {}

        // i opens the inventory sidebar and focuses it. Pressing i again
        // closes it and returns focus to the Actions panel.
        UiAction::ToggleInventory => {
            self.inventory_open = !self.inventory_open;
            if self.inventory_open {
                self.select_focus(Focus::Inventory);
            } else {
                self.select_focus(Focus::Actions);
            }
        }

        UiAction::Confirm | UiAction::None => {}
    }
}
```

Add the inventory movement helpers inside `impl App`:

```rust
/// Moves the inventory selection up one row, clamping at the top.
fn move_inventory_up(&mut self) {
    self.inventory_selected = self.inventory_selected.saturating_sub(INVENTORY_COLUMNS);
}

/// Moves the inventory selection down one row, clamping at the last item.
fn move_inventory_down(&mut self) {
    let max_index = self.inventory_items.len().saturating_sub(1);
    self.inventory_selected = self
        .inventory_selected
        .saturating_add(INVENTORY_COLUMNS)
        .min(max_index);
}

/// Moves the inventory selection left one slot, clamping at the first item.
fn move_inventory_left(&mut self) {
    self.inventory_selected = self.inventory_selected.saturating_sub(1);
}

/// Moves the inventory selection right one slot, clamping at the last item.
fn move_inventory_right(&mut self) {
    let max_index = self.inventory_items.len().saturating_sub(1);
    self.inventory_selected = (self.inventory_selected + 1).min(max_index);
}
```

### Learning checkpoint

You should understand why the match arms are ordered this way. Rust evaluates match arms from top to bottom. The guarded arms for `MoveUp`/`MoveDown` handle the Actions and Inventory panels first; the final `MoveUp | MoveDown => {}` arm catches every other vertical movement and makes it a no-op. If you put the catch-all first, the guarded arms would never run.

### How to verify

Update the existing `app.rs` tests and add new ones:

```rust
#[cfg(test)]
mod tests {
    use crossterm::event::{Event, KeyCode, KeyEvent, KeyModifiers};

    use super::{App, Focus, Screen};
    use crate::action::UiAction;

    #[test]
    fn q_should_request_quit() {
        let mut app = App::default();
        let key = KeyEvent::new(KeyCode::Char('q'), KeyModifiers::NONE);

        app.handle_event(&Event::Key(key));

        assert!(app.should_quit);
    }

    #[test]
    fn j_and_k_should_cycle_focus_and_update_compact_tab() {
        let mut app = App::default();

        // Default focus is Story, so the first detail panel is Actions.
        app.update(UiAction::FocusNext);
        assert_eq!(app.focus, Focus::Actions);
        assert_eq!(app.selected_tab, 0);

        app.update(UiAction::FocusNext);
        assert_eq!(app.focus, Focus::Character);
        assert_eq!(app.selected_tab, 1);

        app.update(UiAction::FocusNext);
        assert_eq!(app.focus, Focus::Quest);
        assert_eq!(app.selected_tab, 2);

        app.update(UiAction::FocusNext);
        assert_eq!(app.focus, Focus::Log);
        assert_eq!(app.selected_tab, 3);

        // Inventory is closed, so focus wraps back to Story.
        app.update(UiAction::FocusNext);
        assert_eq!(app.focus, Focus::Story);
    }

    #[test]
    fn move_down_on_actions_with_story_should_select_next_choice() {
        let mut app = App::default();
        app.select_focus(Focus::Actions);

        app.update(UiAction::MoveDown);

        assert_eq!(app.engine.state().selected_choice, 1);
    }

    #[test]
    fn move_up_on_actions_with_story_should_select_previous_choice() {
        let mut app = App::default();
        app.select_focus(Focus::Actions);
        app.engine.dispatch(storyforge_core::GameCommand::SelectNextChoice {
            choice_count: 2,
        });
        assert_eq!(app.engine.state().selected_choice, 1);

        app.update(UiAction::MoveUp);

        assert_eq!(app.engine.state().selected_choice, 0);
    }

    #[test]
    fn move_right_on_story_should_fallback_to_next_focus() {
        let mut app = App::default();

        app.update(UiAction::MoveRight);

        assert_eq!(app.focus, Focus::Actions);
    }

    #[test]
    fn move_left_on_story_should_fallback_to_previous_focus() {
        let mut app = App::default();

        app.update(UiAction::MoveLeft);

        // Inventory is closed, so the previous panel from Story is Log.
        assert_eq!(app.focus, Focus::Log);
    }

    #[test]
    fn i_should_toggle_inventory_and_focus_it() {
        let mut app = App::default();

        app.update(UiAction::ToggleInventory);
        assert!(app.inventory_open);
        assert_eq!(app.focus, Focus::Inventory);

        app.update(UiAction::ToggleInventory);
        assert!(!app.inventory_open);
        assert_eq!(app.focus, Focus::Actions);
    }

    #[test]
    fn inventory_wasd_should_move_selection_in_the_grid() {
        let mut app = App::default();
        app.select_focus(Focus::Inventory);

        app.update(UiAction::MoveDown);
        assert_eq!(app.inventory_selected, 2);

        app.update(UiAction::MoveRight);
        assert_eq!(app.inventory_selected, 3);

        app.update(UiAction::MoveUp);
        assert_eq!(app.inventory_selected, 1);

        app.update(UiAction::MoveLeft);
        assert_eq!(app.inventory_selected, 0);
    }

    #[test]
    fn inventory_movement_should_clamp_at_the_edges() {
        let mut app = App::default();
        app.select_focus(Focus::Inventory);

        app.update(UiAction::MoveLeft);
        assert_eq!(app.inventory_selected, 0);

        for _ in 0..10 {
            app.update(UiAction::MoveDown);
        }
        assert_eq!(app.inventory_selected, 5);
    }
}
```

Run:

```powershell
cargo test -p storyforge-tui app
```

### Next connection

The logic now works, but the renderer still shows the old borders and does not know about the inventory panel. The next step makes the focused panel visible and adds the sidebar.

---

File: `crates/storyforge-tui/src/ui.rs`

## Step 5: Highlight the focused panel and add the inventory sidebar

### Current state

`ui.rs` draws the dashboard, but the border color is based on `app.screen` for the story panel and the `Actions`/`Log` focus values for the side panels. The Character panel uses `app.screen == Screen::Character` and the Quest panel uses `app.screen == Screen::Journal`, which is a leftover mismatch. The header still shows the old `c/l/m/?` hint and does not mention inventory or focus.

### Why this step matters

The player needs to see which panel has the keyboard. Every panel should use the same `app.focus` source. The header should teach the new controls: `j`/`k` for focus, `w`/`a`/`s`/`d` for movement, and `i` for inventory.

### Before

`render_story_panel` decides border color like this:

```rust
let border_color = if app.screen == Screen::Story {
    theme.focus
} else {
    theme.accent
};
```

`render_character_panel` uses `app.screen == Screen::Character`, and `render_quest_panel` uses `app.screen == Screen::Journal`.

`render_header` shows:

```rust
Span::styled("c/l/m/?", Style::default().fg(theme.focus)),
Span::styled(" screens", Style::default().fg(theme.muted)),
```

### What to change

1. Make every panel use `app.focus == Focus::<Panel>` for its border color.
2. Update the header hint to include `i` and `j`/`k`.
3. Update the Actions panel hint to reflect the new controls.

### Temporary MVP / debug behavior

If you added the temporary `focus:{:?}` debug span to the footer in Step 4, keep it until you can see the focus border change. You will remove it in Step 7.

### After

Update the header:

```rust
fn render_header(frame: &mut Frame, area: Rect, theme: Theme) {
    let hint = Line::from(vec![
        Span::styled("q/Esc", Style::default().fg(theme.focus)),
        Span::styled(" quit  ", Style::default().fg(theme.muted)),
        Span::styled("c/l/m/?", Style::default().fg(theme.focus)),
        Span::styled(" screens  ", Style::default().fg(theme.muted)),
        Span::styled("i", Style::default().fg(theme.focus)),
        Span::styled(" inv  ", Style::default().fg(theme.muted)),
        Span::styled("j/k", Style::default().fg(theme.focus)),
        Span::styled(" focus", Style::default().fg(theme.muted)),
    ]);

    // ... rest of the function is unchanged ...
}
```

Update `render_story_panel`:

```rust
fn render_story_panel(frame: &mut Frame, area: Rect, app: &App) {
    let theme = app.theme;

    let text = match app.screen {
        Screen::Story => "The academy doors stand open. What do you do?",
        Screen::Character => "Character sheet will appear here.",
        Screen::Journal => "Journal entries will appear here.",
        Screen::Map => "Map will appear here.",
        Screen::Help => "Help text will appear here.",
    };

    let border_color = if app.focus == Focus::Story {
        theme.focus
    } else {
        theme.accent
    };

    // ... rest of the function is unchanged ...
}
```

Update `render_character_panel`:

```rust
fn render_character_panel(frame: &mut Frame, area: Rect, app: &App) {
    let theme = app.theme;
    let focused = app.focus == Focus::Character;

    // ... rest of the function is unchanged ...
}
```

Update `render_quest_panel`:

```rust
fn render_quest_panel(frame: &mut Frame, area: Rect, app: &App) {
    let theme = app.theme;
    let focused = app.focus == Focus::Quest;

    // ... rest of the function is unchanged ...
}
```

`render_actions_panel` already uses `app.focus == Focus::Actions`, but update its hint text so the player knows the new controls. On the Story screen, the panel should show:

```rust
format!("\n{first}\n{second}\n\n[w/s] Move  [j/k] Focus  [Enter] Confirm")
```

### Learning checkpoint

You should understand that `render_*` functions only read `app.focus` by shared reference (`&App`). They do not own the focus state, so they cannot accidentally change it. The border color is pure output from pure input.

### How to verify

Run:

```powershell
cargo run -p storyforge-tui
```

At a large terminal size, press `j` and `k`. You should see the border color move around the panels. If you kept the temporary footer debug span, you will also see the `Focus` variant printed there.

### Next connection

Now add the actual inventory sidebar layout and render function.

---

File: `crates/storyforge-tui/src/ui.rs`

## Step 6: Add the inventory sidebar layout and render function

### Current state

There is no inventory rendering code. `render_standard_body` and `render_compact_body` always split the body area between the story panel and the detail panels.

### Why this step matters

The inventory is a sidebar that takes space away from the rest of the dashboard. The split must be responsive, just like the rest of the layout, so the same rendering code works at large, compact, and small sizes.

### Before

`render_standard_body` is:

```rust
fn render_standard_body(frame: &mut Frame, area: Rect, app: &App) {
    let parts = Layout::default()
        .direction(Direction::Vertical)
        .constraints([Constraint::Percentage(45), Constraint::Percentage(55)])
        .split(area);

    render_story_panel(frame, parts[0], app);
    render_detail_grid(frame, parts[1], app);
}
```

`render_compact_body` is:

```rust
fn render_compact_body(frame: &mut Frame, area: Rect, app: &App) {
    let parts = Layout::default()
        .direction(Direction::Vertical)
        .constraints([
            Constraint::Percentage(40),
            Constraint::Length(5),
            Constraint::Min(8),
        ])
        .split(area);

    render_story_panel(frame, parts[0], app);
    render_actions_panel(frame, parts[1], app);
    render_selected_tab(frame, parts[2], app);
}
```

### What to change

1. Add an `inventory_split` helper that returns the main area and the inventory area based on the current layout mode.
2. When `app.inventory_open` is true, both `render_standard_body` and `render_compact_body` should split the body area horizontally and render the inventory panel on the right.
3. Add `render_inventory_panel` that draws the placeholder items and highlights the selected one.

### Temporary MVP / debug behavior

The inventory items are still the dummy list from Step 2. The selected item is just a number. This proves the layout and movement wiring before real items exist.

### After

Add the layout helper:

```rust
/// Splits a body area into a main region and an inventory sidebar.
///
/// The sidebar width follows the requested ratios:
/// - Standard layout: one third of the width.
/// - Compact layout: one half of the width.
/// - Too-small layout: seven eighths of the width.
///
/// The too-small branch is a fallback. The `TooSmall` warning normally
/// renders instead of the body, so this branch is rarely visible.
fn inventory_split(area: Rect, mode: LayoutMode) -> [Rect; 2] {
    let inventory_width = match mode {
        LayoutMode::Standard => area.width / 3,
        LayoutMode::Compact => area.width / 2,
        LayoutMode::TooSmall => area.width * 7 / 8,
    };
    let main_width = area.width.saturating_sub(inventory_width);

    Layout::horizontal([
        Constraint::Length(main_width),
        Constraint::Length(inventory_width),
    ])
    .areas(area)
}
```

Update `render_standard_body`:

```rust
fn render_standard_body(frame: &mut Frame, area: Rect, app: &App) {
    if app.inventory_open {
        let [main_area, inventory_area] = inventory_split(area, mode_for(frame.area()));

        let [story_area, detail_area] = Layout::vertical([
            Constraint::Percentage(45),
            Constraint::Percentage(55),
        ])
        .areas(main_area);

        render_story_panel(frame, story_area, app);
        render_detail_grid(frame, detail_area, app);
        render_inventory_panel(frame, inventory_area, app);
    } else {
        let parts = Layout::vertical([
            Constraint::Percentage(45),
            Constraint::Percentage(55),
        ])
        .split(area);

        render_story_panel(frame, parts[0], app);
        render_detail_grid(frame, parts[1], app);
    }
}
```

Update `render_compact_body`:

```rust
fn render_compact_body(frame: &mut Frame, area: Rect, app: &App) {
    if app.inventory_open {
        let [main_area, inventory_area] = inventory_split(area, mode_for(frame.area()));

        let parts = Layout::vertical([
            Constraint::Percentage(40),
            Constraint::Length(5),
            Constraint::Min(8),
        ])
        .areas(main_area);

        render_story_panel(frame, parts[0], app);
        render_actions_panel(frame, parts[1], app);
        render_selected_tab(frame, parts[2], app);
        render_inventory_panel(frame, inventory_area, app);
    } else {
        let parts = Layout::vertical([
            Constraint::Percentage(40),
            Constraint::Length(5),
            Constraint::Min(8),
        ])
        .split(area);

        render_story_panel(frame, parts[0], app);
        render_actions_panel(frame, parts[1], app);
        render_selected_tab(frame, parts[2], app);
    }
}
```

Add the inventory panel:

```rust
/// Renders the inventory sidebar with a temporary placeholder item list.
fn render_inventory_panel(frame: &mut Frame, area: Rect, app: &App) {
    let theme = app.theme;
    let focused = app.focus == Focus::Inventory;

    let mut lines = vec![String::new()];
    for (index, item) in app.inventory_items.iter().enumerate() {
        let marker = if index == app.inventory_selected {
            ">"
        } else {
            " "
        };
        lines.push(format!("{marker} {item}"));
    }
    lines.push(String::new());
    lines.push("[w/a/s/d] Move  [i] Close".to_owned());

    render_pane(frame, area, "Inventory", lines.join("\n"), theme, focused);
}
```

Make sure the `Focus` enum is imported at the top of `ui.rs`:

```rust
use crate::{
    app::{App, Focus, Screen},
    layout::{LayoutMode, mode_for},
    theme::Theme,
};
```

### Learning checkpoint

You should understand why `inventory_split` is a pure function of the body area and the layout mode. It does not store rectangles in `App`, so resizing the terminal while the inventory is open cannot leave stale rectangles behind. Ratatui recalculates the layout every frame.

### How to verify

Run:

```powershell
cargo run -p storyforge-tui
```

Press `i` at both standard and compact sizes. The inventory sidebar should appear on the right. At standard size it should take roughly one third of the width. At compact size it should take roughly half. At a very small size above the minimum, it should take most of the width, though the too-small warning may hide it.

### Next connection

The inventory panel is visible, but the controls and panel hints need to match. Clean up the temporary debug span and run the full verification suite.

---

File: `crates/storyforge-tui/src/ui.rs`

## Step 7: Remove the temporary focus debug span and update the footer hint

### Current state

If you added the temporary `focus:{:?}` span to `render_footer` in Step 4, it is now redundant because the focused panel border shows the same information. The footer can also give a short reminder of the new controls.

### Why this step matters

Temporary debug output is useful while learning, but it should not ship. The final footer should be useful to the player, not to the developer.

### Before

The footer currently has the temporary debug span, or no movement hint at all.

### What to change

Remove the temporary debug span. Optionally, add a short hint like `[j/k focus]` to the footer if you have room. Keep the existing spell-slot display intact.

### After

```rust
fn render_footer(frame: &mut Frame, area: Rect, app: &App) {
    let theme = app.theme;
    let mut spans: Vec<Span> = Vec::new();

    // Build the compact spell-slot summary.
    spans.push(Span::styled("SLOTS ", Style::default().fg(theme.muted)));
    // ... existing spell-slot loop remains unchanged ...

    // Add a short control reminder after the spell slots.
    spans.push(Span::styled(" | ", Style::default().fg(theme.muted)));
    spans.push(Span::styled("j/k", Style::default().fg(theme.focus)));
    spans.push(Span::styled(" focus  ", Style::default().fg(theme.muted)));
    spans.push(Span::styled("w/a/s/d", Style::default().fg(theme.focus)));
    spans.push(Span::styled(" move", Style::default().fg(theme.muted)));

    // ... existing block and paragraph creation remain unchanged ...
}
```

If you prefer not to add the hint, simply remove the temporary debug span and leave the footer as it was before Step 4.

### Learning checkpoint

You should understand that the footer is just another reader of `app` state. It never changes `app`, and it should not contain leftover debugging code.

### How to verify

Run:

```powershell
cargo run -p storyforge-tui
```

The footer should no longer contain a `focus:Story` style debug span. The focused panel border should still show the active panel.

### Next connection

Run the full verification at the end of this guide to confirm everything is clean before moving on to guide 06.

---

## Full verification

Run the full verification commands and manual checks:

```powershell
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features --locked -- -D warnings
cargo test --workspace --all-features --locked
cargo run -p storyforge-tui
```

Manual checks at a large terminal size (at least 120x36):

1. Press `j` and `k`. The bright border should move around the Story, Actions, Character, Quest, and Log panels in order.
2. Press `i`. The inventory sidebar should open on the right, taking about one third of the width. Focus should move to the inventory panel.
3. Press `w`, `s`, `a`, and `d` while the inventory is focused. The `>` marker should move through the placeholder items.
4. Press `i` again. The inventory should close and focus should return to the Actions panel.
5. Press `j` to focus the Actions panel, then press `s` and `w` on the Story screen. The `>` marker in the Actions panel should move between the two temporary choices.
6. Press `m` to open the Map screen. The story panel text should change to "Map will appear here." and the detail panels should still render.
7. Press `Esc` to return to the Story screen.
8. Press `q` to quit. The terminal should restore normally.

Manual checks at a compact terminal size (80x24):

1. Press `j` and `k`. The bottom tab should change as focus moves through the detail panels, because `select_focus` keeps `selected_tab` in sync.
2. Press `i`. The inventory sidebar should open on the right, taking about half of the width.
3. Press `w`/`a`/`s`/`d` to move inside the inventory.
4. Press `i` again to close the inventory.
5. Press `q` to quit.

## Common mistakes

- Forgetting to update the `action.rs` tests after renaming `Up`/`Down` to `MoveUp`/`MoveDown`.
- Putting the catch-all `MoveUp | MoveDown => {}` arm before the guarded arms for the Actions or Inventory panels. The guarded arms will never match.
- Using `app.screen` to decide the Character or Quest panel border color instead of `app.focus`.
- Storing the inventory sidebar rectangle in `App` instead of recalculating it every frame. The terminal can resize at any time.
- Forgetting to remove the temporary `focus:{:?}` debug span from the footer.
- Making the inventory panel take focus even when it is closed. The focus ring should skip it when `inventory_open` is false.

## Acceptance check

- `j` and `k` move focus to the next or previous layout panel.
- Arrow keys and `w`/`a`/`s`/`d` move inside the focused panel.
- Left and right fall back to focus movement when the focused panel has no horizontal layout.
- The Actions panel still dispatches `SelectNextChoice`/`SelectPreviousChoice` to the engine when it is focused on the Story screen.
- The Inventory panel opens with `i`, focuses itself, and supports full `w`/`a`/`s`/`d` movement.
- The inventory sidebar takes one third of the width in standard layout, one half in compact layout, and seven eighths in the too-small fallback.
- The focused panel is highlighted by its border color.
- `cargo fmt`, `cargo clippy`, and `cargo test --workspace` all pass.
