# 05: Build the command and event engine

## Result

The terminal will stop being the owner of game rules. It will send a command to `storyforge-core`, receive events, and render the resulting state.

The first supported command changes the active choice. The structure is deliberately small, but it is the same path later used for travel, dialogue, items, and combat.

## Add dependencies

```powershell
cargo add -p storyforge-core rand rand_chacha --features rand_chacha/serde
```

Keep Serde and `thiserror` inherited from the workspace.

## Stable IDs

Create `crates/storyforge-core/src/id.rs`:

```rust
use std::{fmt, str::FromStr};

use serde::{Deserialize, Deserializer, Serialize, de::Error as _};

#[derive(
    Debug, Clone, PartialEq, Eq, PartialOrd, Ord, Hash, Serialize,
)]
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
        let valid_characters = value
            .chars()
            .all(|character| character.is_ascii_lowercase()
                || character.is_ascii_digit()
                || matches!(character, '.' | '_' | '-'));

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

Do not expose unchecked ID construction in production. Tests can use a helper that calls `ContentId::new` and returns the result through `?`.

## Game state

Create `state.rs`:

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

The event log will later become bounded. Keeping the complete vector now makes the behavior easy to inspect.

## Commands

Create `command.rs`:

```rust
#[derive(Debug, Clone, PartialEq, Eq)]
pub enum GameCommand {
    SelectNextChoice { choice_count: usize },
    SelectPreviousChoice { choice_count: usize },
}
```

A command describes intent. It does not contain keyboard keys or terminal focus.
Keep this root enum `Clone`, but not `Copy`. Later chapters add owned strings,
vectors, and boxed subsystem commands that cannot implement `Copy`.

## Engine

Create `engine.rs`:

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
        GameEvent::ChoiceSelectionChanged {
            previous,
            current,
        },
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

The `checked_sub(1).unwrap_or(choice_count - 1)` expression is safe production code because it supplies a normal fallback. It does not panic.

`handle_command` only decides which events should occur. `apply_events` is the one place that changes state. This separation makes replay, save migration, scripted playthroughs, and rule tests use the same behavior.

## Public exports

Replace `storyforge-core/src/lib.rs`:

```rust
//! Deterministic rules and state transitions for Storyforge campaigns.

mod command;
mod engine;
mod id;
mod state;

pub use command::GameCommand;
pub use engine::{apply_events, handle_command, GameEngine};
pub use id::{ContentId, IdError};
pub use state::{GameEvent, GameState};
```

## Reducer tests

Add tests beside `reduce`:

```rust
#[cfg(test)]
mod tests {
    use crate::{ContentId, GameCommand, GameEngine, GameEvent, GameState};

    fn engine() -> Result<GameEngine, crate::IdError> {
        let scene = ContentId::new("academy.scene.arrival")?;
        Ok(GameEngine::new(GameState::new(scene), 42))
    }

    #[test]
    fn next_choice_should_wrap_at_end() -> Result<(), crate::IdError> {
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
        Ok(())
    }
}
```

## Connect the TUI

Add a `GameEngine` field to `App`. Map `UiAction::Down` and `UiAction::Up` into `GameCommand` values. Rendering reads `engine.state().selected_choice`.

The TUI may know how many visible choices exist because it is rendering the active scene. The engine still validates zero and out-of-date requests.

## Deterministic randomness

All gameplay randomness comes from `GameEngine`. Rendering never requests a random value.

When save support arrives, record:

- Initial seed.
- ChaCha stream.
- Word position.

Do not use current time as an unrecorded seed. Generate it once at new-game creation, store it, and show it in diagnostics.

## Verify

```powershell
cargo fmt --all
cargo clippy --workspace --all-targets --all-features --locked -- -D warnings
cargo test --workspace --all-features --locked
cargo test --workspace --doc --locked
```

## Common mistakes

- `GameCommand` contains `KeyCode`.
- Rendering changes `selected_choice`.
- Content IDs are plain display names.
- Random rolls happen in widgets.
- A command silently fails without an event.
- Large state structures are cloned for every render.

## Acceptance check

- IDs reject invalid formats.
- Choice selection wraps in both directions.
- Empty choices emit a rejection.
- The TUI renders selection from engine state.
- The same seed and commands produce the same state.

## Suggested commit

```powershell
git add .
git commit -m "Add the deterministic command event engine"
```
