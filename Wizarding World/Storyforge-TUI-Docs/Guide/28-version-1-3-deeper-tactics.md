# 28: Deeper Tactics

## Result

Combat gains reactions and counterspells, terrain, cover, line of sight, summons, stealth openings, companion commands, nonlethal objectives, morale, surrender, and multi-phase bosses. Expanded AI understands encounter goals without reading hidden player state. A simulation command produces balance reports from deterministic fixtures.

Chapter 16 introduced the advanced-combat vocabulary. This chapter turns those pieces into the version 1.3 release sequence. Keep the chapter 10 duel working after every checkpoint.

## Release order

| Checkpoint | Feature | Why this order |
| --- | --- | --- |
| 1 | Reaction resolution and counterspells | Other tactics need a safe interruption model |
| 2 | Terrain, cover, and line of sight | AI and objectives need a shared battlefield view |
| 3 | Nonlethal objectives, morale, and surrender | Proves combat can end without zero HP |
| 4 | Companion commands and summons | Adds actors after objectives work |
| 5 | Stealth openings and boss phases | Adds encounter transitions after normal rounds are stable |
| 6 | Expanded AI goals | Scores every earlier mechanic through one legal-action path |
| 7 | Combat simulation reports | Measures balance before release |

Do not start checkpoint 4 while reaction prompts can duplicate or change after save and load.

## Checkpoint 1: Resolve Reactions without Nested Chaos

Create `storyforge-core/src/combat/reaction_window.rs`:

```rust
use serde::{Deserialize, Serialize};

use crate::{ActorId, CombatEvent, ContentId, EncounterId};

#[derive(Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize)]
pub enum ReactionWindowStatus {
    Open,
    Declined,
    Resolved,
    Expired,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct ReactionOption {
    pub actor: ActorId,
    pub ability: ContentId,
    pub target: Option<ActorId>,
    pub priority: i16,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct ReactionWindow {
    pub id: u64,
    pub encounter: EncounterId,
    pub trigger_event_index: usize,
    pub trigger: ContentId,
    pub options: Vec<ReactionOption>,
    pub status: ReactionWindowStatus,
}

#[derive(Debug, Clone, PartialEq, Eq, thiserror::Error)]
pub enum ReactionError {
    #[error("reaction window is no longer open")]
    Closed,
    #[error("reaction option is not legal for this window")]
    IllegalOption,
}

pub fn choose_reaction(
    window: &ReactionWindow,
    actor: ActorId,
    ability: &ContentId,
    target: Option<ActorId>,
) -> Result<ReactionOption, ReactionError> {
    if window.status != ReactionWindowStatus::Open {
        return Err(ReactionError::Closed);
    }

    window
        .options
        .iter()
        .find(|option| {
            option.actor == actor
                && &option.ability == ability
                && option.target == target
        })
        .cloned()
        .ok_or(ReactionError::IllegalOption)
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct PendingCombatResolution {
    pub triggering_event: CombatEvent,
    pub reaction_window: Option<ReactionWindow>,
    pub continuation_events: Vec<CombatEvent>,
}
```

Resolution order:

1. Validate the original command.
2. Roll and create its triggering event.
3. Find legal reaction definitions from the public combat state.
4. If none exist, apply the trigger and continuation events.
5. If reactions exist, store `PendingCombatResolution`.
6. Show the trigger in the combat log.
7. Ask the player or AI to choose one legal reaction.
8. Resolve that reaction.
9. Modify, cancel, or continue the pending effect as authored.
10. Apply the final event batch and close the window.

Only one reaction window is open at a time in version 1.3. A reaction may produce ordinary events, but it cannot open another reaction window. This rule prevents an unreadable stack of counter-counter-reactions.

### Counterspell Behavior

Create a typed counter definition:

```rust
#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct CounterspellDefinition {
    pub id: ContentId,
    pub counter_tags: Vec<ContentId>,
    pub range: crate::DistanceBand,
    pub ability: crate::Ability,
    pub base_difficulty: i16,
    pub spends_slot_level: Option<u8>,
    pub on_success: CounterspellSuccess,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize)]
pub enum CounterspellSuccess {
    CancelSpell,
    ReduceEffect,
    RedirectToCaster,
}
```

Counter validation checks:

- The reactor can perceive the cast.
- Range and line of sight pass.
- The reactor has a reaction.
- The counter definition matches at least one spell tag.
- Any required slot, focus, or prepared counter exists.
- The reactor is able to act.

The triggering `SpellCast` event remains in the log. A successful cancellation adds `CounterResolved { success: true }` and omits the spell's effect events. A failed counter spends its declared resources and continues the stored effects.

## Checkpoint 2: Add Terrain without a Coordinate Simulator

The terminal needs readable tactical places, not hidden floating-point geometry. Use named zones and explicit visibility links.

Create `storyforge-core/src/combat/terrain.rs`:

```rust
use std::collections::BTreeMap;

use serde::{Deserialize, Serialize};

use crate::{ActorId, ContentId};

#[derive(Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize)]
pub enum CoverLevel {
    None,
    Partial,
    Full,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct TerrainZoneDefinition {
    pub id: ContentId,
    pub name: String,
    pub elevation: i8,
    pub tags: Vec<String>,
    pub capacity: u8,
    pub hazards: Vec<ContentId>,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct TerrainConnection {
    pub from: ContentId,
    pub to: ContentId,
    pub movement_cost: u8,
    pub bidirectional: bool,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct VisibilityLink {
    pub from: ContentId,
    pub to: ContentId,
    pub cover: CoverLevel,
    pub blocks_line_of_sight: bool,
    pub bidirectional: bool,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct TacticalMapDefinition {
    pub id: ContentId,
    pub zones: BTreeMap<ContentId, TerrainZoneDefinition>,
    pub connections: Vec<TerrainConnection>,
    pub visibility: Vec<VisibilityLink>,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct TacticalPositionState {
    pub actors: BTreeMap<ActorId, ContentId>,
}

#[derive(Debug, Clone, PartialEq, Eq)]
pub struct TargetingView {
    pub line_of_sight: bool,
    pub cover: CoverLevel,
    pub elevation_difference: i8,
}

#[must_use]
pub fn targeting_view(
    map: &TacticalMapDefinition,
    positions: &TacticalPositionState,
    attacker: ActorId,
    target: ActorId,
) -> Option<TargetingView> {
    let attacker_zone = positions.actors.get(&attacker)?;
    let target_zone = positions.actors.get(&target)?;
    if attacker_zone == target_zone {
        return Some(TargetingView {
            line_of_sight: true,
            cover: CoverLevel::None,
            elevation_difference: 0,
        });
    }

    let link = map.visibility.iter().find(|link| {
        (&link.from == attacker_zone && &link.to == target_zone)
            || (link.bidirectional
                && &link.to == attacker_zone
                && &link.from == target_zone)
    })?;
    let attacker_elevation = map.zones.get(attacker_zone)?.elevation;
    let target_elevation = map.zones.get(target_zone)?.elevation;

    Some(TargetingView {
        line_of_sight: !link.blocks_line_of_sight,
        cover: link.cover,
        elevation_difference: attacker_elevation.saturating_sub(target_elevation),
    })
}
```

If no visibility link exists, the target is not visible. Content authors must declare links in both directions or mark them bidirectional.

Terrain effects use the existing effect system:

```rust
#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct TerrainRuleDefinition {
    pub required_zone_tags: Vec<String>,
    pub attack_modifier: i8,
    pub save_modifier: i8,
    pub movement_modifier: i8,
    pub public_reason: String,
}
```

Examples:

- High ground grants an authored attack modifier.
- Slippery tiles increase movement cost.
- Dense shelves provide partial cover.
- A warded dais blocks line of sight until disabled.
- Falling glass activates at a stable initiative entry.

The targeting preview prints every relevant reason before confirmation.

## Checkpoint 3: Support Nonlethal Objectives and Morale

Create `storyforge-core/src/combat/objective.rs`:

```rust
use serde::{Deserialize, Serialize};

use crate::{ActorId, ConditionDefinition, ContentId, EffectDefinition};

#[derive(Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize)]
pub enum EncounterObjectiveKind {
    Defeat,
    SurviveRounds,
    ReachZone,
    ProtectActor,
    ProtectObject,
    DisableTargets,
    RetrieveItem,
    CompleteRitual,
    ForceSurrender,
    Escape,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize)]
pub enum ObjectiveRequirement {
    Required,
    Optional,
    Failure,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct EncounterObjectiveDefinition {
    pub id: ContentId,
    pub kind: EncounterObjectiveKind,
    pub requirement: ObjectiveRequirement,
    pub text: String,
    pub complete_when: Vec<ConditionDefinition>,
    pub fail_when: Vec<ConditionDefinition>,
    pub success_effects: Vec<EffectDefinition>,
    pub failure_effects: Vec<EffectDefinition>,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct MoraleDefinition {
    pub actor_definition: ContentId,
    pub maximum: i16,
    pub surrender_threshold: i16,
    pub flee_threshold: i16,
    pub immune_tags: Vec<ContentId>,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct MoraleState {
    pub actor: ActorId,
    pub current: i16,
    pub offered_surrender: bool,
}
```

After each event batch:

1. Evaluate required objective completion.
2. Evaluate failure objectives.
3. Apply morale effects caused by damage, ally defeat, intimidation, or objective loss.
4. Offer surrender or escape when a threshold is crossed.
5. End the encounter once through the highest-priority terminal outcome.

Terminal priority:

1. Authored catastrophic failure.
2. Required nonlethal objective success.
3. Accepted surrender.
4. Escape.
5. Defeat-all victory.
6. Player defeat.

An encounter definition may override this order, but validation requires one unambiguous rule.

Accepting surrender can create prisoners, information, faction reaction, mercy memories, or future betrayal. Refusing surrender is also an explicit event that factions and companions may remember.

## Checkpoint 4: Add Companions and Summons

Companion commands from chapters 15 and 16 continue to use the same `CombatCommand` path as the player.

Summons need separate ownership and lifetime:

```rust
#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct SummonDefinition {
    pub id: ContentId,
    pub actor_definition: ContentId,
    pub maximum_active_per_caster: u8,
    pub duration_rounds: Option<u16>,
    pub concentration: bool,
    pub control: SummonControl,
    pub dismissed_at_zero_hp: bool,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize)]
pub enum SummonControl {
    Direct,
    ActsAfterCaster,
    Autonomous,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct SummonState {
    pub actor: ActorId,
    pub definition: ContentId,
    pub caster: ActorId,
    pub remaining_rounds: Option<u16>,
    pub control: SummonControl,
}
```

Summon validation:

- Definition and actor template exist.
- Caster is allowed to summon it.
- Per-caster and encounter actor caps pass.
- Spawn zone has capacity.
- Concentration rules pass.
- The summon cannot recursively summon itself unless explicitly permitted.

When concentration ends, duration expires, caster dismisses it, or a dismissal condition occurs, emit `SummonDismissed`. Do not leave its hazards, concentration, or pending turn in initiative.

Companion and summon actions display who controls them:

```text
Iren                 Suggested
Ash Owl              Acts after Mara
Chalk Guardian       Autonomous, 3 rounds
```

## Checkpoint 5: Resolve Stealth Openings and Boss Phases

Create `storyforge-core/src/combat/opening.rs`:

```rust
use serde::{Deserialize, Serialize};

use crate::{CheckDefinition, ContentId, EffectDefinition};

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct StealthOpeningDefinition {
    pub approach_check: CheckDefinition,
    pub detection_check: CheckDefinition,
    pub success_position: ContentId,
    pub failure_position: ContentId,
    pub success_effects: Vec<EffectDefinition>,
    pub failure_effects: Vec<EffectDefinition>,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum StealthOpeningOutcome {
    Undetected,
    Partial,
    Detected,
}

#[must_use]
pub fn stealth_opening_outcome(
    approach_total: i16,
    detection_total: i16,
) -> StealthOpeningOutcome {
    let margin = approach_total.saturating_sub(detection_total);
    match margin {
        5.. => StealthOpeningOutcome::Undetected,
        0..=4 => StealthOpeningOutcome::Partial,
        _ => StealthOpeningOutcome::Detected,
    }
}
```

An undetected opening may grant position, information, or one prepared action. It does not grant an unlimited surprise round. A partial result gives a smaller benefit and preserves enemy agency.

Boss phases use chapter 16's definition. Complete the transition contract:

1. Evaluate phase conditions after an event batch.
2. Sort legal phases by authored priority then stable ID.
3. Enter at most one phase.
4. Emit `BossPhaseChanged`.
5. Apply entry effects.
6. Add and remove actions.
7. Replace art and intent text.
8. Start the next normal resolution step.

Phase changes do not refill HP, spell slots, reactions, or conditions unless an entry effect explicitly says so.

## Checkpoint 6: Give AI Visible Goals

Create `storyforge-core/src/combat/ai_goal.rs`:

```rust
use serde::{Deserialize, Serialize};

use crate::ContentId;

#[derive(Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize)]
pub enum AiGoalKind {
    DealDamage,
    Survive,
    Protect,
    ReachZone,
    ControlZone,
    DisableTarget,
    CompleteObjective,
    Escape,
    ForceSurrender,
    ConserveResources,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct AiGoalDefinition {
    pub id: ContentId,
    pub kind: AiGoalKind,
    pub weight: i16,
    pub target: Option<ContentId>,
    pub active_when: Vec<crate::ConditionDefinition>,
}

#[derive(Debug, Clone, PartialEq, Eq)]
pub struct AiScoreBreakdown {
    pub action_label: String,
    pub goal_scores: Vec<(ContentId, i32)>,
    pub survival: i32,
    pub resource_cost: i32,
    pub personality: i32,
}

impl AiScoreBreakdown {
    #[must_use]
    pub fn total(&self) -> i32 {
        let goals = self
            .goal_scores
            .iter()
            .map(|(_, score)| *score)
            .fold(0_i32, i32::saturating_add);
        goals
            .saturating_add(self.survival)
            .saturating_add(self.personality)
            .saturating_sub(self.resource_cost.max(0))
    }
}
```

AI scoring order:

1. Generate legal `CombatCommand` values only.
2. Simulate their immediate public consequences without RNG.
3. Score each active goal.
4. Add survival, personality, and resource-use scores.
5. Select the highest total.
6. Break ties with encounter RNG.
7. Dispatch the chosen command through the normal handler.

AI may read:

- Public battlefield and objective state.
- Its own abilities, resources, memories, and goals.
- Player actions it observed.
- Information granted by its definition.

AI may not read:

- Hidden inventory.
- Uncast player spells.
- Secret quest flags.
- Future RNG.
- Dialogue-only secrets.

The examine panel summarizes intent:

```text
Gate Warden intends to protect the ritual circle.
Likely actions: move to the dais, block a route, or restrain an intruder.
```

It does not reveal the exact score or selected random tie.

## Checkpoint 7: Produce Combat Simulation Reports

Add a development-only simulation module. It calls the real command handler with fixed fixtures and a range of seeds.

Create `tools/storyforge-sim/src/lib.rs`:

```rust
use std::collections::BTreeMap;

use serde::Serialize;
use storyforge_core::ContentId;

#[derive(Debug, Clone, PartialEq, Eq)]
pub struct SimulationRequest {
    pub encounter: ContentId,
    pub first_seed: u64,
    pub runs: u32,
    pub maximum_rounds: u32,
}

#[derive(Debug, Clone, PartialEq, Eq)]
pub struct SimulationOutcome {
    pub victory: bool,
    pub rounds: u32,
    pub player_hp_remaining: i32,
    pub normal_slots_spent: u16,
    pub temporary_slots_spent: u16,
    pub sorcery_points_spent: u16,
    pub reactions_used: u16,
    pub objectives_completed: Vec<ContentId>,
    pub terminal_reason: ContentId,
}

pub trait EncounterSimulator {
    type Error: std::error::Error + Send + Sync + 'static;

    fn run(
        &self,
        encounter: &ContentId,
        seed: u64,
        maximum_rounds: u32,
    ) -> Result<SimulationOutcome, Self::Error>;
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize)]
pub struct SimulationReport {
    pub encounter: String,
    pub runs: u32,
    pub victories: u32,
    pub defeats: u32,
    pub stalled: u32,
    pub minimum_rounds: u32,
    pub maximum_rounds: u32,
    pub total_rounds: u64,
    pub total_player_hp_remaining: i64,
    pub total_slots_spent: u64,
    pub total_sorcery_points_spent: u64,
    pub total_reactions_used: u64,
    pub terminal_reasons: BTreeMap<String, u32>,
}

#[derive(Debug, thiserror::Error)]
pub enum SimulationError<E> {
    #[error("simulation requires at least one run")]
    ZeroRuns,
    #[error("encounter simulation failed")]
    Run(#[source] E),
}

pub fn simulate<S: EncounterSimulator>(
    simulator: &S,
    request: &SimulationRequest,
) -> Result<SimulationReport, SimulationError<S::Error>> {
    if request.runs == 0 {
        return Err(SimulationError::ZeroRuns);
    }

    let mut report = SimulationReport {
        encounter: request.encounter.to_string(),
        runs: request.runs,
        victories: 0,
        defeats: 0,
        stalled: 0,
        minimum_rounds: u32::MAX,
        maximum_rounds: 0,
        total_rounds: 0,
        total_player_hp_remaining: 0,
        total_slots_spent: 0,
        total_sorcery_points_spent: 0,
        total_reactions_used: 0,
        terminal_reasons: BTreeMap::new(),
    };

    for offset in 0..request.runs {
        let seed = request.first_seed.saturating_add(u64::from(offset));
        let outcome = simulator
            .run(&request.encounter, seed, request.maximum_rounds)
            .map_err(SimulationError::Run)?;
        if outcome.rounds >= request.maximum_rounds {
            report.stalled = report.stalled.saturating_add(1);
        } else if outcome.victory {
            report.victories = report.victories.saturating_add(1);
        } else {
            report.defeats = report.defeats.saturating_add(1);
        }
        report.minimum_rounds = report.minimum_rounds.min(outcome.rounds);
        report.maximum_rounds = report.maximum_rounds.max(outcome.rounds);
        report.total_rounds =
            report.total_rounds.saturating_add(u64::from(outcome.rounds));
        report.total_player_hp_remaining = report
            .total_player_hp_remaining
            .saturating_add(i64::from(outcome.player_hp_remaining));
        let slots = u64::from(outcome.normal_slots_spent)
            .saturating_add(u64::from(outcome.temporary_slots_spent));
        report.total_slots_spent =
            report.total_slots_spent.saturating_add(slots);
        report.total_sorcery_points_spent = report
            .total_sorcery_points_spent
            .saturating_add(u64::from(outcome.sorcery_points_spent));
        report.total_reactions_used = report
            .total_reactions_used
            .saturating_add(u64::from(outcome.reactions_used));
        let reason_count = report
            .terminal_reasons
            .entry(outcome.terminal_reason.to_string())
            .or_default();
        *reason_count = reason_count.saturating_add(1);
    }

    Ok(report)
}
```

Keep totals in the serialized report so consumers can calculate averages without losing precision. The terminal renderer may show rounded averages.

Add the command:

```powershell
storyforge simulate combat academy.encounter.gate-warden `
    --runs 1000 `
    --first-seed 1 `
    --max-rounds 30 `
    --output reports\gate-warden.json
```

The report includes:

```text
Runs: 1000
Player victories: 612
Player defeats: 371
Stalled: 17
Rounds: min 3, average 7.8, max 30
Average slots spent: 1.6
Average sorcery points spent: 0.8
Average reactions used: 1.2
Most common terminal reason: ritual disrupted
```

Simulation is a balance tool, not proof that an encounter is fun. Manually play representative winning, losing, surrender, stealth, and objective routes.

## Original Version 1.3 Encounter

Build one `Shattered Observatory` fixture:

- Five named terrain zones.
- Partial cover between broken instruments.
- One unstable-lens hazard.
- One player companion.
- One optional summon fixture.
- Required objective: align two mirrors.
- Optional objective: protect the trapped groundskeeper.
- Enemy objective: complete a three-step ritual.
- One enemy can surrender.
- Boss changes phase when the first mirror aligns.
- Stealth opening can secure high ground.
- Victory does not require defeating every enemy.

This single encounter exercises every version 1.3 system without becoming a full campaign battle.

## TUI Behavior

The tactical panel shows named zones:

```text
SHATTERED OBSERVATORY

[West Gallery] -- [Lens Floor] -- [Ritual Dais]
       |               |
[Upper Walk]     [Collapsed Stairs]

Mara: Upper Walk       Partial cover
Iren: West Gallery
Warden: Ritual Dais    Intent: continue ritual

Objectives
[ ] Align two mirrors                 0 / 2
[ ] Protect Groundskeeper Pell        Optional
Enemy ritual                          1 / 3
```

A reaction prompt preserves context:

```text
The Warden begins Ashen Bind against Iren.

REACTION
1 Counter with Severing Sign       1st-level slot
2 Iren uses Ward                   Reaction
3 Decline
```

The player can inspect range, line of sight, cover, cost, and predicted rule effect before choosing.

## Validator Additions

| Code | Invalid content |
| --- | --- |
| `TAC001` | Reaction trigger or continuation is unsupported |
| `TAC002` | Counter tag, spell, slot, or success behavior is invalid |
| `TAC003` | Terrain connection references an unknown zone |
| `TAC004` | Visibility link is missing for a required targeting fixture |
| `TAC005` | Zone capacity cannot hold required starting actors |
| `TAC006` | Objective has no achievable fixture |
| `TAC007` | Summon cap is zero or recursive summon has no limit |
| `TAC008` | Stealth opening position does not exist |
| `TAC009` | Boss phases have an ambiguous priority or cycle |
| `TAC010` | AI goal target does not exist |
| `TAC011` | Encounter has no terminal outcome |
| `TAC012` | Simulation fixture exceeds its round limit in every seed |

## Automated Test Checklist

1. A reaction window accepts only one legal option.
2. A stale prompt ID changes nothing.
3. A failed counter spends its declared resource and continues the spell.
4. A successful counter cancels only the effect events.
5. Missing visibility link blocks targeting.
6. Partial cover appears in the attack calculation and log.
7. Hazard timing is stable across replay.
8. Nonlethal objective can end combat before all enemies reach zero HP.
9. Surrender triggers at the exact morale threshold.
10. Refusing surrender creates a structured event.
11. Summon cap and zone capacity are enforced.
12. Dismissal removes summon initiative and effects.
13. Stealth margin selects undetected, partial, or detected.
14. Boss phase enters once and does not silently refill resources.
15. Companion direct, suggested, and autonomous control use normal commands.
16. AI candidate commands are legal and use no hidden player knowledge.
17. Same fixture and seed produce the same simulation outcome.
18. Simulation classifies round-limit runs as stalled.
19. Report totals match individual outcomes.
20. Save and load preserve open reaction, terrain position, objectives, summons, morale, and phase.

## Manual Playthrough

1. Enter the observatory through the direct route.
2. Use partial cover and inspect its modifier.
3. Trigger, decline, and then use a reaction.
4. Counter one spell successfully and fail another.
5. Disable the lens hazard.
6. Win by aligning mirrors without defeating every enemy.
7. Replay and accept an enemy surrender.
8. Replay with a stealth opening and compare starting positions.
9. Summon an ally, then break concentration.
10. Command a companion in all three control modes.
11. Trigger the boss phase and inspect the event log.
12. Run 1,000 simulation seeds and investigate stalled outliers.

## Full Verification

```powershell
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features --locked -- -D warnings
cargo test --workspace --all-features --locked
cargo run -p storyforge-tui -- validate --pack campaigns/academy-demo
cargo run -p storyforge-tui -- simulate combat academy.encounter.shattered-observatory --runs 1000 --first-seed 1 --max-rounds 30
```

## Common Mistakes

- Reactions can recursively open more reaction prompts.
- Counterspell erases the original cast from the log.
- Terrain uses hidden coordinates the TUI cannot explain.
- Cover modifies a roll without naming its source.
- Every encounter still ends only at zero HP.
- Summons leave turns behind after dismissal.
- Stealth grants a full free round with no authored limit.
- Boss phases silently refill resources.
- AI reads uncast spells or hidden inventory.
- Simulation uses a simplified combat path.
- A good win rate is treated as proof that the encounter is fun.

## Acceptance Check

- Every tactical rule appears in preview, log, save state, AI, and validation.
- Reactions resolve through one bounded window.
- Terrain uses named zones and explicit visibility.
- Nonlethal objectives, morale, and surrender can end combat.
- Companions and summons use the normal command path.
- Stealth and boss phases make explicit state transitions.
- AI scores visible goals without hidden knowledge.
- Simulation reports run the real engine deterministically.

## Suggested Commit

```powershell
git add .
git commit -m "Ship version 1.3 deeper tactics"
```
