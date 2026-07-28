# 25: Make relationships react and remember

## Result

Gifts, favors, quest decisions, broken promises, and public actions change an NPC's relationship axes. The NPC remembers why, comments on it later, and adopts a warmer or more hostile attitude. Strong positive relationships unlock authored rewards. Strong negative relationships create authored complications, rivalry, a duel, or combat without making the main story impossible.

Rumors can move between NPCs who meet through chapter 24's schedules. A rumor changes what an NPC knows; it does not automatically make the rumor true.

## Build this chapter in five checkpoints

| Checkpoint | Destination | Proof before continuing |
| --- | --- | --- |
| 1 | `storyforge-core/src/social/attitude.rs` | Relationship score and attitude-band boundaries |
| 2 | `storyforge-core/src/social/gift.rs` | Gift preference, cooldown, and diminishing-return tests |
| 3 | `storyforge-core/src/social/milestone.rs` | Positive rewards and hostile consequences fire once |
| 4 | `storyforge-core/src/social/rumor.rs` | Deterministic, bounded rumor transmission |
| 5 | `storyforge-tui/src/screens/relationships.rs` | Gift preview, memory comments, and attitude presentation |

Keep the separate Trust, Respect, Fear, Affection, Rivalry, and Obligation axes from chapter 09. An attitude band is a derived summary, not a replacement.

## Checkpoint 1: Derive an authored attitude

Create `storyforge-core/src/social/attitude.rs`:

```rust
use std::collections::HashMap;

use serde::{Deserialize, Serialize};

use crate::RelationshipAxis;

#[derive(
    Debug, Clone, Copy, PartialEq, Eq, PartialOrd, Ord, Serialize, Deserialize,
)]
pub enum AttitudeBand {
    Hostile,
    Adversarial,
    Cold,
    Neutral,
    Warm,
    Devoted,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct AttitudeWeight {
    pub axis: RelationshipAxis,
    pub weight: i16,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct AttitudeThreshold {
    pub minimum: i16,
    pub band: AttitudeBand,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct AttitudeProfile {
    pub weights: Vec<AttitudeWeight>,
    pub thresholds: Vec<AttitudeThreshold>,
}

#[derive(Debug, Clone, PartialEq, Eq, thiserror::Error)]
pub enum AttitudeError {
    #[error("attitude profile must contain at least one threshold")]
    MissingThreshold,
    #[error("attitude score overflowed")]
    ScoreOverflow,
}

pub fn attitude_score(
    relationship: &HashMap<RelationshipAxis, i8>,
    profile: &AttitudeProfile,
) -> Result<i16, AttitudeError> {
    profile.weights.iter().try_fold(0_i16, |score, entry| {
        let value = i16::from(
            relationship
                .get(&entry.axis)
                .copied()
                .unwrap_or_default(),
        );
        score
            .checked_add(
                value
                    .checked_mul(entry.weight)
                    .ok_or(AttitudeError::ScoreOverflow)?,
            )
            .ok_or(AttitudeError::ScoreOverflow)
    })
}

pub fn attitude_band(
    score: i16,
    profile: &AttitudeProfile,
) -> Result<AttitudeBand, AttitudeError> {
    profile
        .thresholds
        .iter()
        .filter(|threshold| threshold.minimum <= score)
        .max_by_key(|threshold| threshold.minimum)
        .map(|threshold| threshold.band)
        .ok_or(AttitudeError::MissingThreshold)
}
```

Each NPC decides which axes matter. A proud rival may value Respect and Rivalry. A frightened witness may care mostly about Trust and Fear. Do not hardcode one universal formula.

Create a public demo profile:

```ron
(
    weights: [
        (axis: Trust, weight: 3),
        (axis: Respect, weight: 2),
        (axis: Affection, weight: 2),
        (axis: Rivalry, weight: -1),
        (axis: Fear, weight: -2),
    ],
    thresholds: [
        (minimum: -50, band: Hostile),
        (minimum: -20, band: Adversarial),
        (minimum: -5, band: Cold),
        (minimum: 0, band: Neutral),
        (minimum: 12, band: Warm),
        (minimum: 28, band: Devoted),
    ],
)
```

Validation sorts thresholds by minimum and requires coverage of the profile's possible score range.

## Checkpoint 2: Model gifts and quest favors

Create `storyforge-core/src/social/gift.rs`:

```rust
use serde::{Deserialize, Serialize};

use crate::{ContentId, EffectDefinition, GameClock};

#[derive(Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize)]
pub enum GiftPreference {
    Loved,
    Liked,
    Neutral,
    Disliked,
    Hated,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct GiftReactionDefinition {
    pub preference: GiftPreference,
    pub required_tags: Vec<String>,
    pub forbidden_tags: Vec<String>,
    pub first_time_effects: Vec<EffectDefinition>,
    pub repeat_effects: Vec<EffectDefinition>,
    pub comment_scene: ContentId,
    pub cooldown_days: u16,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct NpcGiftProfile {
    pub actor: ContentId,
    pub reactions: Vec<GiftReactionDefinition>,
    pub default_reaction: GiftReactionDefinition,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct GiftHistoryEntry {
    pub actor: ContentId,
    pub item: ContentId,
    pub preference: GiftPreference,
    pub day: u32,
    pub occurrence: u16,
}

#[derive(Debug, Clone, PartialEq, Eq, thiserror::Error)]
pub enum GiftError {
    #[error("the player does not own that gift")]
    ItemNotOwned,
    #[error("this item cannot be given away")]
    ItemNotGiftable,
    #[error("the NPC is not available")]
    NpcUnavailable,
    #[error("the NPC will not accept another gift yet")]
    Cooldown,
}

#[must_use]
pub fn gift_reaction<'a>(
    item_tags: &[String],
    profile: &'a NpcGiftProfile,
) -> &'a GiftReactionDefinition {
    profile
        .reactions
        .iter()
        .find(|reaction| {
            reaction
                .required_tags
                .iter()
                .all(|tag| item_tags.contains(tag))
                && reaction
                    .forbidden_tags
                    .iter()
                    .all(|tag| !item_tags.contains(tag))
        })
        .unwrap_or(&profile.default_reaction)
}

#[must_use]
pub fn gift_is_on_cooldown(
    reaction: &GiftReactionDefinition,
    history: &[GiftHistoryEntry],
    actor: &ContentId,
    clock: &GameClock,
) -> bool {
    history
        .iter()
        .rev()
        .find(|entry| &entry.actor == actor)
        .is_some_and(|entry| {
            clock.day.saturating_sub(entry.day) < u32::from(reaction.cooldown_days)
        })
}
```

The first gift may create a strong memory. Repeats use `repeat_effects`, which should be smaller or purely conversational. This blocks an infinite loop where the player buys the cheapest liked item and farms affection.

Quest help uses the same effect catalog:

```rust
#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct RelationshipImpactDefinition {
    pub id: ContentId,
    pub actor: ContentId,
    pub cause: ContentId,
    pub effects: Vec<EffectDefinition>,
    pub memory: ContentId,
    pub immediate_comment_scene: Option<ContentId>,
    pub later_comment_scene: Option<ContentId>,
}
```

Examples of `cause`:

- Returned a lost item without reading it.
- Helped an NPC's friend during a quest.
- Publicly embarrassed the NPC.
- Broke a promise.
- Protected the NPC in combat.
- Supported or harmed their faction.

The quest outcome points to the impact definition. The relationship engine should not guess intent from quest names.

## Commands and events

Create `storyforge-core/src/social/interaction_command.rs`:

```rust
use crate::{ContentId, EffectDefinition};

use super::{AttitudeBand, GiftPreference};

#[derive(Debug, Clone, PartialEq, Eq)]
pub enum RelationshipCommand {
    OfferGift {
        actor: ContentId,
        item: ContentId,
    },
    ApplyRelationshipImpact {
        impact: ContentId,
    },
    PlayMemoryComment {
        actor: ContentId,
        memory: ContentId,
    },
    ClaimRelationshipMilestone {
        actor: ContentId,
        milestone: ContentId,
    },
}

#[derive(
    Debug, Clone, PartialEq, Eq, serde::Serialize, serde::Deserialize,
)]
pub enum RelationshipEvent {
    GiftOffered {
        actor: ContentId,
        item: ContentId,
        preference: GiftPreference,
        occurrence: u16,
    },
    RelationshipEffectsApplied {
        actor: ContentId,
        cause: ContentId,
        effects: Vec<EffectDefinition>,
    },
    AttitudeChanged {
        actor: ContentId,
        previous: AttitudeBand,
        current: AttitudeBand,
    },
    MemoryCommentRequested {
        actor: ContentId,
        memory: ContentId,
        scene: ContentId,
    },
    RelationshipMilestoneReached {
        actor: ContentId,
        milestone: ContentId,
    },
    RelationshipCommandRejected {
        reason: String,
    },
    AutosaveRequested {
        reason: String,
    },
}
```

Add `Relationship(RelationshipCommand),` to `GameCommand` and `Relationship(RelationshipEvent),` to `GameEvent`.

| Command | Validation | Successful behavior |
| --- | --- | --- |
| `OfferGift` | NPC is present, item is owned and giftable, cooldown expired | Removes item, records reaction and memory, applies effects, requests comment scene |
| `ApplyRelationshipImpact` | Impact exists and has not been applied for this source occurrence | Applies effects once, records memory, recalculates attitude |
| `PlayMemoryComment` | NPC knows the memory and scene conditions pass | Opens immediate or later comment and marks that comment occurrence |
| `ClaimRelationshipMilestone` | Band and milestone conditions pass; milestone is unclaimed | Applies reward or complication once |

Build the entire event list before removing a gift or applying an effect. A rejected gift remains in inventory.

## Checkpoint 3: Reward friendship and make hostility interesting

Create `storyforge-core/src/social/milestone.rs`:

```rust
use serde::{Deserialize, Serialize};

use crate::{ContentId, EffectDefinition};

use super::AttitudeBand;

#[derive(Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize)]
pub enum RelationshipMilestoneKind {
    Reward,
    Complication,
    Duel,
    Combat,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct RelationshipMilestoneDefinition {
    pub id: ContentId,
    pub actor: ContentId,
    pub required_band: AttitudeBand,
    pub kind: RelationshipMilestoneKind,
    pub scene: ContentId,
    pub effects: Vec<EffectDefinition>,
    pub once_per_save: bool,
    pub fallback_scene: Option<ContentId>,
}
```

Positive milestone examples:

- A personal spell lesson.
- A unique item or wand upgrade.
- A companion recruitment route.
- A safe-house or secret passage.
- Help during a later quest.
- A mount introduction.
- A recommendation that improves faction access.

Negative milestone examples:

- The NPC raises shop prices or refuses a favor.
- A rival spreads an authored rumor.
- The NPC competes for the same quest reward.
- A sabotage encounter adds a puzzle or hazard.
- The NPC challenges the player to a nonlethal duel.
- A hostile adult or enemy attacks when the story permits it.

Hostility must remain playable:

1. Main-story information has another source.
2. Essential shops have a fallback merchant or repair path.
3. A school-age rival defaults to nonlethal conflict.
4. Combat requires an authored scene and defeat route.
5. The complication ends, escalates, or can be reconciled.
6. The UI explains the remembered cause without exposing hidden numbers.

Do not reward manipulation by making every devoted NPC a vending machine. Milestones are authored character moments and normally occur once.

## Checkpoint 4: Select memory-aware dialogue

Add these fields to dialogue choices:

```rust
#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct RelationshipDialogueConditions {
    pub minimum_attitude: Option<AttitudeBand>,
    pub maximum_attitude: Option<AttitudeBand>,
    pub required_memories: Vec<ContentId>,
    pub forbidden_memories: Vec<ContentId>,
    pub required_gift_preferences: Vec<GiftPreference>,
}
```

Dialogue selection order:

1. Mandatory quest or danger interruption.
2. A new milestone scene.
3. An immediate memory comment.
4. A later unspoken memory comment.
5. Time-aware greeting for the current attitude.
6. Normal dialogue hub.

If two comments have equal priority, use oldest undisclosed memory first, then stable memory ID. This makes conversation order replayable.

Example attitude differences:

```text
DEVOTED
"I kept the page you returned. I knew you could have read it."

NEUTRAL
"The archive closes at six. Did you need something?"

ADVERSARIAL
"I remember the bridge. Say what you came to say."
```

The NPC comments on an event because a memory definition supplies that line. The engine does not invent dialogue.

## Checkpoint 5: Propagate rumors through scheduled meetings

Create `storyforge-core/src/social/rumor.rs`:

```rust
use std::collections::{BTreeMap, BTreeSet};

use rand::Rng;
use serde::{Deserialize, Serialize};

use crate::ContentId;

#[derive(Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize)]
pub enum RumorConfidence {
    Heard,
    Repeated,
    Credible,
    Confirmed,
    Disproved,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct RumorDefinition {
    pub id: ContentId,
    pub subject: ContentId,
    pub public_summary: String,
    pub variants: Vec<ContentId>,
    pub base_spread_percent: u8,
    pub expires_on_day: Option<u32>,
    pub allowed_factions: BTreeSet<ContentId>,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct RumorKnowledge {
    pub rumor: ContentId,
    pub variant: ContentId,
    pub heard_from: Option<ContentId>,
    pub heard_on_day: u32,
    pub confidence: RumorConfidence,
    pub times_repeated: u16,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct RumorState {
    pub known_by_actor: BTreeMap<ContentId, Vec<RumorKnowledge>>,
}

#[derive(Debug, Clone, PartialEq, Eq)]
pub struct RumorContact {
    pub speaker: ContentId,
    pub listener: ContentId,
    pub shared_location: ContentId,
    pub listener_factions: BTreeSet<ContentId>,
}

#[derive(Debug, Clone, PartialEq, Eq)]
pub struct RumorTransmission {
    pub rumor: ContentId,
    pub variant: ContentId,
    pub speaker: ContentId,
    pub listener: ContentId,
}

pub fn select_rumor_transmissions<R: Rng + ?Sized>(
    rng: &mut R,
    definitions: &BTreeMap<ContentId, RumorDefinition>,
    state: &RumorState,
    contacts: &[RumorContact],
    day: u32,
    maximum: usize,
) -> Vec<RumorTransmission> {
    let mut sorted_contacts = contacts.to_vec();
    sorted_contacts.sort_by(|left, right| {
        left.speaker
            .as_str()
            .cmp(right.speaker.as_str())
            .then_with(|| left.listener.as_str().cmp(right.listener.as_str()))
            .then_with(|| {
                left.shared_location
                    .as_str()
                    .cmp(right.shared_location.as_str())
            })
    });

    let mut transmissions = Vec::new();
    for contact in sorted_contacts {
        let Some(known_rumors) = state.known_by_actor.get(&contact.speaker)
        else {
            continue;
        };

        for knowledge in known_rumors {
            if transmissions.len() >= maximum {
                return transmissions;
            }
            let Some(definition) = definitions.get(&knowledge.rumor) else {
                continue;
            };
            if definition
                .expires_on_day
                .is_some_and(|expiry| day > expiry)
            {
                continue;
            }
            if !definition.allowed_factions.is_empty()
                && definition
                    .allowed_factions
                    .is_disjoint(&contact.listener_factions)
            {
                continue;
            }
            let listener_already_knows = state
                .known_by_actor
                .get(&contact.listener)
                .is_some_and(|known| {
                    known.iter().any(|entry| entry.rumor == knowledge.rumor)
                });
            if listener_already_knows {
                continue;
            }
            if rng.gen_range(0_u8..100) >= definition.base_spread_percent {
                continue;
            }

            transmissions.push(RumorTransmission {
                rumor: knowledge.rumor.clone(),
                variant: knowledge.variant.clone(),
                speaker: contact.speaker.clone(),
                listener: contact.listener.clone(),
            });
            break;
        }
    }
    transmissions
}
```

Create contacts only when scheduled NPCs share a social location during a processed calendar boundary. Do not compare every NPC with every other NPC.

Add rumor events:

```rust
RumorLearned {
    actor: ContentId,
    knowledge: RumorKnowledge,
},
RumorRepeated {
    actor: ContentId,
    rumor: ContentId,
},
RumorDisproved {
    rumor: ContentId,
    evidence: ContentId,
},
```

An authored variant handles distortion. The engine never rewrites text or changes facts at random.

## Original demo content

Create one gift profile and three milestone scenes:

```ron
(
    actor: "academy.npc.tovan-reed",
    reactions: [
        (
            preference: Loved,
            required_tags: ["field-notes"],
            forbidden_tags: ["stolen"],
            first_time_effects: [
                ChangeRelationship(
                    actor: "academy.npc.tovan-reed",
                    axis: Trust,
                    amount: 2,
                ),
            ],
            repeat_effects: [],
            comment_scene: "academy.scene.tovan-loved-gift",
            cooldown_days: 5,
        ),
        (
            preference: Disliked,
            required_tags: ["showy"],
            forbidden_tags: [],
            first_time_effects: [
                ChangeRelationship(
                    actor: "academy.npc.tovan-reed",
                    axis: Respect,
                    amount: -1,
                ),
            ],
            repeat_effects: [],
            comment_scene: "academy.scene.tovan-disliked-gift",
            cooldown_days: 5,
        ),
    ],
    default_reaction: (
        preference: Neutral,
        required_tags: [],
        forbidden_tags: [],
        first_time_effects: [],
        repeat_effects: [],
        comment_scene: "academy.scene.tovan-neutral-gift",
        cooldown_days: 5,
    ),
)
```

The public demo milestones are:

| Band | Scene | Consequence |
| --- | --- | --- |
| Devoted | `tovan-shares-the-key` | Unlocks an archive shortcut and companion research scene |
| Adversarial | `tovan-competes-for-notes` | Adds a timed competitor to the next archive quest |
| Hostile | `tovan-demands-a-duel` | Opens a nonlethal duel with reconciliation and rivalry outcomes |

## TUI behavior

The relationship panel shows words and reasons:

```text
TOVAN REED
Attitude: Warm

Trust      +4
Respect    +2
Fear        0
Affection  +1
Rivalry    +2

Recent:
+ Returned the runaway notes unread
- Supported the bridge-club accusation
```

The gift screen previews social risk without revealing exact hidden effects:

```text
Field Journal
Tovan may appreciate practical research tools.

Give this item?
```

For a hated tag:

```text
Ceremonial Medal
Tovan has spoken against showy status symbols.

This gift may offend him.
```

## Validator additions

| Code | Invalid content |
| --- | --- |
| `REL001` | Attitude thresholds leave a possible score uncovered |
| `REL002` | Duplicate attitude threshold |
| `REL003` | Gift preference has contradictory required and forbidden tags |
| `REL004` | Gift comment scene does not exist |
| `REL005` | Relationship impact references an unknown actor, memory, or cause |
| `REL006` | Milestone has no scene or repeats without an explicit policy |
| `REL007` | Hostile quest-critical NPC has no fallback route |
| `REL008` | Rumor spread percentage exceeds 100 |
| `REL009` | Rumor has no authored variant |
| `REL010` | Rumor subject or allowed faction does not exist |

## Automated test checklist

1. Each NPC profile produces the expected attitude at every threshold.
2. Missing relationship axes count as zero.
3. A loved gift uses first-time effects once.
4. A repeated gift uses smaller repeat effects.
5. Cooldown rejection leaves the gift in inventory.
6. A disliked or hated gift can reduce an axis.
7. Quest help creates one memory and one impact.
8. An NPC selects the oldest undisclosed memory comment.
9. Positive and negative milestones fire once.
10. A hostile milestone cannot remove the only main-story route.
11. The same rumor seed and contacts produce the same transmissions.
12. A listener does not learn the same rumor twice.
13. Expired rumors stop spreading.
14. Disproved rumors remain in history with changed confidence.
15. Save and load preserve gift history, milestones, attitude, and rumor knowledge.

## Manual playthrough

1. Return Tovan's notes without reading them.
2. Speak to him immediately and hear the quest comment.
3. Leave, advance one day, and hear the later greeting variant.
4. Give him a field journal and confirm his attitude warms.
5. Attempt another gift during cooldown and confirm the item is not removed.
6. Give a disliked item after cooldown and confirm his response is negative.
7. Reach a positive milestone and claim its scene once.
8. Replay with hostile choices and trigger the rival quest complication.
9. Let Tovan meet another NPC through their schedules.
10. Confirm the other NPC later repeats the authored rumor.

## Full verification

```powershell
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features --locked -- -D warnings
cargo test --workspace --all-features --locked
cargo run -p storyforge-tui -- validate --pack campaigns/academy-demo
cargo run -p storyforge-tui -- play --pack campaigns/academy-demo --seed 73
```

## Common mistakes

- One relationship number replaces the established axes.
- Gifts grant unlimited affection with no cooldown.
- The TUI applies relationship changes directly.
- The NPC changes attitude but never comments on the cause.
- Maximum hostility silently removes a required quest.
- Every hostile NPC attacks immediately.
- Rumors become true because enough NPCs repeat them.
- Rumor processing compares every NPC in the campaign.

## Acceptance check

- Gifts and quest actions can help or harm a relationship.
- NPC attitude is derived from authored axis weights.
- NPCs comment on remembered events through authored dialogue.
- Positive milestones grant character-specific rewards.
- Negative milestones create fair complications, rivalry, duels, or combat.
- Rumors spread through scheduled contact and remain distinct from facts.
- Every important state survives save and load.

## Suggested commit

```powershell
git add .
git commit -m "Add reactive relationships gifts and rumors"
```
