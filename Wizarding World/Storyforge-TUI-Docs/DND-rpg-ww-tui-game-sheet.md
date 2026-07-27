# Storyforge TUI game design sheet

## One-sentence pitch

A single-player magical-school RPG with visible d20 rules, branching relationships, tactical turn-based combat, an explorable world, and several endings, presented through a responsive terminal dashboard.

## Product shape

Storyforge is a reusable terminal RPG engine. The magical academy campaign is its first game, not a set of special cases baked into Rust.

The public release contains an original campaign called `academy-demo`. A separate local pack can adapt the private wizarding campaign in this vault. Both packs use the same commands, content schemas, save format, combat rules, and terminal widgets.

The game is:

- Single-player.
- Keyboard-first.
- Story-led.
- Turn-based.
- Data-driven.
- Offline.
- Saveable at any safe point.
- Designed for Windows, macOS, and Linux terminals.

The game is not:

- A multiplayer virtual tabletop.
- A real-time action game.
- A full simulation of every person in the world.
- A direct digital copy of a tabletop sourcebook.
- A procedural story generator that replaces authored quests.

## Target player

The first audience is someone who likes choice-heavy RPGs and old terminal games but does not want to memorize terminal commands. Menus, shortcuts, tooltips, a help overlay, and visible results should make the interface learnable.

The writing targets a teen dark-academia tone. The world can cover prejudice, institutional failure, grief, betrayal, danger, and political conflict. Violence is not graphically described. School-age romance stays at friendship, crushes, dances, letters, jealousy, and first dates. Adult relationship content requires an adult-era campaign pack.

## Core promise

The player should be able to say:

> I made a character, found my own route through the school, changed how people saw me, won some fights, talked my way out of others, and reached an ending caused by decisions I remember making.

If the game cannot deliver that sentence, another spell list or monster table will not save it.

## Design pillars

### Story has priority

Combat, loot, puzzles, classes, and exploration must create or resolve story pressure. A fight without context is a short event, not the main diet.

### Choices alter state

A choice can change relationships, faction standing, location access, resources, quest routes, future dialogue, companion loyalty, or an ending condition. Cosmetic choices are fine when labeled through context, but the game must not pretend every line changes the world.

### Rules stay readable

The player sees the die, modifier, target, advantage state, and result. Enemy details are revealed by observation, experience, and character knowledge.

### The terminal feels alive

The screen redraws as a dashboard. Art, borders, color, focus, animation, and sound cues support the scene. The player is never dumped into a raw command prompt.

### Content lives outside the engine

Rust defines what a spell, quest, actor, scene, location, encounter, and ending can do. Campaign files define which ones exist.

### A small loop must work end to end

Every milestone finishes a playable loop and saves its state. Systems do not live as disconnected demonstrations.

## Experience goals

| Goal | Player evidence |
| --- | --- |
| Belonging | The player's affiliation, room, schedule, friends, and rivals change their daily route. |
| Discovery | Exploration reveals shortcuts, secrets, lore, quests, and usable knowledge. |
| Agency | Major outcomes have at least two viable approaches and record the choice. |
| Growth | New spells and traits open options in old and new situations. |
| Tension | Time, resources, injury, reputation, and competing promises force tradeoffs. |
| Mastery | The player learns enemy behavior, spell interactions, routes, recipes, and faction motives. |
| Consequence | Later scenes refer to prior actions without requiring the player to keep notes outside the game. |

## Release layers

### MVP

- Launch and safe terminal restoration.
- Responsive dashboard.
- Character creation.
- One branching scene.
- One visible skill check.
- One short duel.
- One relationship or reputation change.
- One reward.
- Manual save, autosave, quit, and resume.

Target play time: 20 to 35 minutes.

### Vertical slice

- Arrival sequence.
- Affiliation selection.
- One class activity.
- Three connected locations.
- Two NPCs with relationship state.
- Inventory and one shop.
- Three spells and two usable items.
- One optional secret.
- A duel with two possible resolutions.
- A closing scene that reflects earlier choices.

Target play time: 60 to 90 minutes.

### Alpha

- A school hub and grounds.
- Calendar and schedules.
- Main and side quests.
- Party of one player plus up to two companions.
- Shops, loot, equipment, crafting, and potions.
- Classes, exams, clubs, and house competition.
- Factions and reputation.
- Several enemy personalities.
- Content validation and mod loading.

### Beta

- Complete three-arc state model.
- Regional travel and planar destinations.
- Companion loyalty routes.
- Multiple boss structures.
- Multiple ending families.
- Accessibility and balance passes.
- Save migration from every earlier public version.

### Version 1

- Complete original campaign.
- Native release binaries.
- `npx storyforge-tui`.
- Installer smoke tests on every target.
- Crash recovery and player-facing diagnostics.
- Public pack-authoring documentation.

## Main game loop

```text
Read the current situation
          |
          v
Choose dialogue, travel, investigate, prepare, or fight
          |
          v
Resolve commands into visible events
          |
          v
Update character, relationships, quests, and world state
          |
          v
Spend time and trigger consequences
          |
          v
Discover new choices, locations, and problems
```

Combat, puzzles, and shops are temporary modes inside this larger loop. They return the player to the same world state.

## Campaign structure

The engine supports three arcs without understanding their plot.

```text
Arc I: Reconstruction, levels 1-10
  Local access, school systems, first factions, first world changes
                         |
                         v
Arc II: Fracture, levels 11-16
  Regional conflict, alliances, planar travel, companion loyalty
                         |
                         v
Arc III: Convergence, levels 17-20
  Endgame gates, world-scale choices, ending evaluation, epilogues
```

Each campaign defines:

- Arc ID and display name.
- Minimum and maximum level.
- Entry conditions.
- Required main-quest state.
- World-phase effects.
- Available location packs.
- Global encounter deck.
- End conditions.
- Transition scene.

Arc changes are explicit events. Loading a save must never guess the arc from player level alone.

## Character creation

Character creation is a playable prologue, not a configuration form. The player chooses identity and rules options, then visits a magical shopping street to obtain school supplies, meet a pet, and be chosen by a wand. The player can go backward before final confirmation and inspect the mechanical effect of every choice.

The private campaign can call the shopping street Diagon Alley. The public `academy-demo` uses the original location `Lantern Row` and original shops so no private or licensed setting content enters the npm package.

### Required choices

1. Name and pronouns.
2. Portrait glyph or small ASCII silhouette.
3. Origin and family circumstances.
4. Background.
5. Two personality traits.
6. One ideal and one fear.
7. Casting style: Will, Technique, or Intellect.
8. Starting magical-study interest.
9. Affiliation preference.
10. Starting skill proficiencies.
11. School equipment package.
12. Pet or familiar.
13. Wand.
14. Accessibility defaults.

The campaign may assign affiliation through a scene instead of a menu. In that case, the player's preference becomes one input rather than a guarantee.

### Playable creation sequence

Creation uses normal scenes, choices, commands, and autosaves:

```text
Identity and accessibility
  |
  v
Origin, background, traits, ideal, and fear
  |
  v
Casting style and magical-study interest
  |
  v
Arrival at the shopping street
  |
  +--> equipment and robes
  |
  +--> pet shop
  |
  +--> wand shop
  |
  v
Review purchases and character sheet
  |
  v
Confirm character and travel to school
```

The shopping street is a small explorable location graph. The player may choose the pet shop or wand shop first. Shopkeepers react to origin, budget, personality, and prior dialogue. The sequence teaches movement, dialogue, inventory, relationships, and inspection before the main story asks the player to use them under pressure.

Character creation can be suspended and resumed. The save stores a `CharacterCreationState` until final confirmation. It does not create a partly valid `PlayerCharacter`.

### Casting styles

The Spanish campaign guide uses three related caster approaches. Storyforge models them as data-driven magic paths:

| Style | Default casting ability | Play style |
| --- | --- | --- |
| Will | Charisma | Instinct, force of personality, emotional magic, and sorcery features. |
| Technique | Wisdom | Control, timing, reactions, practiced forms, and efficient recovery features. |
| Intellect | Intelligence | Study, ritual casting, broad preparation, and magical theory. |

All three use cantrips and spell slots. A campaign may give them different known-spell progressions, prepared-spell rules, or features, but they share the same spell definitions and command path.

The starting magical-study choice is an interest, not permanent specialization. It grants one dialogue tag, one introductory spell option, and a teacher hook. Formal specialization happens later through classes and story choices.

### Shopping budget

Origin or family circumstances choose a budget policy rather than judging the character:

- Assisted: the school or a sponsor covers required supplies.
- Second-hand: lower cost, worn equipment, and useful social hooks.
- Standard: ordinary school supplies and a small discretionary amount.
- Comfortable: better cosmetic choices but no exclusive combat advantage.

Required equipment can never become impossible to obtain. Money left after the shopping sequence enters the normal inventory.

### Pet shop scene

The source campaign distinguishes a school pet from a supernatural familiar. The engine supports both:

- A pet is an intelligent animal companion with exploration and social abilities.
- A familiar is a campaign-granted magical bond that may gain explicit spell or combat support.

The shopping scene presents three to five authored candidates. Each candidate has a species, temperament, need, exploration ability, complication, and first-impression scene. The player may:

- Observe the animal.
- Ask the keeper about its care.
- Offer a hand or item.
- Use an applicable skill.
- Leave and return.
- Adopt one candidate.

The chosen animal remembers how the meeting happened. That first memory seeds its bond and later behavior. Species is useful, but temperament and relationship should matter more.

### Wand shop scene

The wand is selected through trials rather than a statistical menu:

1. The shopkeeper asks two or three questions based on earlier character choices.
2. The engine selects a deterministic candidate pool from wood, core, length, flexibility, and affinity tags.
3. The player tests a candidate through a short input or choice.
4. The trial produces a harmless magical reaction, shopkeeper interpretation, and compatibility score.
5. The player may test another candidate.
6. A strong match can choose the player, but the player always confirms the final wand.

The candidate seed is stored so save and reload produce the same wands. There is no hidden best wand. A strong affinity opens situational options; a complication creates story and growth hooks.

### Creation completion

Final confirmation checks identity, rules choices, required supplies, one adopted companion, and one selected wand. It then emits:

- `CharacterCreated`
- `StartingInventoryGranted`
- `CompanionBondStarted`
- `WandBondStarted`
- `CreationHistoryRecorded`
- `SceneChanged` to the departure scene
- `AutosaveRequested`

Each event contains stable IDs and the choice that caused it. The history page can therefore explain where the character got their wand and how they met their companion.

### Core statistics

The streamlined rule set keeps the six familiar abilities:

| Ability | Covers |
| --- | --- |
| Strength | Force, climbing, carrying, grappling, and resistance to forced movement. |
| Dexterity | Reflexes, balance, stealth, aim, and initiative. |
| Constitution | Health, fatigue, poison, concentration under physical stress. |
| Intelligence | Magical theory, investigation, history, runes, and potion logic. |
| Wisdom | Perception, insight, survival, creatures, and concentration. |
| Charisma | Persuasion, deception, intimidation, performance, and magical presence. |

Scores normally range from 8 to 18 at creation. The modifier is:

```text
modifier = floor((score - 10) / 2)
```

### Skills

- Acrobatics
- Arcana
- Athletics
- Deception
- Herbology
- History
- Insight
- Intimidation
- Investigation
- Magical Creatures
- Medicine
- Muggle Studies
- Perception
- Performance
- Persuasion
- Potion Making
- Sleight of Hand
- Stealth
- Survival

Campaigns can add skills, but removing a core skill requires a new rules profile and save compatibility identifier.

### Derived values

- Maximum HP
- Defense
- Initiative
- Movement
- Proficiency bonus
- Spell attack bonus
- Spell save difficulty
- Spell-slot progression
- Carry capacity
- Concentration
- Resolve
- Passive perception
- Rest recovery

Derived values are calculated by the engine from level, abilities, background, traits, equipment, and active effects. Content files do not repeat calculated totals.

### Backgrounds

A background provides:

- Two skill proficiencies.
- One background feature.
- Starting items.
- A contact.
- One reputation adjustment.
- Two suggested personal hooks.

A background must open options, not only grant numbers. A groundskeeper background might recognize tracks, request help from staff, and enter work areas without immediately looking suspicious.

### Personality

Traits have a mechanical trigger and a roleplaying prompt.

Example:

```ron
(
    id: "trait.curious",
    name: "Curious",
    prompt: "You have trouble leaving a locked door alone.",
    trigger: FirstDiscoveryInLocation,
    reward: Inspiration(1),
)
```

The engine tracks whether a trigger was used during the current rest period.

### Wand

A wand has:

- Stable content ID.
- Wood or body.
- Core or focus.
- Length category.
- Flexibility.
- Affinity tags.
- One strength.
- One complication.
- Upgrade slots.
- ASCII casting style.

Wands influence a small set of situations. They do not turn character creation into a hidden optimal-build quiz.

### Familiar

A familiar has:

- Species.
- Name.
- Temperament.
- Bond score.
- Exploration ability.
- Social complication.
- Combat support ability, if the campaign enables one.
- Care requirement.

Familiars cannot die permanently in the school-era default rules. Severe harm sends them to care and creates a quest or relationship consequence.

## Character sheet

The character sheet is always reachable with `c`. It has tabs rather than one overcrowded screen:

```text
Overview | Skills | Magic | Inventory | Relationships | Quests | History
```

Overview shows:

- Name, pronouns, affiliation, background, level, and XP.
- HP, remaining spell slots, optional sorcery points, defense, initiative, movement, resolve, and conditions.
- Equipped wand, robe, accessory, and familiar.
- Current location, time, active objective, and currency.

History shows major decisions and the event that caused each one. It is an in-game memory aid and a debugging tool.

## Progression

The default campaign uses levels 1 through 20.

Leveling grants a controlled mix of:

- HP and spell-slot progression.
- Proficiency increases.
- Cantrip scaling.
- Spell access.
- Trait or feat choices.
- Prepared spell capacity.
- Familiar abilities.
- Equipment attunement.
- Social permissions.
- Arc or location access.

XP can be awarded for quests, discovery, relationships, puzzles, peaceful resolutions, and combat. Grinding random encounters is not the intended path.

Campaigns may use milestone leveling, but the save still records XP for analytics and optional display.

## Checks and dice

### Standard check

```text
d20 + ability modifier + proficiency + situational modifiers
```

The UI shows:

```text
Investigation
Roll: 17
Intelligence: +2
Proficiency: +2
Total: 21
Target: 15
SUCCESS
```

### Roll policy

Roll only when:

- Success and failure are both plausible.
- Failure has a consequence.
- The result is not already established by prior action or knowledge.

One check normally resolves one intent. Do not make a player pass three rolls to open one ordinary hidden door.

### Advantage and disadvantage

Roll two d20s and keep the higher or lower result. Multiple sources do not stack. Advantage and disadvantage cancel.

The UI lists the cause:

```text
Advantage: Cedric created a distraction.
Disadvantage: Your hands are burned.
Result: normal roll.
```

### Degrees of outcome

Content can use:

- Critical failure.
- Failure with consequence.
- Success with cost.
- Success.
- Critical success.

Natural 1 and 20 only become automatic outcomes when the specific rule says so. Skill checks use totals and authored degree thresholds.

### Inspiration

Inspiration is a limited player-controlled reroll resource. It is earned by acting on traits, resolving bonds, accepting complications, and discovering secrets.

## Narrative system

### Scene structure

A scene contains:

- ID and title.
- Speaker and prose blocks.
- Optional art.
- Entry conditions.
- Entry effects.
- Choices.
- Skill checks.
- Exit targets.
- Autosave policy.
- Accessibility notes.

Choices can:

- Move to another scene.
- Start a check.
- Start combat.
- Travel.
- Give or remove resources.
- Update a relationship.
- Update faction reputation.
- Advance or fail a quest objective.
- Set a world flag.
- Spend time.
- Reveal a codex entry.

### Conditions

Conditions are typed and composable:

```ron
All([
    HasItem("item.archive_key"),
    RelationshipAtLeast("npc.iren", Trust, 3),
    Not(FlagSet("archive.alarm_raised")),
])
```

Supported groups include:

- Character state.
- Inventory and equipment.
- Spell knowledge.
- Relationship axes.
- Faction reputation.
- Quest and objective state.
- Location discovery.
- Time and calendar.
- World phase and arc.
- Prior decision flags.
- Difficulty and accessibility options.

### Effects

Effects are also typed:

```ron
[
    ChangeRelationship("npc.iren", Trust, 1),
    SetFlag("archive.promised_help"),
    StartQuest("quest.missing_index"),
    AdvanceTime(minutes: 15),
]
```

Every state-changing effect emits a `GameEvent`. That event feeds the visible log, autosave decisions, achievements, and tests.

### Fail forward

Failure should usually change the route:

- The door opens but alerts a guard.
- The NPC refuses but names someone else.
- The player loses time and misses another opportunity.
- The clue is damaged, reducing certainty.
- A companion solves the problem and loses respect.

A dead end is appropriate only when the player can understand why it is closed and has another lead.

## Dialogue and relationships

### Relationship axes

Each tracked NPC may use:

- Trust
- Respect
- Fear
- Affection
- Rivalry
- Obligation

Not every NPC needs every axis. A shopkeeper may only track trust and obligation.

Values use a bounded integer range from -5 to +5. Content checks named thresholds rather than arbitrary raw numbers:

| Range | Meaning |
| --- | --- |
| -5 to -4 | Hostile |
| -3 to -2 | Distrustful |
| -1 to +1 | Uncertain |
| +2 to +3 | Supportive |
| +4 to +5 | Devoted |

### Memories

An NPC memory records:

- Event ID.
- Short player-facing summary.
- Emotional tag.
- Arc created.
- Whether it has been discussed.
- Optional expiration rule.

Memories let dialogue refer to specific events instead of only checking a number.

### Gifts, favors, and attitude

Gifts and quest outcomes use authored relationship impacts. An NPC may love, like, ignore, dislike, or hate an item based on its tags. The first meaningful gift can create a strong memory; repeated gifts have cooldowns and smaller effects so affection cannot be farmed.

An NPC's visible attitude is derived from the relationship axes they care about. One NPC may value Trust and Affection, while a rival weighs Respect and Rivalry more heavily. Attitude bands change greetings, available choices, willingness to help, and later comments.

Strong positive relationships unlock one-time character moments such as lessons, items, shortcuts, introductions, companion routes, faction recommendations, or help during a later quest.

Strong negative relationships create authored complications. These may include higher prices, withheld favors, rivalry, competing quest objectives, rumors, sabotage, a nonlethal duel, or story-appropriate combat. A hostile NPC cannot silently remove the only main-story route.

### Rumors

Rumors move between scheduled NPCs who meet at the same social location. A rumor stores its source, variant, confidence, and listeners. Repetition does not make it true. Evidence may confirm or disprove it later.

The engine propagates authored rumor variants through bounded calendar events. It does not generate or rewrite prose.

### Romance

Romance is opt-in at the campaign and player-settings levels. The school-era default uses age-appropriate crushes and dates. A rejected approach never lowers unrelated combat or quest competence.

### Companion loyalty

Companions track:

- Personal goal.
- Loyalty.
- Stress.
- Approval memories.
- Boundaries.
- Current equipment.
- Prepared abilities.
- Combat behavior.
- Personal quest.
- Departure condition.

A companion can disagree without becoming useless. Loyalty changes whether they take risks, reveal secrets, stay for an ending, or follow a morally difficult order.

## Factions and reputation

A faction has:

- Public identity.
- Internal groups.
- Goals.
- Resources.
- Territory.
- Allies and rivals.
- Reputation thresholds.
- Leadership state.
- Arc-specific agenda.
- Typical quests.
- Failure and transformation states.

No people or species is automatically a single hostile faction. Political conflicts require internal disagreement, civilians, reformers, profiteers, extremists, and people who want to be left alone.

Reputation ranges from -100 to +100 but content checks named ranks:

- Hunted
- Hostile
- Distrusted
- Unknown
- Known
- Respected
- Allied
- Champion

Large reputation changes require a named event. Buying ten cheap items cannot make the player a faction champion.

### Regional control and faction services

Faction influence and player reputation are separate:

- Influence determines which faction can act or govern in a region.
- Reputation determines how that faction treats the player.

A region may be uncontrolled, contested, or controlled. Control can change:

- Skill-check modifiers with a visible source.
- Shop prices and restock rates.
- Passive game-time income with a cap.
- Secret missions.
- Mounts, carriages, guides, or portal permits.
- Route access and travel time.
- Regional danger and travel-event pressure.

Friendly control provides services only when the player meets the required rank or agreement. Hostile control may impose tolls, scarcity, danger, surveillance, or access requirements. Contested regions use their own effects instead of combining every faction's full bonuses and penalties.

Every required story region has an authored fallback route under each tested controller.

## Quest system

### Quest categories

- Main
- Side
- Companion
- Faction
- Class
- Hidden
- Timed
- Repeatable event

### Quest model

A quest contains:

- Stable ID.
- Journal title and summary.
- Arc and level guidance.
- Start conditions.
- Objectives.
- Optional objectives.
- Failure conditions.
- Branch points.
- Rewards.
- Follow-up hooks.
- Cleanup effects.

Objective states are:

```text
Locked -> Active -> Completed
                 -> Failed
                 -> Abandoned
```

Completed and failed are terminal unless a specific effect reopens the objective and records why.

### Timed quests

Timed quests use game time, not wall-clock time. The journal shows the deadline and warns before travel or rest would cross it.

### Hidden quests

Hidden quests do not appear until the player recognizes the problem. Progress can be recorded before discovery so that prior actions still count.

## World and exploration

### Location graph

The world is a graph:

- Locations are nodes.
- Routes are edges.
- Doors, stairs, portals, paths, and travel services are route types.
- Conditions determine whether a route is visible, open, dangerous, or one-way.

This works better than a tile map for an authored terminal RPG. The player chooses meaningful destinations rather than tapping an arrow through empty corridors.

### Location data

A location has:

- ID, name, region, and tags.
- Description variants.
- Art variants.
- Routes.
- Interactions.
- NPC schedule slots.
- Encounter deck.
- Search discoveries.
- Rest rules.
- Ambient log messages.
- Music or sound cue, if enabled.
- Arc and world-phase variants.

### Exploration actions

- Travel.
- Observe.
- Search.
- Talk.
- Interact.
- Cast utility spell.
- Use item.
- Wait.
- Rest.
- Open map.

Search is not a button that repeatedly rolls until success. A search consumes time and uses the best applicable clue state, skill, tool, spell, and companion help.

### Time

The world clock tracks:

- Day.
- Date.
- Time of day.
- School term or campaign season.
- Scheduled events.
- Deadlines.
- Rest periods.

Commands declare their time cost. The engine advances time after their effects resolve, processes crossed schedule boundaries, then emits new events.

### Living-world boundaries

The simulation processes meaningful boundaries rather than individual minutes:

- NPC schedule starts and ends.
- Classes, clubs, exams, ceremonies, and curfew.
- Quest deadlines.
- World-phase changes.
- Shop deliveries and resource-node restocks.
- Companion downtime completion.
- Rumor contacts.

NPC locations are resolved from weekday, time, conditions, and explicit overrides. Dialogue can test time of day, active activities, lateness, and completed events. Large time jumps remain bounded because the engine sorts crossed boundaries and resolves each once.

### Travel events

Travel follows the event-based structure in the private `Traveling Event System TES.md` note. Routes are Close, Far, or Very Far. Distance determines dramatic event slots instead of simulating every mile:

| Distance | Default event slots |
| --- | ---: |
| Close | 1 |
| Far | 2 |
| Very Far | 3 |

A route may override the count for story pacing. Travel time still advances the world clock, but distance math never becomes the main activity.

#### Event pillars

The source note uses color categories. The TUI always adds words and symbols so color is optional:

| Pillar | Source color | Purpose |
| --- | --- | --- |
| Combat | Red | A hostile or dangerous confrontation. |
| Roleplay | Blue | Negotiation, culture, relationships, and social pressure. |
| Exploration | Yellow | Terrain, navigation, discovery, hazards, and problem-solving. |

Combo events contain two or three pillars:

- Purple: roleplay and combat.
- Green: exploration and roleplay.
- Orange: combat and exploration.
- White: all three.

Public content stores the pillar list, not only the color name.

#### Event objectives

The `One-Shot-Wonders.pdf` review suggests a useful second axis: what the player is trying to accomplish. Original Storyforge events use these objective tags:

- Acquisition.
- Competition.
- Confrontation.
- Defence.
- Delivery.
- Escape.
- Investigation.
- Rescue.

Pillar and objective are separate. A rescue may be social, exploratory, combat-focused, or a mixture.

#### Travel event card

Each event definition contains:

- ID, title, theme tags, pillars, objective, and setting tags.
- Eligible regions, routes, arcs, levels, time, weather, and world conditions.
- Weight, cooldown, repeat policy, and once-per-save flag.
- Opening hook.
- Important actors.
- Two to five story beats.
- Key locations or travel-stage variants.
- Secrets and clues that can be discovered in any valid order.
- Resolution paths.
- Rewards and consequences.
- Easier and harder tuning profiles.
- A main-story hook exported on completion.

This card structure is inspired by the PDF's compact one-shot layout, but published events must use original characters, places, prose, art, and encounters. The PDF is a private design reference, not distributable game content.

#### Travel selection

When travel begins:

1. Create the journey state and store the RNG position.
2. Reserve any mandatory route or quest event.
3. Filter random cards by route, arc, level, time, weather, conditions, cooldown, and repeat policy.
4. Prefer pillar and objective variety across the journey.
5. Select remaining cards through deterministic weighted draws without replacement.
6. Store the hidden event queue in the save.
7. Reveal only the current event.

If no event is eligible, use an authored quiet-travel beat. Quiet travel can still include companion conversation, scenery, or a rumor.

#### Connection to the main story

Random does not mean irrelevant. Every event may export one `StoryHook`:

```text
Rumor
FactionLead
NpcContact
LocationClue
ThreatEvidence
ResourceOpportunity
CompanionMemory
```

The hook records its event, outcome, time, region, and involved actors. Main-story scenes may test for the hook later. Repeated related hooks can promote into a side quest, change a faction's knowledge, or alter a future route.

An event cannot overwrite main-story state directly unless its content definition names the exact effect. This prevents a random draw from breaking the campaign.

### Map

The terminal map shows known nodes and routes. Undiscovered nodes do not appear as question marks unless the player has learned that something exists there.

## School-life systems

### Schedule

Classes and activities occupy time blocks. The player can attend, arrive late, skip, or sometimes send an excuse.

Attendance influences:

- Skill training.
- Teacher relationships.
- House standing.
- Detention.
- Available classmates.
- Quest timing.

### Classes

A class is a short authored activity selected from a curriculum deck. It can be:

- Instruction plus check.
- Puzzle.
- Controlled duel.
- Creature care.
- Potion sequence.
- Group project.
- Oral exam.
- Field trip.

Repeated classes use different prompts, complications, and social contexts.

### Exams

Exams summarize learned skills and choices. They do not rely on one random roll. Preparation, attendance, discovered notes, relationships, equipment, and the final check all contribute.

### House or cohort competition

Points are an event ledger. Each entry stores amount, reason, source, time, and responsible actor.

The dashboard shows totals, recent changes, and the next reward threshold. Staff favoritism, cheating, and public disputes can become quests.

### Clubs

Clubs provide:

- Recurring schedules.
- Skill training.
- Relationships.
- Competitions.
- Vendors or equipment.
- Club quest lines.

The player cannot fully advance every club in one playthrough.

### Companion goals and downtime

Companions have multi-stage personal goals that progress from explicit event tags, downtime, quests, and player-help scenes. A goal pauses when it needs the player's decision; it cannot finish a major personal arc invisibly in the background.

When outside the active party, a companion may study, train, work, gather, investigate, socialize, rest, care for a familiar, or pursue a goal. Activities occupy real calendar blocks and respect classes, travel, injuries, faction access, and prior commitments.

Companions may return with progress, resources, stress, a rumor, or a request for help. Recall takes travel time and may interrupt an activity. Arc transitions preserve or deliberately resolve every active companion goal.

## Puzzles

Puzzle families include:

- Rune translation.
- Spell sequencing.
- Potion logic.
- Moving portraits.
- Hidden passages.
- Environmental manipulation.
- Social deduction.
- Scheduling.
- Artifact mechanisms.

Puzzles support:

- Keyboard-only interaction.
- A readable text description.
- Optional hint levels.
- Character-skill assistance.
- More than one valid solution when the fiction supports it.
- A bypass with a cost for players who do not enjoy that puzzle type.

Puzzle state must serialize cleanly. Reloading cannot regenerate a different answer unless the puzzle is explicitly seed-based.

## Inventory and equipment

### Categories

- Wand
- Robe
- Accessory
- Potion
- Tool
- Ingredient
- Artifact
- Book
- Key
- Quest item
- Gift
- Consumable

### Inventory rules

The default game uses slots and bulky tags instead of weight down to the gram. Quest items use a separate unlimited section and cannot be sold or dropped accidentally.

Items can provide:

- Passive modifiers.
- New commands.
- Choice visibility.
- Combat actions.
- Charges.
- Set effects.
- Reputation reactions.
- Quest hooks.

### Equipment

Default slots:

- Wand.
- Robe.
- Accessory one.
- Accessory two.
- Utility item.
- Familiar gear.

Equipment changes emit events and immediately recalculate derived statistics.

## Economy, shops, and loot

Shops have:

- Inventory source.
- Restock schedule.
- Price policy.
- Faction or relationship modifiers.
- Buy restrictions.
- Sell categories.
- Unique stock.
- Rumors.

Prices are integer minor units in engine state. The UI converts them into campaign currency names.

Loot is authored or selected from weighted tables. Random loot records its content IDs and rolls in the event log so save and replay behavior stays deterministic.

Stealing is a choice with observation, evidence, ownership, and consequences. It is not a free inventory command.

## Crafting and potions

A recipe contains:

- Ingredients.
- Required tool.
- Required knowledge.
- Time.
- Check.
- Standard result.
- Flawed result.
- Exceptional result.
- Safety risk.

The crafting preview shows known requirements and keeps undiscovered details hidden. The player can substitute an ingredient only when a recipe defines the substitution.

Potion quality affects duration, strength, side effects, stability, or number of doses. It does not silently change the item's identity after saving.

## Spell system

A spell contains:

- ID and name.
- School or discipline.
- Tags.
- Spell level from 0 through 9, where level 0 means cantrip.
- Character-level requirement.
- Action type.
- Verbal, somatic, focus, and consumed-material requirements.
- Range band.
- Target rule.
- Attack or saving throw.
- Damage or healing expression.
- Duration.
- Concentration.
- Ritual eligibility.
- Conditions.
- Effects.
- Upcast rules.
- Cantrip scaling rules.
- Upgrade paths.
- Exploration uses.
- Counter tags.
- ASCII animation ID.
- Description and concise combat text.

### Cantrips

Cantrips are level-0 spells. They never consume spell slots and remain available whenever the character can satisfy their casting requirements.

- Combat cantrips use an action unless a feature changes their casting time.
- Utility cantrips work outside combat without spending a limited resource.
- Damage and healing cantrips may scale at character levels 5, 11, and 17.
- Scaling belongs to the spell definition, so campaigns can use another progression.
- A silenced, restrained, disarmed, or unfocused caster still has to meet the cantrip's component rules.

### Spell slots

Leveled spells consume spell slots. A character can cast a prepared spell with a slot equal to or higher than the spell's level.

The character sheet tracks remaining and maximum slots separately for levels 1 through 9:

```text
1st  3/4
2nd  2/2
3rd  0/0
4th  0/0
5th  0/0
6th  0/0
7th  0/0
8th  0/0
9th  0/0
```

The rules profile supplies a slot-progression table. Character level, magic path, traits, equipment, and temporary effects choose or modify that table. Content does not calculate slot totals with free-form scripts.

Casting validation checks:

1. The spell is a cantrip or is prepared.
2. The chosen slot is not lower than the spell level.
3. A remaining normal or temporary slot of that level exists.
4. Components, targets, range, action economy, and concentration are legal.
5. The spell does not violate the active rules profile's per-turn casting limit.

A rejected cast spends nothing.

### Upcasting

The cast command includes the chosen slot level. When that level is higher than the spell's base level, the engine applies the spell's authored upcast rule.

Upcast rules may add:

- Damage or healing dice per level above the base.
- Additional targets at named thresholds.
- Longer duration.
- Greater range.
- Stronger dispel or counter rank.
- A condition improvement specifically allowed by the spell.

The selection panel previews the exact difference before confirmation:

```text
Binding Chalk
Base level: 1st
Cast with: 2nd-level slot
Upcast: range increases from Near to Medium
Slots after cast: 1/2 second-level slots
```

A spell with no upcast benefit may still use a higher slot when the rules profile allows it, but the UI warns that the effect will not improve.

### Slot recovery

- A long rest restores normal spell slots.
- Temporary slots disappear at the end of a long rest before normal slots refresh.
- A short rest does not restore slots unless a named character feature says it does.
- Rare items or authored location effects may restore a slot, but ordinary healing potions do not.
- Cantrips remain usable when every slot is empty.
- Mundane actions, movement, items, and interactions never require a spell slot.

### Prepared and known spells

Known spells are learned permanently. Prepared spells are the leveled subset available for normal casting. Known cantrips are always available and do not count against prepared-spell capacity.

A known leveled spell with the ritual tag may be cast unprepared as a ritual when a character feature permits it. Ritual casting takes additional in-game time, cannot be used during combat, and does not consume a slot. Other leveled exploration spells follow the normal preparation and slot rules.

### Sorcery points

Sorcery points are an optional character resource granted by a magic path, trait, or campaign feature. They are not universal.

The state stores current points and the maximum allowed by the character's sorcery progression. Current points never exceed that maximum. A long rest restores all spent points.

The default progression uses the character's sorcery-feature level:

| Feature level | Maximum points | Feature level | Maximum points |
| ---: | ---: | ---: | ---: |
| 1 | 0 | 11 | 11 |
| 2 | 2 | 12 | 12 |
| 3 | 3 | 13 | 13 |
| 4 | 4 | 14 | 14 |
| 5 | 5 | 15 | 15 |
| 6 | 6 | 16 | 16 |
| 7 | 7 | 17 | 17 |
| 8 | 8 | 18 | 18 |
| 9 | 9 | 19 | 19 |
| 10 | 10 | 20 | 20 |

Characters without the feature have no sorcery-point pool. Multiclass or alternate progression packs must provide the feature level explicitly rather than assuming it equals total character level.

#### Flexible casting

As a bonus action, a character can create one temporary spell slot by spending sorcery points:

| Created slot | Sorcery-point cost |
| --- | ---: |
| 1st | 2 |
| 2nd | 3 |
| 3rd | 5 |
| 4th | 6 |
| 5th | 7 |

Flexible casting cannot create slots of levels 6 through 9. Created slots are tracked separately and disappear at the end of a long rest.

As a bonus action, a character can expend one remaining spell slot to regain sorcery points equal to that slot's level. The engine rejects the conversion when it would exceed the character's point maximum, and the UI previews both sides of the exchange.

Conversion commands are atomic. A rejected conversion spends neither the slot nor the bonus action.

#### Metamagic

A cast can normally use one metamagic option. Empowered Spell is the exception and may accompany one other compatible option. Each choice passes through target, component, action, and resource validation before the slot or points are spent.

| Option | Cost | Engine effect |
| --- | ---: | --- |
| Careful Spell | 1 | Choose up to the casting ability modifier in creatures, with a minimum of one. They automatically succeed on the spell's saving throw. |
| Distant Spell | 1 | A ranged spell extends by one supported distance band, up to Far. A touch spell reaches Near. |
| Empowered Spell | 1 | Reroll up to the casting ability modifier in damage dice, with a minimum of one, and keep the new results. It may combine with one other option. |
| Extended Spell | 1 | Double a duration of at least one minute, up to 24 hours. |
| Heightened Spell | 3 | One target has disadvantage on its first saving throw against the spell. |
| Quickened Spell | 2 | A spell with a casting time of one action uses the bonus action for this cast. |
| Subtle Spell | 1 | Ignore verbal and somatic components for this cast. Consumed materials are still required. |
| Twinned Spell | Spell level, minimum 1 | A non-self spell that targets exactly one creature also targets one additional legal creature. A cantrip costs 1 point. |

The default rules allow only one leveled spell during a turn. A quickened leveled spell leaves the action available for a cantrip, movement-independent actions, items, or interaction. Campaign rules can replace this limit, but the active rule must be visible in the cast preview.

Metamagic availability is data-driven. Characters learn a limited set of options rather than receiving every option automatically.

### Upgrades

Spell upgrades choose between behavior changes rather than only larger numbers:

- A new authored upcast benefit.
- Longer range.
- New target shape.
- Safer friendly fire.
- Stronger condition.
- Exploration interaction.

Choices are visible before confirmation and may be retrained through an authored service.

## Tactical combat

### Battlefield model

Combat uses distance zones:

- Engaged.
- Near.
- Medium.
- Far.

Moving normally changes one band. Terrain, spells, and abilities can change that cost.

The battlefield also tracks:

- Cover.
- Hazards.
- Interactable objects.
- Exits.
- Objective zones.
- Visibility.

### Turn structure

Each turn provides:

- One action.
- One bonus action.
- One movement.
- One reaction until the actor's next turn.
- Free inspection and log access.

Actions include:

- Cast.
- Defend.
- Counter-ready.
- Use item.
- Help ally.
- Interact.
- Escape.
- Special objective action.

### Initiative

Initiative is rolled once when combat starts. Ties use initiative bonus, then Dexterity, then stable actor ID. This makes ordering deterministic.

### Attacks and saves

Spell attacks:

```text
d20 + casting ability modifier + proficiency
```

Saving throw spells define ability and target. The combat log shows the roll unless the campaign marks that information hidden.

### Damage

Damage has:

- Dice expression.
- Type.
- Source.
- Resistance, immunity, or vulnerability.
- Critical behavior.

The reducer calculates damage. Content does not write directly to HP.

### Downed and defeat

At zero HP, the default player becomes downed. Allies can stabilize them. If the whole party is downed, the encounter resolves through a campaign-defined defeat scene.

Possible consequences:

- Injury.
- Lost time.
- Lost non-quest item.
- Captivity.
- Reputation loss.
- Changed quest route.

Permanent death is an optional campaign rule and is off for the school-era default.

### Concentration

An actor maintains one concentration effect. Taking damage can trigger a Constitution check. Starting another concentration spell ends the first and emits an explicit event.

### Counterplay

Counters use tags and reactions. A defensive spell may counter `projectile` or `direct_magic` but not an environmental collapse. The UI previews known counter compatibility.

### Encounter goals

Combat can end through:

- Defeat all hostile actors.
- Survive a number of rounds.
- Reach an exit.
- Protect an actor or object.
- Disable a ritual.
- Convince the enemy.
- Retrieve an item.
- Force surrender.

### Version 1.3 tactical expansion

The deeper-tactics release adds one dimension at a time:

1. Bounded reaction windows and counterspells.
2. Named terrain zones with explicit cover and line-of-sight links.
3. Environmental hazards and interactable terrain.
4. Nonlethal objectives, morale, and surrender.
5. Companion commands and capped summons.
6. Stealth openings with undetected, partial, and detected outcomes.
7. Multi-phase bosses with explicit transition events.
8. AI goals for damage, survival, protection, zones, objectives, escape, and resource conservation.

Only one reaction window can be open in version 1.3. Reactions cannot recursively open more reaction prompts.

A development-only combat simulator runs the real command and event engine across deterministic seed ranges. Reports include win and defeat counts, stalled combats, rounds, remaining HP, spell slots, sorcery points, reactions, objectives, and terminal reasons. Simulation measures balance; manual play still decides whether an encounter is clear and fun.

## Enemy AI

AI is a scored choice over legal actions. It cannot access information the actor should not know.

Personality packages include:

- Aggressive: prioritizes damage and closing distance.
- Defensive: values cover, counters, and survival.
- Cowardly: retreats or surrenders when pressure rises.
- Strategic: targets concentration, objectives, and vulnerable roles.
- Chaotic: uses a weighted set of risky actions.
- Protective: guards a leader, ally, or object.

Bosses use phases triggered by visible state:

- HP threshold.
- Objective change.
- Round.
- Destroyed object.
- Dialogue condition.
- Ally defeat.

Phase changes are events and may alter art, actions, terrain, music, or victory conditions.

## Companion combat

Default companion control modes:

- Direct: player selects every action.
- Suggested: AI proposes, player confirms or changes.
- Autonomous: AI acts from selected tactics.

Tactics include:

- Protect player.
- Focus target.
- Conserve high-level spell slots.
- Reserve sorcery points.
- Use support.
- Avoid lethal force.
- Pursue objective.

The player can inspect why the AI chose an action. Companion mistakes should come from limited information, stress, traits, or explicit AI weights, not hidden cheating.

## Status effects

A status defines:

- Duration policy.
- Stack policy.
- Start and end timing.
- Modifiers.
- Granted or blocked commands.
- Periodic effects.
- Removal methods.
- UI icon or glyph.

Core conditions include:

- Burning.
- Charmed.
- Confused.
- Frightened.
- Hidden.
- Poisoned.
- Prone.
- Restrained.
- Silenced.
- Slowed.
- Stunned.
- Vulnerable.

Every duration specifies whether it decrements at turn start, turn end, round end, travel, or rest.

## Rest, injury, and recovery

Short rest:

- Consumes a smaller time block.
- Restores only abilities that explicitly refresh on a short rest.
- Allows item use and companion conversation.
- May be interrupted in unsafe locations.

Long rest:

- Advances to the next valid schedule boundary.
- Restores HP, normal spell slots, sorcery points, and long-rest abilities.
- Removes temporary slots created through flexible casting.
- Processes injuries and timed quests.
- Triggers messages, dreams, familiar care, and world events.

Rest preview lists deadlines and scheduled events that will be crossed.

## World pressure and phases

Long campaigns need the world to change without simulating every citizen.

The engine tracks named meters such as:

- Institutional stability.
- Civil tension.
- Planar instability.
- Public fear.
- Faction resources.

Campaign data chooses names and meanings. Threshold rules change:

- Encounter decks.
- Prices.
- Travel safety.
- NPC schedules.
- Available dialogue.
- Location descriptions.
- Quest starts.
- Ending eligibility.

World phases are discrete. Meters can trigger a proposed phase transition, but the campaign confirms it through a scene or quest event.

## Endings and epilogues

An ending has:

- Eligibility condition.
- Priority.
- Confirmation scene.
- World outcome.
- Companion epilogues.
- Faction epilogues.
- Location epilogues.
- Unresolved-promise notes.
- Achievement.
- New Game Plus unlocks.

The final choice matters, but it does not erase prior state. Ending evaluation combines the final command with the full campaign record.

Ending previews can warn about irreversible action without listing undiscovered outcomes.

## Save system

### Slots

- Three manual slots by default.
- One rotating autosave set.
- One quick-save slot when the current mode allows it.
- One ironman slot when enabled.

### Save envelope

```json
{
  "schema_version": 1,
  "engine_version": "0.1.0",
  "campaign_id": "academy-demo",
  "campaign_version": "0.1.0",
  "content_fingerprint": "sha256-value",
  "saved_at": "2030-01-01T12:00:00Z",
  "slot": 1,
  "rng": {
    "seed": 441993,
    "stream": 0,
    "word_position": 128
  },
  "state": {
    "arc": "reconstruction",
    "level": 1,
    "location": "academy.arrival_hall"
  }
}
```

The real state object contains the full serializable game state. The example shows the envelope fields that remain stable.

Spellcasting save state includes:

- Known cantrips and leveled spells.
- Prepared leveled spells.
- Remaining and maximum normal slots for each level.
- Temporary slots by level.
- Optional sorcery feature level, current points, maximum points, and learned metamagic.
- Active concentration.
- Bonus-action and leveled-spell markers when saving during combat.

Loading validates each count against the active rules profile. It does not refill slots or recalculate spent resources from character level.

### Safety

- Write to a temporary file in the same directory.
- Flush and close it.
- Move the old save to a backup.
- Atomically replace the slot when the platform supports it.
- Reopen and parse the new file.
- Restore the backup if verification fails.

### Migration

Migrations operate one schema version at a time. They are pure transformations with fixture tests. A migration never loads campaign content to invent missing state.

### Ironman

Ironman removes manual reload as a strategy, not backups as a safety measure. The game still keeps recovery data for crashes and filesystem failures.

## Terminal interface

### Standard dashboard

```text
┌──────────────────────────────────────────────────────────────────────────────┐
│ STORYFORGE  Arrival Hall                         Day 1  08:40  [Help: ?]    │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                          RESPONSIVE ASCII ART                                │
│                                                                              │
├──────────────────────────────────────────────────────────────────────────────┤
│ Rain ticks against the glass roof. Iren is waiting beside the sealed gate.  │
│                                                                              │
│ "If we miss the procession, they will remember us for the wrong reason."    │
├──────────────────────────────────────┬───────────────────────────────────────┤
│ ACTIONS                              │ CHARACTER                             │
│ > 1. Ask about the sealed gate       │ HP   ██████████ 12/12                │
│   2. Look for another route          │ SLOTS 1st 3/4   2nd 2/2              │
│   3. Follow Iren                     │ SP 3/3   LVL 3   STATUS Healthy       │
├──────────────────────────────────────┼───────────────────────────────────────┤
│ QUEST: Reach the induction hall      │ LOG                                   │
│                                      │ Iren noticed your hesitation.         │
└──────────────────────────────────────┴───────────────────────────────────────┘
```

### Compact mode

At 80x24:

- Story and choices stay visible.
- Character, quest, map, and log move behind tabs.
- Art uses a short variant or hides.
- The footer always shows controls.

### Focus and input

Global keys:

| Key | Action |
| --- | --- |
| `?` | Help overlay |
| `c` | Character sheet |
| `j` | Journal |
| `m` | Map |
| `l` | Full log |
| `s` | Save menu |
| `Esc` | Back or close overlay |
| `Ctrl+C` | Confirm quit, then restore terminal |

Menu keys:

- Arrow keys or `j` and `k` move selection.
- Enter confirms.
- Number keys choose visible numbered actions.
- Page Up and Page Down scroll.
- Tab and Shift+Tab move focus.

Mouse input may be added later. Nothing requires it.

### Terminal capability fallbacks

- True color when detected.
- 256-color theme.
- 16-color theme.
- No-color theme.
- Unicode borders.
- ASCII borders.
- Animation.
- Reduced motion.
- No animation.

Player settings override automatic detection.

## ASCII art and animation

Art assets are text files with metadata:

- ID.
- Width and height.
- Size category.
- Color regions or style spans.
- Frame duration for animation.
- Fallback ID.
- Attribution and source status.

Key scenes use hand-edited art. Image conversion can create a rough first pass, but the result must be cleaned at each target size.

Animations stay short:

- Spell flash.
- Wand trail.
- Door opening.
- Damage impact.
- Weather motion.
- Boss phase transition.

Animations cannot delay input for more than a short configurable duration. Pressing a key skips them.

## Audio

Audio is optional and disabled by default in version 1 planning.

If added:

- It is feature-gated.
- The game works fully without an audio device.
- Volume is configurable by category.
- Errors disable audio instead of crashing.
- Public releases include only licensed or original files.

## Accessibility

Required settings:

- ASCII-only mode.
- No-color mode.
- Color-blind palettes.
- Reduced motion.
- Instant text.
- Adjustable log length.
- Persistent control hints.
- Larger minimum prose panel through compact layout.
- Puzzle hints.
- Combat confirmation prompts.
- Screen-reader transcript mode outside the alternate screen.
- Content filters for campaign-defined sensitive topics.

Every important state uses text or shape in addition to color.

## Data-driven content

### Pack layout

```text
campaigns/academy-demo/
├── campaign.toml
├── art/
├── actors/
├── encounters/
├── endings/
├── factions/
├── items/
├── locations/
├── quests/
├── scenes/
├── spells/
└── tests/
```

### Manifest

```toml
id = "academy-demo"
name = "The Ashen Academy"
version = "0.1.0"
schema-version = 1
rules-profile = "storyforge-default"
default-locale = "en"
entry-scene = "academy.arrival"
starting-arc = "reconstruction"
```

### Stable IDs

IDs use a namespace:

```text
academy.actor.iren
academy.location.arrival_hall
academy.scene.sealed_gate
academy.spell.ember_thread
```

The display name can change. Once an ID appears in a released save, it is never reused for a different concept.

### Validation

The validator reports:

- File and line when available.
- Duplicate IDs.
- Missing references.
- Invalid dice expressions.
- Invalid ranges.
- Impossible conditions.
- Unreachable scenes.
- Choices with no outcome.
- Quest objectives with no completion route.
- Missing art fallback.
- Unsupported schema version.
- Ending priority conflicts.

Warnings do not block development runs unless strict mode is enabled. Errors always block play.

## Modding

Mods are campaign packs or dependency packs. Version 1 does not load arbitrary native libraries.

A pack declares dependencies and compatible schema versions. The loader resolves them in order, rejects cycles, and shows conflicts before play.

Override rules:

- A pack may extend registries.
- Replacement requires an explicit `replaces` declaration.
- Core demo IDs cannot be silently changed.
- Save files record every loaded pack and version.

## Private campaign workflow

The private pack lives outside the public release tree:

```text
projects/
├── storyforge-tui/             public engine and original demo
└── wizarding-world-private/    separate private Git repository
```

The public repository uses an ignored `.storyforge.local.toml` to point at the sibling pack during development. Private story files and their Git history never enter the public repository. A clean public checkout and release audit build the npm artifact. Deleting files or rewriting Git history immediately before publication is an emergency-cleanup technique, not the normal privacy model.

The guide will convert material deliberately:

1. Add a source entry to the provenance ledger.
2. Decide whether it is user-owned, licensed, or private-reference-only.
3. Create an original or private RON record.
4. Validate IDs and references.
5. Add a focused content test.
6. Play the smallest scene that uses it.

PDF ingestion is not a runtime feature. Sourcebooks are read by the author and converted through human judgment. The game does not scrape or redistribute them.

## Rust architecture

```text
Input event
    |
    v
TUI maps input to GameCommand
    |
    v
storyforge-core validates and reduces command
    |
    +----> GameEvent list ----> log, animation, autosave policy
    |
    v
Updated GameState
    |
    v
TUI renders state and current content
```

### `storyforge-core`

Owns:

- IDs.
- Game state.
- Commands and events.
- Dice and deterministic RNG.
- Character math.
- Relationships.
- Quests.
- Time.
- Exploration.
- Combat.
- Arc and ending rules.

Does not know about:

- Terminal cells.
- Keyboard keys.
- Paths.
- JSON files.
- RON files.
- Logging subscribers.

### `storyforge-content`

Owns:

- Serde content structures.
- RON loading.
- TOML manifests.
- Registries.
- Dependency resolution.
- Content fingerprints.
- Validation.
- Conversion into core definitions.

### `storyforge-tui`

Owns:

- CLI.
- Terminal lifecycle.
- Input.
- Rendering.
- Focus.
- Animation scheduling.
- Save paths.
- Save files and migration orchestration.
- Logging.
- Pack selection.

### Why no ECS yet

Most turns involve a few actors in one active scene. Plain structs, maps, and reducers are easier to learn, serialize, test, and debug.

Evaluate Bevy ECS only when all are true:

- The game has thousands of active simulated entities.
- Systems need independent scheduling over large groups.
- Plain updates are measured as a bottleneck.
- Save conversion has a documented plan.
- A prototype proves the improvement.

## Error handling

Library crates return typed errors with `thiserror`.

The binary uses `color-eyre` to add context and format top-level reports.

Recoverable errors become player screens:

- Invalid campaign.
- Missing save.
- Incompatible save.
- Corrupt save with backup available.
- Terminal too small.
- Read-only save directory.

Programming errors may stop the application, but the terminal must restore first and the crash report must point to the log.

Production code does not use `unwrap` or `expect`.

## Logging and diagnostics

Use `tracing` with a rolling log file outside the terminal output.

Log:

- Engine and campaign versions.
- Pack load and validation summary.
- Save and migration operations.
- Command type and resulting event types.
- Terminal size changes.
- Recoverable errors.
- Performance spans over slow operations.

Do not log:

- Full save content by default.
- Player-entered names in analytics.
- Secrets or tokens.
- Every render frame.

`storyforge doctor` reports terminal capability, user directories, pack status, and save write access.

## Testing strategy

### Unit tests

- Ability modifiers.
- Dice parsing.
- Advantage.
- Derived statistics.
- Command legality.
- Relationship bounds.
- Quest transitions.
- Time crossing schedules.
- Damage and resistance.
- Status durations.
- Arc transitions.
- Ending priority.

### Property tests

- HP never exceeds maximum after clamping.
- Remaining normal slots never exceed their slot-level maximum.
- Cantrips never consume spell slots.
- Temporary slots disappear at long rest.
- Sorcery points remain between zero and their character maximum.
- A rejected flexible-casting command changes no resource.
- Relationship and reputation remain in bounds.
- Invalid dice expressions never panic.
- A combat turn cannot spend the same action twice.
- Saving and loading preserves equivalent state.

### Content tests

- Every pack validates.
- Every scene has a reachable exit unless marked terminal.
- Every main quest can reach a terminal state in its authored test route.
- Every ending fixture selects the expected ending.

### Snapshot tests

- Compact dashboard.
- Standard dashboard.
- Character creation.
- Skill roll reveal.
- Combat menu.
- No-color mode.
- ASCII-border mode.

Snapshots remain small and named. Numeric combat rules use direct assertions instead.

### Integration tests

- New game to confirmed character.
- Branching scene to check.
- Duel to reward.
- Save to process exit.
- Reload to continued scene.
- Old save migration.
- Missing private pack message.
- Bundled demo launch.

## Performance

Performance work follows measurements.

Targets:

- Input response under 50 ms for normal commands.
- Render update under 16 ms at recommended size, without requiring 60 frames per second.
- Campaign validation under 2 seconds for the version 1 demo on a normal laptop.
- Save under 250 ms for ordinary state.
- Idle CPU close to zero when no animation or timed input is active.

Use release builds for benchmarks. Do not introduce shared threads, locks, caches, arenas, or ECS because they look fast.

## Release and npm

`dist` builds native binaries and generates an npm installer. The npm package downloads the correct release artifact. It does not run the Rust application through JavaScript.

Initial targets:

- `x86_64-pc-windows-msvc`
- `x86_64-unknown-linux-gnu`
- `x86_64-apple-darwin`
- `aarch64-apple-darwin`

Release checks:

- Clean Git tag.
- Locked dependencies.
- All tests pass.
- Demo pack validates.
- Private directories absent from archives.
- `--version` works.
- `doctor` works.
- Launch and quit restore the terminal.
- npm tarball contents are inspected.
- Each native archive has a checksum.

## Licensing boundary

The engine and original demo need their own clear license.

If SRD text is copied, include the required CC BY attribution and track the exact source. Rules inspired by familiar mechanics should still use original wording whenever practical.

The flexible-casting costs and named metamagic options in this design follow the older 5e-style rules requested for the campaign. If the public demo ships those names or values, record the exact SRD edition in `THIRD_PARTY_LICENSES.md` and include its CC BY 4.0 attribution. Keep the rules-profile version in campaign data so a later SRD revision cannot silently change existing saves.

Do not publish:

- Canon setting names or artwork without permission.
- Third-party sourcebook text.
- PDFs from this vault.
- Maps or portraits without redistribution rights.
- Private player information.

A disclaimer does not replace permission.

## Open design risks

These risks are known and have planned controls:

| Risk | Control |
| --- | --- |
| Too much scope | Milestone gates and a complete small MVP. |
| Content breaks saves | Stable IDs, schema versions, migrations, and fingerprints. |
| TUI becomes unreadable | Responsive layouts and snapshot tests at fixed sizes. |
| Random results make tests flaky | Deterministic seeded RNG. |
| Branching story becomes impossible to reason about | Typed conditions/effects, validator, history log, and route fixtures. |
| Combat dominates development | Encounter goals and noncombat routes in the vertical slice. |
| Private material leaks | Separate ignored directory and release archive inspection. |
| Beginner architecture becomes too abstract | Three crates only, plain types, no ECS, no general scripting. |
| Later arcs require a rewrite | Arc, world-phase, faction, planar-pack, and ending concepts exist before MVP saves ship. |

## Product roadmap

The [Storyforge product roadmap](ROADMAP.md) turns this design into a dependency
order from terminal boot through the public npm release, living-world updates,
and deeper tactics. It also records fifty later story-first systems, including
trained familiars, delayed mail, evidence boards, secrecy and corruption,
meaningful food, deeper crafting and loot, feats, long-form transformation,
cursor-driven ASCII combat maps, travel networks, aerial school sport, public
news, social commitments, heist planning, rituals, and return-to-game
briefings.

Those systems are candidates, not version-one requirements. Each must first
prove one small story use through the existing command, event, save, content,
and TUI path.

## Final design test

Before adding a feature, answer:

1. Which player decision does it create?
2. Which state does it read?
3. Which event does it emit?
4. How is it saved?
5. How is it shown at 80x24?
6. How is it validated?
7. Which test proves it works?

If those answers are unclear, the feature is not ready to implement.
