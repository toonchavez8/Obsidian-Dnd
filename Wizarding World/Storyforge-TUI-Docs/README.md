# Storyforge TUI

Storyforge TUI is the working plan for a single-player, story-driven terminal RPG engine written in Rust. The first game built with it is a magical academy campaign with character creation, exploration, choices, visible dice rolls, tactical combat, companions, factions, and several endings.

The terminal is the screen, not the limit. The game should feel like a small RPG that happens to render through text.

This folder contains the design and the build-along guide. It does not contain the Rust project yet. Follow the guide in order to create that project without having to decide the architecture while you are still learning the language.

## What we are building

The public project has three parts:

- `storyforge-core`: deterministic game rules and state transitions.
- `storyforge-content`: RON and TOML campaign loading and validation.
- `storyforge-tui`: the executable, terminal interface, saves, logs, and command-line tools.

The public release includes an original campaign named `academy-demo`. Your existing wizarding campaign stays in a separate local pack named `wizarding-world-private`. Keeping that line clear lets the reusable engine and original demo ship through npm while your private campaign can use the names, notes, maps, and reference material already in this vault.

The intended launch command is:

```powershell
npx storyforge-tui
```

During development, use:

```powershell
cargo run -p storyforge-tui -- play --pack campaigns/academy-demo
```

## The first playable MVP

The MVP is intentionally small. It is done when a player can:

1. Open a polished, responsive terminal dashboard.
2. Create and confirm a character through the Lantern Row shopping-day prologue.
3. Read a branching scene and make a meaningful choice.
4. See a d20 skill check with its modifier and result.
5. Complete one short tactical duel using a cantrip and a leveled spell.
6. Receive a consequence or reward.
7. Save, quit, relaunch, and continue.

This slice proves the risky parts: terminal lifecycle, input, rendering, deterministic rules, content loading, save compatibility, and a complete game loop. More locations and quests do not matter until these parts work together.

## Magic resource rule

Storyforge uses cantrips and spell slots:

- Cantrips are level-0 spells and do not consume slots.
- Leveled spells spend a slot of the spell's level or higher.
- Higher slots can upcast spells through authored effects.
- Long rests restore normal slots.
- Optional sorcery features add flexible casting and metamagic.

The engine tracks normal and temporary slots separately. Flexible casting can create temporary slots through 5th level or convert an available slot into sorcery points. The combat engine also tracks bonus actions so Quickened Spell and both conversion directions follow the same command and event rules as every other action.

## The three-arc foundation

The existing campaign README uses three large arcs. Storyforge keeps that scale as an engine requirement while leaving the final plot open for rewriting.

### Arc I: reconstruction, levels 1-10

The player enters a magical society repairing itself after a war. School life, local mysteries, new friendships, political pressure, and small signs of planar instability share the spotlight.

The engine must support:

- School schedules, classes, exams, clubs, and house competition.
- A connected school and grounds map with locked or hidden routes.
- Relationship memory and companion recruitment.
- Factions with internal disagreements rather than a single alignment.
- Local quests whose outcomes alter later scenes.
- The first world-phase changes and the first irreversible decisions.

### Arc II: fracture, levels 11-16

The conflict moves outside school. The player deals with institutions, resistance groups, competing communities, open violence, and travel through unstable portals.

The engine must add:

- Regional and planar location packs.
- Faction alliances, ceasefires, betrayals, and leadership changes.
- Companion loyalty missions and the possibility of departure.
- Large encounters resolved through combat, negotiation, sabotage, or rescue.
- World pressure that changes travel events, prices, NPC schedules, and available quests.
- Save-state continuity from every important Arc I decision.

### Arc III: convergence, levels 17-20

The boundaries between worlds fail. The player decides which alliances survive and what the magical world becomes.

The engine must add:

- Arc-gated endgame locations.
- Multi-stage plans and boss encounters.
- Artifact and alliance requirements.
- Endings evaluated from accumulated state rather than one final menu.
- Epilogues for companions, factions, locations, and unresolved promises.
- New Game Plus or an ending archive for exploring other paths.

The detailed plot remains campaign data. The engine only knows about arcs, conditions, effects, world variables, and ending rules.

## Milestone ladder

| Milestone | Result |
| --- | --- |
| M0: boot | A Rust binary starts, draws, receives input, and restores the terminal. |
| M1: character | A responsive shell can create and display a valid player character. |
| M2: playable MVP | One scene, one check, one duel, one reward, and save/resume work together. |
| M3: explorable alpha | Locations, travel, inventory, quests, shops, classes, NPC schedules, flexible casting, and metamagic work. |
| M4: campaign beta | Factions, companions, deep combat, world phases, arc gates, and endings work. |
| M5: public release | Original content ships as checksummed native binaries through npm and `npx`. |
| Version 1.2: living school | Schedules, relationship reactions, rumors, faction-controlled regions, and companion downtime work. |
| Version 1.3: deeper tactics | Reactions, terrain, nonlethal objectives, expanded AI, and combat reports work. |
| Versions 1.5-1.9: story-life candidates | Investigation, correspondence, consequence, crafting, spatial travel, school activities, and campaign continuity grow in dependency order. |

Do not skip milestone acceptance checks. A small feature that works from launch through save and reload is worth more than ten disconnected systems.

## Documentation map

Start with the [expanded game sheet](DND-rpg-ww-tui-game-sheet.md) when you want to understand the whole game. The [product roadmap](ROADMAP.md) shows the implementation order and fifty future story-first systems. Use the numbered guide when you are coding.

1. [How to use this guide](Guide/00-how-to-use-this-guide.md)
2. [Development environment](Guide/01-development-environment.md)
3. [Create the workspace](Guide/02-create-the-workspace.md)
4. [First terminal application](Guide/03-first-terminal-application.md)
5. [Responsive dashboard](Guide/04-responsive-dashboard.md)
6. [Engine command and event model](Guide/05-engine-command-event-model.md)
7. [Content pack format](Guide/06-content-pack-format.md)
8. [Character creation](Guide/07-character-creation.md)
9. [Save, load, and migrations](Guide/08-save-load-and-migrations.md)
10. [Scenes, dialogue, and skill checks](Guide/09-scenes-dialogue-and-skill-checks.md)
11. [Tactical combat MVP](Guide/10-tactical-combat-mvp.md)
12. [First playable MVP](Guide/11-first-playable-mvp.md)
13. [World exploration and time](Guide/12-world-exploration-and-time.md)
14. [Magic, inventory, and economy](Guide/13-magic-inventory-economy.md)
15. [Quests, puzzles, and school life](Guide/14-quests-puzzles-and-school-life.md)
16. [NPCs, factions, and companions](Guide/15-npcs-factions-and-companions.md)
17. [Deeper combat and enemy AI](Guide/16-deeper-combat-and-enemy-ai.md)
18. [Three-arc campaign foundation](Guide/17-three-arc-campaign-foundation.md)
19. [ASCII art, animation, and accessibility](Guide/18-ascii-art-animation-and-accessibility.md)
20. [Private content and modding](Guide/19-private-content-and-modding.md)
21. [Testing, observability, and performance](Guide/20-testing-observability-and-performance.md)
22. [Release with npm and npx](Guide/21-release-with-npm-and-npx.md)
23. [Growing after version 1](Guide/22-growing-after-v1.md)
24. [Repeat the documentation review loop](Guide/23-documentation-review-loop.md)
25. [Living-world schedules and restocking](Guide/24-living-world-schedules-and-restocking.md)
26. [Relationships, gifts, and rumors](Guide/25-relationships-gifts-and-rumors.md)
27. [Faction control and regional consequences](Guide/26-faction-control-and-regional-consequences.md)
28. [Companion goals and downtime](Guide/27-companion-goals-and-downtime.md)
29. [Version 1.3 deeper tactics](Guide/28-version-1-3-deeper-tactics.md)

The [review ledger](REVIEW-LEDGER.md) records which chapters received a structure pass, code-contract pass, and manual verification pass.

## Working rules

These rules run through every chapter:

- Keep rules deterministic. Rendering and input must not secretly change game state.
- Send player intent into the engine as a `GameCommand`.
- Record outcomes as `GameEvent` values.
- Save state with explicit schema and campaign versions.
- Use stable content IDs. Names can change; IDs cannot.
- Validate all content before a player can load it.
- Return `Result` for recoverable failures.
- Do not use `unwrap` or `expect` in production code.
- Use `thiserror` in libraries and `color-eyre` at the executable boundary.
- Borrow data when a function only needs to read it.
- Measure before optimizing.
- Keep public APIs documented and test their examples.
- Run formatting, Clippy, tests, and content validation before every milestone commit.

## Public and private content

The files already in this vault are useful, but they do not all have the same release status.

| Material | How the guide treats it |
| --- | --- |
| Your session notes and campaign ideas | Design reference and private campaign input. |
| Your original NPC and character concepts | Balance and story reference; publish only after renaming and confirming ownership. |
| Canon names, places, spells, and characters | Private-pack content only. |
| Wands & Wizards PDFs and character sheets | Reference only; do not copy them into the public package. |
| One-shot PDFs, catalogues, and third-party tables | Reference only unless their license clearly allows redistribution. |
| Existing maps and portraits | Private reference unless you own them or have a redistribution license. |
| Potion and trinket spreadsheets | Convert selected material only for the private pack unless each row's license is known. |
| New `academy-demo` content | Original and suitable for the public package. |

The current design pass consulted these private vault references:

- [Traveling Event System TES](../World/Wizarding%20World/Traveling%20Event%20System%20TES.md) for story-distance tiers and combat, roleplay, and exploration combinations.
- [Wizarding World Spanish guide](../World/Wizarding%20World/Wizarding-World-esp.md) for casting approaches, wand traits, school shopping, pets, and familiars.
- [One-Shot Wonders](../World/Wizarding%20World/One-Shot-Wonders.pdf) for scenario-card structure and objective categories.

These are design references only. Their prose and named adventures do not enter `academy-demo` or the npm package.

The private pack lives in its own sibling private Git repository and is selected explicitly:

```powershell
cargo run -p storyforge-tui -- play --pack ..\wizarding-world-private
```

An ignored `.storyforge.local.toml` may store that path for local development. The program should print the pack ID and version on the load screen. That makes it hard to accidentally test one campaign and release another. Chapter 19 explains why deleting files or rewriting Git history before npm publication is not a safe substitute for repository separation.

## Planned command-line interface

```text
storyforge
storyforge play
storyforge play --pack <path>
storyforge play --slot <number>
storyforge validate --pack <path>
storyforge new-pack <name> --output <path>
storyforge doctor
storyforge --version
```

`play` opens the TUI. `validate` checks content without starting the game. `new-pack` creates an original empty campaign template. `doctor` checks terminal features, directories, bundled content, and save permissions.

## Recommended terminal targets

| Size | Behavior |
| --- | --- |
| Smaller than 80x24 | Show a clear size warning and a quit key. |
| 80x24 | Compact single-column mode with tabbed secondary panels. |
| 120x36 | Standard dashboard with story, character, actions, quest, and log panels. |
| 160x45 or larger | Expanded art and side panels without stretching prose too wide. |

The UI must also work with ASCII-only borders, no color, and reduced motion. Those modes are functional requirements, not late polish.

## Technology choices

- Rust edition 2024
- Ratatui with Crossterm
- Serde
- RON campaign records
- TOML campaign manifests and configuration
- JSON save envelopes
- `rand_chacha` for deterministic random rolls
- `thiserror` for library errors
- `color-eyre` for top-level reports
- `tracing` and rolling file logs
- `insta` for small structural and terminal snapshots
- `proptest` for rule invariants
- `clap` for command-line parsing
- `directories` for platform-correct save paths
- `dist` for release archives and npm installers

There is no ECS in the first version. A turn-based game with a location graph and a few active actors does not need one. Chapter 22 defines the evidence required before adopting Bevy ECS.

There is no general-purpose story scripting in the first version either. Typed conditions and effects are easier to validate, save, test, and explain. A sandboxed Rhai extension is reserved for content that genuinely cannot be expressed declaratively.

## Definition of done for this documentation

This document set is complete when:

- The game sheet describes the full intended system and its boundaries.
- Every guide chapter has a concrete result and acceptance checkpoint.
- The MVP can be built by following chapters 00 through 11 in order.
- Later chapters grow the same architecture without replacing it.
- The three arcs have technical support in the data model and save format.
- The npm release path is native, cross-platform, and testable.
- Private campaign files never enter a public release by accident.
- The review ledger and chapter 23 make vague guide steps visible and correctable.
