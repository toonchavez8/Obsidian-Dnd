			# 05: Build the command and event engine

## Result

The TUI will stop owning gameplay selection rules. It will translate player input into a `GameCommand`, send that command to `storyforge-core`, receive `GameEvent` values, and render the updated `GameState`.

This guide keeps the first engine slice small. That is deliberate. The first supported gameplay commands only move the selected story choice up and down. That same command/event path later supports travel, dialogue, checks, combat, inventory, and character creation.

The current repo state matters:

- `storyforge-core` currently only exposes `engine_name()`.
- `storyforge-content` currently only exposes `schema_version()`.
- `storyforge-tui` already has `UiAction`, `App`, and rendering.

This guide builds the first real core reducer without touching content loading yet.

## Flow to prove

The flow uses these functions and types:

```text
UiAction::from
    -> App::update
    -> GameEngine::dispatch
    -> handle_command
    -> apply_events
    -> ui::render reads GameEngine::state
```

First prove the engine receives a command. Then prove it emits events. Then prove events update state. Then connect the TUI.

---

File: `crates/storyforge-core/Cargo.toml`

## Step 1: Add deterministic RNG dependencies

### Current state

`storyforge-core` already depends on workspace `serde` and `thiserror`:

```toml
[dependencies]
serde.workspace = true
thiserror.workspace = true
```

It does not yet have a random number generator.

### Why this step matters

The game sheet requires visible dice and reproducible behavior. Randomness should belong to the engine, not the renderer. Adding the RNG now establishes the rule before dice checks exist.

### Before

```toml
[dependencies]
serde.workspace = true
thiserror.workspace = true
```

### What to change

Add `rand` and `rand_chacha`. Use ChaCha because it is deterministic and easy to seed in tests.

### Temporary MVP / debug behavior

Do not roll dice yet. First expose a tiny `next_random_u32` method in Step 5 so you can prove the engine owns randomness.

### After

```toml
[dependencies]
rand = "0.9"
rand_chacha = { version = "0.9", features = ["serde"] }
serde.workspace = true
thiserror.workspace = true
```

If `cargo add` is preferred, run:

```powershell
cargo add -p storyforge-core rand rand_chacha --features rand_chacha/serde
```

### Learning checkpoint

You should understand that deterministic game systems need a stored seed. The TUI may display a roll, but it should not create the roll.

### How to verify

Run:

```powershell
cargo check -p storyforge-core
```

### Next connection

Next, create a stable ID type so state and events can refer to content without using display names.

---

File: `crates/storyforge-core/src/id.rs`

## Step 2: Add validated content IDs

### Current state

There is no `id.rs` yet. The only core file is `lib.rs`, and it currently contains `engine_name()`.

### Why this step matters

The game sheet says content IDs are stable and namespaced, such as `academy.scene.arrival`. Display text can change; IDs in saves must not silently change meaning.

### Before

Create a new file:

```text
crates/storyforge-core/src/id.rs
```

### What to change

Add a `ContentId` wrapper around `String`. It validates IDs at construction and during Serde deserialization.

### Temporary MVP / debug behavior

Start by validating only the ID shape:

1. It is not empty.
2. It contains at least one `.` namespace separator.
3. No namespace segment is empty.
4. It uses lowercase ASCII letters, digits, `.`, `_`, or `-`.

Do not validate whether the ID exists in a campaign yet. Guide 06 does that.

### After

```rust
use std::{fmt, str::FromStr};

use serde::{Deserialize, Deserializer, Serialize, de::Error as _};

#[derive(Debug, Clone, PartialEq, Eq, PartialOrd, Ord, Hash, Serialize)]
#[serde(transparent)]
pub struct ContentId(String);

impl ContentId {
    /// Creates a validated namespaced content identifier.
    ///
    /// # Errors
    ///
    /// Returns [`IdError`] when the value is empty, lacks a namespace, or
    /// contains unsupported characters.
    pub fn new(value: impl Into<String>) -> Result<Self, IdError> {
        let value = value.into();
        let has_namespace = value.contains('.') && !value.split('.').any(str::is_empty);
        let valid_characters = value.chars().all(|character| {
            character.is_ascii_lowercase()
                || character.is_ascii_digit()
                || matches!(character, '.' | '_' | '-')
        });

        if value.is_empty() || !has_namespace || !valid_characters {
            return Err(IdError::Invalid(value));
        }

        Ok(Self(value))
    }

    #[must_use]
    pub fn as_str(&self) -> &str {
        &self.0
    }
}

impl fmt::Display for ContentId {
    fn fmt(&self, formatter: &mut fmt::Formatter<'_>) -> fmt::Result {
        self.0.fmt(formatter)
    }
}

impl FromStr for ContentId {
    type Err = IdError;

    fn from_str(value: &str) -> Result<Self, Self::Err> {
        Self::new(value)
    }
}

impl<'de> Deserialize<'de> for ContentId {
    fn deserialize<D>(deserializer: D) -> Result<Self, D::Error>
    where
        D: Deserializer<'de>,
    {
        let value = String::deserialize(deserializer)?;
        Self::new(value).map_err(D::Error::custom)
    }
}

#[derive(Debug, Clone, PartialEq, Eq, thiserror::Error)]
pub enum IdError {
    #[error("content ID `{0}` must be lowercase, namespaced, and ASCII")]
    Invalid(String),
}
```

### Learning checkpoint

You should understand why `ContentId::new` returns `Result`. Invalid IDs are data errors, not panics. Rust makes that failure explicit.

### How to verify

You cannot compile yet until `lib.rs` exports the module in Step 6. Continue to the next step.

### Next connection

Now create the game state that stores an active scene ID and selected choice.

---

File: `crates/storyforge-core/src/state.rs`

## Step 3: Add the first game state and event types

### Current state

There is no `GameState` yet. The TUI currently holds demo UI values, but the selected story choice does not exist in core.

### Why this step matters

State is what the game remembers. Events are how the engine explains what changed. Later, the log, autosave policy, tests, and animations can all read the same event list.

### Before

Create a new file:

```text
crates/storyforge-core/src/state.rs
```

### What to change

Add a minimal `GameState` and a minimal `GameEvent` enum.

### Temporary MVP / debug behavior

Store the full `event_log` for now. Later you may cap or split logs, but the complete vector is useful while learning and debugging.

### After

```rust
use serde::{Deserialize, Serialize};

use crate::ContentId;

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct GameState {
    pub active_scene: ContentId,
    pub selected_choice: usize,
    pub turn: u64,
    pub event_log: Vec<GameEvent>,
}

impl GameState {
    #[must_use]
    pub fn new(active_scene: ContentId) -> Self {
        Self {
            active_scene,
            selected_choice: 0,
            turn: 0,
            event_log: Vec::new(),
        }
    }
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub enum GameEvent {
    ChoiceSelectionChanged { previous: usize, current: usize },
    CommandRejected { reason: String },
    TurnAdvanced { turn: u64 },
}
```

### Learning checkpoint

You should understand that `GameState` stores facts. `GameEvent` describes changes. The renderer reads facts; the log displays changes.

### How to verify

Compilation waits until `lib.rs` is updated in Step 6.

### Next connection

Next define the commands that ask the engine to change state.

---

File: `crates/storyforge-core/src/command.rs`

## Step 4: Add explicit game commands

### Current state

There is no command type yet. The TUI currently handles Up and Down as UI-only behavior.

### Why this step matters

A command describes player intent in game terms. It should not contain keyboard keys, terminal rectangles, or widget focus.

### Before

Create a new file:

```text
crates/storyforge-core/src/command.rs
```

### What to change

Add a `GameCommand` enum with two choice-navigation commands.

### Temporary MVP / debug behavior

Pass `choice_count` in the command for now because content loading does not exist yet. In guide 06, this count can come from the active loaded scene.

### After

```rust
#[derive(Debug, Clone, PartialEq, Eq)]
pub enum GameCommand {
    SelectNextChoice { choice_count: usize },
    SelectPreviousChoice { choice_count: usize },
}
```

### Learning checkpoint

You should understand that `GameCommand` is not `Copy`. Keeping it `Clone` only is the better default because future commands may contain owned strings, vectors, or boxed subsystem commands.

### How to verify

Compilation waits until `lib.rs` is updated in Step 6.

### Next connection

Now add the reducer: one function decides events, another applies events.

---

File: `crates/storyforge-core/src/engine.rs`

## Step 5: Build the reducer and engine wrapper

### Current state

There is no engine module. `storyforge-core` does not yet own state transitions.

### Why this step matters

This is the step to slow down on. It separates:

- `handle_command`: read state and command, then decide which events should happen.
- `apply_events`: mutate state by applying those events.
- `GameEngine::dispatch`: run both steps and remember the event log.

That separation makes replay, save migration, scripted tests, and debugging easier.

### Before

Create a new file:

```text
crates/storyforge-core/src/engine.rs
```

### What to change

Add `GameEngine`, `handle_command`, and `apply_events`.

### Temporary MVP / debug behavior

First dispatch a command and log the returned event list in a test. Then check that state changed. That proves data reaches the engine before you connect the TUI.

### After

```rust
use rand::{RngCore, SeedableRng};
use rand_chacha::ChaCha8Rng;

use crate::{GameCommand, GameEvent, GameState};

#[derive(Debug)]
pub struct GameEngine {
    state: GameState,
    rng: ChaCha8Rng,
}

impl GameEngine {
    #[must_use]
    pub fn new(state: GameState, seed: u64) -> Self {
        Self {
            state,
            rng: ChaCha8Rng::seed_from_u64(seed),
        }
    }

    #[must_use]
    pub fn state(&self) -> &GameState {
        &self.state
    }

    pub fn dispatch(&mut self, command: GameCommand) -> Vec<GameEvent> {
        let events = handle_command(&self.state, command);
        apply_events(&mut self.state, &events);
        self.state.event_log.extend(events.iter().cloned());
        events
    }

    #[must_use]
    pub fn next_random_u32(&mut self) -> u32 {
        self.rng.next_u32()
    }
}

#[must_use]
pub fn handle_command(state: &GameState, command: GameCommand) -> Vec<GameEvent> {
    let choice_count = match command {
        GameCommand::SelectNextChoice { choice_count }
        | GameCommand::SelectPreviousChoice { choice_count } => choice_count,
    };

    if choice_count == 0 {
        return vec![GameEvent::CommandRejected {
            reason: "the active scene has no choices".to_owned(),
        }];
    }

    let previous = state.selected_choice;
    let current = match command {
        GameCommand::SelectNextChoice { .. } => (previous + 1) % choice_count,
        GameCommand::SelectPreviousChoice { .. } => {
            previous.checked_sub(1).unwrap_or(choice_count - 1)
        }
    };
    let next_turn = state.turn.saturating_add(1);

    vec![
        GameEvent::ChoiceSelectionChanged { previous, current },
        GameEvent::TurnAdvanced { turn: next_turn },
    ]
}

pub fn apply_events(state: &mut GameState, events: &[GameEvent]) {
    for event in events {
        match event {
            GameEvent::ChoiceSelectionChanged { current, .. } => {
                state.selected_choice = *current;
            }
            GameEvent::TurnAdvanced { turn } => {
                state.turn = *turn;
            }
            GameEvent::CommandRejected { .. } => {}
        }
    }
}
```

### Learning checkpoint

You should understand why rejection is an event instead of a silent no-op. If a command fails, the player, log, test, or debug view should be able to explain why.

### How to verify

Compilation waits until Step 6. After exporting the module, reducer tests will verify this behavior.

### Next connection

Expose the new modules from `lib.rs` so other crates and tests can use them.

---

File: `crates/storyforge-core/src/lib.rs`

## Step 6: Export the engine API

### Current state

`lib.rs` currently contains:

```rust
//! Deterministic rules and state transitions for Storyforge campaigns.

/// Returns the engine name used in diagnostics.
#[must_use]
pub const fn engine_name() -> &'static str {
    "Storyforge"
}
```

There is also a small test for `engine_name`.

### Why this step matters

Rust modules are private by default. Adding files does nothing until `lib.rs` declares and exports them.

### Before

Keep the crate-level docs and `engine_name()` unless you deliberately decide it is obsolete. It is still useful for diagnostics, so keep it.

### What to change

Declare the new modules and publicly re-export the types the TUI and tests need.

### Temporary MVP / debug behavior

Export only the small public surface needed now. You can keep helper details private inside their modules.

### After

```rust
//! Deterministic rules and state transitions for Storyforge campaigns.

mod command;
mod engine;
mod id;
mod state;

pub use command::GameCommand;
pub use engine::{GameEngine, apply_events, handle_command};
pub use id::{ContentId, IdError};
pub use state::{GameEvent, GameState};

/// Returns the engine name used in diagnostics.
#[must_use]
pub const fn engine_name() -> &'static str {
    "Storyforge"
}
```

Keep or update the existing `engine_name_should_be_stable` test below this.

### Learning checkpoint

You should understand the difference between `mod engine;` and `pub use engine::GameEngine;`. The first makes the file part of the crate. The second chooses what outside crates can access.

### How to verify

Run:

```powershell
cargo check -p storyforge-core
```

### Next connection

Now add tests that prove command handling before the TUI uses it.

---

File: `crates/storyforge-core/src/engine.rs`

## Step 7: Test the reducer before connecting the TUI

### Current state

`engine.rs` now contains the reducer, but no tests yet.

### Why this step matters

The engine should be correct without a terminal. If a rule fails, you want a small core test to fail instead of debugging a full TUI run.

### Before

Add tests at the bottom of `engine.rs`.

### What to change

Test wrapping selection and rejected empty choice lists.

### Temporary MVP / debug behavior

First call `dispatch`, inspect the returned events, and then inspect state. That proves both event creation and event application.

### After

```rust
#[cfg(test)]
mod tests {
    use crate::{ContentId, GameCommand, GameEngine, GameEvent, GameState};

    fn engine() -> Result<GameEngine, crate::IdError> {
        let scene = ContentId::new("academy.scene.arrival")?;
        Ok(GameEngine::new(GameState::new(scene), 42))
    }

    #[test]
    fn next_choice_should_advance_selection() -> Result<(), crate::IdError> {
        let mut engine = engine()?;

        let events = engine.dispatch(GameCommand::SelectNextChoice { choice_count: 3 });

        assert!(matches!(
            events.as_slice(),
            [
                GameEvent::ChoiceSelectionChanged { previous: 0, current: 1 },
                GameEvent::TurnAdvanced { turn: 1 }
            ]
        ));
        assert_eq!(engine.state().selected_choice, 1);
        Ok(())
    }

    #[test]
    fn previous_choice_should_wrap_at_start() -> Result<(), crate::IdError> {
        let mut engine = engine()?;

        engine.dispatch(GameCommand::SelectPreviousChoice { choice_count: 3 });

        assert_eq!(engine.state().selected_choice, 2);
        Ok(())
    }

    #[test]
    fn empty_choice_list_should_emit_rejection() -> Result<(), crate::IdError> {
        let mut engine = engine()?;

        let events = engine.dispatch(GameCommand::SelectNextChoice { choice_count: 0 });

        assert!(matches!(
            events.as_slice(),
            [GameEvent::CommandRejected { reason }]
                if reason == "the active scene has no choices"
        ));
        assert_eq!(engine.state().selected_choice, 0);
        Ok(())
    }
}
```

### Learning checkpoint

You should understand that `handle_command` does not mutate state by itself. The state changes only after `apply_events` runs inside `dispatch`.

### How to verify

Run:

```powershell
cargo test -p storyforge-core engine
```

### Next connection

Now connect the TUI to the core engine.

---

File: `crates/storyforge-tui/src/app.rs`

## Step 8: Store a `GameEngine` inside `App`

### Current state

`App` stores TUI-only fields like `screen`, `focus`, `selected_tab`, and demo spell resources. It does not yet contain core state.

### Why this step matters

The TUI needs one owner for gameplay state. `App` can own the engine because `App` owns the event loop, but the rules still live inside `storyforge-core`.

### Before

At the top of `app.rs`, imports currently look like this:

```rust
use crossterm::event::{self, Event};
use ratatui::DefaultTerminal;
use std::io;

use crate::{action::UiAction, theme::Theme, ui};
```

The `App` struct has no engine field.

### What to change

Import core types, add an engine field, and initialize it with a temporary hard-coded scene ID and seed.

### Temporary MVP / debug behavior

Hard-code `academy.scene.arrival` and seed `42` for now. Guide 06 replaces the hard-coded scene with the campaign manifest entry scene. Later save support will store the seed.

### After

Add this import:

```rust
use storyforge_core::{ContentId, GameEngine, GameState};
```

Add this field to `App`:

```rust
/// Deterministic gameplay engine. The TUI sends commands here instead of
/// changing game rules directly.
pub(crate) engine: GameEngine,
```

Initialize it in `Default`:

```rust
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
}
```

### Learning checkpoint

You should understand that `App` owns the engine, but it should not duplicate engine facts. If core has `selected_choice`, the TUI should read it from core instead of storing a second selected story choice.

### How to verify

Run:

```powershell
cargo check -p storyforge-tui
```

### Next connection

Now route `Up` and `Down` into `GameCommand` values.

---

File: `crates/storyforge-tui/src/app.rs`

## Step 9: Dispatch choice-selection commands from `App::update`

### Current state

After guide 04, `UiAction::Up` and `UiAction::Down` move `selected_tab`. That proved input changed visible UI state. Now gameplay selection should move into core.

### Why this step matters

This is the first real TUI-to-engine connection:

```text
UiAction::Down -> GameCommand::SelectNextChoice -> GameEvent -> GameState.selected_choice
```

### Before

You may currently have:

```rust
UiAction::Down => {
    self.selected_tab = (self.selected_tab + 1) % 4;
}
UiAction::Up => {
    self.selected_tab = self.selected_tab.checked_sub(1).unwrap_or(3);
}
```

### What to change

For Story screen movement, dispatch engine commands. For non-story compact tabs, you can keep tab movement as a UI behavior.

Start with a helper that returns the temporary number of visible story choices.

### Temporary MVP / debug behavior

Use `2` as the temporary choice count because the current UI shows placeholder actions and guide 06 will replace this with loaded scene data.

Log or inspect events before relying on the rendered selection:

```rust
let events = self.engine.dispatch(command);
tracing::debug!(?events, "engine command dispatched");
```

If `tracing` is not set up yet, skip the log and verify with tests instead.

### After

Add imports:

```rust
use storyforge_core::{ContentId, GameCommand, GameEngine, GameState};
```

Add this helper inside `impl App`:

```rust
fn visible_choice_count(&self) -> usize {
    2
}
```

Change the movement arms:

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
```

### Learning checkpoint

You should understand that `choice_count` is temporary glue. The final owner of available choices is loaded scene content, not the TUI.

### How to verify

Add a focused test in `app.rs`:

```rust
#[test]
fn down_on_story_should_dispatch_to_the_engine() {
    let mut app = App::default();

    app.update(crate::action::UiAction::Down);

    assert_eq!(app.engine.state().selected_choice, 1);
}
```

Run:

```powershell
cargo test -p storyforge-tui app
```

### Next connection

Now the renderer needs to read `engine.state().selected_choice` when drawing actions.

---

File: `crates/storyforge-tui/src/ui.rs`

## Step 10: Render story selection from engine state

### Current state

`render_actions_panel` currently shows static shortcut text:

```rust
let text = "\n[c] Character\n[l] Journal\n[m] Map\n[?] Help\n[Esc] Back/Quit";
```

It does not show selected story choices.

### Why this step matters

The renderer should display engine state without changing it. This proves the complete loop:

```text
input -> command -> event -> state -> render
```

### Before

Find `render_actions_panel` in `ui.rs`.

### What to change

When `app.screen == Screen::Story`, render two temporary story choices and highlight the selected one from `app.engine.state().selected_choice`.

### Temporary MVP / debug behavior

The choices can still be hard-coded in this guide:

1. `Ask about the sealed gate.`
2. `Look for another route.`

Guide 06 replaces this list with loaded scene choices.

### After

One simple version is:

```rust
let text = if app.screen == Screen::Story {
    let selected = app.engine.state().selected_choice;
    let first = if selected == 0 {
        "> Ask about the sealed gate."
    } else {
        "  Ask about the sealed gate."
    };
    let second = if selected == 1 {
        "> Look for another route."
    } else {
        "  Look for another route."
    };
    format!("\n{first}\n{second}\n\n[j/k] Choose  [Enter] Confirm")
} else {
    "\n[c] Character\n[l] Journal\n[m] Map\n[?] Help\n[Esc] Back/Quit".to_owned()
};

render_pane(frame, area, "Actions", text, theme, focused);
```

### Learning checkpoint

You should understand that rendering reads `app.engine.state()` by shared reference. It does not call `dispatch`, does not mutate state, and does not decide rules.

### How to verify

Run:

```powershell
cargo run -p storyforge-tui
```

On the Story screen, press `j` and `k`. The `>` marker should move between the two temporary choices.

### Next connection

Guide 06 replaces temporary hard-coded choices with loaded content records.

## Full verification

Run:

```powershell
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features --locked -- -D warnings
cargo test --workspace --all-features --locked
cargo run -p storyforge-tui
```

Manual checks:

1. The app starts on the Story screen.
2. `j` moves the story selection down.
3. `k` moves the story selection up and wraps.
4. `l`, `c`, `m`, and `?` still open their screens.
5. Movement on non-story compact tabs still changes UI tabs if you kept that behavior.

## Common mistakes

- Putting `KeyCode` inside `GameCommand`.
- Letting `ui.rs` call `dispatch`.
- Storing a second gameplay selected-choice field in `App`.
- Silently ignoring rejected commands.
- Using display names as stable IDs.
- Rolling random values from render functions.

## Acceptance check

- `ContentId` rejects invalid ID strings.
- `GameCommand` contains gameplay intent, not keyboard input.
- `handle_command` emits events without mutating state.
- `apply_events` is the only function that changes `GameState`.
- Empty choice lists emit `CommandRejected`.
- TUI movement dispatches commands to the engine on the Story screen.
- `ui.rs` renders selection from `engine.state()`.

