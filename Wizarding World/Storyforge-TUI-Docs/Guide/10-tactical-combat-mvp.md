# 10: Build tactical combat

## Result

The player can enter a duel, roll initiative, move between distance zones, cast one of three spells, defend, use an item, observe enemy intent, win, lose, or escape.

## Combat scope

The MVP duel uses:

- One player.
- One enemy.
- Four distance bands.
- One action, one bonus action, one movement, and one reaction per turn.
- HP, spell slots, optional sorcery points, defense, and concentration.
- Three spells.
- Two conditions.
- One potion.
- One nonlethal defeat scene.

Do not add companions or boss phases in this chapter.

## Core combat types

Create `combat.rs`:

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq, serde::Serialize, serde::Deserialize)]
pub enum DistanceBand {
    Engaged,
    Near,
    Medium,
    Far,
}

impl DistanceBand {
    #[must_use]
    pub const fn step_toward(self) -> Self {
        match self {
            Self::Engaged | Self::Near => Self::Engaged,
            Self::Medium => Self::Near,
            Self::Far => Self::Medium,
        }
    }

    #[must_use]
    pub const fn step_away(self) -> Self {
        match self {
            Self::Engaged => Self::Near,
            Self::Near => Self::Medium,
            Self::Medium | Self::Far => Self::Far,
        }
    }
}

#[derive(Debug, Clone, Copy, PartialEq, Eq, serde::Serialize, serde::Deserialize)]
pub struct TurnResources {
    pub action_available: bool,
    pub bonus_action_available: bool,
    pub movement_available: bool,
    pub reaction_available: bool,
    pub leveled_spell_cast: bool,
}

impl Default for TurnResources {
    fn default() -> Self {
        Self {
            action_available: true,
            bonus_action_available: true,
            movement_available: true,
            reaction_available: true,
            leveled_spell_cast: false,
        }
    }
}

#[derive(Debug, Clone, Copy, PartialEq, Eq, serde::Serialize, serde::Deserialize)]
pub enum CombatOutcome {
    InProgress,
    Victory,
    Defeat,
    Escaped,
}
```

## Actors and initiative

Combat actors use runtime IDs separate from content IDs:

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash, serde::Serialize, serde::Deserialize)]
pub struct ActorId(pub u32);
```

An encounter may use the same enemy definition twice, so content ID alone cannot identify a combat participant.

Initiative record:

```rust
pub struct InitiativeEntry {
    pub actor: ActorId,
    pub total: i16,
    pub dexterity_modifier: i8,
}
```

Sort by:

1. Higher initiative total.
2. Higher Dexterity modifier.
3. Lower stable `ActorId`.

Record each initiative roll in a combat-start event.

## Commands

Add these runtime and command types to `combat.rs`:

```rust
use crate::ContentId;

#[derive(
    Debug, Clone, Copy, PartialEq, Eq, Hash, serde::Serialize, serde::Deserialize,
)]
pub struct EncounterId(pub u64);

#[derive(
    Debug, Clone, Copy, PartialEq, Eq, serde::Serialize, serde::Deserialize,
)]
pub enum MoveDirection {
    Toward,
    Away,
}

#[derive(
    Debug, Clone, PartialEq, Eq, serde::Serialize, serde::Deserialize,
)]
pub enum TargetSelection {
    SelfOnly,
    Actor(ActorId),
    Actors(Vec<ActorId>),
}

#[derive(
    Debug, Clone, PartialEq, Eq, serde::Serialize, serde::Deserialize,
)]
pub enum CombatCommand {
    StartEncounter {
        definition: ContentId,
    },
    Move {
        encounter: EncounterId,
        actor: ActorId,
        direction: MoveDirection,
    },
    CastSpell {
        encounter: EncounterId,
        actor: ActorId,
        spell: SpellCommand,
    },
    Defend {
        encounter: EncounterId,
        actor: ActorId,
    },
    UseItem {
        encounter: EncounterId,
        actor: ActorId,
        item: ContentId,
        target: TargetSelection,
    },
    ExamineActor {
        encounter: EncounterId,
        observer: ActorId,
        target: ActorId,
    },
    AttemptEscape {
        encounter: EncounterId,
        actor: ActorId,
    },
    EndTurn {
        encounter: EncounterId,
        actor: ActorId,
    },
}
```

`StartEncounter` creates the runtime ID. Every command after it carries that ID. Reject a stale ID instead of applying an old keypress to a new fight.

Command behavior:

| Command | Validate before emitting events | Successful state change |
| --- | --- | --- |
| `StartEncounter` | Definition exists and no encounter is active | Creates actors, rolls initiative, starts first turn |
| `Move` | Encounter and active actor match; movement remains | Moves exactly one band and spends movement |
| `CastSpell` | Full spell legality checklist below | Spends action or bonus action, slots and points, then resolves effects |
| `Defend` | Actor owns turn and has an action | Spends action and grants defended state |
| `UseItem` | Item exists, target is legal, action remains | Spends item and action, resolves item effect |
| `ExamineActor` | Observer and target exist | Reveals authored facts; costs no action in the default profile |
| `AttemptEscape` | Escape is allowed and action remains | Rolls the authored check; ends encounter only on success |
| `EndTurn` | Actor owns the turn | Applies end-turn effects and advances initiative |

Every rejection emits one `CombatCommandRejected`. A rejected command emits no slot, point, action, movement, item, HP, or condition event.

## Events

Add the event payloads to `combat.rs`:

```rust
#[derive(
    Debug, Clone, Copy, PartialEq, Eq, serde::Serialize, serde::Deserialize,
)]
pub enum SpellSlotSource {
    Normal,
    Temporary,
}

#[derive(
    Debug, Clone, PartialEq, Eq, serde::Serialize, serde::Deserialize,
)]
pub enum CombatEvent {
    EncounterStarted {
        encounter: EncounterId,
        definition: ContentId,
        actors: Vec<ActorId>,
    },
    InitiativeRolled {
        encounter: EncounterId,
        actor: ActorId,
        die: u8,
        modifier: i8,
        total: i16,
    },
    TurnStarted {
        encounter: EncounterId,
        actor: ActorId,
        round: u32,
    },
    ActorMoved {
        encounter: EncounterId,
        actor: ActorId,
        from: DistanceBand,
        to: DistanceBand,
    },
    SpellCast {
        encounter: EncounterId,
        caster: ActorId,
        spell: ContentId,
        power: SpellPower,
        targets: Vec<ActorId>,
    },
    SpellSlotSpent {
        actor: ActorId,
        level: u8,
        source: SpellSlotSource,
    },
    TemporarySpellSlotCreated {
        actor: ActorId,
        level: u8,
        point_cost: u8,
    },
    SpellSlotConverted {
        actor: ActorId,
        level: u8,
        points_gained: u8,
        source: SpellSlotSource,
    },
    SorceryPointsSpent {
        actor: ActorId,
        amount: u8,
        reason: ContentId,
    },
    SorceryPointsGained {
        actor: ActorId,
        amount: u8,
    },
    MetamagicApplied {
        actor: ActorId,
        option: ContentId,
        point_cost: u8,
    },
    AttackRolled {
        attacker: ActorId,
        target: ActorId,
        die: u8,
        modifier: i16,
        total: i16,
        defense: i16,
        hit: bool,
    },
    SavingThrowRolled {
        actor: ActorId,
        ability: crate::Ability,
        die: u8,
        modifier: i16,
        total: i16,
        difficulty: i16,
        success: bool,
    },
    DamageApplied {
        source: ActorId,
        target: ActorId,
        amount: i32,
        hp_after: i32,
    },
    HealingApplied {
        source: ActorId,
        target: ActorId,
        amount: i32,
        hp_after: i32,
    },
    ConditionApplied {
        target: ActorId,
        condition: ContentId,
        source: ActorId,
    },
    ConditionExpired {
        target: ActorId,
        condition: ContentId,
    },
    ConcentrationStarted {
        actor: ActorId,
        spell: ContentId,
    },
    ConcentrationEnded {
        actor: ActorId,
        spell: ContentId,
        reason: String,
    },
    ActorDefended {
        actor: ActorId,
        defense_bonus: i16,
    },
    TurnEnded {
        encounter: EncounterId,
        actor: ActorId,
    },
    EncounterEnded {
        encounter: EncounterId,
        outcome: CombatOutcome,
        transition_scene: ContentId,
    },
    CombatCommandRejected {
        encounter: Option<EncounterId>,
        actor: Option<ActorId>,
        reason: String,
    },
}
```

The log renderer converts events into sentences. Core events keep structured values, including slot level, slot source, point cost, and metamagic choice.

## Spell-resource commands

Add these resource command types to `storyforge-core/src/combat.rs`:

```rust
#[derive(
    Debug, Clone, Copy, PartialEq, Eq, serde::Serialize, serde::Deserialize,
)]
pub enum SpellPower {
    Cantrip,
    Slot { level: u8 },
}

#[derive(
    Debug, Clone, PartialEq, Eq, serde::Serialize, serde::Deserialize,
)]
pub enum SpellCommand {
    Cast {
        spell: ContentId,
        target: TargetSelection,
        power: SpellPower,
        metamagic: Vec<ContentId>,
    },
    CreateTemporarySlot {
        level: u8,
    },
    ConvertSlotToSorceryPoints {
        level: u8,
    },
}
```

`SpellPower::Cantrip` is legal only for a level-0 spell. `SpellPower::Slot` accepts levels 1 through 9 and must be at least the spell's base level.

Normal slot state has remaining and maximum counts for all nine levels. Temporary slots have counts only for levels 1 through 5. When both kinds exist at the selected level, spend a temporary slot first because it disappears at the next long rest. Record the source in `SpellSlotSpent`.

Flexible casting uses the bonus action:

| Temporary slot | Point cost |
| --- | ---: |
| 1st | 2 |
| 2nd | 3 |
| 3rd | 5 |
| 4th | 6 |
| 5th | 7 |

Converting a remaining slot grants points equal to its level. Reject conversion if the actor lacks the slot, has no sorcery feature, lacks a bonus action, or would exceed their point maximum.

Do not mutate slot or point state during validation. Build the complete event list first, then apply it through the same event path used by every other command.

## Spell definitions

Create three original demo spells:

### Ember Thread

- Level: Cantrip.
- Range: Near or Medium.
- Spell attack.
- Damage: `1d6 + casting modifier`.
- Gains one additional `d6` at character levels 5, 11, and 17.
- Applies Burning for two turn ends on a critical hit.

### Ward

- Level: 1st.
- Range: Self.
- Grants +2 defense until next turn.
- Enables a reaction that reduces direct magical damage by `1d4 + slot level`.
- Upcasting increases the fixed reduction because the chosen slot level is part of the expression.

### Binding Chalk

- Level: 1st.
- Range: Near.
- Dexterity save.
- Applies Restrained until the target succeeds at an end-turn save.
- Requires concentration.
- A 2nd-level slot extends the range to Medium.
- A 3rd-level or higher slot may target one additional creature in range.

Use parsed dice expressions. A spell definition never calls RNG itself.

## Metamagic validation

A character with a sorcery feature can choose learned metamagic while preparing a cast:

| Option | Cost | Validation and event result |
| --- | ---: | --- |
| Careful Spell | 1 | Select between one creature and the casting ability modifier in creatures. Each selected creature automatically succeeds on the spell's save. |
| Distant Spell | 1 | Move a ranged spell one band farther, up to Far, or change touch range to Near. |
| Empowered Spell | 1 | Choose up to the casting ability modifier in damage dice, with a minimum of one, reroll them, and keep the new values. |
| Extended Spell | 1 | Require a duration of at least one minute and double it without exceeding 24 hours. |
| Heightened Spell | 3 | Choose one target. Its first save against this cast has disadvantage. |
| Quickened Spell | 2 | Require a one-action casting time and consume the bonus action instead. |
| Subtle Spell | 1 | Ignore verbal and somatic requirements. Material and focus requirements remain. |
| Twinned Spell | Spell level, minimum 1 | Require a non-self spell that targets one creature, then select one additional legal target. A cantrip costs 1 point. |

Normally accept at most one option. Empowered Spell may accompany one other compatible option. Reject duplicate options.

The cast preview shows:

- Base spell level.
- Selected slot level and remaining slots.
- Upcast changes.
- Point cost.
- Action or bonus action consumed.
- Targets protected, added, or hindered.
- Components removed by Subtle Spell.

The default profile allows one leveled spell per turn. After Quickened Spell casts a leveled spell, the actor may use their action for a cantrip or a non-spell action.

## Dice parser

The parser must accept these exact forms:

```text
d20
1d4
2d6
1d8+2
2d6-1
```

Parse into:

```rust
use std::{num::ParseIntError, str::FromStr};

#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub struct DiceExpression {
    pub count: u16,
    pub sides: u16,
    pub modifier: i16,
}

#[derive(Debug, thiserror::Error)]
pub enum DiceError {
    #[error("dice expression must look like `2d6+1`")]
    InvalidFormat,
    #[error("dice expression contains an invalid number: {0}")]
    InvalidNumber(#[from] ParseIntError),
    #[error("dice count must be between 1 and 100")]
    CountOutOfRange,
    #[error("die sides must be between 2 and 1000")]
    SidesOutOfRange,
    #[error("dice modifier must be between -10000 and 10000")]
    ModifierOutOfRange,
}

impl FromStr for DiceExpression {
    type Err = DiceError;

    fn from_str(input: &str) -> Result<Self, Self::Err> {
        let input = input.trim();
        let (count_text, rest) = input.split_once('d').ok_or(DiceError::InvalidFormat)?;
        if rest.contains('d') {
            return Err(DiceError::InvalidFormat);
        }

        let modifier_index = rest
            .char_indices()
            .find_map(|(index, character)| matches!(character, '+' | '-').then_some(index));
        let (sides_text, modifier_text) = match modifier_index {
            Some(index) => (&rest[..index], Some(&rest[index..])),
            None => (rest, None),
        };

        let count = if count_text.is_empty() {
            1
        } else {
            count_text.parse::<u16>()?
        };
        let sides = sides_text.parse::<u16>()?;
        let modifier = modifier_text.map_or(Ok(0), str::parse::<i16>)?;

        if !(1..=100).contains(&count) {
            return Err(DiceError::CountOutOfRange);
        }
        if !(2..=1000).contains(&sides) {
            return Err(DiceError::SidesOutOfRange);
        }
        if !(-10_000..=10_000).contains(&modifier) {
            return Err(DiceError::ModifierOutOfRange);
        }

        Ok(Self {
            count,
            sides,
            modifier,
        })
    }
}
```

The validation limits are:

- Count 1 through 100.
- Sides 2 through 1000.
- Modifier -10,000 through 10,000.

Never panic on player or content input.

Add focused parser tests:

```rust
#[test]
fn parses_supported_dice_forms() {
    let cases = [
        ("d20", (1, 20, 0)),
        ("1d4", (1, 4, 0)),
        ("2d6", (2, 6, 0)),
        ("1d8+2", (1, 8, 2)),
        ("2d6-1", (2, 6, -1)),
    ];

    for (input, expected) in cases {
        let parsed = input
            .parse::<DiceExpression>()
            .expect("supported expression should parse");
        assert_eq!(
            (parsed.count, parsed.sides, parsed.modifier),
            expected,
            "{input}"
        );
    }
}

#[test]
fn rejects_out_of_range_or_malformed_dice() {
    for input in ["0d6", "101d6", "2d1", "2d1001", "2d6+10001", "2dd6"] {
        assert!(input.parse::<DiceExpression>().is_err(), "{input}");
    }
}
```

## Command legality

Before mutation, verify:

- Encounter is active.
- Actor owns the turn.
- Actor is able to act.
- Action or movement is available.
- Target exists and is legal.
- Target is in range.
- Spell is prepared.
- A cantrip uses no slot, or the chosen spell slot exists.
- The chosen slot level is equal to or higher than the spell level.
- Sorcery points and learned metamagic are sufficient.
- Action or bonus action matches the final casting time.
- The actor has not already cast a leveled spell this turn.
- Concentration rules allow the spell.
- Item exists and is usable.

On rejection, emit one event with a player-safe reason. Do not spend a slot, sorcery points, action, or bonus action.

## Damage order

1. Roll base damage.
2. Add modifiers.
3. Apply critical rule.
4. Apply immunity.
5. Apply resistance or vulnerability.
6. Apply reaction reduction.
7. Clamp final damage to at least zero.
8. Reduce HP without going below zero.
9. Test concentration.
10. Check encounter outcome.

Each stage that changes the result belongs in structured event details or the expanded combat log.

## Enemy AI

The MVP enemy has three scored actions:

- Cast a basic attack when in range.
- Move toward the player when out of range.
- Defend when below one-third HP and unable to finish the player.

AI receives a read-only combat view. It chooses one legal `GameCommand` and sends it through the same reducer as player commands.

Tie scores with engine RNG and record the candidate actions and chosen action at debug log level.

## Combat UI

Show:

```text
DUEL

Player                    Gate Warden
HP     12/12              HP  10/10
Slots  1st 1/2            Intent: Ember Bolt
SP     3/3
Status: Healthy           Status: Guarded

Distance: Medium

1 Cast spell
2 Move
3 Defend
4 Use item
5 Examine
6 Escape
7 End turn
```

The intent is a readable hint, not the exact random roll.

Spell selection shows spell level, available slots, upcast effect, range, chance information the character knows, metamagic, and concentration warning.

## Turn state

Start-turn sequence:

1. Expire start-turn conditions.
2. Apply periodic effects.
3. Check whether actor is downed.
4. Reset action, bonus action, movement, and the leveled-spell marker.
5. Restore reaction.
6. Emit `TurnStarted`.

End-turn sequence:

1. Apply end-turn effects.
2. Attempt defined condition saves.
3. Expire durations.
4. Emit `TurnEnded`.
5. Select next active actor.

## Encounter ending

On victory:

- Emit outcome.
- Apply authored rewards.
- Clear combat-only effects.
- Retain injuries and campaign-duration conditions.
- Transition to victory scene.
- Request autosave.

On defeat:

- Emit outcome.
- Transition to the encounter's defeat scene.
- Apply time, injury, relationship, or resource consequence.
- Request autosave after the defeat scene begins.

## Tests

Required unit tests:

- Distance movement clamps at both ends.
- Action cannot be spent twice.
- A cantrip does not spend a slot.
- A rejected cast does not spend a slot or sorcery points.
- A lower-level slot cannot cast a higher-level spell.
- Upcasting applies the selected spell's authored improvement.
- Temporary slots are spent before normal slots.
- Flexible casting costs match the five supported slot levels.
- Slot conversion never exceeds maximum sorcery points.
- Quickened Spell consumes a bonus action.
- Empowered Spell can combine with one other option.
- Other metamagic pairs are rejected.
- Twinned Spell rejects self and multi-target spells.
- Subtle Spell retains material requirements.
- Resistance applies after modifiers.
- HP stops at zero.
- Ward expires at next turn.
- Concentration ends when another concentration spell begins.
- Initiative ties resolve by Dexterity then actor ID.
- Enemy AI returns only legal commands.
- Victory and defeat transition once.

Use direct assertions for combat math. Use snapshots only for structured logs and terminal layout.

## Manual test

1. Start at Medium range.
2. Try an out-of-range leveled spell and confirm no slot is spent.
3. Move to Near.
4. Cast Binding Chalk with a 1st-level slot.
5. Replay the cast with a 2nd-level slot and inspect the upcast range.
6. Observe concentration.
7. Cast Ember Thread after all slots are empty.
8. Use flexible casting in the sorcery-enabled fixture.
9. Quick-cast Ember Thread, then use the action to defend.
10. Use a potion below maximum HP.
11. Win once.
12. Replay and lose once.
13. Load each resulting autosave.

## Verify

```powershell
cargo run -p storyforge-tui -- validate --pack campaigns/academy-demo --strict
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features --locked -- -D warnings
cargo test --workspace --all-features --locked
```

## Acceptance check

- All commands pass through core validation.
- Player and AI use the same command path.
- The combat log explains every HP, spell-slot, and sorcery-point change.
- Cantrips, upcasting, flexible casting, and all eight metamagic options have rule tests.
- Victory and defeat return to narrative scenes.
- Save and reload preserve active combat if manual saving is allowed there.

## Suggested commit

```powershell
git add .
git commit -m "Add the tactical duel engine"
```
