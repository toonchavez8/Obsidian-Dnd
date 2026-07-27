# Storyforge documentation review ledger

## How to read this ledger

`Complete` means the documentation pass was performed. `Pending runtime` means the lesson still needs to be compiled and played in the future Rust repository. Editorial review is not a substitute for runtime proof.

The first complete-set inventory was performed on 2026-07-27. It covered every Markdown file in this folder and the three private design references named in chapter 12 and the game sheet.

## File status

| File | Structure | Contracts | Continuity | Junior pass | Runtime | Review note |
| --- | --- | --- | --- | --- | --- | --- |
| `README.md` | Complete | Complete | Complete | Complete | Not applicable | Roadmap, three arcs, spell slots, shopping-day MVP, sibling private repo, and chapter map agree. |
| `DND-rpg-ww-tui-game-sheet.md` | Complete | Complete | Complete | Complete | Not applicable | Expanded character creation, TES-style travel events, story hooks, spell slots, and private-public boundary. |
| `ROADMAP.md` | Complete | Complete | Complete | Complete | Not applicable | Consolidates the designed build order and fifty substantial story-first candidate systems with reasons and playable proofs. |
| `REVIEW-LEDGER.md` | Complete | Complete | Complete | Complete | Not applicable | Includes every documentation file and keeps editorial proof separate from future runtime proof. |
| `Guide/00-how-to-use-this-guide.md` | Complete | Complete | Complete | Complete | Not applicable | Defines the required anatomy of every implementation step and the review loop. |
| `Guide/01-development-environment.md` | Complete | Complete | Complete | Complete | Pending runtime | Commands and environment checks are explicit; rerun against the selected Rust toolchain. |
| `Guide/02-create-the-workspace.md` | Complete | Complete | Complete | Complete | Pending runtime | Public workspace no longer creates an in-repository private campaign directory. |
| `Guide/03-first-terminal-application.md` | Complete | Complete | Complete | Complete | Pending runtime | Terminal lifecycle code, panic restoration, tests, and manual exit checks are present. |
| `Guide/04-responsive-dashboard.md` | Complete | Complete | Complete | Complete | Pending runtime | Layout breakpoints and widget responsibilities are specified with visual checks. |
| `Guide/05-engine-command-event-model.md` | Complete | Complete | Complete | Complete | Pending runtime | Initial command, event, reducer, deterministic RNG, and tests are implemented. |
| `Guide/06-content-pack-format.md` | Complete | Complete | Complete | Complete | Pending runtime | Loader, validation report, fixtures, and CLI behavior are explicit. |
| `Guide/07-character-creation.md` | Complete | Complete | Complete | Complete | Pending runtime | Rewritten as six checkpoints with shopping scenes, companion meetings, wand trials, handlers, reducers, and tests. |
| `Guide/08-save-load-and-migrations.md` | Complete | Complete | Complete | Complete | Pending runtime | Envelope, atomic write, migration, slot behavior, and failure recovery are specified. |
| `Guide/09-scenes-dialogue-and-skill-checks.md` | Complete | Complete | Complete | Complete | Pending runtime | Scene records, condition/effect ownership, visible rolls, and dialogue tests are present. |
| `Guide/10-tactical-combat-mvp.md` | Complete | Complete | Complete | Complete | Pending runtime | Naked command and event lists were replaced with typed payloads and behavior contracts. |
| `Guide/11-first-playable-mvp.md` | Complete | Complete | Complete | Complete | Pending runtime | Integrates the earlier systems into one bounded launch-to-resume slice. |
| `Guide/12-world-exploration-and-time.md` | Complete | Complete | Complete | Complete | Pending runtime | Rewritten with distance tiers, semantic pillars, event cards, deterministic selection, journey state, commands, reducers, content, UI, and tests. |
| `Guide/13-magic-inventory-economy.md` | Complete | Complete | Complete | Complete | Pending runtime | Uses cantrips, spell slots, upcasting, Flexible Casting, and Metamagic instead of mana. |
| `Guide/14-quests-puzzles-and-school-life.md` | Complete | Complete | Complete | Complete | Pending runtime | Quest state, puzzle attempts, schedules, classes, and failure consequences are separated. |
| `Guide/15-npcs-factions-and-companions.md` | Complete | Complete | Complete | Complete | Pending runtime | Relationship axes, memory, factions, companion state, and departure behavior are specified. |
| `Guide/16-deeper-combat-and-enemy-ai.md` | Complete | Complete | Complete | Complete | Pending runtime | Multi-actor positioning, AI scoring, bosses, objectives, and performance gates are explicit. |
| `Guide/17-three-arc-campaign-foundation.md` | Complete | Complete | Complete | Complete | Pending runtime | Reconstruction, Fracture, and Convergence are represented as data, state, gates, and ending facts. |
| `Guide/18-ascii-art-animation-and-accessibility.md` | Complete | Complete | Complete | Complete | Pending runtime | Art fallbacks, animation clock, color-independent meaning, and terminal sizes are covered. |
| `Guide/19-private-content-and-modding.md` | Complete | Complete | Complete | Complete | Pending runtime | Sibling private repository, provenance, safe pack loading, release-audit code, and recovery policy are explicit. |
| `Guide/20-testing-observability-and-performance.md` | Complete | Complete | Complete | Complete | Pending runtime | Test layers, properties, traces, replay diagnostics, budgets, and profiling gates are present. |
| `Guide/21-release-with-npm-and-npx.md` | Complete | Complete | Complete | Complete | Pending runtime | Clean checkout, allowlisted staging, release audit, native targets, npm publishing, and smoke tests are connected. |
| `Guide/22-growing-after-v1.md` | Complete | Complete | Complete | Complete | Pending runtime | Growth decisions require measured pressure and preserve the engine-content boundary. |
| `Guide/23-documentation-review-loop.md` | Complete | Complete | Complete | Complete | Not applicable | Defines the repeatable five-pass process, scans, link check, repair standard, and cadence. |
| `Guide/24-living-world-schedules-and-restocking.md` | Complete | Complete | Complete | Complete | Pending runtime | Implements scheduled NPC locations, school calendar boundaries, time-aware dialogue, and bounded restocking. |
| `Guide/25-relationships-gifts-and-rumors.md` | Complete | Complete | Complete | Complete | Pending runtime | Implements gifts, quest impacts, attitude bands, memory comments, milestones, and rumor transmission. |
| `Guide/26-faction-control-and-regional-consequences.md` | Complete | Complete | Complete | Complete | Pending runtime | Implements tagged faction reactions, regional control, visible benefits and penalties, income, missions, and travel assets. |
| `Guide/27-companion-goals-and-downtime.md` | Complete | Complete | Complete | Complete | Pending runtime | Implements multi-stage companion goals, downtime planning, recall, return summaries, and arc continuity. |
| `Guide/28-version-1-3-deeper-tactics.md` | Complete | Complete | Complete | Complete | Pending runtime | Implements the v1.3 order for reactions, terrain, objectives, summons, stealth, bosses, AI goals, and simulation reports. |

## Cross-cutting decisions verified

| Decision | Verified locations |
| --- | --- |
| Cantrips and spell slots replace mana | README, game sheet, character, combat, travel/rest, magic, saves, release licensing |
| Shopping-day character creation | README, game sheet, Guide 07 |
| Story-first travel distance and pillar combinations | Game sheet, Guide 12 |
| Random events create optional story hooks | Game sheet, Guide 12, three-arc foundation |
| Private story history stays private | Guide 00, Guide 02, README, game sheet, Guide 19, Guide 21 |
| Three-arc foundation remains an engine requirement | README, game sheet, Guide 17, Guide 22 |
| Living-world time advances through bounded boundaries | Game sheet, Guides 12, 24-27 |
| NPC attitudes remember gifts and quest actions | Game sheet, Guides 15 and 25 |
| Faction control changes regions through visible modifiers | Game sheet, Guides 17 and 26 |
| Version 1.3 tactics preserve one command/event path | Game sheet, Guides 10, 16, and 28 |
| Later story-life systems remain staged candidates | ROADMAP, README, game sheet, and Guide 22 |

## Next runtime pass

When the Rust repository reaches each chapter, replace `Pending runtime` with the milestone tag or commit that passed:

```text
Runtime: Complete at m1-character
```

If the code does not compile exactly as written, leave runtime pending, record the compiler error in the row's review note, and correct both the guide and implementation before continuing.
