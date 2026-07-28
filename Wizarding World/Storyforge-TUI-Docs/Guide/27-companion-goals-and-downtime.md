# 27: Give companions lives between quests

## Result

Companions follow schedules, choose downtime activities, pursue long-running goals, bring back rumors or resources, and ask the player for help at meaningful stages. Their progress survives arc changes and time skips.

Companions do not become background resource generators. Their goals create scenes, choices, risks, and boundaries.

## Build this chapter in five checkpoints

| Checkpoint | Destination | Proof before continuing |
| --- | --- | --- |
| 1 | `storyforge-core/src/companion/goal.rs` | Goal stages progress once and wait for required scenes |
| 2 | `storyforge-core/src/companion/downtime.rs` | Activity requirements, time, outcome, and cap tests |
| 3 | `storyforge-core/src/companion/planner.rs` | Schedule conflicts and deterministic autonomous choices |
| 4 | `storyforge-content/src/model/companion.rs` | Goal, downtime, and fallback validation |
| 5 | `storyforge-tui/src/screens/companion_planner.rs` | Assignment, return summary, and interruption playthrough |

## Checkpoint 1: Model long-running goals

Create `storyforge-core/src/companion/goal.rs`:

```rust
use serde::{Deserialize, Serialize};

use crate::{ConditionDefinition, ContentId, EffectDefinition};

#[derive(Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize)]
pub enum CompanionGoalStatus {
    Locked,
    Active,
    WaitingForScene,
    Completed,
    Failed,
    Abandoned,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct CompanionGoalDefinition {
    pub id: ContentId,
    pub actor: ContentId,
    pub title: String,
    pub summary: String,
    pub start_conditions: Vec<ConditionDefinition>,
    pub stages: Vec<CompanionGoalStageDefinition>,
    pub completion_effects: Vec<EffectDefinition>,
    pub failure_effects: Vec<EffectDefinition>,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct CompanionGoalStageDefinition {
    pub id: String,
    pub text: String,
    pub required_progress: u16,
    pub progress_tags: Vec<ContentId>,
    pub player_help_scene: Option<ContentId>,
    pub deadline_day: Option<u32>,
    pub success_effects: Vec<EffectDefinition>,
    pub failure_effects: Vec<EffectDefinition>,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct CompanionGoalState {
    pub goal: ContentId,
    pub status: CompanionGoalStatus,
    pub stage_index: usize,
    pub progress: u16,
    pub applied_source_events: Vec<u64>,
    pub started_day: u32,
    pub last_progress_day: Option<u32>,
}

#[derive(Debug, Clone, PartialEq, Eq, thiserror::Error)]
pub enum CompanionGoalError {
    #[error("goal has no stages")]
    MissingStages,
    #[error("goal stage index is invalid")]
    InvalidStage,
    #[error("goal progress overflowed")]
    ProgressOverflow,
}

#[derive(Debug, Clone, PartialEq, Eq)]
pub enum GoalProgress {
    Unchanged,
    Progressed {
        previous: u16,
        current: u16,
    },
    WaitingForScene {
        scene: ContentId,
    },
    AdvancedStage {
        previous_stage: usize,
        current_stage: usize,
    },
    Completed,
}

pub fn apply_goal_progress(
    definition: &CompanionGoalDefinition,
    state: &CompanionGoalState,
    source_event: u64,
    event_tags: &[ContentId],
    amount: u16,
    current_day: u32,
) -> Result<(CompanionGoalState, GoalProgress), CompanionGoalError> {
    if definition.stages.is_empty() {
        return Err(CompanionGoalError::MissingStages);
    }
    if state.applied_source_events.contains(&source_event)
        || state.status != CompanionGoalStatus::Active
    {
        return Ok((state.clone(), GoalProgress::Unchanged));
    }

    let stage = definition
        .stages
        .get(state.stage_index)
        .ok_or(CompanionGoalError::InvalidStage)?;
    if !stage
        .progress_tags
        .iter()
        .any(|tag| event_tags.contains(tag))
    {
        return Ok((state.clone(), GoalProgress::Unchanged));
    }

    let mut next = state.clone();
    next.applied_source_events.push(source_event);
    next.last_progress_day = Some(current_day);
    let previous = next.progress;
    next.progress = next
        .progress
        .checked_add(amount)
        .ok_or(CompanionGoalError::ProgressOverflow)?
        .min(stage.required_progress);

    if next.progress < stage.required_progress {
        return Ok((
            next,
            GoalProgress::Progressed {
                previous,
                current: previous.saturating_add(amount).min(stage.required_progress),
            },
        ));
    }

    if let Some(scene) = &stage.player_help_scene {
        next.status = CompanionGoalStatus::WaitingForScene;
        return Ok((
            next,
            GoalProgress::WaitingForScene {
                scene: scene.clone(),
            },
        ));
    }

    let previous_stage = next.stage_index;
    next.stage_index = next.stage_index.saturating_add(1);
    next.progress = 0;
    if next.stage_index >= definition.stages.len() {
        next.status = CompanionGoalStatus::Completed;
        Ok((next, GoalProgress::Completed))
    } else {
        Ok((
            next,
            GoalProgress::AdvancedStage {
                previous_stage,
                current_stage: previous_stage.saturating_add(1),
            },
        ))
    }
}
```

A goal that reaches `WaitingForScene` cannot continue in the background. The player may help, refuse, postpone, or allow the companion to choose an autonomous route defined by content.

Examples:

- Research a family mystery over several terms.
- Earn a club leadership role.
- Repair a damaged relationship.
- Investigate a faction contact.
- Train a difficult spell.
- Save for equipment or a mount.
- Find evidence against a rival.

## Goal commands and events

Create `storyforge-core/src/companion/goal_command.rs`:

```rust
use crate::ContentId;

use super::{CompanionGoalStatus, DowntimeAssignment};

#[derive(Debug, Clone, PartialEq, Eq)]
pub enum CompanionWorldCommand {
    StartCompanionGoal {
        actor: ContentId,
        goal: ContentId,
    },
    ApplyCompanionGoalProgress {
        actor: ContentId,
        goal: ContentId,
        source_event: u64,
        tags: Vec<ContentId>,
        amount: u16,
    },
    ResolveCompanionGoalScene {
        actor: ContentId,
        goal: ContentId,
        outcome: ContentId,
    },
    AssignCompanionDowntime {
        assignment: DowntimeAssignment,
    },
    ConfirmCompanionPlan {
        actor: ContentId,
    },
    ResolveCompanionDowntime {
        actor: ContentId,
        assignment: u64,
    },
    RecallCompanion {
        actor: ContentId,
    },
}

#[derive(
    Debug, Clone, PartialEq, Eq, serde::Serialize, serde::Deserialize,
)]
pub enum CompanionWorldEvent {
    CompanionGoalStarted {
        actor: ContentId,
        goal: ContentId,
    },
    CompanionGoalProgressed {
        actor: ContentId,
        goal: ContentId,
        previous: u16,
        current: u16,
    },
    CompanionGoalStatusChanged {
        actor: ContentId,
        goal: ContentId,
        previous: CompanionGoalStatus,
        current: CompanionGoalStatus,
    },
    CompanionGoalSceneRequested {
        actor: ContentId,
        goal: ContentId,
        scene: ContentId,
    },
    CompanionDowntimeAssigned {
        assignment: DowntimeAssignment,
    },
    CompanionDowntimeResolved {
        actor: ContentId,
        assignment: u64,
        outcome: ContentId,
    },
    CompanionReturned {
        actor: ContentId,
        summary_scene: ContentId,
    },
    CompanionWorldCommandRejected {
        reason: String,
    },
    AutosaveRequested {
        reason: String,
    },
}
```

Add `CompanionWorld(CompanionWorldCommand),` to `GameCommand` and `CompanionWorld(CompanionWorldEvent),` to `GameEvent`.

## Checkpoint 2: Define downtime activities

Create `storyforge-core/src/companion/downtime.rs`:

```rust
use serde::{Deserialize, Serialize};

use crate::{
    CheckDefinition, ConditionDefinition, ContentId, EffectDefinition,
};

#[derive(Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize)]
pub enum DowntimeActivityKind {
    Rest,
    Study,
    Train,
    Work,
    Gather,
    Investigate,
    Socialize,
    PursueGoal,
    CareForFamiliar,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct DowntimeActivityDefinition {
    pub id: ContentId,
    pub title: String,
    pub kind: DowntimeActivityKind,
    pub duration_minutes: u32,
    pub location: ContentId,
    pub conditions: Vec<ConditionDefinition>,
    pub check: Option<CheckDefinition>,
    pub success_effects: Vec<EffectDefinition>,
    pub failure_effects: Vec<EffectDefinition>,
    pub maximum_repeats_per_week: u8,
    pub risk_tags: Vec<ContentId>,
    pub return_scene: ContentId,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct DowntimeAssignment {
    pub id: u64,
    pub actor: ContentId,
    pub activity: ContentId,
    pub starts_day: u32,
    pub starts_minute: u16,
    pub ends_day: u32,
    pub ends_minute: u16,
    pub goal: Option<ContentId>,
    pub status: DowntimeStatus,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize)]
pub enum DowntimeStatus {
    Planned,
    Active,
    Completed,
    Interrupted,
    Failed,
}

#[derive(Debug, Clone, PartialEq, Eq, thiserror::Error)]
pub enum DowntimeError {
    #[error("downtime duration must be greater than zero")]
    ZeroDuration,
    #[error("downtime date overflowed")]
    DateOverflow,
}

pub fn create_assignment(
    id: u64,
    actor: ContentId,
    activity: &DowntimeActivityDefinition,
    start_day: u32,
    start_minute: u16,
    goal: Option<ContentId>,
) -> Result<DowntimeAssignment, DowntimeError> {
    if activity.duration_minutes == 0 {
        return Err(DowntimeError::ZeroDuration);
    }

    let start = u64::from(start_day) * 1440 + u64::from(start_minute);
    let end = start
        .checked_add(u64::from(activity.duration_minutes))
        .ok_or(DowntimeError::DateOverflow)?;

    Ok(DowntimeAssignment {
        id,
        actor,
        activity: activity.id.clone(),
        starts_day: start_day,
        starts_minute: start_minute,
        ends_day: u32::try_from(end / 1440)
            .map_err(|_| DowntimeError::DateOverflow)?,
        ends_minute: u16::try_from(end % 1440)
            .map_err(|_| DowntimeError::DateOverflow)?,
        goal,
        status: DowntimeStatus::Planned,
    })
}
```

Downtime resolves once at its end boundary:

1. Confirm the companion remained available.
2. Confirm the location and world phase still permit the activity.
3. Read the stored assignment and activity definition.
4. Roll the optional check through engine RNG.
5. Apply success or failure effects.
6. Apply goal progress from explicit tags.
7. Add earned rumor knowledge through chapter 25 events.
8. Add stress, recovery, relationship, items, or currency.
9. Request the return-summary scene.
10. Mark the assignment completed and autosave.

Do not run one update per downtime minute.

## Checkpoint 3: Plan around schedules

Create `storyforge-core/src/companion/planner.rs`:

```rust
use crate::ContentId;

use super::DowntimeAssignment;

#[derive(Debug, Clone, PartialEq, Eq)]
pub struct ScheduleConflict {
    pub assignment: u64,
    pub conflicting_activity: ContentId,
    pub overlap_minutes: u32,
}

#[must_use]
pub fn assignments_overlap(
    left: &DowntimeAssignment,
    right_start_day: u32,
    right_start_minute: u16,
    right_end_day: u32,
    right_end_minute: u16,
) -> bool {
    let left_start =
        u64::from(left.starts_day) * 1440 + u64::from(left.starts_minute);
    let left_end =
        u64::from(left.ends_day) * 1440 + u64::from(left.ends_minute);
    let right_start =
        u64::from(right_start_day) * 1440 + u64::from(right_start_minute);
    let right_end =
        u64::from(right_end_day) * 1440 + u64::from(right_end_minute);
    left_start < right_end && right_start < left_end
}
```

Validation before assignment:

- Companion is recruited and not in the active party.
- Activity conditions pass.
- Companion schedule has an open window.
- No class, club, personal scene, injury care, or prior assignment overlaps.
- Weekly repeat cap is not reached.
- Required tool, location, faction access, and travel time are available.
- A `PursueGoal` activity names an active goal.

Companion control determines who chooses:

| Control | Downtime behavior |
| --- | --- |
| Direct | Player selects an activity and sees every known requirement |
| Suggested | Engine offers up to three legal activities with plain reasons |
| Autonomous | Companion chooses by personality, current goal, stress, and promises |

Autonomous choice uses scored legal candidates and deterministic tie-breaking. It cannot spend a unique player item, accept a dangerous faction mission, abandon a goal, or cross a stated boundary without an authored permission.

## Checkpoint 4: Handle interruption and recall

Recall is not teleportation:

1. Check whether the companion can leave.
2. Calculate travel time from their scheduled location.
3. Preview the interrupted activity and lost progress.
4. Confirm.
5. Mark assignment `Interrupted`.
6. Apply only authored partial effects.
7. Schedule the companion's return.

Emergency story events may interrupt automatically, but must emit the same events and show the reason.

If the player starts a major quest while a companion is away:

- The party screen shows their return time.
- Required-companion quests can wait, offer a recall, or provide another route.
- The main story cannot assume every recruited companion is physically present.

## Checkpoint 5: Connect goals across arcs

At an arc transition:

1. Resolve completed assignments.
2. Pause assignments that cannot cross the transition.
3. Evaluate goal stage deadlines.
4. Request mandatory waiting scenes.
5. Apply time-skip variants.
6. Preserve active goals and source-event history.
7. Migrate unavailable locations to authored alternatives.
8. Record an arc-transition summary for each companion.

Each goal stage can define a transition fallback:

```rust
#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct GoalTransitionRule {
    pub from_arc: ContentId,
    pub to_arc: ContentId,
    pub incomplete_outcome: ContentId,
    pub replacement_location: Option<ContentId>,
    pub effects: Vec<EffectDefinition>,
}
```

A long-running goal may succeed, fail, change direction, or remain unresolved. It should not silently vanish when the campaign changes location packs.

## Original demo goal

Create `campaigns/academy-demo/companions/goals/iren-bell-history.ron`:

```ron
(
    id: "academy.companion-goal.iren-bell-history",
    actor: "academy.npc.iren",
    title: "A Bell Without a Voice",
    summary: "Iren wants to learn why the old-road bell no longer rings.",
    start_conditions: [
        FlagSet("academy.flag.silent-bell-found"),
    ],
    stages: [
        (
            id: "collect-accounts",
            text: "Collect two independent accounts of the bell.",
            required_progress: 2,
            progress_tags: ["academy.progress.bell-account"],
            player_help_scene: None,
            deadline_day: None,
            success_effects: [],
            failure_effects: [],
        ),
        (
            id: "inspect-ward",
            text: "Help Iren inspect the failed ward.",
            required_progress: 1,
            progress_tags: ["academy.progress.bell-ward-found"],
            player_help_scene: Some("academy.scene.iren-inspects-bell"),
            deadline_day: Some(20),
            success_effects: [],
            failure_effects: [],
        ),
        (
            id: "choose-disclosure",
            text: "Decide who should receive the recovered history.",
            required_progress: 1,
            progress_tags: ["academy.progress.bell-truth-ready"],
            player_help_scene: Some("academy.scene.iren-bell-disclosure"),
            deadline_day: None,
            success_effects: [],
            failure_effects: [],
        ),
    ],
    completion_effects: [
        SetFlag("academy.flag.bell-history-resolved"),
    ],
    failure_effects: [],
)
```

The companion can gather one account through downtime. The player must find or unlock another source, then participate in the ward and disclosure scenes.

## TUI behavior

The companion planner shows:

```text
IREN - DOWNTIME

Monday 14:00-17:00

> Research the silent bell       Goal progress
  Attend Glasswing Club          Relationship opportunity
  Recover in the infirmary       Stress -20
  Help at the Brass Trunk        Earn 8 sparks

Conflict: Practical Runes begins at 16:30
Available window: 150 minutes
```

The return summary shows authored outcomes:

```text
IREN RETURNED

Iren found a groundskeeper who remembers the old bell.

Goal: A Bell Without a Voice  1 / 2 accounts
Rumor learned: "The bell was silenced before the ward failed."
Stress: +4

Continue
```

## Validator additions

| Code | Invalid content |
| --- | --- |
| `COM001` | Goal has no stages |
| `COM002` | Stage progress requirement is zero |
| `COM003` | Goal scene, tag, actor, or effect does not exist |
| `COM004` | Deadline occurs before the goal can start in fixtures |
| `COM005` | Downtime duration is zero |
| `COM006` | Downtime location, check, return scene, or risk tag is invalid |
| `COM007` | Weekly repeat cap is zero for a repeatable activity |
| `COM008` | Dangerous autonomous activity lacks permission conditions |
| `COM009` | Arc-crossing goal has no fallback |
| `COM010` | Main quest requires a companion who can be unavailable with no alternate route |

## Automated test checklist

1. A matching tag advances an active goal.
2. The same source event cannot advance twice.
3. A nonmatching tag changes nothing.
4. A required scene pauses background progress.
5. Completing the final stage applies completion once.
6. Goal deadline uses the exact day boundary.
7. Downtime assignment calculates cross-midnight end time.
8. Overlap detection catches partial and complete overlaps.
9. Recall applies interruption once and schedules travel.
10. Weekly activity cap blocks farming.
11. Autonomous choice uses only legal actions.
12. Downtime rumor enters chapter 25's knowledge state.
13. Arc transition preserves or resolves every active goal.
14. Save and load preserve assignments, goal source events, stress, and return summaries.

## Manual playthrough

1. Recruit Iren and discover the silent bell.
2. Assign Iren to research during a free schedule window.
3. Attempt an overlapping assignment and confirm it is rejected.
4. Advance time and view the return summary.
5. Confirm the goal gains one account and a rumor.
6. Assign Iren to the same activity until the weekly cap is reached.
7. Send Iren away, begin a party quest, and use recall.
8. Confirm travel time and interruption consequences.
9. Reach the player-help stage and confirm background progress pauses.
10. Save before an arc transition and verify the goal's transition rule.

## Full verification

```powershell
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features --locked -- -D warnings
cargo test --workspace --all-features --locked
cargo run -p storyforge-tui -- validate --pack campaigns/academy-demo
cargo run -p storyforge-tui -- play --pack campaigns/academy-demo --seed 97
```

## Common mistakes

- Companion progress runs once per simulated minute.
- Autonomous companions spend unique items or accept major quests.
- A waiting goal scene completes in the background.
- Recall teleports a companion to the player.
- The main story assumes every companion is available.
- Time skips delete incomplete goals.
- Downtime grants unlimited money, items, or relationship points.

## Acceptance check

- Companions have multi-stage goals with explicit progress sources.
- Downtime fits into schedules and resolves at one boundary.
- Companions can return with resources, rumors, stress, or goal progress.
- Required scenes stop autonomous background progression.
- Recall has time and interruption consequences.
- Arc transitions preserve or deliberately resolve active goals.
- The game remains playable when a companion is away.

## Suggested commit

```powershell
git add .
git commit -m "Add companion goals and downtime"
```
