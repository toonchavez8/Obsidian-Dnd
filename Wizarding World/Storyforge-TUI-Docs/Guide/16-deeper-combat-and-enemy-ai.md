# 16: Deepen combat and enemy AI

## Result

Combat will support companions, reactions, cover, hazards, objectives, surrender, boss phases, and reusable AI personalities without giving enemies hidden player knowledge.

## Build this chapter in six checkpoints

| Checkpoint | Destination | Proof before continuing |
| --- | --- | --- |
| 1 | `storyforge-core/src/combat/battlefield.rs` | Pairwise range, cover, and movement tests |
| 2 | `storyforge-core/src/combat/reaction.rs` | Trigger, decline, counter, and resource tests |
| 3 | `storyforge-core/src/combat/objective.rs` | Hazard and objective transition tests |
| 4 | `storyforge-core/src/combat/ai.rs` | Knowledge boundary, scoring, and deterministic tie tests |
| 5 | `storyforge-core/src/combat/boss.rs` | Phase transition exactly-once tests |
| 6 | `storyforge-tui/src/screens/combat.rs` | Multi-actor, reaction prompt, objective, and surrender playthrough |

Keep the chapter 10 one-versus-one duel passing after every checkpoint.

## Advanced commands and events

Create `storyforge-core/src/combat/advanced_command.rs`:

```rust
use crate::{ActorId, CombatCommand, ContentId, EncounterId};

#[derive(Debug, Clone, PartialEq, Eq)]
pub enum AdvancedCombatCommand {
    UseInteractable {
        encounter: EncounterId,
        actor: ActorId,
        interactable: ContentId,
    },
    ChooseReaction {
        encounter: EncounterId,
        actor: ActorId,
        prompt: u64,
        choice: ReactionChoice,
    },
    PrepareCounter {
        encounter: EncounterId,
        actor: ActorId,
        counter_tag: ContentId,
    },
    Surrender {
        encounter: EncounterId,
        actor: ActorId,
    },
    RespondToSurrender {
        encounter: EncounterId,
        actor: ActorId,
        response: SurrenderResponse,
    },
    DirectCompanion {
        encounter: EncounterId,
        companion: ActorId,
        command: Box<CombatCommand>,
    },
}

#[derive(Debug, Clone, PartialEq, Eq)]
pub enum ReactionChoice {
    Decline,
    UseReaction {
        ability: ContentId,
        target: Option<ActorId>,
    },
}

#[derive(
    Debug, Clone, Copy, PartialEq, Eq, serde::Serialize, serde::Deserialize,
)]
pub enum SurrenderResponse {
    Accept,
    Refuse,
    DemandTerms,
}

#[derive(
    Debug, Clone, Copy, PartialEq, Eq, serde::Serialize, serde::Deserialize,
)]
pub enum CombatObjectiveStatus {
    Hidden,
    Active,
    Completed,
    Failed,
}

#[derive(
    Debug, Clone, PartialEq, Eq, serde::Serialize, serde::Deserialize,
)]
pub enum AdvancedCombatEvent {
    InteractableUsed {
        actor: ActorId,
        interactable: ContentId,
    },
    CoverChanged {
        actor: ActorId,
        previous: Cover,
        current: Cover,
    },
    ReactionPrompted {
        prompt: u64,
        actor: ActorId,
        trigger: ContentId,
        legal_abilities: Vec<ContentId>,
    },
    ReactionDeclined {
        prompt: u64,
        actor: ActorId,
    },
    ReactionUsed {
        prompt: u64,
        actor: ActorId,
        ability: ContentId,
    },
    CounterPrepared {
        actor: ActorId,
        counter_tag: ContentId,
    },
    CounterResolved {
        actor: ActorId,
        triggering_spell: ContentId,
        roll_total: i16,
        difficulty: i16,
        success: bool,
    },
    HazardActivated {
        hazard: ContentId,
        affected: Vec<ActorId>,
    },
    ObjectiveStateChanged {
        objective: ContentId,
        previous: CombatObjectiveStatus,
        current: CombatObjectiveStatus,
    },
    SurrenderOffered {
        actor: ActorId,
    },
    SurrenderResolved {
        actor: ActorId,
        response: SurrenderResponse,
    },
    BossPhaseChanged {
        boss: ActorId,
        previous: ContentId,
        current: ContentId,
    },
    AdvancedCombatCommandRejected {
        reason: String,
    },
}
```

Add `AdvancedCombat(AdvancedCombatCommand),` to the top-level `GameCommand` and `AdvancedCombat(AdvancedCombatEvent),` to `GameEvent`.

| Command | Validation | Successful behavior |
| --- | --- | --- |
| `UseInteractable` | Encounter, turn, range, usage count, and action cost pass | Applies authored battlefield change and spends the declared resource |
| `ChooseReaction` | Prompt ID is active; actor has reaction; ability is in prompt list | Declines for free or spends reaction and resolves ability |
| `PrepareCounter` | Actor owns turn, has action and reaction, and knows counter tag | Spends action and stores prepared counter |
| `Surrender` | Actor may communicate and encounter permits surrender | Opens response scene or AI response |
| `RespondToSurrender` | Offer is active and responder has authority | Accepts, refuses, or opens authored terms |
| `DirectCompanion` | Companion is active, directly controlled, and owns turn | Sends the nested command through the normal combat legality path |

A prompt ID prevents a delayed keypress from accepting a newer reaction. Rejection changes no action, reaction, HP, condition, hazard, or objective state.

## Battlefield

Create `storyforge-core/src/combat/battlefield.rs`:

```rust
pub struct BattlefieldState {
    pub distance: HashMap<ActorPair, DistanceBand>,
    pub cover: HashMap<ActorId, Cover>,
    pub hazards: Vec<HazardInstance>,
    pub interactables: Vec<InteractableInstance>,
    pub objectives: Vec<CombatObjectiveState>,
    pub exits: Vec<ExitState>,
}

#[derive(
    Debug, Clone, Copy, PartialEq, Eq, serde::Serialize, serde::Deserialize,
)]
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
    pub action: CombatCommand,
    pub objective: i32,
    pub survival: i32,
    pub damage: i32,
    pub support: i32,
    pub personality: i32,
    pub resource_cost: i32,
}
```

Implement its total rather than repeating the sum in every personality:

```rust
impl ActionScore {
    #[must_use]
    pub fn total(&self) -> i32 {
        [
            self.objective,
            self.survival,
            self.damage,
            self.support,
            self.personality,
        ]
        .into_iter()
        .fold(0_i32, i32::saturating_add)
        .saturating_sub(self.resource_cost.max(0))
    }
}
```

Log the components and total at debug level. Generate only legal `CombatCommand` candidates, score them, select the highest total, and break equal scores with the encounter RNG. Record the candidate list and winning index so a replay can explain the choice.

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
