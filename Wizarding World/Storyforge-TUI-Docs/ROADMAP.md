# Storyforge product roadmap

## What this roadmap is for

This file turns the design sheet and implementation guide into one build order.
It separates work that is already designed from ideas that still need their own
implementation chapters.

The documentation is designed. The Rust game has not been built yet. A feature
does not become complete because it has a data model in a Markdown file. It is
complete when it works from a clean launch, survives save and load, appears
clearly at 80x24, and has automated tests.

Version numbers after 1.4 are planning labels, not promised release dates.

## The dependency path

```text
Toolchain and workspace
        |
        v
Terminal boot and responsive shell
        |
        v
Character creation and content loading
        |
        v
One complete story, check, combat, reward, save loop
        |
        v
Exploration, time, inventory, magic, quests, and school life
        |
        v
NPC memory, factions, companions, campaign arcs, and endings
        |
        v
Public native release through npm and npx
        |
        v
Living-world simulation and deeper tactics
        |
        v
Investigation, consequence, travel, crafting, and campaign-life expansions
```

Do not build a later box to avoid finishing an earlier one. Dynamic combat maps
will not rescue a game that cannot safely save one completed duel.

## Roadmap for the systems already designed

### Foundation: learn the tools and protect the terminal

| Stage | Guides | Playable result | Exit gate |
| --- | --- | --- | --- |
| Project rules | 00-02 | The workspace, content boundary, commands, and milestone habits are established. | A clean checkout can build the empty workspace. |
| M0: terminal boot | 03-04 | The program enters the alternate screen, draws a responsive dashboard, reads input, and restores the terminal. | Normal exit, error exit, and panic restoration all leave the terminal usable. |
| Engine foundation | 05-06 | Input becomes typed commands, results become serializable events, and original RON content validates before play. | The same seed and command sequence produce the same event sequence. |

### M1: character creation

Guide 07 builds a complete shopping-day prologue instead of a form:

- Identity, background, traits, casting style, and study interests.
- A visit to the equipment shops.
- A companion or familiar meeting.
- Wand trials with deterministic candidates.
- A final review that creates a valid level-one character.

The stage is complete when every choice can be revisited before confirmation,
invalid combinations explain themselves, and the finished character survives a
save round trip.

### M2: first playable story loop

Guides 08-11 connect:

```text
Launch
  -> load original campaign
  -> enter a scene
  -> make a consequential choice
  -> show a d20 check
  -> fight one tactical duel
  -> receive a consequence or reward
  -> save
  -> quit
  -> resume at the exact resulting state
```

This is the first release candidate worth handing to another player. Every
later feature must keep this playthrough passing.

### M3: explorable alpha

Guides 12-14 add the daily game loop:

- A location graph and resumable travel journeys.
- Story-first random travel events and optional story hooks.
- A game clock, rest rules, and schedule crossings.
- Inventory, equipment, shops, loot, crafting, and potions.
- Cantrips, spell slots, upcasting, sorcery points, and metamagic.
- Quests, classes, exams, clubs, school competition, and puzzles.

The alpha gate is one small region that supports a week of in-game decisions.
The player must be able to miss an activity, spend resources, travel, solve a
problem in more than one way, and see those decisions affect a later scene.

### M4: campaign beta

Guides 15-18 add the systems needed for the three-arc story:

- NPC memories and relationship axes.
- Faction reputation and internal leadership.
- Recruitable companions with combat control and departure rules.
- Reactions, hazards, objectives, boss phases, and enemy goals.
- Campaign arcs, world phases, irreversible decisions, and ending evaluation.
- ASCII art, animation timing, compact layouts, and accessibility fallbacks.

The beta gate is one route through Reconstruction, Fracture, and Convergence
using small fixture content. It must produce an ending and epilogues from saved
decisions rather than a final alignment choice.

### M5: public version 1

Guides 19-21 prepare the engine and original demo for release:

- The private campaign remains in a sibling private repository.
- Public packs contain original names, prose, art, and scenario content.
- Saves and content have explicit versions and migrations.
- Tests, traces, replay diagnostics, and performance budgets run in CI.
- Native archives are checksummed and selected by a small npm launcher.
- `npx storyforge-tui` installs and runs the correct binary.

Version 1 is complete only after inspecting the npm tarball and release archives
for private files.

### Version 1.1: author comfort

This release reduces the cost of writing and debugging campaign content:

- Better validation diagnostics with file paths and content IDs.
- Content search and reference inspection commands.
- Faster development reloads.
- More original fixture encounters.
- Accessibility refinements based on playtests.

### Version 1.2: living school and world

Guides 24-27 implement:

- Scheduled NPC locations.
- A school calendar with classes, clubs, curfew, exams, and ceremonies.
- Time-aware dialogue and bounded restocking.
- Gifts, quest favors, attitude changes, remembered comments, and rumors.
- Faction reactions and regional control.
- Regional modifiers, income, secret missions, mounts, danger, and route access.
- Long-running companion goals and scheduled downtime.

The release gate is a seven-day simulation that can skip forward, save, reload,
and reach the same schedules, stock, rumor knowledge, income, and companion
state without replaying every missed minute.

### Version 1.3: deeper tactics

Guide 28 implements:

- Reactions and counterspells.
- Terrain zones, cover, line of sight, and hazards.
- Nonlethal objectives, morale, and surrender.
- Companion commands and summons.
- Stealth openings and multi-phase bosses.
- Enemy goals that do not read hidden player state.
- Deterministic combat simulation reports.

The release gate includes one rescue, one escape, one nonlethal duel, and one
multi-phase boss. All four must be understandable through the ASCII map,
examine panel, and combat log.

### Version 1.4: creator ecosystem

This release turns the public content format into a supported author surface:

- Pack templates and example fixtures.
- Content and save migration tools.
- Localization workflow and coverage reports.
- Mod dependency and conflict reports.
- A published schema and condition/effect reference.

## The three-arc content roadmap

Engine milestones and story arcs are separate. The engine can be tested with
small original fixtures while the private campaign is written in parallel.

| Arc | Levels | Story work | Systems that must already work |
| --- | ---: | --- | --- |
| Reconstruction | 1-10 | School life, local mysteries, friendships, institutional repair, and the first irreversible decisions. | M3, relationships, schedules, rumors, and faction reputation. |
| Fracture | 11-16 | Regional conflict, changing alliances, companion loyalty, secrecy, and competing solutions to open threats. | M4, regional control, companion goals, deep travel, and world pressure. |
| Convergence | 17-20 | Failing boundaries, endgame plans, artifacts, alliance commitments, and several ending families. | Arc gates, boss phases, ending evaluation, epilogues, and save migrations. |

Each arc needs at least one combat route and one noncombat route through its
central conflict. Neither route may require one specific companion to remain
recruited.

## Fifty more story-first quality-of-life systems

These are substantial systems, not cosmetic rewards. "Quality of life" here
means that they help the player live in the story, remember consequences, make
plans, and express a character without turning the game into bookkeeping.

Every feature below must use typed commands and events. Time-based systems
advance through the bounded clock processor from chapter 24. Content supplies
names, dialogue, thresholds, and outcomes; Rust enforces rules and saves state.

### Version 1.5 candidate: investigation, familiars, and correspondence

| # | Feature | Why it belongs | Smallest playable proof |
| ---: | --- | --- | --- |
| 1 | Familiar aptitude and training | A familiar becomes a relationship and progression path instead of an inventory bonus. Training can develop scent, scouting, retrieval, distraction, or magical-sense aptitudes. | Train scent twice, then let the familiar reveal an optional clue during one investigation. |
| 2 | Contextual familiar field orders | The player can ask a trained familiar to search, follow, carry, distract, guard, or assist. This creates new solutions to puzzles and encounters without replacing the player's skills. | One scene accepts `Search`, `Distract`, and `Stay Safe`, with different risks and outcomes. |
| 3 | Familiar confidence, fatigue, and recovery | Sending a companion animal into danger should carry emotional and tactical weight. Confidence affects difficult orders; fatigue prevents repeatedly using the familiar as a free tool. | A risky search can succeed, refuse, or return injured, followed by a recovery scene. |
| 4 | Delayed courier mail | Distant relationships remain active when the player leaves school. Letters take in-game time, can arrive during other quests, and let contacts answer research or personal questions. | Send one letter, advance two days, receive a reply whose contents depend on relationship state. |
| 5 | Mail security and interception | Sensitive correspondence creates decisions about codes, trusted couriers, and delivery routes. Factions can intercept careless messages without making every letter randomly fail. | Send one ordinary and one encoded letter through a dangerous region; show the exposure preview before confirming. |
| 6 | Correspondence-led quests | A letter can open, advance, complicate, or quietly close a quest. This supports slow-burn plots that develop while the player works elsewhere. | A colleague requests help by mail, reacts to the reply, and changes a later meeting scene. |
| 7 | Evidence board and case notebook | Branching mysteries become easier to reason about when clues, claims, suspects, and unanswered questions are visible in one place. The board should record facts, not solve the case automatically. | Link two clues to a suspect, present a theory, and receive a consequence for a weak or strong case. |
| 8 | Clue provenance and contradictions | The player needs to know who supplied a claim, when it was learned, and whether later evidence conflicts with it. This makes investigation about judgment instead of collecting glowing objects. | Two witnesses disagree; the notebook marks the contradiction and unlocks a follow-up question. |
| 9 | Delegated research requests | Friends, teachers, librarians, and faction contacts can investigate questions during their own schedules. Strong relationships improve access or speed, while asking too much can create obligations. | Ask one contact to research a rune, wait through their next free period, then receive a sourced answer or refusal. |
| 10 | Witness memory and interview pressure | NPC testimony should reflect what the witness saw, what they believe, what they fear, and how the player treats them. Repeated questioning can build trust or make a witness shut down. | Interview one witness politely or aggressively and carry the resulting memory into a later hearing. |

### Version 1.6 candidate: secrecy, morality, and daily life

| # | Feature | Why it belongs | Smallest playable proof |
| ---: | --- | --- | --- |
| 11 | Magical exposure and secrecy | Magic used near non-magical communities can create witnesses, evidence, panic, opportunists, or official intervention. Exposure becomes a regional story pressure, not an instant game-over meter. | Cast openly during one rescue, then choose whether to explain, alter evidence, flee, or accept an inquiry. |
| 12 | Cover stories and aftermath repair | Secrecy is more interesting when the player can repair harm through persuasion, favors, mundane explanations, or restitution. Cleanup costs time and may conflict with another commitment. | Resolve one exposed incident through two distinct cleanup plans with different witnesses and costs. |
| 13 | Magical law, inquiries, and hearings | Institutions need procedures that can challenge powerful characters without defaulting to combat. Evidence, reputation, testimony, and legal allies can alter sanctions. | Play one disciplinary hearing where the outcome reads the evidence board, witness memories, and faction standing. |
| 14 | Dark-magic corruption | Repeated use of cruel or lethal magic can change available dialogue, dreams, spell behavior, relationships, and ending conditions. Corruption must be visible enough to support an informed choice. | Use a forbidden spell, gain a corruption mark, and see one companion confront the player about it. |
| 15 | Mercy, lethality, and responsibility | The world should remember whether the player captures, abandons, injures, or kills defeated enemies. Context matters: self-defense and execution are not treated as the same event. | End one encounter by surrender, knockout, escape, or lethal force and show four later reactions. |
| 16 | Atonement and recovery paths | Corruption should create a difficult story path, not permanently lock the player into an evil route after one mistake. Recovery can require confession, restitution, restraint, or sacrifice. | Complete a short restitution quest that removes one symptom but preserves the historical memory. |
| 17 | Food as preparation, not a hunger chore | Meals can affect recovery, concentration, morale, travel readiness, and social scenes. The player should plan meaningful meals without feeding the character every few real-world minutes. | Choose a quick snack or shared meal before travel and see different time, recovery, and relationship results. |
| 18 | Magical confection outcome tables | Sweets can produce short, readable benefits, drawbacks, sensory clues, or social complications. The existing [private candy notes](../World/Shops/Candy.md) can inspire private-pack tables; the public demo needs original products and wording. | Eat one mystery sweet, roll a seeded outcome, apply a timed effect, and let an NPC comment on it. |
| 19 | Shared meals and hospitality | Inviting someone to eat creates a natural place for rumors, reconciliation, cultural details, and difficult conversations. Food quality matters less than the invitation, setting, and remembered preferences. | Invite one NPC, choose a meal and topic, then create a memory that changes the next greeting. |
| 20 | Dietary, potion, poison, and condition interactions | Ingredients should matter to characters with allergies, curses, transformations, or active potions. Clear previews prevent this from becoming surprise punishment. | A meal can soothe one potion side effect, worsen another condition, or be declined based on known preferences. |

### Version 1.7 candidate: crafting, progression, and economy

| # | Feature | Why it belongs | Smallest playable proof |
| ---: | --- | --- | --- |
| 21 | Ingredient sourcing and resource ecology | Crafting gains story value when ingredients have seasons, habitats, owners, harvesting methods, and faction consequences. Gathering from a protected grove is a decision, not a button press. | Source one ingredient by purchase, ethical harvest, theft, or faction favor. |
| 22 | Recipe discovery and experimentation | Players can learn recipes from books, mentors, observation, and controlled experiments. Failed experiments should reveal information or create manageable complications. | Discover one recipe fragment, test a substitution, and record the result in the crafting notebook. |
| 23 | Crafting quality, substitutions, and mishaps | Skill, tools, ingredient condition, and preparation can affect duration or side effects without producing an endless item-rarity ladder. The result remains deterministic from recorded rolls and inputs. | Brew one potion at standard quality and one with a risky substitute, then compare their disclosed effects. |
| 24 | Loot provenance and legal ownership | Stolen, cursed, faction-marked, or historically important objects should draw different buyers and reactions. Loot becomes evidence and story leverage instead of anonymous gold. | Find a marked artifact and choose to return, conceal, fence, study, or present it to a faction. |
| 25 | Repair, modification, and item history | Important equipment can accumulate repairs, enchantments, owners, and memories. Breaking an item creates a quest or cost without deleting a beloved tool without warning. | Damage one focus, repair it through two specialists, and preserve the chosen repair in its examine text. |
| 26 | Multiple currencies and exchange | School tokens, common coin, faction scrip, favors, and rare materials can gate different economies. The wallet must show conversion and acceptance rules before purchase. | Buy the same supply with coin, school credit, or a favor and record the social consequence. |
| 27 | Debts, favors, bargains, and contracts | Help received now can become a request later. Explicit terms and a promise ledger keep bargains dramatic instead of arbitrary. | Accept one favor with a due date, then repay, renegotiate, or break it. |
| 28 | Shortages, smuggling, and black markets | Regional danger and faction control can change supply without merely raising every price. Scarcity can open alternate suppliers, moral choices, and investigative leads. | A blocked route removes one ingredient from normal stock and creates legal and illicit acquisition paths. |
| 29 | Character feats with narrative prerequisites | Feats give characters mechanical identity beyond spell choice. Some can require training, a mentor, a hard decision, or repeated behavior rather than appearing in a level-up list. | Unlock one feat through play, preview its rule change, and use it in both a check and encounter. |
| 30 | Long-form beast-shape apprenticeship | A path comparable to becoming an Animagus should take research, a mentor or forbidden source, risky practice, and identity consequences. Public content should use an original tradition; the private pack can apply its setting-specific name. | Complete the first transformation, choose a stable form trait, and handle one complication caused by being recognized. |

### Version 1.8 candidate: spatial play, travel, and school activities

| # | Feature | Why it belongs | Smallest playable proof |
| ---: | --- | --- | --- |
| 31 | Cursor-driven ASCII combat map | The player can select a reachable tile or zone directly on the map instead of choosing only "move closer" or "move away." The panel must still explain movement cost, danger, cover, and line of sight. | Move a cursor, preview a route, confirm movement, and reproduce the same path after save and load. |
| 32 | Manipulable terrain and spell-shaped battlefields | Doors, furniture, fire, ice, smoke, water, and fragile structures can change movement and objectives. This lets utility spells matter during combat. | Freeze water to create a route, burn a barrier, or move cover in one encounter. |
| 33 | Vertical movement and falling risk | Stairs, balconies, rooftops, pits, and flying actors add tactical choices without requiring a graphical engine. Height affects access and hazards, not every attack roll by default. | Climb to a balcony, push an enemy from it, and offer a visible falling-risk preview. |
| 34 | Light, noise, and concealment | Stealth becomes spatial and understandable when actions produce light and noise that specific actors can perceive. These signals can also drive investigation scenes. | Extinguish a light, distract a patrol with noise, and cross one watched room. |
| 35 | Scheduled academy rail travel | A fixed-route train can simplify long trips, create carriage scenes, and preserve the feeling of distance. The public engine uses an original railway; a private pack may skin it for its own setting. | Buy or earn a ticket, choose a departure, play one onboard event, and arrive with less route danger. |
| 36 | Discovered-location hearth network | A fireplace or hearth-gate network can skip routine travel between visited major hubs. It costs a resource or permission and does not work for unknown, sealed, or minor locations. | Unlock two hubs, travel instantly between them, and reject an undiscovered destination with a clear reason. |
| 37 | Broom and personal flight travel | Flight shortens journeys but can increase weather, pursuit, visibility, and aerial-event chances. Skill, broom condition, passengers, and regional law affect the route. | Fly one known route, compare time and event odds with ground travel, and resolve one aerial complication. |
| 38 | Travel permissions, customs, and checkpoints | Restricted destinations can require papers, disguises, reputation, bribes, secret routes, or an escort. Access becomes a story problem instead of a locked menu item. | Enter one controlled district through legal, deceptive, and covert routes. |
| 39 | Aerial school sport season | A team sport comparable to Quidditch can combine positioning, roles, checks, rivalries, injuries, training, and a season table. The public demo needs original rules and names. | Play one short match with three meaningful decisions and carry its rivalry result into school dialogue. |
| 40 | Reusable minigame contract | Potion timing, rune tracing, lock mechanisms, card games, music, and sports need a shared way to suspend a scene, accept commands, emit results, save, and resume. It prevents every minigame from inventing a second engine. | Implement one rune puzzle and one pub game through the same `MinigameCommand` and `MinigameEvent` boundary. |

### Version 1.9 candidate: public narrative and long-term continuity

| # | Feature | Why it belongs | Smallest playable proof |
| ---: | --- | --- | --- |
| 41 | Living newspapers and public reports | The world can publish partial or biased accounts of player actions, faction changes, disappearances, and disasters. A paper shows what society believes, not the engine's hidden truth. | Generate one issue from authored article templates after a regional event. |
| 42 | Propaganda and competing public narratives | Factions can frame the same event differently and spend influence spreading their version. The player may provide evidence, remain silent, or exploit the confusion. | Show two headlines for one event and let evidence shift which account dominates a region. |
| 43 | Calendar planner and commitment preview | A solo player needs to see travel time, classes, meetings, deadlines, companion plans, and expected mail in one place. The planner warns about conflicts but never makes choices automatically. | Schedule a meeting, preview a missed class, and confirm or cancel the conflict. |
| 44 | Promises, oaths, and missed commitments | Dialogue promises become tracked obligations with recipients, terms, and dates. NPCs can distinguish refusal, honest failure, renegotiation, and betrayal. | Promise to attend an event, miss or renegotiate it, and receive the correct remembered response. |
| 45 | Heist and mission planning board | Complex missions can record roles, entry points, timing, disguises, escape plans, and known risks before play begins. Preparation changes available commands rather than only granting a flat bonus. | Plan one archive break-in, assign a companion role, then adapt when one assumption proves false. |
| 46 | Councils, trials, debates, and votes | Social conflicts need encounter structure, objectives, initiative-like speaking opportunities, evidence, interruptions, and several possible outcomes. This gives high-level characters problems that cannot be solved by damage. | Resolve one council vote through testimony, bargaining, evidence, or intimidation. |
| 47 | Sanctuary and safehouse network | Safe locations can store items, shelter NPCs, host meetings, support recovery, and unlock routes. Improvements come from relationships and quests, not decorative furniture placement. | Secure one safehouse, choose its function, and see it change a later pursuit or recovery scene. |
| 48 | Mentor, apprentice, and teaching lineages | Characters can learn advanced techniques from mentors and eventually teach others. Teaching tests mastery and creates responsibility for how an apprentice uses that knowledge. | Learn one technique, teach its basic form to an NPC, and react to their later use of it. |
| 49 | Long rituals and group projects | Some magic should require several stages, rare components, calendars, safe locations, helpers, and interruption handling. This supports major plans without reducing them to one skill check. | Prepare a three-stage ward over several days and recover from one interrupted stage. |
| 50 | Return-to-game briefing and intention log | A solo story can span months of real time. On load, the game should summarize where the player is, why they are there, recent consequences, promises, incoming mail, and the next timed risk without revealing secrets. | Resume a week-old save and receive a generated briefing sourced only from recorded events and visible quest state. |

## How the fifty features should be prioritized

Use this order inside each candidate release:

1. Pick one story problem in the original demo.
2. Implement the smallest feature proof listed above.
3. Add save fields and migration before adding more content.
4. Add validation errors for every referenced content ID and impossible state.
5. Test the command, event, reducer, save round trip, and 80x24 presentation.
6. Play the proof inside a real scene.
7. Keep, revise, or remove it based on whether it created a meaningful decision.

Do not implement all ten features in a release at once. Ship two or three through
the full path, then continue. Candidate version labels can be split when a
feature needs more content or migration work than expected.

## Shared architecture for the future systems

Most of these features fit the existing crates:

| Concern | Owner |
| --- | --- |
| Familiar orders, corruption, promises, currencies, movement, mail timing | `storyforge-core` |
| NPC lines, recipes, articles, train routes, foods, feats, rituals | `storyforge-content` |
| Maps, planners, evidence boards, mailboxes, briefings, minigame screens | `storyforge-tui` |
| Private setting names, canon locations, private NPCs, and campaign prose | Sibling private campaign pack |

Prefer extending the existing condition and effect vocabulary. Add a new Rust
primitive only after several content examples need the same rule.

Long-running work uses boundary events such as:

```text
MailDeliveryDue
TrainingSessionCompleted
MealEffectExpired
ShopRestockDue
RitualStageDue
AppointmentDue
NewspaperIssueDue
```

The engine processes crossed boundaries in stable order. It never simulates
each missed minute after a long rest or a loaded save.

## Public naming and licensing boundary

The reusable engine may support the mechanic of a magical train, fireplace
travel, beast transformation, or aerial school sport. The public demo must use
original names, rules, teams, locations, dialogue, and art.

The sibling private pack can provide setting-specific records for personal use.
Those records must not enter npm staging, release archives, fixtures, snapshots,
or Git history in the public repository.

## Roadmap acceptance rules

A roadmap feature is ready to become an implementation chapter only when:

- It creates a player decision or removes story-state confusion.
- Its first playable proof is small enough to finish.
- The state owner and event boundary are named.
- Save and migration effects are understood.
- Failure and refusal remain playable.
- The TUI can explain the rule at 80x24.
- Original public content can demonstrate it.
- At least one automated test can prove its central invariant.

If a proposal only adds a collectible, animation, skin, or numerical bonus, it
does not belong on this roadmap without a stronger story use.
