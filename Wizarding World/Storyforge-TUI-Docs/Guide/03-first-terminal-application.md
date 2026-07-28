# 03: Build the first terminal application

## Result

The executable will enter the alternate screen, draw a bordered message, wait for `q` or `Esc`, and restore the terminal even when the application returns an error.

## Add dependencies

```powershell
cargo add -p storyforge-tui ratatui crossterm color-eyre
```

Commit the updated lockfile with the code.

## Application module

Create `crates/storyforge-tui/src/app.rs`:

```rust
use std::io;

use crossterm::event::{self, Event, KeyCode, KeyEventKind};
use ratatui::{
    DefaultTerminal, Frame,
    layout::Alignment,
    widgets::{Block, Borders, Paragraph},
};

#[derive(Debug, Default)]
pub struct App {
    should_quit: bool,
}

impl App {
    pub fn run(mut self, terminal: &mut DefaultTerminal) -> io::Result<()> {
        while !self.should_quit {
            terminal.draw(|frame| self.render(frame))?;
            self.handle_event(event::read()?)?;
        }

        Ok(())
    }

    fn render(&self, frame: &mut Frame) {
        let message = Paragraph::new("Storyforge is awake.\n\nPress q or Esc to leave.")
            .alignment(Alignment::Center)
            .block(Block::default().title(" Storyforge ").borders(Borders::ALL));

        frame.render_widget(message, frame.area());
    }

    fn handle_event(&mut self, event: Event) -> io::Result<()> {
        let Event::Key(key) = event else {
            return Ok(());
        };

        if key.kind != KeyEventKind::Press {
            return Ok(());
        }

        if matches!(key.code, KeyCode::Char('q') | KeyCode::Esc) {
            self.should_quit = true;
        }

        Ok(())
    }
}

#[cfg(test)]
mod tests {
    use crossterm::event::{Event, KeyCode, KeyEvent, KeyModifiers};

    use super::App;

    #[test]
    fn q_should_request_quit() {
        let mut app = App::default();
        let key = KeyEvent::new(KeyCode::Char('q'), KeyModifiers::NONE);

        app.handle_event(Event::Key(key))
            .expect("key handling should succeed");

        assert!(app.should_quit);
    }
}
```

Tests may use `expect` because a failure should stop the test with context.

## Main entry point

Replace `crates/storyforge-tui/src/main.rs`:

```rust
mod app;

use color_eyre::Result;

use app::App;

fn main() -> Result<()> {
    color_eyre::install()?;
    ratatui::run(|terminal| App::default().run(terminal))?;
    Ok(())
}
```

`ratatui::run` owns setup and cleanup around the closure. It enables raw mode, enters the alternate screen, installs cleanup behavior, and restores the terminal when the closure exits.

## Why the loop blocks on input

This version calls `event::read()`, which sleeps until an event arrives. Idle CPU stays low. Chapter 18 introduces timed polling only when animations need it.

## Resize behavior

Ratatui checks terminal size during each draw. Crossterm sends resize events, but this application does not need special state yet. The next loop iteration redraws using the new area.

## Verify

```powershell
cargo fmt --all
cargo clippy --workspace --all-targets --all-features --locked -- -D warnings
cargo test --workspace --all-features --locked
cargo run -p storyforge-tui
```

Manual test:

1. Confirm the program takes over the terminal.
2. Resize the window.
3. Confirm the border redraws.
4. Press `q`.
5. Confirm the shell prompt is usable and typed characters do not appear one per line.
6. Run again and exit with `Esc`.

## Test a returned error

Temporarily change `App::run` to return an error before the loop, run the program, and confirm the prompt returns normally. Revert that local experiment before committing.

Do not add a deliberate panic to committed production code.

## Common mistakes

### Terminal remains in raw mode

Make sure `main` uses `ratatui::run` and does not call `std::process::exit` inside the app loop.

### One key triggers twice

Filter for `KeyEventKind::Press`. Windows terminals may send press and release events.

### `q` cannot be typed in a later text field

Global quit handling must become screen-aware once text input exists. Chapter 04 introduces actions and focus so `q` is only quit when appropriate.

## Acceptance check

- `cargo test` passes.
- `q` and `Esc` leave the app.
- Resize redraws the border.
- The terminal is restored after normal exit.
- Production code contains no `unwrap` or `expect`.

## Suggested commit

```powershell
git add .
git commit -m "Build the first safe terminal loop"
```

