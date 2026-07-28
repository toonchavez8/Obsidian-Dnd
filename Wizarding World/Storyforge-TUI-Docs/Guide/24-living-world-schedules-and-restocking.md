# 24: Build the living-world clock

## Result

NPCs move between authored locations, dialogue changes with time and calendar state, classes and clubs follow a school calendar, and shops or gathering nodes restock without simulating every missed minute.

This chapter extends the `GameClock`, NPC schedules, classes, and shops from chapters 12 through 15. It does not create a second clock.

## Build this chapter in five checkpoints

| Checkpoint | Destination | Proof before continuing |
| --- | --- | --- |
| 1 | `storyforge-core/src/simulation/schedule.rs` | NPC location changes at exact boundaries |
| 2 | `storyforge-core/src/simulation/calendar.rs` | School dates and activity occurrences are stable |
| 3 | `storyforge-core/src/simulation/dialogue_time.rs` | Time-aware dialogue conditions select the right variant |
| 4 | `storyforge-core/src/simulation/restock.rs` | Shops and resources restock once per due boundary |
| 5 | `storyforge-tui/src/screens/schedule.rs` | Planner, location panel, and closed-shop states work |

Create `storyforge-core/src/simulation/mod.rs`:

```rust
mod calendar;
mod dialogue_time;
mod processor;
mod restock;
mod schedule;

pub use calendar::{
    ActivityOccurrence, CalendarDate, CalendarDefinition, CalendarError,
    CalendarEventDefinition, SchoolActivityKind,
};
pub use dialogue_time::{DialogueTimeCondition, TimeOfDay};
pub use processor::{
    SimulationBoundary, SimulationBoundaryKind, SimulationError,
    process_due_boundaries,
};
pub use restock::{RestockPolicy, RestockState};
pub use schedule::{
    NpcScheduleDefinition, NpcScheduleSlot, ScheduleOverride, Weekday,
    resolve_npc_location,
};
```

Export the module from `storyforge-core/src/lib.rs`:

```rust
pub mod simulation;
```

## Checkpoint 1: Resolve scheduled NPC locations

Create `storyforge-core/src/simulation/schedule.rs`:

```rust
use serde::{Deserialize, Serialize};

use crate::{ContentId, GameClock, WorldCondition};

#[derive(
    Debug, Clone, Copy, PartialEq, Eq, PartialOrd, Ord, Hash, Serialize,
    Deserialize,
)]
pub enum Weekday {
    Monday,
    Tuesday,
    Wednesday,
    Thursday,
    Friday,
    Saturday,
    Sunday,
}

impl Weekday {
    #[must_use]
    pub const fn from_day(day: u32) -> Self {
        match day % 7 {
            0 => Self::Monday,
            1 => Self::Tuesday,
            2 => Self::Wednesday,
            3 => Self::Thursday,
            4 => Self::Friday,
            5 => Self::Saturday,
            _ => Self::Sunday,
        }
    }

    #[must_use]
    pub const fn day_offset(self) -> u64 {
        match self {
            Self::Monday => 0,
            Self::Tuesday => 1,
            Self::Wednesday => 2,
            Self::Thursday => 3,
            Self::Friday => 4,
            Self::Saturday => 5,
            Self::Sunday => 6,
        }
    }
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct NpcScheduleDefinition {
    pub actor: ContentId,
    pub default_location: ContentId,
    pub slots: Vec<NpcScheduleSlot>,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct NpcScheduleSlot {
    pub id: String,
    pub weekdays: Vec<Weekday>,
    pub start_minute: u16,
    pub end_minute: u16,
    pub location: ContentId,
    pub priority: i16,
    pub conditions: Vec<WorldCondition>,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct ScheduleOverride {
    pub location: ContentId,
    pub begins_day: u32,
    pub begins_minute: u16,
    pub ends_day: u32,
    pub ends_minute: u16,
    pub reason: ContentId,
}

impl ScheduleOverride {
    #[must_use]
    pub fn is_active(&self, clock: &GameClock) -> bool {
        let current = absolute_minute(clock.day, clock.minute_of_day);
        let start = absolute_minute(self.begins_day, self.begins_minute);
        let end = absolute_minute(self.ends_day, self.ends_minute);
        (start..end).contains(&current)
    }
}

#[must_use]
pub const fn absolute_minute(day: u32, minute_of_day: u16) -> u64 {
    u64::from(day) * 1440 + minute_of_day as u64
}

#[must_use]
pub fn resolve_npc_location<F>(
    definition: &NpcScheduleDefinition,
    schedule_override: Option<&ScheduleOverride>,
    clock: &GameClock,
    condition_is_met: F,
) -> ContentId
where
    F: Fn(&WorldCondition) -> bool,
{
    if let Some(schedule_override) = schedule_override
        && schedule_override.is_active(clock)
    {
        return schedule_override.location.clone();
    }

    let weekday = Weekday::from_day(clock.day);
    definition
        .slots
        .iter()
        .filter(|slot| {
            slot.weekdays.contains(&weekday)
                && (slot.start_minute..slot.end_minute)
                    .contains(&clock.minute_of_day)
                && slot.conditions.iter().all(&condition_is_met)
        })
        .max_by(|left, right| {
            left.priority
                .cmp(&right.priority)
                .then_with(|| right.id.cmp(&left.id))
        })
        .map_or_else(
            || definition.default_location.clone(),
            |slot| slot.location.clone(),
        )
}
```

The stable tie rule is highest priority, then alphabetically lowest slot ID. The expression uses `right.id.cmp(&left.id)` because `max_by` selects the greater ordering.

Do not move every NPC each minute. Resolve a location when:

- The player enters a location.
- A scene asks whether an NPC is present.
- Time crosses a schedule boundary.
- The map or planner needs a known location.

Add `storyforge-core/tests/npc_schedule.rs`:

```rust
use storyforge_core::{
    ContentId, GameClock,
    simulation::{
        NpcScheduleDefinition, NpcScheduleSlot, ScheduleOverride, Weekday,
        resolve_npc_location,
    },
};

fn id(value: &str) -> ContentId {
    ContentId::new(value).expect("test ID should be valid")
}

#[test]
fn class_slot_should_replace_default_location_during_its_window() {
    let schedule = NpcScheduleDefinition {
        actor: id("academy.npc.iren"),
        default_location: id("academy.location.staff-room"),
        slots: vec![NpcScheduleSlot {
            id: "monday-class".to_owned(),
            weekdays: vec![Weekday::Monday],
            start_minute: 9 * 60,
            end_minute: 10 * 60,
            location: id("academy.location.rune-class"),
            priority: 10,
            conditions: Vec::new(),
        }],
    };
    let clock = GameClock {
        day: 0,
        minute_of_day: 9 * 60 + 15,
        term: id("academy.term.autumn"),
    };

    let location = resolve_npc_location(&schedule, None, &clock, |_| true);

    assert_eq!(location, id("academy.location.rune-class"));
}

#[test]
fn active_override_should_win_over_the_normal_schedule() {
    let schedule = NpcScheduleDefinition {
        actor: id("academy.npc.iren"),
        default_location: id("academy.location.staff-room"),
        slots: Vec::new(),
    };
    let clock = GameClock {
        day: 2,
        minute_of_day: 600,
        term: id("academy.term.autumn"),
    };
    let schedule_override = ScheduleOverride {
        location: id("academy.location.infirmary"),
        begins_day: 2,
        begins_minute: 500,
        ends_day: 3,
        ends_minute: 500,
        reason: id("academy.reason.recovering"),
    };

    let location =
        resolve_npc_location(&schedule, Some(&schedule_override), &clock, |_| true);

    assert_eq!(location, id("academy.location.infirmary"));
}
```

Run:

```powershell
cargo test -p storyforge-core --test npc_schedule
```

## Checkpoint 2: Add the school calendar

Create `storyforge-core/src/simulation/calendar.rs`:

```rust
use serde::{Deserialize, Serialize};

use crate::{ContentId, GameClock, WorldCondition};

use super::Weekday;

#[derive(Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize)]
pub struct CalendarDate {
    pub day: u32,
    pub minute_of_day: u16,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize)]
pub enum SchoolActivityKind {
    Class,
    Club,
    Exam,
    Meal,
    Curfew,
    Holiday,
    Match,
    Ceremony,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct CalendarDefinition {
    pub wake_minute: u16,
    pub terms: Vec<ContentId>,
    pub events: Vec<CalendarEventDefinition>,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct CalendarEventDefinition {
    pub id: ContentId,
    pub title: String,
    pub kind: SchoolActivityKind,
    pub weekdays: Vec<Weekday>,
    pub specific_days: Vec<u32>,
    pub start_minute: u16,
    pub duration_minutes: u16,
    pub location: ContentId,
    pub scene: Option<ContentId>,
    pub conditions: Vec<WorldCondition>,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct ActivityOccurrence {
    pub definition: ContentId,
    pub begins: CalendarDate,
    pub ends: CalendarDate,
}

#[derive(Debug, Clone, PartialEq, Eq, thiserror::Error)]
pub enum CalendarError {
    #[error("calendar minute must be lower than 1440")]
    InvalidMinute,
    #[error("calendar duration must be greater than zero")]
    ZeroDuration,
    #[error("calendar occurrence overflowed the supported day range")]
    DateOverflow,
}

impl CalendarEventDefinition {
    pub fn occurrence_on<F>(
        &self,
        day: u32,
        condition_is_met: F,
    ) -> Result<Option<ActivityOccurrence>, CalendarError>
    where
        F: Fn(&WorldCondition) -> bool,
    {
        if self.start_minute >= 1440 {
            return Err(CalendarError::InvalidMinute);
        }
        if self.duration_minutes == 0 {
            return Err(CalendarError::ZeroDuration);
        }

        let weekday_matches = self.weekdays.contains(&Weekday::from_day(day));
        let date_matches = self.specific_days.contains(&day);
        if (!weekday_matches && !date_matches)
            || !self.conditions.iter().all(condition_is_met)
        {
            return Ok(None);
        }

        let start = u64::from(day) * 1440 + u64::from(self.start_minute);
        let end = start + u64::from(self.duration_minutes);
        let end_day =
            u32::try_from(end / 1440).map_err(|_| CalendarError::DateOverflow)?;
        let end_minute =
            u16::try_from(end % 1440).map_err(|_| CalendarError::DateOverflow)?;

        Ok(Some(ActivityOccurrence {
            definition: self.id.clone(),
            begins: CalendarDate {
                day,
                minute_of_day: self.start_minute,
            },
            ends: CalendarDate {
                day: end_day,
                minute_of_day: end_minute,
            },
        }))
    }
}

#[must_use]
pub fn occurrences_crossed<F>(
    calendar: &CalendarDefinition,
    from: &GameClock,
    to: &GameClock,
    condition_is_met: F,
) -> Vec<ActivityOccurrence>
where
    F: Fn(&WorldCondition) -> bool + Copy,
{
    let from_absolute = u64::from(from.day) * 1440 + u64::from(from.minute_of_day);
    let to_absolute = u64::from(to.day) * 1440 + u64::from(to.minute_of_day);
    let mut occurrences = Vec::new();

    for day in from.day..=to.day {
        for event in &calendar.events {
            let Ok(Some(occurrence)) = event.occurrence_on(day, condition_is_met)
            else {
                continue;
            };
            let begins = u64::from(occurrence.begins.day) * 1440
                + u64::from(occurrence.begins.minute_of_day);
            if begins > from_absolute && begins <= to_absolute {
                occurrences.push(occurrence);
            }
        }
    }

    occurrences.sort_by(|left, right| {
        left.begins
            .day
            .cmp(&right.begins.day)
            .then(left.begins.minute_of_day.cmp(&right.begins.minute_of_day))
            .then_with(|| {
                left.definition
                    .as_str()
                    .cmp(right.definition.as_str())
            })
    });
    occurrences
}
```

This loops over crossed days and authored events, not crossed minutes. A long rest spanning eight hours has the same cost whether it starts at 21:00 or 21:01.

Calendar invitations become normal events:

```rust
#[derive(
    Debug, Clone, PartialEq, Eq, serde::Serialize, serde::Deserialize,
)]
pub enum LivingWorldEvent {
    CalendarActivityBegan {
        occurrence: ActivityOccurrence,
    },
    NpcLocationChanged {
        actor: ContentId,
        previous: ContentId,
        current: ContentId,
    },
    ShopRestocked {
        shop: ContentId,
        stock: Vec<(ContentId, u32)>,
    },
    ResourceNodeRestocked {
        node: ContentId,
        quantity: u32,
    },
    LivingWorldCommandRejected {
        reason: String,
    },
}
```

Add `LivingWorld(LivingWorldEvent),` to `GameEvent`. Chapter 14's class invitation consumes `CalendarActivityBegan` when the occurrence kind is `Class`. Clubs, exams, ceremonies, and curfew use the same event path.

## Checkpoint 3: Make dialogue aware of time

Create `storyforge-core/src/simulation/dialogue_time.rs`:

```rust
use serde::{Deserialize, Serialize};

use crate::{ContentId, GameClock};

use super::Weekday;

#[derive(Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize)]
pub enum TimeOfDay {
    Dawn,
    Morning,
    Afternoon,
    Evening,
    Night,
}

impl TimeOfDay {
    #[must_use]
    pub const fn from_minute(minute: u16) -> Self {
        match minute {
            300..=419 => Self::Dawn,
            420..=719 => Self::Morning,
            720..=1019 => Self::Afternoon,
            1020..=1259 => Self::Evening,
            _ => Self::Night,
        }
    }
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub enum DialogueTimeCondition {
    TimeOfDay(TimeOfDay),
    Weekday(Weekday),
    DayRange {
        first: u32,
        last: u32,
    },
    BeforeMinute(u16),
    AfterMinute(u16),
    DuringActivity(ContentId),
    ActivityCompleted(ContentId),
}

#[must_use]
pub fn dialogue_time_condition_is_met(
    condition: &DialogueTimeCondition,
    clock: &GameClock,
    active_activity: Option<&ContentId>,
    completed_activities: &[ContentId],
) -> bool {
    match condition {
        DialogueTimeCondition::TimeOfDay(expected) => {
            TimeOfDay::from_minute(clock.minute_of_day) == *expected
        }
        DialogueTimeCondition::Weekday(expected) => {
            Weekday::from_day(clock.day) == *expected
        }
        DialogueTimeCondition::DayRange { first, last } => {
            (*first..=*last).contains(&clock.day)
        }
        DialogueTimeCondition::BeforeMinute(minute) => {
            clock.minute_of_day < *minute
        }
        DialogueTimeCondition::AfterMinute(minute) => {
            clock.minute_of_day >= *minute
        }
        DialogueTimeCondition::DuringActivity(activity) => {
            active_activity == Some(activity)
        }
        DialogueTimeCondition::ActivityCompleted(activity) => {
            completed_activities.contains(activity)
        }
    }
}
```

Add this variant to the chapter 09 condition catalog:

```rust
Time(DialogueTimeCondition),
```

Examples:

- A professor comments when the player arrives after class starts.
- A groundskeeper gives different warnings after curfew.
- A club leader discusses tonight's meeting only on the correct day.
- A shopkeeper says the delivery arrives tomorrow instead of showing stock that does not exist.

Time-aware dialogue is still authored. The engine exposes conditions; it does not generate prose.

## Checkpoint 4: Restock shops and resource nodes

Create `storyforge-core/src/simulation/restock.rs`:

```rust
use serde::{Deserialize, Serialize};

use crate::ContentId;

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub enum RestockPolicy {
    Never,
    DailyAt {
        minute: u16,
    },
    Weekly {
        weekday: super::Weekday,
        minute: u16,
    },
    EveryDays {
        days: u16,
        minute: u16,
    },
    OnWorldPhase {
        phase: ContentId,
    },
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct RestockState {
    pub last_restock_absolute_minute: Option<u64>,
    pub last_world_phase: Option<ContentId>,
}

#[must_use]
pub fn restock_boundaries_crossed(
    policy: &RestockPolicy,
    state: &RestockState,
    from_absolute: u64,
    to_absolute: u64,
    current_phase: &ContentId,
) -> Vec<u64> {
    if to_absolute <= from_absolute {
        return Vec::new();
    }

    match policy {
        RestockPolicy::Never => Vec::new(),
        RestockPolicy::DailyAt { minute } => periodic_boundaries(
            0,
            1440,
            u64::from(*minute),
            state,
            from_absolute,
            to_absolute,
        ),
        RestockPolicy::Weekly { weekday, minute } => {
            periodic_boundaries(
                0,
                7 * 1440,
                weekday.day_offset() * 1440 + u64::from(*minute),
                state,
                from_absolute,
                to_absolute,
            )
        }
        RestockPolicy::EveryDays { days, minute } => {
            if *days == 0 {
                return Vec::new();
            }
            periodic_boundaries(
                0,
                u64::from(*days) * 1440,
                u64::from(*minute),
                state,
                from_absolute,
                to_absolute,
            )
        }
        RestockPolicy::OnWorldPhase { phase } => {
            let changed = phase == current_phase
                && state.last_world_phase.as_ref() != Some(current_phase);
            changed.then_some(to_absolute).into_iter().collect()
        }
    }
}

fn periodic_boundaries(
    origin: u64,
    period: u64,
    offset: u64,
    state: &RestockState,
    from_absolute: u64,
    to_absolute: u64,
) -> Vec<u64> {
    let effective_from = state
        .last_restock_absolute_minute
        .map_or(from_absolute, |last| last.max(from_absolute));
    let first_period = effective_from.saturating_sub(origin) / period;
    let mut boundary = origin + first_period * period + offset;
    if boundary <= effective_from {
        boundary = boundary.saturating_add(period);
    }

    let mut crossed = Vec::new();
    while boundary <= to_absolute {
        crossed.push(boundary);
        boundary = boundary.saturating_add(period);
    }
    crossed
}
```

A stock entry defines its reset behavior:

```rust
#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct RestockEntryDefinition {
    pub item: ContentId,
    pub target_quantity: u32,
    pub add_per_restock: u32,
}
```

At each boundary:

1. Find every entry below its target.
2. Add at most `add_per_restock`.
3. Clamp to `target_quantity`.
4. Apply regional scarcity modifiers from chapter 26.
5. Emit one `ShopRestocked` event containing the final changes.
6. Store the boundary in `RestockState`.

Resource nodes use the same policy but track harvestable quantity. A greenhouse herb bed, mine, fishing spot, or library request desk does not need a separate timer system.

## Checkpoint 5: Wire time advancement through one processor

Create `storyforge-core/src/simulation/processor.rs`:

```rust
use crate::{ContentId, GameEvent, GameClock};

#[derive(Debug, Clone, Copy, PartialEq, Eq, PartialOrd, Ord)]
pub enum SimulationBoundaryKind {
    QuestDeadline,
    WorldPhase,
    CalendarActivity,
    NpcSchedule,
    Restock,
    Ambient,
}

#[derive(Debug, Clone, PartialEq, Eq)]
pub struct SimulationBoundary {
    pub id: ContentId,
    pub absolute_minute: u64,
    pub kind: SimulationBoundaryKind,
}

#[derive(Debug, Clone, PartialEq, Eq, thiserror::Error)]
pub enum SimulationError {
    #[error("simulation cannot move backward in time")]
    BackwardTime,
    #[error("simulation boundary limit exceeded")]
    BoundaryLimit,
}

pub fn process_due_boundaries<F>(
    candidates: &[SimulationBoundary],
    from: &GameClock,
    to: &GameClock,
    maximum_boundaries: usize,
    mut resolve: F,
) -> Result<Vec<GameEvent>, SimulationError>
where
    F: FnMut(&SimulationBoundary) -> Vec<GameEvent>,
{
    let from_absolute =
        u64::from(from.day) * 1440 + u64::from(from.minute_of_day);
    let to_absolute = u64::from(to.day) * 1440 + u64::from(to.minute_of_day);
    if to_absolute < from_absolute {
        return Err(SimulationError::BackwardTime);
    }

    let mut boundaries = candidates
        .iter()
        .filter(|boundary| {
            boundary.absolute_minute > from_absolute
                && boundary.absolute_minute <= to_absolute
        })
        .collect::<Vec<_>>();
    boundaries.sort_by(|left, right| {
        left.absolute_minute
            .cmp(&right.absolute_minute)
            .then(left.kind.cmp(&right.kind))
            .then_with(|| left.id.as_str().cmp(right.id.as_str()))
    });

    if boundaries.len() > maximum_boundaries {
        return Err(SimulationError::BoundaryLimit);
    }

    let mut events = Vec::new();
    for boundary in boundaries {
        events.extend(resolve(boundary));
    }
    Ok(events)
}
```

Build `SimulationBoundary` candidates from `occurrences_crossed`, NPC slot start and end times, `restock_boundaries_crossed`, quest deadlines, and world-phase events. The resolver closure looks up the boundary ID and returns the typed events for that boundary. Tests can pass a small closure without constructing a full `GameState`; the engine passes its normal read-only catalogs.

Boundary priority is:

1. Quest deadlines.
2. Arc or world-phase transitions.
3. Calendar activities.
4. NPC schedule changes.
5. Shop and resource restocks.
6. Ambient events.

The processor builds the event list from read-only state. Apply that list through the chapter 05 reducer after it succeeds.

## Original calendar content

Create `campaigns/academy-demo/calendar/autumn-term.ron`:

```ron
(
    wake_minute: 420,
    terms: ["academy.term.autumn"],
    events: [
        (
            id: "academy.activity.rune-class-monday",
            title: "Practical Runes",
            kind: Class,
            weekdays: [Monday],
            specific_days: [],
            start_minute: 540,
            duration_minutes: 60,
            location: "academy.location.rune-class",
            scene: Some("academy.scene.rune-class"),
            conditions: [],
        ),
        (
            id: "academy.activity.glasswing-club",
            title: "Glasswing Club",
            kind: Club,
            weekdays: [Wednesday],
            specific_days: [],
            start_minute: 1020,
            duration_minutes: 90,
            location: "academy.location.greenhouse",
            scene: Some("academy.scene.glasswing-club"),
            conditions: [],
        ),
        (
            id: "academy.activity.weeknight-curfew",
            title: "Weeknight Curfew",
            kind: Curfew,
            weekdays: [Monday, Tuesday, Wednesday, Thursday, Friday],
            specific_days: [],
            start_minute: 1260,
            duration_minutes: 1,
            location: "academy.location.dormitory",
            scene: None,
            conditions: [],
        ),
    ],
)
```

## TUI behavior

Add a planner tab:

```text
DAY 8 - MONDAY

07:00  Breakfast                Main Hall
09:00  Practical Runes         Rune Classroom
17:00  Glasswing Club          Wednesday
21:00  Curfew                  Dormitory

Professor Iren
Current: Rune Classroom
Next known: Staff Room at 10:00
```

The planner shows only information the player knows. Secret meetings and unknown schedules remain hidden.

At a closed shop:

```text
THE BRASS TRUNK IS CLOSED

Hours: 08:00-18:00
Next restock: Tuesday morning

1 Wait until opening
2 Inspect the notice board
3 Leave
```

Waiting previews crossed classes, deadlines, curfew, and travel consequences before confirmation.

## Validator additions

| Code | Invalid content |
| --- | --- |
| `SIM001` | Schedule minute is outside `0..1440` |
| `SIM002` | Schedule end is not after start |
| `SIM003` | Equal-priority schedule slots overlap under the same conditions |
| `SIM004` | Scheduled actor or location does not exist |
| `SIM005` | Calendar duration is zero |
| `SIM006` | Calendar scene or location does not exist |
| `SIM007` | Restock period is zero |
| `SIM008` | Restock minute is outside `0..1440` |
| `SIM009` | Restock item or resource node does not exist |
| `SIM010` | Dialogue activity condition references no calendar activity |

## Automated test checklist

1. Default NPC location applies outside all slots.
2. Highest-priority legal schedule slot wins.
3. Stable slot ID breaks an equal-priority tie in test fixtures.
4. An active override beats the normal schedule.
5. An expired override stops applying.
6. A weekly class occurs only on its authored weekday.
7. A specific-date ceremony works without a weekday rule.
8. A long rest collects crossed boundaries without minute iteration.
9. Time-aware dialogue changes at the exact minute.
10. Daily restock fires once when several days are skipped.
11. Weekly restock uses the correct weekday.
12. Restocking never exceeds target quantity.
13. Loading before a restock and replaying produces the same event list.
14. A boundary limit failure changes no state.

## Manual playthrough

1. Start Monday at 08:45.
2. Find Professor Iren in the staff room.
3. Wait until 09:05 and confirm Iren moves to the rune classroom.
4. Speak to Iren and confirm the late-class dialogue appears.
5. Skip to Wednesday afternoon and open the planner.
6. Attend Glasswing Club.
7. Buy the last stocked notebook at the Brass Trunk.
8. Wait past the next restock boundary.
9. Confirm the notebook returns once and does not duplicate after save and load.
10. Wait until curfew and confirm known consequences appear before time advances.

## Full verification

```powershell
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features --locked -- -D warnings
cargo test --workspace --all-features --locked
cargo run -p storyforge-tui -- validate --pack campaigns/academy-demo
cargo run -p storyforge-tui -- play --pack campaigns/academy-demo --seed 61
```

## Common mistakes

- A second clock is added for school activities.
- Every NPC is mutated once per simulated minute.
- Dialogue generates text from time instead of selecting authored variants.
- A long rest runs thousands of tiny update ticks.
- Restock resets sold unique items that should remain unique.
- Shop inventory changes without an event, making replays diverge.
- The UI reveals secret NPC locations.

## Acceptance check

- NPC location is deterministic from clock, conditions, and override.
- The school calendar supports classes, clubs, exams, ceremonies, and curfew.
- Dialogue can test time and activity state.
- Shops and resource nodes restock at exact boundaries.
- Large time jumps process bounded boundary events rather than individual minutes.
- Saves replay the same schedule and restock results.

## Suggested commit

```powershell
git add .
git commit -m "Add living-world schedules and restocking"
```
