# 22: Grow the engine after version 1

## Result

This stage gives you decision rules for expanding the engine without turning the first release into a permanent prototype or replacing working systems because a different architecture sounds more advanced.

Version 1 is the beginning of content production. Protect its save files, content packs, and author workflow while you add depth.

## Start with player evidence

Before adding a major system, answer:

```text
What player experience is missing?
Which existing system prevents that experience?
Can content solve it without engine work?
What is the smallest engine change that unlocks it?
How will we test and measure the result?
Does it change content or save schemas?
```

Keep a short architecture decision record for choices that affect multiple crates or public formats:

```text
docs/decisions/
  0001-command-event-core.md
  0002-ron-content-json-saves.md
  0003-no-ecs-for-v1.md
```

Each record states the context, decision, consequences, and conditions that would justify revisiting it.

## Grow content before adding machinery

The engine is healthy when a campaign author can create:

- New scenes without Rust changes.
- New checks using existing skills and conditions.
- New enemies by composing actions and behavior weights.
- New spells from existing targeting and effect primitives.
- New quests from objectives and world flags.
- New endings from accumulated state.

When content repeatedly needs the same awkward workaround, capture three concrete examples. Those examples form the requirements for a new primitive.

## Add a scripting language only when needed

RON conditions and effects should cover the first release. Consider Rhai when authors need calculations or branching that would otherwise create a large collection of one-off Rust effect types.

Before embedding Rhai, build a prototype with:

- No filesystem access.
- No network access.
- No process execution.
- A strict operation budget.
- A maximum call depth.
- A small allowlisted game API.
- Deterministic random access supplied by the engine.
- Script source locations in error messages.
- Versioned script API documentation.

Expose intentions, not internal state:

```text
get_flag("arc1.archive_opened")
relationship("npc.mara", "trust")
emit("quest.advance", "quest.lost_tome")
roll("2d6")
```

Do not expose arbitrary mutable Rust objects. Commands emitted by scripts must pass the same validation as commands from the TUI.

Reject or suspend a script that exceeds its budget. A broken optional scene should produce a useful content error, not freeze the terminal.

## Reconsider ECS only with evidence

The plain typed state and command/event reducer are the default architecture.

Evaluate an ECS when profiling and content requirements show several of these conditions:

- Thousands of active entities need the same systems each tick.
- Dynamic component composition is common.
- Queries across changing component sets dominate engine code.
- World simulation costs exceed agreed budgets.
- Adding entity kinds requires duplicated update loops.
- A small ECS prototype is clearer and faster for a representative area.

Run the prototype in a branch or isolated crate. Compare:

```text
Implementation size
Content author impact
Save migration complexity
Runtime measurements
Debuggability
Testing ergonomics
Mod compatibility
```

Do not migrate dialogue, quests, and static content into an ECS merely for consistency. A hybrid architecture may be the right result.

## Add audio as an optional adapter

Terminal play must remain complete without sound. If audio is added:

- Put playback behind a trait.
- Keep cues in content data.
- Offer independent music, ambience, and effects volume.
- Provide captions or log messages for meaningful cues.
- Handle missing output devices gracefully.
- Ship only licensed audio.
- Do not make audio initialization block game startup.

Example content:

```ron
(
    id: "academy-demo.audio.archive-rumble",
    channel: Ambience,
    file: "assets/audio/archive-rumble.ogg",
    caption: "Stone shifts somewhere beneath the archive.",
)
```

An audio feature flag can keep minimal builds lightweight.

## Prepare localization before translating

Separate stable text IDs from rendered text:

```text
scene.arrival.gatekeeper.greeting
item.healing_draught.name
combat.status.burning.applied
```

Design for:

- Variable interpolation.
- Plural rules.
- Gender and grammatical context where required.
- Dialogue variants.
- Text expansion.
- Unicode width in the terminal.
- Fonts and terminals that cannot display every glyph.
- Translator notes and screenshots.

Do not split sentences into fragments that translators must reconstruct. Keep fallback English and report missing keys in validation.

## Build author tools from real pain

Useful post-v1 tools may include:

- A content ID browser.
- Reference search.
- Dialogue graph visualization.
- Quest-state inspection.
- A combat simulator.
- A save flag editor for development.
- An ASCII animation preview.
- A localization coverage report.
- A pack migration command.

Start each tool as a subcommand in the existing `storyforge` executable. Split author tools into a separate crate only when their dependencies or release cycle make that boundary useful. Create a separate graphical editor only when terminal and text workflows demonstrably slow authors down.

Development tools that mutate saves must create a backup and mark the save as modified.

## Deepen the world simulation

Add simulation in layers:

1. Scheduled NPC locations.
2. Time-aware dialogue.
3. Faction reactions.
4. Resource and shop restocking.
5. Rumor propagation.
6. Regional danger and access changes.
7. Long-running companion goals.

Each layer should consume and emit explicit events. Keep simulation work bounded when time advances; do not replay every missed minute individually after a long rest.

The implementation is split across:

- [Chapter 24](24-living-world-schedules-and-restocking.md): NPC schedules, school calendar, time-aware dialogue, shops, and resource restocking.
- [Chapter 25](25-relationships-gifts-and-rumors.md): gifts, quest favors, attitudes, remembered comments, milestones, and rumor propagation.
- [Chapter 26](26-faction-control-and-regional-consequences.md): faction reactions, regional control, benefits, penalties, income, missions, mounts, danger, and route access.
- [Chapter 27](27-companion-goals-and-downtime.md): companion goals, autonomous plans, scheduled downtime, recall, and arc continuity.

Use summaries for distant regions:

```text
Region pressure
Faction control
Available resources
Unresolved threats
Named NPC state
```

Materialize detailed actors only when the player enters a relevant location.

## Expand combat without losing clarity

Potential post-v1 systems include:

- Reactions and counterspells.
- Environmental hazards.
- Cover and line of sight.
- Summons.
- Multi-phase bosses.
- Companion commands.
- Nonlethal objectives.
- Stealth openings.
- Morale and surrender.

Add one dimension at a time. Every action needs:

```text
Rules resolution
AI evaluation
TUI presentation
Keyboard path
Content validation
Save representation
Tests
Combat log wording
```

If a rule cannot be explained in the examine panel and combat log, it is not ready for players.

[Chapter 28](28-version-1-3-deeper-tactics.md) implements these systems in release order and adds deterministic combat simulation reports.

## Support New Game Plus carefully

Define exactly what carries forward:

```text
Unlocked lore
Cosmetic themes
Achievements
Selected spell variants
Difficulty modifiers
Ending history
```

Do not carry quest flags or relationships unless the campaign explicitly supports them. Create a new save with imported unlocks instead of rewriting the completed save.

## Stabilize public extension points

Not every Rust module is a public API. The supported extension surface is:

- Content manifest.
- RON schemas.
- Namespaced IDs.
- Condition and effect catalog.
- Save import and export policy.
- Tool commands intended for pack authors.

Mark experimental fields and commands. Publish schema changes with migration notes and examples. Keep unknown optional fields forward-compatible where it is safe, but reject unknown rule kinds that could change gameplay silently.

## Use a staged roadmap

The root [product roadmap](../ROADMAP.md) consolidates the full M0 through
version 1.4 build order and groups fifty later story-first systems into
dependency waves. This chapter supplies the rules for deciding when one of
those candidates deserves engine work.

### Version 1.1: Author comfort

- Better validation diagnostics.
- Content search commands.
- Faster pack reload during development.
- More demo encounters.
- Accessibility refinements.

### Version 1.2: Living school

- Scheduled NPC locations and a school calendar from chapter 24.
- Club, class, curfew, and time-aware dialogue from chapter 24.
- Relationship reactions, remembered gifts, favors, and rumors from chapter 25.
- Faction-controlled regional services and consequences from chapter 26.
- Companion goals and downtime from chapter 27.

### Version 1.3: Deeper tactics

- Bounded reactions and counterspells.
- Named terrain zones, cover, line of sight, and hazards.
- Summons, companion commands, stealth openings, and boss phases.
- Expanded AI goals with a strict knowledge boundary.
- Nonlethal objectives, morale, and surrender.
- Deterministic combat simulation reports.

Chapter 28 contains the complete version 1.3 implementation sequence.

### Version 1.4: Creator ecosystem

- Pack templates.
- Migration tools.
- Localization workflow.
- Mod dependency reports.
- Published schema reference.

### Version 2.0 candidates

- A deliberately versioned scripting API.
- Large-scale regional simulation.
- Optional graphical author tools.
- Breaking schema cleanup with automated migration.

A version number is not a deadline. Promote a candidate only when its player value and compatibility cost are understood.

### Versions 1.5-1.9: story-life candidates

The detailed candidates live in the [product roadmap](../ROADMAP.md):

- Investigation, trained familiars, and delayed correspondence.
- Secrecy, dark-magic consequences, mercy, food, and hospitality.
- Ingredient sourcing, crafting depth, item provenance, currencies, feats, and a long-form transformation path.
- Cursor-driven ASCII combat maps, terrain interaction, magical travel networks, flight, school sport, and reusable minigames.
- Newspapers, propaganda, commitment planning, heists, councils, safehouses, teaching, rituals, and return-to-game briefings.

These labels may be split. Finish a small playable proof and its save migration
before starting the next candidate.

## Protect the three-arc campaign

As systems grow, maintain automated campaign coverage:

- Every arc has at least one playable route.
- Arc transitions validate required state.
- Optional factions can change the route without blocking completion.
- A failed check has a consequence and a continuation.
- Companion loss does not make the main story impossible.
- Each ending declares the flags and relationships it reads.
- Old saves migrate across content updates or receive a clear compatibility message.

Keep an arc matrix in the private campaign:

| Route | Reconstruction | Fracture | Convergence | Ending family |
| --- | --- | --- | --- | --- |
| Reformer | Rebuild trust | Hold the coalition | Share power | Renewal |
| Loyalist | Restore order | Defend institutions | Centralize control | Stewardship |
| Dissident | Expose buried harm | Break alliances | Reject the old system | Liberation |
| Opportunist | Gain influence | Trade loyalties | Claim authority | Ascendance |

These are route functions, not fixed plots. Individual scenes and endings should emerge from accumulated choices, relationships, faction standing, and unresolved costs.

## Definition of healthy growth

The engine is growing well when:

- A small content change remains a small review.
- Invalid data fails before play.
- A new system has an observable player benefit.
- Save compatibility is deliberate.
- Performance claims have measurements.
- Public and private material remain separated.
- Terminal accessibility is part of each feature.
- The original MVP playthrough still passes.
- The three-arc campaign gains paths without duplicating the engine.

## Acceptance check

- Major architecture changes require evidence and a recorded decision.
- Scripting and ECS have explicit adoption gates.
- Audio and localization remain optional, testable adapters.
- Author tools grow from measured workflow problems.
- Public extension points are documented and versioned.
- The roadmap improves content production before adding spectacle.
- Three-arc route coverage remains automated.

## Suggested commit

```powershell
git add .
git commit -m "docs: define post-v1 engine growth strategy"
```
