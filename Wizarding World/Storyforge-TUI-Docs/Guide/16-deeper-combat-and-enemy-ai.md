# 16: Deepen combat and enemy AI

## Result

Combat will support companions, reactions, cover, hazards, objectives, surrender, boss phases, and reusable AI personalities without giving enemies hidden player knowledge.

## Battlefield

Add:

```rust
pub struct BattlefieldState {
    pub distance: HashMap<ActorPair, DistanceBand>,
    pub cover: HashMap<ActorId, Cover>,
    pub hazards: Vec<HazardInstance>,
    pub interactables: Vec<InteractableInstance>,
    pub objectives: Vec<CombatObjectiveState>,
    pub exits: Vec<ExitState>,
}

pub enum Cover {
    None,
    Partial,
    Full,
}
```

For a small party, pairwise distance remains manageable. If encounter size later makes it cumbersome, profile and redesign with explicit zones. Do not change representation solely for elegance.

## Reactions

A reaction definition declares:

- Trigger.
- Condition.
- Cost.
- Effect.
- Prompt policy.
- AI score.

Triggers:

- Targeted by spell.
- Hit by attack.
- Actor moves away from Engaged.
- Ally takes damage.
- Enemy begins concentration.
- Hazard activates.

Direct-control player reactions open a timed-off prompt only after the triggering event is shown. There is no real-time countdown.

## Counter-ready action

The actor spends their action to prepare a counter tag and keeps their reaction. When a compatible spell begins, they may counter.

Record:

- Prepared counter.
- Triggering spell.
- Reaction decision.
- Counter roll.
- Result.
- Resources spent.

A failed counter does not erase the original cast event; it adds resolution events before spell effects.

## Cover and line of effect

Partial cover grants a defense or save bonus defined by the rules profile. Full cover blocks direct targeting but not every area or environmental effect.

The UI explains:

```text
Target has partial cover: +2 Defense.
```

Movement and interactable commands can change cover.

## Hazards

Hazards have:

- Trigger timing.
- Visible warning.
- Affected bands or actors.
- Check or save.
- Effect.
- Duration.
- Disable interaction.

Examples for the original demo:

- Falling glass.
- Unstable rune line.
- Spreading ember patch.
- Rotating ward beam.

Hazards act at a stable initiative entry.

## Objectives

Supported objective kinds:

- Defeat targets.
- Survive rounds.
- Reach exit.
- Protect actor.
- Protect object.
- Disable interactables.
- Retrieve item.
- Force surrender.
- Complete ritual steps.

An encounter can have required and optional objectives. Victory evaluation runs after every relevant event.

## Surrender and morale

Actors can have morale:

```rust
pub struct MoraleState {
    pub current: i16,
    pub maximum: i16,
    pub surrender_threshold: i16,
    pub flee_threshold: i16,
}
```

Damage, ally defeat, intimidation, leader state, and completed objectives can change morale. Mindless constructs omit morale.

The AI may surrender, flee, negotiate, or become desperate based on its personality and authored boundaries.

## AI scoring

Each legal action receives components:

```rust
pub struct ActionScore {
    pub action: GameCommand,
    pub objective: i32,
    pub survival: i32,
    pub damage: i32,
    pub support: i32,
    pub personality: i32,
    pub resource_cost: i32,
}
```

Sum with saturating arithmetic. Log components at debug level.

AI receives:

- Public battlefield state.
- Its own state.
- Observed player actions.
- Authored knowledge.
- Current objective.

AI does not receive hidden inventory, uncast spells, secret quest flags, or future RNG.

## Personality packages

Create reusable weights:

- Aggressive.
- Defensive.
- Cowardly.
- Strategic.
- Chaotic.
- Protective.

An actor can override individual weights. Use content validation to keep values in a documented range.

## Boss phases

```rust
pub struct BossPhaseDefinition {
    pub id: String,
    pub enter_when: Vec<ConditionDefinition>,
    pub once: bool,
    pub entry_effects: Vec<EffectDefinition>,
    pub add_actions: Vec<ContentId>,
    pub remove_actions: Vec<ContentId>,
    pub art: Option<ContentId>,
}
```

Evaluate phases after an event batch. Enter at most one phase at a time unless the definition explicitly allows a chain.

Phase entry emits a named event before new actions become available.

## Boss example

Create an Arc I training boss:

- Phase one uses direct rune attacks.
- At 60 percent HP, activates rotating wards and partial cover.
- At 25 percent morale, offers surrender terms.
- Destroying two ward anchors opens a non-damage victory.

This encounter tests objectives and choices without requiring endgame numbers.

## Balance fixtures

Create deterministic simulations for:

- Level 1 basic duel.
- Level 5 party versus three enemies.
- Level 10 boss.
- Party with only cantrips and one 1st-level slot remaining.
- Sorcery-enabled caster with several flexible-casting choices.
- Support companion.
- Escape objective.

Record:

- Rounds.
- Damage taken.
- Spell slots spent by level.
- Temporary slots created and converted.
- Sorcery points spent by metamagic and flexible casting.
- Downed actors.
- Outcome.

Use broad assertions, not exact full transcripts, for balance:

```text
victory rate across seeds 1..100 should be between 55% and 80%
median rounds should be between 3 and 8
```

Keep exact tests for rule correctness.

## Validator additions

- Reaction triggers supported.
- Counter tags resolve.
- Hazards have warning and timing.
- Objectives reference actors or objects that spawn.
- Boss phase IDs unique.
- At least one victory route exists.
- AI weight range valid.
- Morale thresholds ordered.

## Acceptance check

- A companion acts in all three control modes.
- Cover affects a visible roll.
- One reaction changes an outcome.
- One hazard can be disabled.
- One encounter supports damage and objective victories.
- One enemy surrenders through morale.
- Boss phase transition is saved and resumes once.
- AI debug output explains action selection.

## Suggested commit

```powershell
git add .
git commit -m "Deepen tactical combat and enemy AI"
```
