# 05c: House themes and focus dimming

## Guide order

These six guides build one learning path from the current repo state:

1. `04-responsive-dashboard.md` finishes the visible TUI shell and proves keyboard input reaches app state.
2. `05-engine-command-event-model.md` moves gameplay choice selection out of the TUI and into `storyforge-core`.
3. `05b-layout-focus-navigation-and-inventory.md` separates focus from movement, adds directional navigation, and adds a temporary inventory sidebar.
4. `05c-house-themes-and-focus-dimming.md` (this guide) makes `theme.rs` the source of truth for all colors, adds four house-inspired themes, and dims unfocused panels.
5. `06-content-pack-format.md` loads scene data from files so the engine and TUI stop depending on hard-coded story text.
6. `07-character-creation.md` starts the playable character-creation prologue using the same command/event path.

This guide does not build the final theming system from the end-goal game sheet. It only covers the first useful slice: multiple hard-coded themes, a key to cycle them, and a uniform dimming effect for panels that are not focused.

## Result

`theme.rs` will become the only place that owns RGB color values. Every other file will ask for `theme.background`, `theme.text`, `theme.focus`, and so on instead of inventing its own colors.

The file will contain four house-inspired themes:

- `Lion` (Gryffindor): deep red and gold.
- `Raven` (Ravenclaw): dark blue and bronze.
- `Badger` (Hufflepuff): black and yellow.
- `Serpent` (Slytherin): dark green and silver.

The app will launch in the `Lion` theme and let the player press `t` to cycle forward through the four houses. The header will show the current house name so the change is visible even before you learn the colors.

The `Theme` struct will also gain `primary` and `secondary` colors for future buttons, and a `dim()` method that returns a 25% darker copy of the whole palette. `render_pane` will use that dimmed palette for every panel that does not currently have focus, so the focused panel stands out.

## Flow to prove

The new flow uses these functions and types:

```text
KeyEvent 't' -> UiAction::NextTheme -> App::update -> App.theme_id -> Theme::for_id -> App.theme
    -> ui::render_header shows the current house name
    -> ui::render_pane dims the palette when focused == false
```

First prove the theme file owns all the colors. Then prove the app can switch themes. Then prove the focused panel is brighter than the others.

---

File: `crates/storyforge-tui/src/theme.rs`

## Step 1: Make `theme.rs` the source of truth for colors, themes, and dimming

### Current state

`theme.rs` only has a single `Theme` struct with seven colors and one `Default` implementation. There is no way to switch themes, no house palettes, and no dimming helper.

### Why this step matters

If every panel invents its own RGB values, theming becomes impossible to maintain. Centralizing every color in `theme.rs` means a new theme is just a new constructor, and the rest of the UI does not need to change. Adding `primary` and `secondary` now also gives future buttons a stable color pair.

### Before

`theme.rs` currently looks like this:

```rust
use ratatui::style::Color;

/// Semantic color palette for every widget in the TUI.
///
/// Widgets ask for named colors like `accent` or `danger` instead of inventing
/// their own RGB values. Centralizing the palette makes theming and snapshot
/// testing straightforward.
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
            background: Color::Rgb(10, 10, 21),
            text: Color::Rgb(51, 152, 219),
            muted: Color::Rgb(112, 127, 245),
            accent: Color::Rgb(231, 126, 35),
            success: Color::Rgb(26, 188, 156),
            danger: Color::Rgb(231, 37, 35),
            focus: Color::Rgb(241, 196, 14),
        }
    }
}
```

### What to change

1. Add `primary` and `secondary` fields to `Theme`.
2. Add a `ThemeId` enum that names the four houses.
3. Add a constructor for each house theme and a `Theme::for_id` helper.
4. Add a `ThemeId::name` method for the header title and a `ThemeId::next` method for cycling.
5. Add a `dim()` method that returns a 25% darker copy of the theme. The dimming should be done with integer math so the render loop stays cheap.
6. Keep the neutral `Default` theme as a fallback for tests and snapshots.

### Temporary MVP / debug behavior

The house themes are hard-coded constructors. Later guides can load them from campaign data or a settings file, but the first step is to prove that `theme.rs` can serve multiple palettes and the rest of the UI can read them.

### After

Replace the entire contents of `theme.rs` with this:

```rust
use ratatui::style::Color;

/// Identifies one of the built-in house themes.
///
/// The names are deliberately neutral versions of the house animals so the
/// public engine does not ship copyrighted names. Private campaign packs can
/// display whatever names they want in their own content.
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum ThemeId {
    /// Deep red and gold.
    Lion,

    /// Dark blue and bronze.
    Raven,

    /// Black and yellow.
    Badger,

    /// Dark green and silver.
    Serpent,
}

impl ThemeId {
    /// Returns the display name used in the UI header.
    #[must_use]
    pub fn name(self) -> &'static str {
        match self {
            Self::Lion => "Lion",
            Self::Raven => "Raven",
            Self::Badger => "Badger",
            Self::Serpent => "Serpent",
        }
    }

    /// Returns the next theme in the cycle, wrapping back to the start.
    #[must_use]
    pub fn next(self) -> Self {
        match self {
            Self::Lion => Self::Raven,
            Self::Raven => Self::Badger,
            Self::Badger => Self::Serpent,
            Self::Serpent => Self::Lion,
        }
    }
}

/// Semantic color palette for every widget in the TUI.
///
/// Widgets ask for named colors like `accent` or `danger` instead of inventing
/// their own RGB values. Centralizing the palette makes theming and snapshot
/// testing straightforward.
///
/// `primary` and `secondary` are reserved for future buttons and highlighted
/// controls. They are not used anywhere yet, but they live here so no other
/// file invents a one-off color when that time comes.
#[derive(Debug, Clone, Copy)]
pub struct Theme {
    pub background: Color,
    pub text: Color,
    pub muted: Color,
    pub accent: Color,
    pub success: Color,
    pub danger: Color,
    pub focus: Color,
    pub primary: Color,
    pub secondary: Color,
}

impl Theme {
    /// Returns the theme that matches the given `ThemeId`.
    #[must_use]
    pub fn for_id(id: ThemeId) -> Self {
        match id {
            ThemeId::Lion => Self::lion(),
            ThemeId::Raven => Self::raven(),
            ThemeId::Badger => Self::badger(),
            ThemeId::Serpent => Self::serpent(),
        }
    }

    /// Lion theme: deep red and gold.
    #[must_use]
    pub fn lion() -> Self {
        Self {
            background: Color::Rgb(12, 0, 0),
            text: Color::Rgb(248, 226, 209),
            muted: Color::Rgb(135, 0, 1),
            accent: Color::Rgb(211, 166, 37),
            success: Color::Rgb(26, 188, 156),
            danger: Color::Rgb(231, 37, 35),
            focus: Color::Rgb(255, 186, 51),
            primary: Color::Rgb(211, 166, 37),
            secondary: Color::Rgb(135, 0, 1),
        }
    }

    /// Raven theme: dark blue and bronze.
    #[must_use]
    pub fn raven() -> Self {
        Self {
            background: Color::Rgb(10, 12, 30),
            text: Color::Rgb(230, 240, 255),
            muted: Color::Rgb(60, 80, 120),
            accent: Color::Rgb(184, 115, 51),
            success: Color::Rgb(26, 188, 156),
            danger: Color::Rgb(231, 37, 35),
            focus: Color::Rgb(70, 130, 240),
            primary: Color::Rgb(30, 60, 130),
            secondary: Color::Rgb(184, 115, 51),
        }
    }

    /// Badger theme: black and yellow.
    #[must_use]
    pub fn badger() -> Self {
        Self {
            background: Color::Rgb(25, 20, 5),
            text: Color::Rgb(255, 242, 180),
            muted: Color::Rgb(160, 140, 60),
            accent: Color::Rgb(255, 215, 0),
            success: Color::Rgb(26, 188, 156),
            danger: Color::Rgb(231, 37, 35),
            focus: Color::Rgb(255, 235, 80),
            primary: Color::Rgb(255, 215, 0),
            secondary: Color::Rgb(20, 15, 5),
        }
    }

    /// Serpent theme: dark green and silver.
    #[must_use]
    pub fn serpent() -> Self {
        Self {
            background: Color::Rgb(5, 20, 10),
            text: Color::Rgb(220, 245, 235),
            muted: Color::Rgb(40, 90, 60),
            accent: Color::Rgb(192, 192, 192),
            success: Color::Rgb(26, 188, 156),
            danger: Color::Rgb(231, 37, 35),
            focus: Color::Rgb(80, 220, 120),
            primary: Color::Rgb(30, 120, 60),
            secondary: Color::Rgb(192, 192, 192),
        }
    }

    /// Returns a copy of this theme with every RGB color multiplied by 3/4.
    ///
    /// This dims unfocused panels by 25%. Non-RGB colors are left unchanged.
    /// The fraction is easy to change later if you want a subtler 15% or 20%
    /// effect.
    #[must_use]
    pub fn dim(self) -> Self {
        const NUMERATOR: u16 = 3;
        const DENOMINATOR: u16 = 4;

        Self {
            background: dim_color(self.background, NUMERATOR, DENOMINATOR),
            text: dim_color(self.text, NUMERATOR, DENOMINATOR),
            muted: dim_color(self.muted, NUMERATOR, DENOMINATOR),
            accent: dim_color(self.accent, NUMERATOR, DENOMINATOR),
            success: dim_color(self.success, NUMERATOR, DENOMINATOR),
            danger: dim_color(self.danger, NUMERATOR, DENOMINATOR),
            focus: dim_color(self.focus, NUMERATOR, DENOMINATOR),
            primary: dim_color(self.primary, NUMERATOR, DENOMINATOR),
            secondary: dim_color(self.secondary, NUMERATOR, DENOMINATOR),
        }
    }
}

impl Default for Theme {
    /// A neutral academy theme used for tests and snapshots.
    ///
    /// The play path starts with the Lion theme, but keeping a neutral default
    /// means snapshot tests do not need to know which house is currently active.
    fn default() -> Self {
        Self {
            background: Color::Rgb(10, 10, 21),
            text: Color::Rgb(51, 152, 219),
            muted: Color::Rgb(112, 127, 245),
            accent: Color::Rgb(231, 126, 35),
            success: Color::Rgb(26, 188, 156),
            danger: Color::Rgb(231, 37, 35),
            focus: Color::Rgb(241, 196, 14),
            primary: Color::Rgb(231, 126, 35),
            secondary: Color::Rgb(112, 127, 245),
        }
    }
}

/// Dims a single RGB color by an integer fraction.
///
/// Uses `u16` for the intermediate multiplication so channel values up to
/// 255 do not overflow.
fn dim_color(color: Color, numerator: u16, denominator: u16) -> Color {
    match color {
        Color::Rgb(r, g, b) => Color::Rgb(
            ((r as u16 * numerator) / denominator) as u8,
            ((g as u16 * numerator) / denominator) as u8,
            ((b as u16 * numerator) / denominator) as u8,
        ),
        other => other,
    }
}

#[cfg(test)]
mod tests {
    use super::{Color, Theme, ThemeId, dim_color};

    #[test]
    fn dim_color_should_darken_rgb_by_the_given_fraction() {
        let color = Color::Rgb(100, 120, 140);
        let dimmed = dim_color(color, 3, 4);

        assert_eq!(dimmed, Color::Rgb(75, 90, 105));
    }

    #[test]
    fn dim_color_should_leave_non_rgb_colors_unchanged() {
        assert_eq!(dim_color(Color::Reset, 3, 4), Color::Reset);
    }

    #[test]
    fn theme_dim_should_darken_every_channel() {
        let bright = Theme::lion();
        let dimmed = bright.dim();

        // The background is the easiest channel to reason about.
        assert_eq!(dimmed.background, Color::Rgb(9, 0, 0));
    }

    #[test]
    fn theme_id_should_cycle_through_houses() {
        assert_eq!(ThemeId::Lion.next(), ThemeId::Raven);
        assert_eq!(ThemeId::Raven.next(), ThemeId::Badger);
        assert_eq!(ThemeId::Badger.next(), ThemeId::Serpent);
        assert_eq!(ThemeId::Serpent.next(), ThemeId::Lion);
    }

    #[test]
    fn theme_id_name_should_match_the_house() {
        assert_eq!(ThemeId::Lion.name(), "Lion");
        assert_eq!(ThemeId::Raven.name(), "Raven");
        assert_eq!(ThemeId::Badger.name(), "Badger");
        assert_eq!(ThemeId::Serpent.name(), "Serpent");
    }
}
```

### Learning checkpoint

You should understand why `Theme` is `Copy`: every field is a small `Color` value, so passing the whole palette by value is cheap. The `dim` method consumes `self` and returns a new `Theme` rather than mutating the original, which keeps the renderer pure and makes the effect easy to test.

### How to verify

Run:

```powershell
cargo test -p storyforge-tui theme
```

### Next connection

Now add a `UiAction` that lets the player cycle themes with a key press.

---

File: `crates/storyforge-tui/src/action.rs`

## Step 2: Add a `NextTheme` action mapped to `t`

### Current state

`UiAction` has focus, movement, inventory, and screen actions, but no theme action. The `From<KeyEvent>` mapping does not handle `t`.

### Why this step matters

The UI layer should turn the physical key into a semantic action. `App` will decide what `NextTheme` means, but the mapping lives here in one place.

### Before

`UiAction` currently ends with:

```rust
pub enum UiAction {
    // ... movement, focus, inventory, confirm, back, screens, quit, none ...
}
```

And the `From<KeyEvent>` match does not include `t`.

### What to change

Add a `NextTheme` variant and map `KeyCode::Char('t')` to it.

### Temporary MVP / debug behavior

`NextTheme` only cycles forward for now. A future guide can add a previous-theme key or a settings menu.

### After

Add the variant to the enum:

```rust
pub enum UiAction {
    // ... existing movement, focus, inventory, confirm, back variants ...

    /// Switch to the next color theme.
    NextTheme,

    OpenCharacter,
    OpenJournal,
    OpenMap,
    OpenHelp,
    Quit,
    None,
}
```

Add the mapping in `From<KeyEvent>`:

```rust
KeyCode::Char('t') => Self::NextTheme,
```

A good place to put it is near the inventory toggle, just before the screen-open keys.

Update the tests to include the new key:

```rust
#[cfg(test)]
mod tests {
    use crossterm::event::{KeyCode, KeyEvent, KeyModifiers};

    use super::UiAction;

    // ... existing tests ...

    #[test]
    fn t_should_cycle_to_the_next_theme() {
        let key = KeyEvent::new(KeyCode::Char('t'), KeyModifiers::NONE);

        assert_eq!(UiAction::from(key), UiAction::NextTheme);
    }
}
```

### Learning checkpoint

You should understand that adding a new UI action is a two-step process: add the variant, then map the key. The rest of the app only sees the semantic action, so the key can change later without touching `App` or `ui.rs`.

### How to verify

Run:

```powershell
cargo test -p storyforge-tui action
```

### Next connection

Now store the selected `ThemeId` in `App` and update it when `NextTheme` is received.

---

File: `crates/storyforge-tui/src/app.rs`

## Step 3: Store the current theme and cycle it from `App::update`

### Current state

`App` stores `theme: Theme` but no `theme_id`. The `Theme` is created with `Theme::default()` in the `Default` impl. `App::update` does not handle `NextTheme`.

### Why this step matters

The app needs to know which house theme is active so it can cycle forward and so the renderer can display the house name in the header. Keeping `theme_id` separate from `theme` lets the id be the small persistent fact, while the full `Theme` is derived from it.

### Before

`App` imports only `Theme`:

```rust
use crate::{action::UiAction, theme::Theme, ui};
```

`App` has this field:

```rust
pub(crate) theme: Theme,
```

And `Default` initializes it like this:

```rust
theme: Theme::default(),
```

### What to change

1. Import `ThemeId` alongside `Theme`.
2. Add a `theme_id` field to `App`.
3. Start the app in the `Lion` theme.
4. Handle `UiAction::NextTheme` by advancing the id and refreshing the theme.

### Temporary MVP / debug behavior

The theme cycles in a fixed order: Lion → Raven → Badger → Serpent → Lion. The full theme comes from `Theme::for_id`, so `theme.rs` stays the source of truth.

### After

Update the import:

```rust
use crate::{action::UiAction, theme::Theme, theme::ThemeId, ui};
```

Add the field:

```rust
/// Currently selected house theme.
///
/// The full palette is recomputed from this id whenever it changes. The id is
/// the small piece of state that is saved and compared; the RGB values live in
/// `theme.rs`.
pub(crate) theme_id: ThemeId,
```

Update `Default`:

```rust
// Start in the Lion theme so the first screen has a distinct house palette.
// Snapshots and tests that do not care about the house can use Theme::default().
theme_id: ThemeId::Lion,
theme: Theme::for_id(ThemeId::Lion),
```

Add the action arm in `App::update`:

```rust
UiAction::NextTheme => {
    self.theme_id = self.theme_id.next();
    self.theme = Theme::for_id(self.theme_id);
}
```

A good place for this arm is right after the screen-opening actions and before the `Confirm | None` catch-all.

### Learning checkpoint

You should understand why `theme` and `theme_id` are both stored in `App` even though `theme` can be derived. The renderer reads `app.theme` directly, so storing the derived value avoids calling `Theme::for_id` on every frame. The id is stored so the next cycle knows where to start.

### How to verify

Add a test inside the existing `app.rs` test module:

```rust
use crate::action::UiAction;
use crate::theme::{Theme, ThemeId};

// ... existing imports ...

#[test]
fn next_theme_should_cycle_theme_id_and_refresh_theme() {
    let mut app = App::default();

    assert_eq!(app.theme_id, ThemeId::Lion);
    assert_eq!(app.theme.background, Theme::lion().background);

    app.update(UiAction::NextTheme);
    assert_eq!(app.theme_id, ThemeId::Raven);
    assert_eq!(app.theme.background, Theme::raven().background);

    app.update(UiAction::NextTheme);
    assert_eq!(app.theme_id, ThemeId::Badger);
    assert_eq!(app.theme.background, Theme::badger().background);

    app.update(UiAction::NextTheme);
    assert_eq!(app.theme_id, ThemeId::Serpent);
    assert_eq!(app.theme.background, Theme::serpent().background);

    app.update(UiAction::NextTheme);
    assert_eq!(app.theme_id, ThemeId::Lion);
    assert_eq!(app.theme.background, Theme::lion().background);
}
```

Run:

```powershell
cargo test -p storyforge-tui app
```

### Next connection

Now make the renderer show the current theme name and dim the panels that are not focused.

---

File: `crates/storyforge-tui/src/ui.rs`

## Step 4: Show the theme name and dim unfocused panels

### Current state

`render_header` receives `theme` directly, so it cannot read the current `ThemeId`. The title is always `"Storyforge"`. `render_pane` uses the full palette for every panel, so focused and unfocused panels look equally bright. `render_header` and `render_footer` already show the focus and inventory hints, but they do not mention the theme key.

### Why this step matters

This is the visible payoff. The header proves the theme is active, and the dimming proves the focus model from guide 05b is now affecting the whole palette. All the color logic stays in `theme.rs`; `ui.rs` only decides *when* to use the bright or dimmed palette.

### Before

`render_header` is declared like this:

```rust
fn render_header(frame: &mut Frame, area: Rect, theme: Theme) {
    let hint = Line::from(vec![
        // ... spans ...
    ]);

    let block = Block::default()
        .title("Storyforge")
        // ... rest ...
}
```

`render_pane` is declared like this:

```rust
fn render_pane(
    frame: &mut Frame,
    area: Rect,
    title: &str,
    content: impl Into<Text<'static>>,
    theme: Theme,
    focused: bool,
) {
    let border_color = if focused { theme.focus } else { theme.accent };

    let block = Block::default()
        .title(title)
        .borders(Borders::ALL)
        .border_style(Style::default().fg(border_color));

    let paragraph = Paragraph::new(content)
        .style(Style::default().fg(theme.text).bg(theme.background))
        .block(block);

    frame.render_widget(paragraph, area);
}
```

### What to change

1. Change `render_header` to take `app: &App` so it can read both `theme` and `theme_id`.
2. Update the header title to include the current house name.
3. Add a `t` theme hint to the header and the footer.
4. Update `render_pane` to use `theme.dim()` when the panel is not focused, and to style the title with the border color.
5. Update the two callers of `render_header` to pass `app` instead of `app.theme`.

### Temporary MVP / debug behavior

Dimming is applied uniformly to every color in the palette. If you want a subtler effect later, you can keep the background unchanged and only dim the foreground colors. For now, uniform dimming is the easiest way to make the focused panel pop.

### After

Update `render_header`:

```rust
fn render_header(frame: &mut Frame, area: Rect, app: &App) {
    let theme = app.theme;

    let hint = Line::from(vec![
        Span::styled("q/Esc", Style::default().fg(theme.focus)),
        Span::styled(" quit  ", Style::default().fg(theme.muted)),
        Span::styled("c/l/m/?", Style::default().fg(theme.focus)),
        Span::styled(" screens  ", Style::default().fg(theme.muted)),
        Span::styled("i", Style::default().fg(theme.focus)),
        Span::styled(" inv  ", Style::default().fg(theme.muted)),
        Span::styled("j/k", Style::default().fg(theme.focus)),
        Span::styled(" focus  ", Style::default().fg(theme.muted)),
        Span::styled("t", Style::default().fg(theme.focus)),
        Span::styled(" theme", Style::default().fg(theme.muted)),
    ]);

    let title = format!("Storyforge - {}", app.theme_id.name());

    let block = Block::default()
        .title(title.as_str())
        .borders(Borders::ALL)
        .border_style(Style::default().fg(theme.accent))
        .border_type(BorderType::Rounded);

    let header = Paragraph::new(hint)
        .alignment(Alignment::Left)
        .style(Style::default().fg(theme.text).bg(theme.background))
        .block(block);

    frame.render_widget(header, area);
}
```

Update `render_pane`:

```rust
fn render_pane(
    frame: &mut Frame,
    area: Rect,
    title: &str,
    content: impl Into<Text<'static>>,
    theme: Theme,
    focused: bool,
) {
    // Use the full palette when this panel is focused. Dim every color when it
    // is not, so the focused panel stands out.
    let theme = if focused { theme } else { theme.dim() };
    let border_color = if focused { theme.focus } else { theme.accent };

    let block = Block::default()
        .title(title)
        .title_style(Style::default().fg(border_color))
        .borders(Borders::ALL)
        .border_style(Style::default().fg(border_color));

    let paragraph = Paragraph::new(content)
        .style(Style::default().fg(theme.text).bg(theme.background))
        .block(block);

    frame.render_widget(paragraph, area);
}
```

Update `render_standard` and `render_compact` to pass `app` to the header:

```rust
fn render_standard(frame: &mut Frame, app: &App) {
    // ... existing body area split ...

    render_header(frame, header_area, app);
    // ... rest unchanged ...
}

fn render_compact(frame: &mut Frame, app: &App) {
    // ... existing body area split ...

    render_header(frame, header_area, app);
    // ... rest unchanged ...
}
```

Add a `t` theme hint to the footer. Find the section where the focus and movement hints were added in guide 05b and append the theme hint:

```rust
// Add a short control reminder after the spell slots.
spans.push(Span::styled(" | ", Style::default().fg(theme.muted)));
spans.push(Span::styled("j/k", Style::default().fg(theme.focus)));
spans.push(Span::styled(" focus  ", Style::default().fg(theme.muted)));
spans.push(Span::styled("w/a/s/d", Style::default().fg(theme.focus)));
spans.push(Span::styled(" move  ", Style::default().fg(theme.muted)));
spans.push(Span::styled("t", Style::default().fg(theme.focus)));
spans.push(Span::styled(" theme", Style::default().fg(theme.muted)));
```

If you did not add the focus/movement hints in guide 05b, just add the two theme spans at the end of the existing spell-slot section instead.

### Learning checkpoint

You should understand that `render_pane` is the single place where the dimming decision is made. Every other panel just tells `render_pane` whether it is focused. Because `Theme` is `Copy`, the dimming creates a new, cheap palette on the stack each frame for unfocused panels. No other function needs to know how dimming works.

### How to verify

Run:

```powershell
cargo run -p storyforge-tui
```

At a large terminal size:

1. The header should show `Storyforge - Lion`.
2. Press `t`. The header should change to `Storyforge - Raven`, then `Badger`, then `Serpent`, then back to `Lion`.
3. The background, text, and border colors should change with each house.
4. Press `j` and `k` to move focus between panels. The panel with the bright border should be visibly brighter than the dimmed panels. The story panel, action panel, character panel, quest panel, log panel, and inventory panel should all participate.
5. Press `i` to open the inventory. The inventory panel should be bright while the rest of the dashboard dims.
6. Press `q` to quit. The terminal should restore normally.

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

1. The header title is `Storyforge - Lion`.
2. `t` cycles the header title through `Raven`, `Badger`, `Serpent`, and back to `Lion`.
3. The focused panel has a bright border; the other panels are dimmed.
4. `j` and `k` move the bright border around the panels.
5. `i` opens the inventory. The inventory panel is bright and the rest dims.
6. `i` again closes the inventory.
7. `c`, `l`, `m`, and `?` still open their screens, and the theme colors stay active.
8. `q` quits and restores the terminal.

Manual checks at a compact terminal size (80x24):

1. The header still shows `Storyforge - Lion` and the theme hint is visible.
2. `t` cycles the theme.
3. The focused bottom tab is bright; the story panel and the other tabs are dimmed.
4. `i` opens the inventory sidebar. The inventory is bright and the rest of the screen is dimmed.
5. `q` quits and restores the terminal.

## Common mistakes

- Hard-coding a new RGB value in `ui.rs` instead of adding it to `Theme`.
- Storing the full `Theme` palette in a save file or config instead of the small `ThemeId`. The palette should always be derived from `theme.rs`.
- Forgetting to update the `Theme::default()` implementation after adding `primary` and `secondary`. Every constructor must initialize every field.
- Using `as u8` multiplication without casting to a larger integer first. `255 * 3` overflows `u8`, so the dimming math must use `u16`.
- Dimming the focused panel instead of the unfocused ones. The bright border should be on the panel that currently has focus.
- Passing `app.theme` to `render_header` and then trying to read `app.theme_id` inside it. The signature needs to take `app: &App`.

## Acceptance check

- `theme.rs` defines all four house themes, `ThemeId`, `Theme::for_id`, `ThemeId::next`, and `Theme::dim`.
- `Theme` has `primary` and `secondary` fields, even though they are not used yet.
- No other file invents RGB values; all colors come from `app.theme` or `Theme` constructors.
- `t` maps to `UiAction::NextTheme`.
- `App` stores `theme_id` and updates `theme` from `Theme::for_id`.
- `render_header` shows the current house name.
- `render_pane` dims the palette when the panel is not focused.
- `cargo fmt`, `cargo clippy`, and `cargo test --workspace` all pass.
