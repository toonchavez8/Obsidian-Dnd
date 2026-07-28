# 04: Build the responsive dashboard

## Result

The application will render three layouts:

- A size warning below 80x24.
- Compact tabs at 80x24 through 119x35.
- The standard dashboard at 120x36 or larger.

Input will map to semantic `UiAction` values instead of mutating state directly from key codes.

## Add modules

Create this module tree inside `crates/storyforge-tui/src`:

```text
crates/storyforge-tui/src/
├── action.rs
├── app.rs
├── layout.rs
├── main.rs
├── theme.rs
└── ui.rs
```

## Semantic actions

Create `action.rs`:

```rust
use crossterm::event::{KeyCode, KeyEvent, KeyEventKind};

#[derive(Debug, Clone, Copy, PartialEq, Eq)]
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

impl From<KeyEvent> for UiAction {
    fn from(key: KeyEvent) -> Self {
        if key.kind != KeyEventKind::Press {
            return Self::None;
        }

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
    }
}
```

Use `l` for journal in this early shell because `j` moves down. A later configurable keymap can use a chord or uppercase key.

## Layout modes

Create `layout.rs`:

```rust
use ratatui::layout::Rect;

#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum LayoutMode {
    TooSmall,
    Compact,
    Standard,
}

#[must_use]
pub const fn mode_for(area: Rect) -> LayoutMode {
    if area.width < 80 || area.height < 24 {
        LayoutMode::TooSmall
    } else if area.width < 120 || area.height < 36 {
        LayoutMode::Compact
    } else {
        LayoutMode::Standard
    }
}

#[cfg(test)]
mod tests {
    use ratatui::layout::Rect;

    use super::{LayoutMode, mode_for};

    #[test]
    fn mode_should_be_too_small_below_minimum_width() {
        assert_eq!(mode_for(Rect::new(0, 0, 79, 24)), LayoutMode::TooSmall);
    }

    #[test]
    fn mode_should_be_compact_at_minimum_size() {
        assert_eq!(mode_for(Rect::new(0, 0, 80, 24)), LayoutMode::Compact);
    }

    #[test]
    fn mode_should_be_standard_at_recommended_size() {
        assert_eq!(mode_for(Rect::new(0, 0, 120, 36)), LayoutMode::Standard);
    }
}
```

## App screen and focus

Add to `app.rs`:

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq, Default)]
pub enum Screen {
    #[default]
    Story,
    Character,
    Journal,
    Map,
    Help,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq, Default)]
pub enum Focus {
    #[default]
    Actions,
    Story,
    Log,
}
```

Change event handling so it converts the key to `UiAction`, then sends it to:

```rust
fn update(&mut self, action: UiAction) {
    match action {
        UiAction::OpenCharacter => self.screen = Screen::Character,
        UiAction::OpenJournal => self.screen = Screen::Journal,
        UiAction::OpenMap => self.screen = Screen::Map,
        UiAction::OpenHelp => self.screen = Screen::Help,
        UiAction::Back if self.screen != Screen::Story => self.screen = Screen::Story,
        UiAction::Back | UiAction::Quit => self.should_quit = true,
        UiAction::Up
        | UiAction::Down
        | UiAction::Confirm
        | UiAction::None => {}
    }
}
```

Import `UiAction` from `crate::action`.

## Rendering responsibilities

`ui.rs` should expose:

```rust
pub fn render(frame: &mut Frame, app: &App)
```

It chooses `mode_for(frame.area())` and delegates to:

- `render_size_warning`
- `render_compact`
- `render_standard`
- `render_overlay`

Keep layout calculation inside rendering functions. Do not store rectangles in `App`; they become stale after resize.

Use Ratatui `Layout` with vertical header, body, and footer constraints. In standard mode, split the lower body into actions, character, quest, and log panels. In compact mode, render story and actions plus one selected tab.

Spell resources use discrete counts, not progress bars:

```text
Slots  1st 3/4  2nd 2/2
SP     3/3
```

Only show slot levels whose maximum or temporary count is nonzero. Hide the sorcery-point row for characters without that feature. Compact mode may use `SLOTS 1:3/4 2:2/2 | SP 3/3`; the Magic tab always shows all nine slot levels, temporary slots, and the next long-rest recovery.

The minimum-size screen must include:

```text
Storyforge needs at least 80 columns x 24 rows.
Current size: 72 x 20
Resize the terminal or press q to quit.
```

## Theme

Create `theme.rs` with semantic colors:

```rust
use ratatui::style::Color;

#[derive(Debug, Clone, Copy)]
pub struct Theme {
    pub background: Color,
    pub text: Color,
    pub muted: Color,
    pub accent: Color,
    pub success: Color,
    pub danger: Color,
    pub focus: Color,
}

impl Default for Theme {
    fn default() -> Self {
        Self {
            background: Color::Rgb(10, 14, 20),
            text: Color::Rgb(224, 228, 235),
            muted: Color::Rgb(132, 143, 156),
            accent: Color::Rgb(117, 170, 219),
            success: Color::Rgb(105, 190, 132),
            danger: Color::Rgb(215, 102, 115),
            focus: Color::Rgb(238, 191, 94),
        }
    }
}
```

Widgets ask for semantic colors. They do not invent local RGB values.

## Snapshot-ready rendering

Move all visible state into `App` and keep `render` free of I/O and random behavior. That allows chapter 20 to render into `TestBackend`.

## Verify

```powershell
cargo fmt --all
cargo clippy --workspace --all-targets --all-features --locked -- -D warnings
cargo test --workspace --all-features --locked
cargo run -p storyforge-tui
```

Manual sizes:

1. Shrink below 80x24 and verify the warning.
2. Set 80x24 and verify compact mode.
3. Set at least 120x36 and verify standard mode.
4. Open and close each overlay.
5. Confirm `q` exits from the story screen.

## Common mistakes

- Rendering directly from key handlers.
- Storing terminal rectangles in persistent app state.
- Using color as the only focus indicator.
- Hiding the quit key on the too-small screen.
- Stretching prose across the entire width on a large display.

## Acceptance check

- The three layout modes are selected by tested thresholds.
- Every overlay opens and returns.
- Focus is visible through border or marker as well as color.
- Render functions do not read files, roll dice, or mutate game state.
- Full checks pass.

## Suggested commit

```powershell
git add .
git commit -m "Add the responsive Storyforge dashboard"
```
