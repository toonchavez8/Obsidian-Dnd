# 18: Add ASCII art, animation, and accessibility

## Result

Scenes can select hand-edited art by terminal size, short animations can play without busy-waiting, and all important UI works with ASCII borders, no color, and reduced motion.

## Art format

Use UTF-8 text files for frames and RON metadata:

```text
campaigns/academy-demo/art/rain-gate/
├── art.ron
├── compact.txt
├── standard.txt
├── wide.txt
├── frame-01.txt
├── frame-02.txt
└── frame-03.txt
```

`art.ron`:

```ron
(
    id: "academy.art.rain_gate",
    variants: [
        (name: "compact", file: "compact.txt", min_width: 40, min_height: 6),
        (name: "standard", file: "standard.txt", min_width: 72, min_height: 12),
        (name: "wide", file: "wide.txt", min_width: 100, min_height: 16),
    ],
    animation: Some((
        frames: ["frame-01.txt", "frame-02.txt", "frame-03.txt"],
        frame_ms: 90,
        loops: 1,
    )),
    ascii_fallback: Some("academy.art.rain_gate_ascii"),
    attribution: "Original Storyforge artwork",
    public: true,
)
```

## Hybrid art workflow

For private source images:

1. Record source path and rights status.
2. Crop around the subject.
3. Convert to monochrome ASCII as a rough draft with a local tool.
4. Edit silhouette, contrast, and focal points by hand.
5. Remove details that become noise.
6. Create separate compact, standard, and wide compositions.
7. Confirm the result does not copy a protected logo or embedded text.
8. Keep private outputs in the private pack.

For public art, start from original sketches, public-domain material, or assets with a compatible redistribution license. Record attribution.

## Loader validation

Validate:

- Every declared file exists.
- Lines fit declared target width.
- Frame dimensions match.
- Frame duration is 30 through 2000 ms.
- Loop count is bounded.
- ASCII fallback exists when the art uses non-ASCII glyphs.
- Public pack art has nonempty attribution.
- Private source paths are not referenced by public metadata.

## Rendering

Art selection chooses the largest variant that fits the allocated art panel. If no variant fits, hide art and give space to prose.

Never crop text art silently. An incomplete face or gate looks like corruption.

## Animation loop

Replace blocking `event::read()` with timed polling only when animation is active:

```rust
let timeout = app
    .animation_deadline()
    .map_or(MAX_IDLE_POLL, |deadline| deadline.saturating_duration_since(Instant::now()));

if crossterm::event::poll(timeout)? {
    app.handle_event(crossterm::event::read()?)?;
}

app.advance_animation(Instant::now());
```

Set `MAX_IDLE_POLL` to a long duration such as 250 ms only if background coordination requires it. With no background work, the non-animation path can block on input.

Use `Instant` for animation timing and game `GameClock` for story time. They are unrelated.

## Skip behavior

Any key during a noninteractive animation:

- Completes the animation.
- Does not also choose a menu item.
- Requires a second key for the next action.

Reduced-motion mode renders the final frame immediately.

## Semantic theme

Expand theme settings:

```rust
pub enum ColorMode {
    TrueColor,
    Ansi256,
    Ansi16,
    NoColor,
}

pub enum BorderMode {
    Unicode,
    Ascii,
}

pub struct AccessibilitySettings {
    pub color_mode: ColorMode,
    pub border_mode: BorderMode,
    pub reduced_motion: bool,
    pub instant_text: bool,
    pub persistent_hints: bool,
    pub confirm_combat_actions: bool,
}
```

Persist settings separately from campaign saves so they apply before the load menu.

## No-color rules

Every state also uses:

- Text label.
- Glyph.
- Border style.
- Position.

Examples:

```text
[SUCCESS] Investigation 20 vs 15
[FAILED] Stealth 9 vs 12
> selected choice
! dangerous action
* new journal entry
```

## ASCII-only mode

Replace:

```text
┌ ┐ └ ┘ ─ │ ├ ┤ ┬ ┴ ┼ █ ░
```

with:

```text
+ + + + - | + + + + + # .
```

Do not only replace borders. Progress bars, map lines, status icons, and art need fallbacks.

## Transcript mode

Add a non-alternate-screen mode:

```powershell
cargo run -p storyforge-tui -- play --transcript
```

Transcript mode:

- Prints scene heading and prose once.
- Prints numbered actions.
- Reads a line.
- Prints complete check and combat events.
- Avoids cursor movement and animation.

It is useful for screen readers, debugging, CI playthroughs, and terminals without TUI support.

## Content filters

Campaign manifest declares content tags such as:

- Spiders.
- Mind control.
- Imprisonment.
- Body transformation.
- Romance.

Player settings:

- Allow.
- Reduce detail.
- Skip optional content.

Required main content needs an authored alternate presentation or a clear warning before the campaign begins.

## Snapshot matrix

Render:

- 80x24 compact Unicode.
- 80x24 compact ASCII.
- 120x36 standard true color.
- 120x36 no color.
- Reduced motion final frame.
- Too-small warning.
- Transcript text.

## Manual test

1. Use Windows Terminal.
2. Use a second available terminal.
3. Test 80x24.
4. Test no color.
5. Test ASCII-only.
6. Test reduced motion.
7. Hold a key during animation.
8. Test transcript mode with redirected output.

## Acceptance check

- Art never clips at supported sizes.
- Animation does not busy-wait.
- Any key skips without selecting.
- All information survives no-color mode.
- ASCII mode contains no required Unicode glyph.
- Transcript mode can complete the MVP.
- Public art has recorded provenance.

## Suggested commit

```powershell
git add .
git commit -m "Add responsive ASCII art and accessibility modes"
```

