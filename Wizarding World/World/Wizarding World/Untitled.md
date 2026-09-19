- Our Caracter seet should ve sub blocks or under scores under tittles
- abilites should be in 1 line consider using a horizontable table 
like this below 

| Dex | Con | STR |
| --- | --- | --- |
| 8   | 15  | 16  |
| -1  | +2  | +3  |
- instead of those checkboxes use this ## Tui-checkbox [Crate](https://crates.io/crates/tui-checkbox) [Docs](https://docs.rs/tui-checkbox) [GitHub](https://github.com/sorinirimies/tui-checkbox)

[`tui-checkbox`](https://crates.io/crates/tui-checkbox) is a customizable checkbox widget with support for custom styling, symbols (unicode, emoji, ASCII), and optional block wrappers.

- use a canvas instead on the starting menu for the logo create in teh canvas a version of this in pixal art style  
- ![[Pasted image 20260918153855.png]]
- the health bar should not exist in the side panel 
- the health bar should be right after hp in like the example from rattui # Gauge

Demonstrates the [`Gauge`](https://docs.rs/ratatui/0.30.2/ratatui/widgets/struct.Gauge.html) widget. Source [gauge.rs](https://github.com/ratatui/ratatui/blob/ratatui-v0.30.2/ratatui-widgets/examples/gauge.rs).

run

```
git clone https://github.com/ratatui/ratatui.git --branch ratatui-v0.30.2 --depth 1cd ratatuicargo run -p ratatui-widgets --example gauge
```

![gauge](https://github.com/ratatui/ratatui/blob/images/widget-examples/gauge.gif?raw=true)

gauge.rs

```
//! # [Ratatui] `Gauge` example//!//! The latest version of this example is available in the [widget examples] folder in the//! repository.//!//! Please note that the examples are designed to be run against the `main` branch of the Github//! repository. This means that you may not be able to compile with the latest release version on//! crates.io, or the one that you have installed locally.//!//! See the [examples readme] for more information on finding examples that match the version of the//! library you are using.//!//! [Ratatui]: https://github.com/ratatui/ratatui//! [widget examples]: https://github.com/ratatui/ratatui/blob/main/ratatui-widgets/examples//! [examples readme]: https://github.com/ratatui/ratatui/blob/main/examples/README.md
use color_eyre::Result;use crossterm::event;use ratatui::layout::{Constraint, Layout, Rect};use ratatui::style::{Modifier, Style, Stylize};use ratatui::text::{Line, Span};use ratatui::widgets::{Gauge, LineGauge};use ratatui::{Frame, symbols};
fn main() -> Result<()> {    color_eyre::install()?;    ratatui::run(|terminal| {        loop {            terminal.draw(render)?;            if event::read()?.is_key_press() {                break Ok(());            }        }    })}
/// Render the UI with various progress bars.fn render(frame: &mut Frame) {    let constraints = [        Constraint::Length(1),        Constraint::Max(2),        Constraint::Fill(1),    ];    let layout = Layout::vertical(constraints).spacing(1);    let [top, first, second] = frame.area().layout(&layout);
    let title = Line::from_iter([        Span::from("Gauge Widget").bold(),        Span::from(" (Press 'q' to quit)"),    ]);    frame.render_widget(title.centered(), top);
    render_gauge(frame, first);    render_line_gauge(frame, second);}
/// Render a gauge with a custom style.pub fn render_gauge(frame: &mut Frame, area: Rect) {    let gauge = Gauge::default()        .style(Modifier::BOLD)        .gauge_style(Style::new().blue().on_black())        .label("Year Progress")        .percent(80);    frame.render_widget(gauge, area);}
/// Render a line gauge (compact progress bar).pub fn render_line_gauge(frame: &mut Frame, area: Rect) {    let line_gauge = LineGauge::default()        .filled_style(Style::new().white().on_red().bold())        .unfilled_style(Style::new().gray().on_black())        .label("❤️ HP")        .ratio(0.42)        .filled_symbol(symbols::line::THICK_HORIZONTAL)        .unfilled_symbol(symbols::line::THICK_HORIZONTAL);    frame.render_widget(line_gauge, area);}
```