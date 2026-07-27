# 12: Build world travel, time, and story events

## Result

The player moves across a validated location graph. A route's story distance determines how many travel moments occur. Those moments can emphasize combat, roleplay, exploration, or a combination of those pillars. Travel can pause for a scene or encounter, survive a save, and resume at the correct point.

This system is inspired by the private `Traveling Event System TES.md` notes. It does not simulate every mile. It asks a more useful story question: how much can happen before the party arrives?

The event-card structure also borrows a general lesson from the private `One-Shot-Wonders.pdf`: a usable scenario needs an objective, opening hook, characters, beats, locations, clues, rewards, and difficulty adjustments. Do not copy its prose or adventures into the public campaign. All public event records in this guide are original.

## Checkpoint 1: Represent locations and routes

Create `crates/storyforge-core/src/world.rs`:

```rust
use std::collections::{BTreeSet, HashSet, VecDeque};

use serde::{Deserialize, Serialize};

use crate::ContentId;

#[derive(Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize)]
pub enum TravelDistance {
    Close,
    Far,
    VeryFar,
}

impl TravelDistance {
    #[must_use]
    pub const fn event_slots(self) -> u8 {
        match self {
            Self::Close => 1,
            Self::Far => 2,
            Self::VeryFar => 3,
        }
    }
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct LocationDefinition {
    pub id: ContentId,
    pub name: String,
    pub region: ContentId,
    pub tags: BTreeSet<String>,
    pub routes: Vec<RouteDefinition>,
    pub event_deck: Option<ContentId>,
    pub rest: RestPolicy,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct RouteDefinition {
    pub id: String,
    pub destination: ContentId,
    pub minutes: u32,
    pub distance: TravelDistance,
    pub visibility: Vec<WorldCondition>,
    pub access: Vec<WorldCondition>,
    pub event_deck_override: Option<ContentId>,
    pub one_way: bool,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize)]
pub enum RestPolicy {
    Unsafe,
    ShortRestOnly,
    Safe,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub enum WorldCondition {
    FlagSet(ContentId),
    RouteDiscovered(String),
    MinimumRelationship {
        npc: ContentId,
        trust: i16,
    },
}
```

The public content file for a location stores routes leaving that location. A normal two-way route therefore has two records. The validator checks that a return route exists unless `one_way` is true.

Export the types from `crates/storyforge-core/src/lib.rs`:

```rust
mod world;

pub use world::{
    LocationDefinition, RestPolicy, RouteDefinition, TravelDistance,
    WorldCondition,
};
```

Add this focused unit test inside `world.rs`:

```rust
#[cfg(test)]
mod tests {
    use super::TravelDistance;

    #[test]
    fn story_distance_controls_event_count() {
        assert_eq!(TravelDistance::Close.event_slots(), 1);
        assert_eq!(TravelDistance::Far.event_slots(), 2);
        assert_eq!(TravelDistance::VeryFar.event_slots(), 3);
    }
}
```

Run:

```powershell
cargo test -p storyforge-core story_distance_controls_event_count
```

Continue when the test passes.

## Checkpoint 2: Define travel pillars and event cards

Add the following types to `world.rs`:

```rust
#[derive(
    Debug, Clone, Copy, PartialEq, Eq, PartialOrd, Ord, Hash, Serialize, Deserialize,
)]
pub enum TravelPillar {
    Combat,
    Roleplay,
    Exploration,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize)]
pub enum TravelObjective {
    Acquisition,
    Competition,
    Confrontation,
    Defence,
    Delivery,
    Escape,
    Investigation,
    Rescue,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct TravelEventDefinition {
    pub id: ContentId,
    pub title: String,
    pub pillars: BTreeSet<TravelPillar>,
    pub objective: TravelObjective,
    pub opening_hook: String,
    pub scene: ContentId,
    pub conditions: Vec<WorldCondition>,
    pub weight: u16,
    pub cooldown_journeys: u8,
    pub once_per_save: bool,
    pub required_route_tags: BTreeSet<String>,
    pub story_hooks: Vec<StoryHookTemplate>,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct TravelEventDeck {
    pub id: ContentId,
    pub entries: Vec<ContentId>,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct StoryHookTemplate {
    pub kind: StoryHookKind,
    pub subject: ContentId,
    pub summary: String,
    pub expires_on_day: Option<u32>,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize)]
pub enum StoryHookKind {
    Rumor,
    FactionLead,
    NpcContact,
    LocationClue,
    ThreatEvidence,
    ResourceOpportunity,
    CompanionMemory,
}
```

Pillar combinations replace the color-only vocabulary from the tabletop notes:

| Tabletop color | Stored pillars | Typical use |
| --- | --- | --- |
| Red | Combat | Immediate physical or magical danger |
| Blue | Roleplay | Negotiation, conflict, favor, or confession |
| Yellow | Exploration | Navigation, discovery, puzzle, or environment |
| Purple | Roleplay + Combat | A tense conversation can become a fight |
| Green | Exploration + Roleplay | A discovery belongs to or affects someone |
| Orange | Combat + Exploration | The environment changes the tactical problem |
| White | All three | A larger event with several approaches |

The engine stores semantic pillar names. The TUI may use colors, but it must always print the words too so color-blind players and plain terminals receive the same information.

Event cards need enough material to become scenes. Add content-side authoring fields in `storyforge-content`:

```rust
#[derive(Debug, Clone, serde::Deserialize)]
pub struct TravelEventCard {
    pub definition: TravelEventDefinition,
    pub important_characters: Vec<ContentId>,
    pub story_beats: Vec<ContentId>,
    pub key_locations: Vec<ContentId>,
    pub clues: Vec<ContentId>,
    pub rewards: Vec<ContentId>,
    pub easier_variant: Option<ContentId>,
    pub harder_variant: Option<ContentId>,
}
```

The core selection system reads `TravelEventDefinition`. The scene system reads the rest of the card. This keeps selection fast and lets the story remain data-driven.

## Checkpoint 3: Add runtime journey state

Add these runtime types to `storyforge-core/src/world.rs`:

```rust
#[derive(Debug, Clone, PartialEq, Eq, Hash, Serialize, Deserialize)]
pub struct RouteKey {
    pub source: ContentId,
    pub route_id: String,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct GameClock {
    pub day: u32,
    pub minute_of_day: u16,
    pub term: ContentId,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct ScheduledEvent {
    pub id: ContentId,
    pub day: u32,
    pub minute_of_day: u16,
    pub priority: u16,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct StoryHook {
    pub source_event: ContentId,
    pub kind: StoryHookKind,
    pub subject: ContentId,
    pub summary: String,
    pub discovered_on_day: u32,
    pub expires_on_day: Option<u32>,
    pub promoted_quest: Option<ContentId>,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct JourneyState {
    pub source: ContentId,
    pub destination: ContentId,
    pub route_id: String,
    pub total_minutes: u32,
    pub remaining_minutes: u32,
    pub selected_events: Vec<ContentId>,
    pub next_event_index: usize,
}

impl JourneyState {
    #[must_use]
    pub fn is_complete(&self) -> bool {
        self.next_event_index >= self.selected_events.len()
    }
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct WorldState {
    pub current_location: ContentId,
    pub discovered_locations: HashSet<ContentId>,
    pub discovered_routes: HashSet<RouteKey>,
    pub clock: GameClock,
    pub scheduled_events: Vec<ScheduledEvent>,
    pub recent_travel_events: VecDeque<ContentId>,
    pub completed_once_events: HashSet<ContentId>,
    pub pending_journey: Option<JourneyState>,
    pub journey: Option<JourneyState>,
    pub story_hooks: Vec<StoryHook>,
}
```

`JourneyState` is saved. If a travel event opens combat and the player quits, loading the save restores the event scene and the journey cursor. It does not reroll the route.

Add clock advancement:

```rust
#[derive(Debug, Clone, PartialEq, Eq, thiserror::Error)]
pub enum TimeError {
    #[error("minute_of_day must be lower than 1440")]
    InvalidMinute,
    #[error("advancing time overflowed the supported day counter")]
    DayOverflow,
}

pub fn advance_clock(clock: &mut GameClock, minutes: u32) -> Result<(), TimeError> {
    if clock.minute_of_day >= 1440 {
        return Err(TimeError::InvalidMinute);
    }

    let current = u64::from(clock.day)
        .checked_mul(1440)
        .and_then(|value| value.checked_add(u64::from(clock.minute_of_day)))
        .ok_or(TimeError::DayOverflow)?;
    let advanced = current
        .checked_add(u64::from(minutes))
        .ok_or(TimeError::DayOverflow)?;
    let day = advanced / 1440;
    let minute = advanced % 1440;

    clock.day = u32::try_from(day).map_err(|_| TimeError::DayOverflow)?;
    clock.minute_of_day = u16::try_from(minute).map_err(|_| TimeError::DayOverflow)?;
    Ok(())
}
```

Create a test:

```rust
#[test]
fn advancing_across_midnight_wraps_the_clock() {
    let mut clock = GameClock {
        day: 3,
        minute_of_day: 1430,
        term: ContentId::new("academy.term.autumn").expect("valid test ID"),
    };

    advance_clock(&mut clock, 45).expect("clock should advance");

    assert_eq!(clock.day, 4);
    assert_eq!(clock.minute_of_day, 35);
}
```

Run:

```powershell
cargo test -p storyforge-core advancing_across_midnight_wraps_the_clock
```

## Checkpoint 4: Select events deterministically

Add `rand` to `storyforge-core` if chapter 05 did not already add it:

```powershell
cargo add -p storyforge-core rand
```

The selector receives already-loaded event definitions and an engine-owned RNG. It never creates a thread RNG, so saves and tests remain replayable.

Add the selection context and error to `storyforge-core/src/world.rs`:

```rust
use rand::Rng;

#[derive(Debug)]
pub struct TravelSelectionContext<'a> {
    pub route_tags: &'a BTreeSet<String>,
    pub recent_events: &'a VecDeque<ContentId>,
    pub completed_once_events: &'a HashSet<ContentId>,
}

#[derive(Debug, Clone, PartialEq, Eq, thiserror::Error)]
pub enum TravelSelectionError {
    #[error("travel event `{0}` has zero weight")]
    ZeroWeight(ContentId),
    #[error("the legal travel-event weights overflowed")]
    WeightOverflow,
}

fn event_is_legal(
    event: &TravelEventDefinition,
    context: &TravelSelectionContext<'_>,
) -> bool {
    if event.once_per_save && context.completed_once_events.contains(&event.id) {
        return false;
    }

    let recent_count = context
        .recent_events
        .iter()
        .rev()
        .take(usize::from(event.cooldown_journeys))
        .any(|recent| recent == &event.id);
    if recent_count {
        return false;
    }

    event
        .required_route_tags
        .iter()
        .all(|tag| context.route_tags.contains(tag))
}

pub fn select_travel_events<R: Rng + ?Sized>(
    rng: &mut R,
    distance: TravelDistance,
    events: &[TravelEventDefinition],
    context: &TravelSelectionContext<'_>,
) -> Result<Vec<ContentId>, TravelSelectionError> {
    for event in events {
        if event.weight == 0 {
            return Err(TravelSelectionError::ZeroWeight(event.id.clone()));
        }
    }

    let mut legal = events
        .iter()
        .filter(|event| event_is_legal(event, context))
        .collect::<Vec<_>>();
    legal.sort_by(|left, right| left.id.as_str().cmp(right.id.as_str()));

    let mut selected = Vec::new();
    for _ in 0..distance.event_slots() {
        if legal.is_empty() {
            break;
        }

        let total = legal.iter().try_fold(0_u32, |sum, event| {
            sum.checked_add(u32::from(event.weight))
                .ok_or(TravelSelectionError::WeightOverflow)
        })?;
        let mut roll = rng.gen_range(0..total);
        let selected_index = legal
            .iter()
            .position(|event| {
                let weight = u32::from(event.weight);
                if roll < weight {
                    true
                } else {
                    roll -= weight;
                    false
                }
            })
            .unwrap_or(legal.len() - 1);

        let event = legal.remove(selected_index);
        selected.push(event.id.clone());
    }

    Ok(selected)
}
```

The exact order is:

1. Reject invalid zero weights.
2. Remove once-per-save events that already occurred.
3. Remove events still on cooldown.
4. Remove events whose route tags do not match.
5. Sort by stable content ID.
6. Roll a deterministic weighted index.
7. Remove the selected event so it cannot fill two slots in one journey.
8. Repeat for the route's event count.

World flags and relationship conditions belong in `event_is_legal`. Add them when the condition evaluator from chapter 09 is available. Do not create a second condition language here.

If no event is legal, returning fewer events is valid. The TUI renders a short quiet-travel message for the empty slot; quiet travel is not stored as fake content history.

Add a deterministic test:

```rust
#[test]
fn the_same_seed_selects_the_same_far_journey() {
    use rand::{SeedableRng, rngs::StdRng};

    fn event(id: &str, weight: u16) -> TravelEventDefinition {
        TravelEventDefinition {
            id: ContentId::new(id).expect("valid test ID"),
            title: id.to_owned(),
            pillars: BTreeSet::from([TravelPillar::Exploration]),
            objective: TravelObjective::Investigation,
            opening_hook: "A test hook.".to_owned(),
            scene: ContentId::new("academy.scene.test").expect("valid test ID"),
            conditions: Vec::new(),
            weight,
            cooldown_journeys: 0,
            once_per_save: false,
            required_route_tags: BTreeSet::new(),
            story_hooks: Vec::new(),
        }
    }

    let events = [
        event("academy.travel.a", 1),
        event("academy.travel.b", 3),
        event("academy.travel.c", 2),
    ];
    let tags = BTreeSet::new();
    let recent = VecDeque::new();
    let completed = HashSet::new();
    let context = TravelSelectionContext {
        route_tags: &tags,
        recent_events: &recent,
        completed_once_events: &completed,
    };
    let mut first_rng = StdRng::seed_from_u64(44);
    let mut second_rng = StdRng::seed_from_u64(44);

    let first = select_travel_events(
        &mut first_rng,
        TravelDistance::Far,
        &events,
        &context,
    )
    .expect("selection should work");
    let second = select_travel_events(
        &mut second_rng,
        TravelDistance::Far,
        &events,
        &context,
    )
    .expect("selection should work");

    assert_eq!(first, second);
    assert_eq!(first.len(), 2);
    assert_ne!(first[0], first[1]);
}
```

Run:

```powershell
cargo test -p storyforge-core the_same_seed_selects_the_same_far_journey
```

## Checkpoint 5: Define travel commands and events

Add the travel command and event enums to `storyforge-core/src/world.rs`:

```rust
#[derive(Debug, Clone, PartialEq, Eq)]
pub enum TravelCommand {
    BeginTravel {
        route_id: String,
    },
    ConfirmDeadlineCrossing,
    ResolveCurrentEvent {
        outcome: ContentId,
    },
    ResumeJourney,
    CancelBeforeDeparture,
    Wait {
        minutes: u32,
    },
    Rest {
        kind: RestKind,
    },
    PromoteStoryHook {
        hook_index: usize,
        quest: ContentId,
    },
}

#[derive(
    Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize,
)]
pub enum RestKind {
    Short,
    Long,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub enum TravelEvent {
    TravelConfirmationRequired {
        journey: JourneyState,
        crossed_schedule: ContentId,
    },
    JourneyStarted {
        journey: JourneyState,
    },
    TravelSceneRequested {
        event: ContentId,
        scene: ContentId,
    },
    TravelEventResolved {
        event: ContentId,
        outcome: ContentId,
        once_per_save: bool,
    },
    StoryHookDiscovered {
        hook: StoryHook,
    },
    JourneyResumed {
        next_event_index: usize,
    },
    LocationEntered {
        location: ContentId,
    },
    RouteDiscovered {
        route: RouteKey,
    },
    TimeAdvanced {
        minutes: u32,
    },
    JourneyCompleted,
    JourneyCancelled,
    RestCompleted {
        kind: RestKind,
    },
    StoryHookPromoted {
        hook_index: usize,
        quest: ContentId,
    },
    TravelRejected {
        reason: String,
    },
    AutosaveRequested {
        reason: String,
    },
}
```

The command contracts are:

| Command | Validation | Emitted result |
| --- | --- | --- |
| `BeginTravel` | Route exists, is visible, accessible, and no journey is active | Confirmation event if a known schedule is crossed; otherwise `JourneyStarted` |
| `ConfirmDeadlineCrossing` | A pending confirmation exists | Starts the previously prepared journey |
| `ResolveCurrentEvent` | A journey and current event exist; outcome belongs to that scene | Records resolution, hooks, time, and next cursor |
| `ResumeJourney` | A journey exists and no child scene is active | Requests the next travel scene or completes arrival |
| `CancelBeforeDeparture` | Journey has not started | Clears confirmation only |
| `Wait` | Minutes are one of the offered choices and location permits waiting | Advances clock and processes crossed schedules |
| `Rest` | Location permits the rest kind | Applies chapter 13 resource rules after confirmation |
| `PromoteStoryHook` | Hook exists, is not expired, and has no quest yet | Links the hook to a new or authored quest |

`BeginTravel` performs these steps in order:

1. Find the route on the current location.
2. Evaluate visibility.
3. Evaluate access.
4. Determine the event deck from the route override or location default.
5. Select and save all event IDs using the engine RNG.
6. Preview schedule boundaries and deadlines.
7. Ask for confirmation if a known consequence is crossed.
8. Store `JourneyState`.
9. Dispatch the first event or complete quiet travel.

Implement the route lookup and journey construction as a separate function. This keeps the command match arm short and gives it a focused test:

```rust
#[derive(Debug, Clone, PartialEq, Eq, thiserror::Error)]
pub enum TravelStartError {
    #[error("another journey is already active")]
    JourneyAlreadyActive,
    #[error("route `{0}` does not exist at the current location")]
    UnknownRoute(String),
    #[error("route `{0}` is not visible")]
    HiddenRoute(String),
    #[error("route `{0}` is currently locked")]
    LockedRoute(String),
    #[error(transparent)]
    Selection(#[from] TravelSelectionError),
}

pub fn prepare_journey<R, F>(
    rng: &mut R,
    world: &WorldState,
    location: &LocationDefinition,
    route_id: &str,
    route_tags: &BTreeSet<String>,
    deck_events: &[TravelEventDefinition],
    condition_is_met: F,
) -> Result<JourneyState, TravelStartError>
where
    R: Rng + ?Sized,
    F: Fn(&WorldCondition) -> bool,
{
    if world.journey.is_some() {
        return Err(TravelStartError::JourneyAlreadyActive);
    }

    let route = location
        .routes
        .iter()
        .find(|candidate| candidate.id == route_id)
        .ok_or_else(|| TravelStartError::UnknownRoute(route_id.to_owned()))?;

    if !route.visibility.iter().all(&condition_is_met) {
        return Err(TravelStartError::HiddenRoute(route_id.to_owned()));
    }
    if !route.access.iter().all(condition_is_met) {
        return Err(TravelStartError::LockedRoute(route_id.to_owned()));
    }

    let selection_context = TravelSelectionContext {
        route_tags,
        recent_events: &world.recent_travel_events,
        completed_once_events: &world.completed_once_events,
    };
    let selected_events = select_travel_events(
        rng,
        route.distance,
        deck_events,
        &selection_context,
    )?;

    Ok(JourneyState {
        source: location.id.clone(),
        destination: route.destination.clone(),
        route_id: route.id.clone(),
        total_minutes: route.minutes,
        remaining_minutes: route.minutes,
        selected_events,
        next_event_index: 0,
    })
}
```

The engine's `BeginTravel` match arm calls `prepare_journey`. Convert each `TravelStartError` into `TravelRejected { reason: error.to_string() }`. If `crossed_known_schedule(&journey, world)` returns an ID, emit `TravelConfirmationRequired { journey, crossed_schedule }`. The event applier stores the journey in `world.pending_journey`. Otherwise emit:

```rust
vec![
    TravelEvent::JourneyStarted {
        journey: journey.clone(),
    },
    TravelEvent::AutosaveRequested {
        reason: "journey started".to_owned(),
    },
]
```

`ConfirmDeadlineCrossing` clones `world.pending_journey`. If it is `None`, emit `TravelRejected`. If it exists, emit `JourneyStarted` and `AutosaveRequested`. `CancelBeforeDeparture` emits only `JourneyCancelled`; its reducer clears `pending_journey` without changing time or location.

After those events are applied, dispatch `ResumeJourney`. Its handler reads `journey.selected_events[journey.next_event_index]`. If an event remains, it emits `TravelSceneRequested`. If none remains, it emits, in this exact order:

```rust
vec![
    TravelEvent::TimeAdvanced {
        minutes: journey.remaining_minutes,
    },
    TravelEvent::RouteDiscovered {
        route: RouteKey {
            source: journey.source.clone(),
            route_id: journey.route_id.clone(),
        },
    },
    TravelEvent::LocationEntered {
        location: journey.destination.clone(),
    },
    TravelEvent::JourneyCompleted,
    TravelEvent::AutosaveRequested {
        reason: "journey completed".to_owned(),
    },
]
```

`ResolveCurrentEvent` performs these steps:

1. Validate the active event and its allowed outcome.
2. Add any earned `StoryHook` records.
3. Mark a once-per-save event complete.
4. Add its ID to bounded recent history.
5. Emit `TimeAdvanced` with the event's authored time.
6. Increment `next_event_index`.
7. Autosave.
8. Resume the next event or arrival.

Keep the handler pure, as in chapter 05. Route lookup, condition evaluation, and event lookup arrive through a `TravelCatalog<'_>` argument. The handler emits events; `apply_travel_events` is the only function that mutates `WorldState`.

## Checkpoint 6: Apply travel events

Add this applier:

```rust
pub fn apply_travel_events(
    world: &mut WorldState,
    events: &[TravelEvent],
) -> Result<(), TimeError> {
    for event in events {
        match event {
            TravelEvent::TravelConfirmationRequired { journey, .. } => {
                world.pending_journey = Some(journey.clone());
            }
            TravelEvent::JourneyStarted { journey } => {
                world.pending_journey = None;
                world.journey = Some(journey.clone());
            }
            TravelEvent::TravelEventResolved {
                event,
                once_per_save,
                ..
            } => {
                world.recent_travel_events.push_back(event.clone());
                while world.recent_travel_events.len() > 12 {
                    world.recent_travel_events.pop_front();
                }
                if *once_per_save {
                    world.completed_once_events.insert(event.clone());
                }
            }
            TravelEvent::StoryHookDiscovered { hook } => {
                world.story_hooks.push(hook.clone());
            }
            TravelEvent::JourneyResumed { next_event_index } => {
                if let Some(journey) = &mut world.journey {
                    journey.next_event_index = *next_event_index;
                }
            }
            TravelEvent::TimeAdvanced { minutes } => {
                advance_clock(&mut world.clock, *minutes)?;
                if let Some(journey) = &mut world.journey {
                    journey.remaining_minutes =
                        journey.remaining_minutes.saturating_sub(*minutes);
                }
            }
            TravelEvent::LocationEntered { location } => {
                world.current_location.clone_from(location);
                world.discovered_locations.insert(location.clone());
            }
            TravelEvent::RouteDiscovered { route } => {
                world.discovered_routes.insert(route.clone());
            }
            TravelEvent::StoryHookPromoted { hook_index, quest } => {
                if let Some(hook) = world.story_hooks.get_mut(*hook_index) {
                    hook.promoted_quest = Some(quest.clone());
                }
            }
            TravelEvent::JourneyCompleted => {
                world.journey = None;
            }
            TravelEvent::JourneyCancelled => {
                world.pending_journey = None;
            }
            TravelEvent::TravelSceneRequested { .. }
            | TravelEvent::RestCompleted { .. }
            | TravelEvent::TravelRejected { .. }
            | TravelEvent::AutosaveRequested { .. } => {}
        }
    }

    Ok(())
}
```

The handler copies `once_per_save` from the validated definition into `TravelEventResolved`. The reducer never infers rules from an event's name.

Export the travel types and functions from `lib.rs` after this checkpoint.

## Checkpoint 7: Author an original public event deck

Create `campaigns/academy-demo/travel/decks/castle-grounds.ron`:

```ron
(
    id: "academy.travel-deck.castle-grounds",
    entries: [
        "academy.travel.runaway-notes",
        "academy.travel.bridge-argument",
        "academy.travel.glasswing-trail",
        "academy.travel.silent-bell",
    ],
)
```

Create `campaigns/academy-demo/travel/events/runaway-notes.ron`:

```ron
(
    definition: (
        id: "academy.travel.runaway-notes",
        title: "The Runaway Notes",
        pillars: [Exploration, Roleplay],
        objective: Acquisition,
        opening_hook: "Loose pages race uphill against the wind while their owner gives chase.",
        scene: "academy.scene.runaway-notes",
        conditions: [],
        weight: 4,
        cooldown_journeys: 2,
        once_per_save: false,
        required_route_tags: ["outdoor"],
        story_hooks: [
            (
                kind: FactionLead,
                subject: "academy.faction.night-archive",
                summary: "One recovered page bears the seal of a closed student society.",
                expires_on_day: None,
            ),
        ],
    ),
    important_characters: ["academy.npc.tovan-reed"],
    story_beats: [
        "academy.beat.notes-scatter",
        "academy.beat.choose-rescue-method",
        "academy.beat.return-or-read",
    ],
    key_locations: ["academy.location.rain-bridge"],
    clues: ["academy.clue.night-archive-seal"],
    rewards: ["academy.reward.tovan-trust"],
    easier_variant: Some("academy.variant.notes-dry-weather"),
    harder_variant: Some("academy.variant.notes-storm"),
)
```

The event has at least three approaches:

- Exploration: predict where the pages will collect.
- Roleplay: coordinate the frightened owner and bystanders.
- Magic: use an appropriate cantrip or leveled spell, with authored consequences.

Create the other three deck records so validation never depends on a named but missing event.

`campaigns/academy-demo/travel/events/bridge-argument.ron`:

```ron
(
    definition: (
        id: "academy.travel.bridge-argument",
        title: "Terms at the Rain Bridge",
        pillars: [Roleplay, Combat],
        objective: Confrontation,
        opening_hook: "Two student groups block the bridge while a ward crackles between them.",
        scene: "academy.scene.bridge-argument",
        conditions: [],
        weight: 3,
        cooldown_journeys: 3,
        once_per_save: false,
        required_route_tags: ["bridge"],
        story_hooks: [
            (
                kind: FactionLead,
                subject: "academy.faction.bridge-club",
                summary: "Someone supplied both groups with the same false warning.",
                expires_on_day: Some(6),
            ),
        ],
    ),
    important_characters: [],
    story_beats: [],
    key_locations: ["academy.location.rain-bridge"],
    clues: [],
    rewards: [],
    easier_variant: None,
    harder_variant: None,
)
```

`campaigns/academy-demo/travel/events/glasswing-trail.ron`:

```ron
(
    definition: (
        id: "academy.travel.glasswing-trail",
        title: "The Glasswing Trail",
        pillars: [Exploration],
        objective: Investigation,
        opening_hook: "A line of luminous moths marks a path that was not here this morning.",
        scene: "academy.scene.glasswing-trail",
        conditions: [],
        weight: 4,
        cooldown_journeys: 2,
        once_per_save: false,
        required_route_tags: ["outdoor"],
        story_hooks: [
            (
                kind: LocationClue,
                subject: "academy.location.sealed-orchard",
                summary: "The moths vanish through a wall beside the sealed orchard.",
                expires_on_day: None,
            ),
        ],
    ),
    important_characters: [],
    story_beats: [],
    key_locations: ["academy.location.rain-path"],
    clues: [],
    rewards: [],
    easier_variant: None,
    harder_variant: None,
)
```

`campaigns/academy-demo/travel/events/silent-bell.ron`:

```ron
(
    definition: (
        id: "academy.travel.silent-bell",
        title: "The Bell Without a Sound",
        pillars: [Combat, Exploration, Roleplay],
        objective: Defence,
        opening_hook: "A fallen handbell shakes in the mud while nearby shadows move against the light.",
        scene: "academy.scene.silent-bell",
        conditions: [],
        weight: 1,
        cooldown_journeys: 6,
        once_per_save: true,
        required_route_tags: ["old-road"],
        story_hooks: [
            (
                kind: ThreatEvidence,
                subject: "academy.threat.hollow-shadow",
                summary: "The silent bell carries residue from a ward that failed years ago.",
                expires_on_day: None,
            ),
        ],
    ),
    important_characters: [],
    story_beats: [],
    key_locations: ["academy.location.old-road"],
    clues: [],
    rewards: [],
    easier_variant: None,
    harder_variant: None,
)
```

Create these four scene files with the scene model from chapter 09:

| File | Required choices |
| --- | --- |
| `scenes/travel/runaway-notes.ron` | Predict the pages, coordinate students, cast a suitable spell, or leave |
| `scenes/travel/bridge-argument.ron` | Mediate, investigate the warning, threaten passage, or accept a duel |
| `scenes/travel/glasswing-trail.ron` | Follow, observe safely, collect a trace, or continue travelling |
| `scenes/travel/silent-bell.ron` | Raise a defence, examine the bell, call for help, or retreat |

Every choice has an authored outcome ID. Combat is one possible branch in `bridge-argument` and `silent-bell`, not the only valid resolution. Add each referenced scene and location before expecting `storyforge validate` to pass.

The event does not automatically become a main quest. It produces a `FactionLead`. Later scenes may:

- Ignore it.
- Add context to it.
- Expire it.
- Convert it into a side quest.
- Promote it into a main-arc objective when the current arc and faction state allow that transition.

This separation lets random travel content enrich the central story without making the main plot depend on one random roll.

## Checkpoint 8: Add map and travel-event UI

The map must provide a text route list even when it also draws spatial ASCII:

```text
                    [North Tower]
                          |
[Greenhouses] -- [Main Hall] -- [Library]
                          |
                    [Rain Gate]
```

The selected route panel shows:

```text
Rain Gate -> Lantern Row
Distance: Far
Travel moments: 2
Time: 45 minutes
Pillars in deck: Exploration, Roleplay, Combat
Known consequence: Archives close before arrival
```

Render a triggered event in the living dashboard:

```text
┌──────────────────────────────────────────────────────────────────────┐
│ TRAVEL EVENT                         EXPLORATION + ROLEPLAY           │
├──────────────────────────────────────────────────────────────────────┤
│ THE RUNAWAY NOTES                                                    │
│                                                                      │
│ Loose pages race uphill against the wind while their owner gives     │
│ chase.                                                               │
│                                                                      │
│ > Read the wind and cut them off                                     │
│   Organize the nearby students                                       │
│   Cast a suitable spell                                              │
│   Let the pages go                                                    │
├────────────────────────────────────┬─────────────────────────────────┤
│ JOURNEY                            │ CLOCK                           │
│ Main Hall -> Lantern Row           │ Day 3  16:35                   │
│ Event 1 of 2                       │ Archives close at 17:00        │
└────────────────────────────────────┴─────────────────────────────────┘
```

Never show only “green event.” Print `Exploration + Roleplay`; color is secondary styling.

When a scene or combat interrupts travel:

1. Keep the active `JourneyState`.
2. Save before opening the child scene.
3. Resolve the child scene into one authored outcome ID.
4. Dispatch `ResolveCurrentEvent`.
5. Save the incremented cursor.
6. Dispatch `ResumeJourney`.

## Checkpoint 9: Waiting, schedules, and rest

Waiting offers only explicit values:

- 10 minutes.
- 30 minutes.
- 60 minutes.
- Until the next known scheduled event.

Preview crossed deadlines before dispatch. Waiting in an unsafe location may select a local event, but it does not count as a route journey.

Schedule ordering is stable:

1. Deadlines.
2. Required world-phase transitions.
3. Quest events.
4. NPC schedule changes.
5. Ambient events.
6. Stable event ID for ties.

Do not advance one minute at a time. Calculate crossed boundaries between the old and new absolute time, sort them, then emit them in order.

Rest rules:

| Rest | Default time | Spell slots | Sorcery points | Temporary slots |
| --- | --- | --- | --- | --- |
| Short | 30 minutes | Unchanged unless a feature says otherwise | Unchanged unless a feature says otherwise | Unchanged |
| Long | Until campaign wake time | Restore normal slots to maximum | Restore to feature maximum | Remove before restoring normal slots |

The confirmation dialog lists known deadline and night-event consequences. Chapter 13 owns the resource mutations; this chapter owns time and location permission.

## Validator additions

Implement these diagnostics in `storyforge-content`:

| Code | Condition |
| --- | --- |
| `WORLD001` | Location ID is duplicated |
| `WORLD002` | Route destination does not exist |
| `WORLD003` | Route minutes are zero |
| `WORLD004` | Two-way route has no return definition |
| `WORLD005` | Event deck ID does not resolve |
| `WORLD006` | Travel-event weight is zero |
| `WORLD007` | Event scene does not exist |
| `WORLD008` | Event has no pillar |
| `WORLD009` | Event references an unknown clue, reward, beat, or actor |
| `WORLD010` | Entry location is unreachable in the validation fixture |
| `WORLD011` | Schedule minute is outside `0..1440` |
| `WORLD012` | NPC has ambiguous equal-priority schedule entries |

Add fixture states for hidden routes and arc-locked routes. A route need not be open in a fresh save, but the validator must prove that required locations are reachable in at least one declared fixture.

## Automated test checklist

Add one focused test for each behavior:

1. Close, Far, and Very Far select at most one, two, and three unique events.
2. The same seed and state produce the same selection.
3. A zero-weight event is rejected.
4. A once-per-save event never repeats.
5. A cooldown event returns after leaving recent history.
6. A required route tag filters the event.
7. A locked route rejects `BeginTravel`.
8. Crossing a known deadline requests confirmation.
9. Cancelling before departure leaves location and time unchanged.
10. Combat interruption preserves `JourneyState`.
11. Loading mid-journey does not reroll selected event IDs.
12. Arrival discovers the route and destination.
13. Midnight wraps correctly.
14. Crossed schedules emit in stable order.
15. A story hook can be promoted only once.
16. Short rest leaves spell slots and sorcery points unchanged by default.
17. Long rest removes temporary slots before normal slot restoration.

## Manual playthrough

1. Start the public demo with seed `44`.
2. Open the map at Main Hall.
3. Inspect a Close, Far, and Very Far route and confirm their event counts are 1, 2, and 3.
4. Choose the Far route to Lantern Row.
5. Decline a deadline warning and confirm nothing changes.
6. Begin the route again and accept.
7. Resolve the first event, then quit during the second.
8. Load the save and confirm the second event is still active.
9. Finish the journey and confirm the destination, clock, and discovered route.
10. Open the journal and inspect any earned story hook.
11. Rest in an unsafe location and confirm long rest is unavailable.
12. Return to a safe location, long rest, and confirm spell slots restore without any mana field.

## Full verification

```powershell
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features --locked -- -D warnings
cargo test --workspace --all-features --locked
cargo run -p storyforge-tui -- validate --pack campaigns/academy-demo
cargo run -p storyforge-tui -- play --pack campaigns/academy-demo --seed 44
```

## Common mistakes

- Distance is treated as realistic miles instead of story-event capacity.
- The UI relies on red, blue, and yellow without pillar labels.
- The selector uses thread-local randomness and cannot replay a save.
- Event IDs are rerolled after loading mid-journey.
- An event can occupy two slots in the same journey.
- A random clue is required to finish the main story.
- Public event content copies a private or published adventure.
- Waiting advances one minute at a time.
- Arrival occurs before the final travel scene resolves.
- Long rest restores a mana field that no longer exists.

## Acceptance check

- Locations and routes validate as a graph.
- Story distance produces one, two, or three travel moments.
- Events support semantic pillar combinations and objective tags.
- Selection is weighted, filtered, stable, and save-safe.
- Travel survives scene and combat interruption.
- Random events create optional hooks that later story systems can promote.
- The TUI shows routes and pillars without depending on ASCII position or color.
- Rest restores spell slots according to the rules profile.

## Suggested commit

```powershell
git add .
git commit -m "Build deterministic story-first travel events"
```
