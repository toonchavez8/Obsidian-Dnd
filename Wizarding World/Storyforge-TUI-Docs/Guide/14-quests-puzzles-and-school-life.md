# 14: Add quests, puzzles, and school life

## Result

The player can track multi-objective quests, attend or skip scheduled classes, earn cohort points, solve a stateful puzzle, and complete a small exam.

## Build this chapter in five checkpoints

| Checkpoint | Destination | Proof before continuing |
| --- | --- | --- |
| 1 | `storyforge-core/src/quest.rs` | Quest activation, objective, failure, and reward-once tests |
| 2 | `storyforge-core/src/school.rs` | Attendance, grace period, training, and point-ledger tests |
| 3 | `storyforge-core/src/puzzle.rs` | Submit, hint, bypass, reset, and save round-trip tests |
| 4 | `storyforge-content/src/model/` | Quest, class, puzzle, and exam validators pass |
| 5 | `storyforge-tui/src/screens/journal.rs` and `puzzle.rs` | Manual class, journal, puzzle, and exam playthrough passes |

Do not build the exam before a class and a puzzle work independently.

## Commands and events

Create `storyforge-core/src/quest_command.rs`:

```rust
use crate::ContentId;

#[derive(Debug, Clone, PartialEq, Eq)]
pub enum QuestSchoolCommand {
    StartQuest {
        quest: ContentId,
    },
    AbandonQuest {
        quest: ContentId,
    },
    ReevaluateQuests,
    RespondToClassInvitation {
        session: ContentId,
        response: AttendanceResponse,
    },
    SubmitPuzzle {
        puzzle: ContentId,
        answer: Vec<String>,
    },
    RequestPuzzleHint {
        puzzle: ContentId,
        method: HintMethod,
    },
    ResetPuzzle {
        puzzle: ContentId,
    },
    BypassPuzzleStep {
        puzzle: ContentId,
        step: usize,
        item: ContentId,
    },
    SubmitExamPreparation {
        exam: ContentId,
        choice: ContentId,
    },
}

#[derive(
    Debug, Clone, Copy, PartialEq, Eq, serde::Serialize, serde::Deserialize,
)]
pub enum AttendanceResponse {
    Attend,
    ArriveLate,
    Skip,
    UseExcuse,
}

#[derive(
    Debug, Clone, Copy, PartialEq, Eq, serde::Serialize, serde::Deserialize,
)]
pub enum HintMethod {
    Normal,
    Investigation,
    Arcana,
    Companion,
}

#[derive(
    Debug, Clone, PartialEq, Eq, serde::Serialize, serde::Deserialize,
)]
pub enum QuestSchoolEvent {
    QuestStarted {
        quest: ContentId,
    },
    ObjectiveStateChanged {
        quest: ContentId,
        objective: String,
        previous: ObjectiveState,
        current: ObjectiveState,
    },
    QuestStateChanged {
        quest: ContentId,
        previous: QuestState,
        current: QuestState,
    },
    QuestRewardApplied {
        quest: ContentId,
        reward_index: usize,
    },
    ClassAttendanceRecorded {
        session: ContentId,
        response: AttendanceResponse,
        minutes_late: u16,
    },
    TrainingProgressChanged {
        subject: ContentId,
        previous: u16,
        current: u16,
    },
    CohortPointsRecorded {
        cohort: ContentId,
        amount: i16,
        reason: ContentId,
    },
    PuzzleAttempted {
        puzzle: ContentId,
        correct: bool,
    },
    PuzzleHintRevealed {
        puzzle: ContentId,
        fact: ContentId,
        method: HintMethod,
    },
    PuzzleReset {
        puzzle: ContentId,
    },
    PuzzleStepBypassed {
        puzzle: ContentId,
        step: usize,
        item: ContentId,
    },
    ExamScored {
        exam: ContentId,
        attendance_points: i16,
        preparation_points: i16,
        check_points: i16,
        consequence_points: i16,
        total: i16,
    },
    QuestSchoolCommandRejected {
        reason: String,
    },
    AutosaveRequested {
        reason: String,
    },
}
```

Add `QuestSchool(QuestSchoolCommand),` to `GameCommand` and `QuestSchool(QuestSchoolEvent),` to `GameEvent`.

| Command | Validation | Successful behavior |
| --- | --- | --- |
| `StartQuest` | Definition exists, start conditions pass, state is hidden | Starts quest, evaluates initial objectives, journal event |
| `AbandonQuest` | Quest is active and abandonment is allowed | Fails or abandons active objectives, applies cleanup once |
| `ReevaluateQuests` | Always legal after direct command effects | Reaches a stable quest state or reports a content cycle |
| `RespondToClassInvitation` | Invitation is active; response and excuse are legal | Records attendance, advances time, applies attendance or absence effects |
| `SubmitPuzzle` | Puzzle is active; answer shape and symbols are legal | Records attempt, success effects or authored failure pressure |
| `RequestPuzzleHint` | Hint exists; skill or companion requirement passes; cost is available | Reveals one stable fact without changing seed |
| `ResetPuzzle` | Reset is allowed and its cost can be paid | Clears entered answer, preserves puzzle instance and seed |
| `BypassPuzzleStep` | Step is unresolved; item is owned and supports bypass | Consumes the item once and marks that step resolved |
| `SubmitExamPreparation` | Exam is active and choice is offered | Stores preparation, makes one final check, emits a complete score breakdown |

The handlers build all events before applying any effect. A rejected hint does not consume time; a rejected bypass does not consume the item; a repeated quest evaluation does not apply rewards twice.

## Quest definitions

```rust
pub struct QuestDefinition {
    pub id: ContentId,
    pub category: QuestCategory,
    pub title: String,
    pub summary: String,
    pub arc: ContentId,
    pub start_conditions: Vec<ConditionDefinition>,
    pub objectives: Vec<ObjectiveDefinition>,
    pub failure_conditions: Vec<ConditionDefinition>,
    pub rewards: Vec<EffectDefinition>,
}

pub struct ObjectiveDefinition {
    pub id: String,
    pub text: String,
    pub optional: bool,
    pub activate_when: Vec<ConditionDefinition>,
    pub complete_when: Vec<ConditionDefinition>,
    pub fail_when: Vec<ConditionDefinition>,
}
```

Runtime state:

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq, serde::Serialize, serde::Deserialize)]
pub enum ObjectiveState {
    Locked,
    Active,
    Completed,
    Failed,
    Abandoned,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq, serde::Serialize, serde::Deserialize)]
pub enum QuestState {
    Hidden,
    Active,
    Completed,
    Failed,
    Abandoned,
}
```

Objective IDs are stable within a quest. Saves use the pair of quest ID and objective ID.

## Quest evaluation

After each command's direct effects:

1. Check active objective completion.
2. Check active objective failure.
3. Activate newly unlocked objectives.
4. Determine quest terminal state.
5. Apply terminal rewards or cleanup once.
6. Emit journal events.

Use a loop with a strict iteration limit based on objective count. A quest whose effects repeatedly reactivate each other is a content error.

## Journal

Tabs:

- Main.
- Side.
- Companion.
- Faction.
- Class.
- Completed.

Show active objectives, optional marker, known deadline, region, and the last relevant event. Hidden objectives remain absent.

## Timed quests

Deadlines use `GameClock`. The journal shows:

```text
Due: Day 3 at 18:00
Time remaining: 1 day, 4 hours
```

Travel, wait, craft, and rest previews report if they cross a known deadline.

## Class definitions

A class is a scheduled scene or activity:

```rust
pub struct ClassSessionDefinition {
    pub id: ContentId,
    pub course: ContentId,
    pub start: ScheduleTime,
    pub duration_minutes: u32,
    pub location: ContentId,
    pub scene: ContentId,
    pub late_grace_minutes: u16,
    pub attendance_effects: Vec<EffectDefinition>,
    pub absence_effects: Vec<EffectDefinition>,
}
```

At class start, schedule an invitation event. The player may attend, arrive late, skip, or use a valid excuse effect.

## Learning

Classes award progress toward skills or spells:

```rust
pub struct TrainingProgress {
    pub subject: ContentId,
    pub points: u16,
    pub threshold: u16,
}
```

Crossing a threshold emits a learning event and unlocks the defined ability. Extra points carry only when the definition allows it.

## Cohort competition

Store every point change:

```rust
pub struct PointEntry {
    pub cohort: ContentId,
    pub amount: i16,
    pub reason: String,
    pub source_event: u64,
    pub day: u32,
}
```

Totals are derived from the ledger. Do not store a second mutable total.

The dashboard shows recent entries and ranking. Point changes have an authored reason.

## Puzzle interface

Puzzles implement a data-driven core kind rather than arbitrary code:

```rust
pub enum PuzzleKind {
    Sequence(SequencePuzzle),
    Matching(MatchingPuzzle),
    LogicGrid(LogicGridPuzzle),
    RuneTranslation(RunePuzzle),
}
```

Every puzzle supports:

- Serializable runtime state.
- Submit command.
- Reset command with authored cost.
- Hint command.
- Skill-assisted hint.
- Success effects.
- Bypass effects and cost.

## First puzzle

Create a three-rune sequence at the side passage:

- Four available rune commands.
- Three-step solution.
- Incorrect submission raises alarm pressure.
- Investigation reveals one position.
- Arcana reveals the ordering rule.
- A companion hint can reveal another position.
- The player can spend a consumable chalk to bypass one unknown position.

The seed and resulting sequence are stored when the puzzle instance is created.

## Exam

The first exam score combines:

- Attendance progress.
- Learned notes.
- One preparation choice.
- One final check.
- Optional help or cheating consequence.

Use explicit weighted integer points. Show the breakdown after the result.

## Validator additions

- Quest objective IDs unique.
- Main quests have a tested terminal route.
- Deadlines are after start conditions in fixtures.
- Class location and scene exist.
- Schedule overlaps are intentional.
- Training threshold greater than zero.
- Puzzle has at least one solution.
- Hints reference valid puzzle information.
- Cohort point reason is not empty.

## Tests

- Quest rewards apply once.
- Hidden quest can record pre-discovery progress.
- Timed quest fails at the exact boundary.
- Skipping class applies absence effect.
- Late arrival uses grace correctly.
- Point total equals ledger sum.
- Puzzle save round-trip keeps seed and input.
- Hint does not reroll puzzle.
- Exam breakdown totals correctly.

## Acceptance check

- One main and one side quest work.
- One class can be attended, missed, or joined late.
- Cohort points update with a visible reason.
- One puzzle supports solve, hint, and bypass.
- One exam reflects preparation and final check.
- Save/resume works during quest, class invitation, and puzzle.

## Suggested commit

```powershell
git add .
git commit -m "Add quests puzzles and school life"
```
