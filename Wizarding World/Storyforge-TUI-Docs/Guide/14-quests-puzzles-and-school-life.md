# 14: Add quests, puzzles, and school life

## Result

The player can track multi-objective quests, attend or skip scheduled classes, earn cohort points, solve a stateful puzzle, and complete a small exam.

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
pub enum ObjectiveState {
    Locked,
    Active,
    Completed,
    Failed,
    Abandoned,
}

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

