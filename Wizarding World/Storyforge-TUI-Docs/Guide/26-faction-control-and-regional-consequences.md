# 26: Make factions change regions

## Result

Factions react to player and world events. Their influence can make a region controlled, contested, or unstable. Control changes prices, passive income, checks, shop stock, secret missions, mounts, routes, and travel danger through visible authored rules.

A faction benefit is never global simply because the player's reputation is high. It comes from a faction service, controlled region, agreement, rank, or completed mission.

## Build this chapter in five checkpoints

| Checkpoint | Destination | Proof before continuing |
| --- | --- | --- |
| 1 | `storyforge-core/src/faction/reaction.rs` | Tagged events produce bounded faction reactions |
| 2 | `storyforge-core/src/faction/control.rs` | Controller and contested-state boundary tests |
| 3 | `storyforge-core/src/faction/regional_effect.rs` | Benefits and penalties compose in stable order |
| 4 | `storyforge-core/src/faction/income.rs` | Passive income accrues once and respects its cap |
| 5 | `storyforge-tui/src/screens/region.rs` | Region, controller, danger, access, and benefits are readable |

## Checkpoint 1: React to tagged game events

Create `storyforge-core/src/faction/reaction.rs`:

```rust
use serde::{Deserialize, Serialize};

use crate::{ConditionDefinition, ContentId, EffectDefinition};

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct FactionReactionDefinition {
    pub id: ContentId,
    pub faction: ContentId,
    pub event_tags: Vec<ContentId>,
    pub conditions: Vec<ConditionDefinition>,
    pub effects: Vec<EffectDefinition>,
    pub once_per_source_event: bool,
    pub public_reason: ContentId,
}

#[derive(Debug, Clone, PartialEq, Eq)]
pub struct TaggedWorldEvent {
    pub source_event: u64,
    pub tags: Vec<ContentId>,
    pub region: Option<ContentId>,
    pub actors: Vec<ContentId>,
}

#[must_use]
pub fn matching_faction_reactions<'a, F>(
    event: &TaggedWorldEvent,
    definitions: &'a [FactionReactionDefinition],
    condition_is_met: F,
) -> Vec<&'a FactionReactionDefinition>
where
    F: Fn(&ConditionDefinition) -> bool + Copy,
{
    let mut matching = definitions
        .iter()
        .filter(|definition| {
            definition
                .event_tags
                .iter()
                .any(|tag| event.tags.contains(tag))
                && definition.conditions.iter().all(condition_is_met)
        })
        .collect::<Vec<_>>();
    matching.sort_by(|left, right| left.id.as_str().cmp(right.id.as_str()));
    matching
}
```

Examples of event tags:

```text
academy.event.protected-students
academy.event.exposed-corruption
academy.event.damaged-public-property
academy.event.aided-smugglers
academy.event.broke-ceasefire
academy.event.rescued-faction-member
```

Several factions may react to one event. Each reaction has its own conditions and effects. One faction may reward exposing a secret while another loses trust because it wanted the secret contained.

Add an application record so a reaction cannot fire twice for the same source event:

```rust
#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct AppliedFactionReaction {
    pub definition: ContentId,
    pub source_event: u64,
}
```

## Checkpoint 2: Derive regional control

Create `storyforge-core/src/faction/control.rs`:

```rust
use std::collections::BTreeMap;

use serde::{Deserialize, Serialize};

use crate::ContentId;

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct RegionalControlPolicy {
    pub minimum_control: i16,
    pub required_lead: i16,
    pub minimum_influence: i16,
    pub maximum_influence: i16,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub enum RegionControl {
    Uncontrolled,
    Controlled {
        faction: ContentId,
        influence: i16,
    },
    Contested {
        leaders: Vec<ContentId>,
    },
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct RegionFactionState {
    pub region: ContentId,
    pub influence: BTreeMap<ContentId, i16>,
    pub control: RegionControl,
    pub unresolved_conflicts: Vec<ContentId>,
}

#[derive(Debug, Clone, PartialEq, Eq, thiserror::Error)]
pub enum ControlError {
    #[error("regional control policy has an invalid influence range")]
    InvalidRange,
}

pub fn derive_region_control(
    influence: &BTreeMap<ContentId, i16>,
    policy: &RegionalControlPolicy,
) -> Result<RegionControl, ControlError> {
    if policy.minimum_influence > policy.maximum_influence {
        return Err(ControlError::InvalidRange);
    }

    let mut ranked = influence
        .iter()
        .map(|(faction, value)| {
            (
                faction.clone(),
                value.clamp(policy.minimum_influence, policy.maximum_influence),
            )
        })
        .collect::<Vec<_>>();
    ranked.sort_by(|left, right| {
        right
            .1
            .cmp(&left.1)
            .then_with(|| left.0.as_str().cmp(right.0.as_str()))
    });

    let Some((leader, leader_value)) = ranked.first() else {
        return Ok(RegionControl::Uncontrolled);
    };
    if *leader_value < policy.minimum_control {
        return Ok(RegionControl::Uncontrolled);
    }

    let second_value = ranked.get(1).map_or(policy.minimum_influence, |entry| entry.1);
    if leader_value.saturating_sub(second_value) < policy.required_lead {
        let leaders = ranked
            .iter()
            .take_while(|(_, value)| {
                leader_value.saturating_sub(*value) < policy.required_lead
            })
            .map(|(faction, _)| faction.clone())
            .collect();
        return Ok(RegionControl::Contested { leaders });
    }

    Ok(RegionControl::Controlled {
        faction: leader.clone(),
        influence: *leader_value,
    })
}
```

Influence is not reputation. Influence asks who can act in a region. Reputation asks how that faction treats the player.

Control changes only after applying an explicit influence event:

```rust
#[derive(Debug, Clone, PartialEq, Eq)]
pub enum FactionRegionCommand {
    ApplyFactionReaction {
        reaction: ContentId,
        source_event: u64,
    },
    ChangeRegionalInfluence {
        region: ContentId,
        faction: ContentId,
        amount: i16,
        reason: ContentId,
    },
    RecalculateRegion {
        region: ContentId,
    },
    ClaimFactionIncome {
        source: ContentId,
    },
    AcceptSecretMission {
        mission: ContentId,
    },
    RequestTravelAsset {
        asset: ContentId,
    },
}

#[derive(
    Debug, Clone, PartialEq, Eq, serde::Serialize, serde::Deserialize,
)]
pub enum FactionRegionEvent {
    FactionReactionApplied {
        reaction: ContentId,
        source_event: u64,
    },
    RegionalInfluenceChanged {
        region: ContentId,
        faction: ContentId,
        requested: i16,
        applied: i16,
        current: i16,
        reason: ContentId,
    },
    RegionControlChanged {
        region: ContentId,
        previous: RegionControl,
        current: RegionControl,
    },
    RegionalEffectsChanged {
        region: ContentId,
        active_effects: Vec<ContentId>,
    },
    FactionIncomeClaimed {
        source: ContentId,
        amount: i64,
    },
    SecretMissionAccepted {
        mission: ContentId,
    },
    TravelAssetGranted {
        asset: ContentId,
    },
    FactionRegionCommandRejected {
        reason: String,
    },
    AutosaveRequested {
        reason: String,
    },
}
```

Add `FactionRegion(FactionRegionCommand),` to `GameCommand` and `FactionRegion(FactionRegionEvent),` to `GameEvent`.

| Command | Validation | Successful behavior |
| --- | --- | --- |
| `ApplyFactionReaction` | Definition matches source tags and has not already fired | Applies faction effects and records source occurrence |
| `ChangeRegionalInfluence` | Region, faction, and reason exist; arithmetic is safe | Clamps influence and requests control recalculation |
| `RecalculateRegion` | Region and policy exist | Emits control change and replaces active regional effects |
| `ClaimFactionIncome` | Source is unlocked, due, and below stored cap | Adds exact currency and advances claim boundary |
| `AcceptSecretMission` | Mission is unlocked, available, and not accepted | Starts the authored quest |
| `RequestTravelAsset` | Rank, region, relationship, and availability pass | Adds mount, permit, guide, or transport service |

## Checkpoint 3: Apply benefits and penalties

Create `storyforge-core/src/faction/regional_effect.rs`:

```rust
use serde::{Deserialize, Serialize};

use crate::ContentId;

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub enum StandingRequirement {
    Any,
    ReputationAtLeast(i16),
    ReputationAtMost(i16),
    Rank(String),
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub enum RegionalEffectKind {
    SkillModifier {
        skill: ContentId,
        amount: i8,
    },
    ShopPriceBasisPoints {
        category: Option<ContentId>,
        amount: i32,
    },
    RestockPercent {
        category: Option<ContentId>,
        amount: i16,
    },
    PassiveIncome {
        source: ContentId,
    },
    SecretMission {
        mission: ContentId,
    },
    TravelAsset {
        asset: ContentId,
    },
    TravelMinutesPercent {
        route_tag: String,
        amount: i16,
    },
    DangerDelta {
        amount: i16,
    },
    RoutePermission {
        permission: ContentId,
    },
    RouteBlocked {
        route_tag: String,
    },
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct RegionalEffectDefinition {
    pub id: ContentId,
    pub controller: ContentId,
    pub standing: StandingRequirement,
    pub kind: RegionalEffectKind,
    pub public_description: String,
}
```

Signed values make benefits and penalties explicit:

- `SkillModifier +2` helps.
- `SkillModifier -2` hinders.
- `ShopPriceBasisPoints -1000` is a 10% discount.
- `ShopPriceBasisPoints +1500` is a 15% surcharge.
- `DangerDelta -1` makes travel safer.
- `DangerDelta +2` adds danger.

Use one stable modifier order:

1. Base rule.
2. World phase.
3. Regional control.
4. Faction standing.
5. Relationship.
6. Equipment or mount.
7. Difficulty profile.
8. Final authored clamp.

Every check preview lists the sources:

```text
Investigation
Ability                      +3
Archive Coalition control   +2
Night Court hostility       -1
Total modifier              +4
```

Do not hide a regional penalty in the final number.

## Regional danger and route access

Extend the route access check from chapter 12:

```rust
#[derive(Debug, Clone, PartialEq, Eq)]
pub struct RegionalTravelView {
    pub base_danger: i16,
    pub danger_delta: i16,
    pub permissions: Vec<ContentId>,
    pub blocked_tags: Vec<String>,
    pub travel_minutes_percent: i16,
}

impl RegionalTravelView {
    #[must_use]
    pub fn final_danger(&self) -> i16 {
        self.base_danger.saturating_add(self.danger_delta).clamp(0, 10)
    }

    #[must_use]
    pub fn route_is_blocked(&self, route_tags: &[String]) -> bool {
        self.blocked_tags
            .iter()
            .any(|blocked| route_tags.contains(blocked))
    }

    #[must_use]
    pub fn adjusted_minutes(&self, base_minutes: u32) -> u32 {
        let percent = i64::from(100_i16.saturating_add(self.travel_minutes_percent))
            .clamp(10, 300);
        let adjusted = i64::from(base_minutes)
            .saturating_mul(percent)
            .div_euclid(100);
        u32::try_from(adjusted).unwrap_or(u32::MAX).max(1)
    }
}
```

`unwrap_or` supplies a fallback and does not panic. A mount may reduce travel time or unlock a route tag. It does not remove all travel events unless its own definition says so.

Create a travel-asset definition:

```rust
#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct TravelAssetDefinition {
    pub id: ContentId,
    pub name: String,
    pub granting_faction: ContentId,
    pub required_rank: String,
    pub route_tags: Vec<String>,
    pub travel_minutes_percent: i16,
    pub passenger_capacity: u8,
    pub upkeep: Option<ContentId>,
}
```

Travel assets can represent a mount, carriage pass, portal permit, river guide, or trained flying creature.

## Checkpoint 4: Accrue passive income safely

Create `storyforge-core/src/faction/income.rs`:

```rust
use serde::{Deserialize, Serialize};

use crate::ContentId;

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct IncomeDefinition {
    pub id: ContentId,
    pub faction: ContentId,
    pub amount_per_period: i64,
    pub period_days: u16,
    pub maximum_unclaimed: i64,
    pub claim_location: Option<ContentId>,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct IncomeState {
    pub source: ContentId,
    pub last_accrual_day: u32,
    pub unclaimed: i64,
}

#[derive(Debug, Clone, PartialEq, Eq, thiserror::Error)]
pub enum IncomeError {
    #[error("income period must be greater than zero")]
    ZeroPeriod,
    #[error("income arithmetic overflowed")]
    Overflow,
}

pub fn accrue_income(
    definition: &IncomeDefinition,
    state: &IncomeState,
    current_day: u32,
) -> Result<IncomeState, IncomeError> {
    if definition.period_days == 0 {
        return Err(IncomeError::ZeroPeriod);
    }

    let elapsed = current_day.saturating_sub(state.last_accrual_day);
    let periods = elapsed / u32::from(definition.period_days);
    if periods == 0 {
        return Ok(state.clone());
    }

    let earned = definition
        .amount_per_period
        .checked_mul(i64::from(periods))
        .ok_or(IncomeError::Overflow)?;
    let unclaimed = state
        .unclaimed
        .checked_add(earned)
        .ok_or(IncomeError::Overflow)?
        .clamp(0, definition.maximum_unclaimed);
    let advanced_days = periods
        .checked_mul(u32::from(definition.period_days))
        .ok_or(IncomeError::Overflow)?;

    Ok(IncomeState {
        source: state.source.clone(),
        last_accrual_day: state.last_accrual_day.saturating_add(advanced_days),
        unclaimed,
    })
}
```

The cap prevents players from loading a save after a large time skip and receiving campaign-breaking wealth. Income advances on game days, not real-world time.

## Checkpoint 5: Author one contested region

Create `campaigns/academy-demo/regions/lantern-district.ron`:

```ron
(
    id: "academy.region.lantern-district",
    policy: (
        minimum_control: 40,
        required_lead: 15,
        minimum_influence: -100,
        maximum_influence: 100,
    ),
    starting_influence: {
        "academy.faction.archive-coalition": 42,
        "academy.faction.night-court": 35,
        "academy.faction.independent-merchants": 28,
    },
    control_effects: [
        (
            id: "academy.region-effect.archive-research",
            controller: "academy.faction.archive-coalition",
            standing: ReputationAtLeast(20),
            kind: SkillModifier(
                skill: "academy.skill.investigation",
                amount: 2,
            ),
            public_description: "Coalition archivists share local records with trusted allies.",
        ),
        (
            id: "academy.region-effect.merchant-carriage",
            controller: "academy.faction.independent-merchants",
            standing: Rank("partner"),
            kind: TravelAsset(
                asset: "academy.travel-asset.lantern-carriage",
            ),
            public_description: "Merchant partners may use the district carriage network.",
        ),
        (
            id: "academy.region-effect.night-toll",
            controller: "academy.faction.night-court",
            standing: ReputationAtMost(-20),
            kind: ShopPriceBasisPoints(
                category: None,
                amount: 1500,
            ),
            public_description: "Night Court agents impose a surcharge on known opponents.",
        ),
    ],
)
```

When the region is contested, apply only effects explicitly marked for contested control. Do not combine every contender's full benefits and penalties.

## TUI behavior

The map side panel shows:

```text
LANTERN DISTRICT
Control: Contested

Archive Coalition       42
Night Court             35
Independent Merchants   28

Danger: 4 / 10  (+1 from unrest)
Access: Main roads open

Your active effects:
+ Coalition research network: +2 Investigation
- Night Court toll: shop prices +15%
```

The faction screen shows benefits as conditions, not promises:

```text
ARCHIVE COALITION - TRUSTED

Active
+ Investigation support in Coalition-controlled regions
+ Secret mission: The Missing Catalogue

Locked
Carriage network
Requires: Partner rank and Merchant-controlled district
```

## Validator additions

| Code | Invalid content |
| --- | --- |
| `FAC001` | Reaction has no event tag |
| `FAC002` | Reaction effect or reason does not exist |
| `FAC003` | Regional influence starts outside policy range |
| `FAC004` | Control lead or minimum is negative |
| `FAC005` | Regional effect controller is unknown |
| `FAC006` | Skill, shop category, route tag, mission, or asset does not exist |
| `FAC007` | Income period is zero or amount exceeds campaign limit |
| `FAC008` | Income cap is lower than one period |
| `FAC009` | Travel adjustment is outside campaign limits |
| `FAC010` | Required main-story region can become permanently inaccessible |

## Automated test checklist

1. One tagged event can produce different faction reactions.
2. A reaction fires once per source event.
3. Influence clamps and reports the applied amount.
4. A clear leader becomes controller.
5. A narrow lead produces contested control.
6. A low-influence region remains uncontrolled.
7. Control ties resolve to contested, not alphabetical ownership.
8. Regional modifiers apply in stable order.
9. Check preview lists each signed source.
10. Hostile control can increase danger, prices, or access requirements.
11. Friendly control can grant a modifier, mission, income, or travel asset.
12. Passive income accrues by game day and stops at its cap.
13. A mount changes only routes whose tags it supports.
14. Required story routes retain an authored fallback.
15. Save and load preserve influence, control, effects, income, and assets.

## Manual playthrough

1. Enter the contested Lantern District and inspect all active effects.
2. Complete an event tagged `protected-students`.
3. Confirm two factions react differently.
4. Change Coalition influence enough to establish control.
5. Confirm danger, available mission, and Investigation preview update.
6. Lower Coalition reputation and confirm the trusted benefit disappears.
7. Establish Merchant control and earn the carriage asset.
8. Compare route time with and without the carriage.
9. Establish hostile Night Court control and confirm the visible surcharge.
10. Advance several game days and claim capped faction income.

## Full verification

```powershell
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features --locked -- -D warnings
cargo test --workspace --all-features --locked
cargo run -p storyforge-tui -- validate --pack campaigns/academy-demo
cargo run -p storyforge-tui -- play --pack campaigns/academy-demo --seed 89
```

## Common mistakes

- Influence and player reputation use the same number.
- Regional effects are hidden inside final modifiers.
- Every contender applies full effects in a contested region.
- Passive income uses real-world elapsed time.
- A mount removes all travel danger by default.
- Hostile control permanently blocks the only main-story route.
- Factions react to a quest name instead of explicit event tags.

## Acceptance check

- Factions react independently to tagged world events.
- Regional control derives from bounded influence and a visible policy.
- Friendly control can grant checks, income, missions, stock, or travel assets.
- Hostile control can impose fair prices, danger, access, or resource penalties.
- Contested regions have their own authored state.
- Every modifier and access change has a visible source.
- The main story remains completable under every tested controller.

## Suggested commit

```powershell
git add .
git commit -m "Add faction control and regional consequences"
```
