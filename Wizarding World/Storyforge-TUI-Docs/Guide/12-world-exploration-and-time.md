# 12: Add world exploration and time

## Result

The player can move across a validated location graph, discover routes, spend time, meet scheduled NPCs, trigger travel events, wait, and rest.

## Location definitions

Add:

```rust
pub struct LocationDefinition {
    pub id: ContentId,
    pub name: String,
    pub region: ContentId,
    pub tags: Vec<String>,
    pub descriptions: Vec<DescriptionVariant>,
    pub routes: Vec<RouteDefinition>,
    pub interactions: Vec<InteractionDefinition>,
    pub event_deck: Option<ContentId>,
    pub rest: RestPolicy,
}

pub struct RouteDefinition {
    pub id: String,
    pub destination: ContentId,
    pub minutes: u32,
    pub visibility: Vec<ConditionDefinition>,
    pub access: Vec<ConditionDefinition>,
    pub event_budget: u8,
    pub one_way: bool,
}
```

Routes are declared from each source location. For a two-way passage, validate that the return route exists unless the author explicitly marks it one-way.

## Runtime world state

```rust
pub struct WorldState {
    pub current_location: ContentId,
    pub discovered_locations: HashSet<ContentId>,
    pub discovered_routes: HashSet<RouteKey>,
    pub clock: GameClock,
    pub scheduled_events: Vec<ScheduledEvent>,
    pub recent_travel_events: VecDeque<ContentId>,
}

pub struct GameClock {
    pub day: u32,
    pub minute_of_day: u16,
    pub term: ContentId,
}
```

Keep `minute_of_day` in `0..1440`. Advancing across midnight increments day and wraps minutes.

## Travel command

`Travel { route_id }` resolves:

1. Find route from current location.
2. Check visibility.
3. Check access.
4. Warn if travel crosses a known deadline or class.
5. Confirm when needed.
6. Select travel events from the route budget.
7. Resolve each event.
8. Advance remaining time.
9. Move to destination.
10. Discover destination and route.
11. Process arrival schedules.
12. Emit `LocationEntered`.
13. Autosave.

Travel interrupted by combat or a scene keeps a `JourneyState` with source, destination, route, completed events, and remaining minutes.

## Time advancement

Use one core function:

```rust
pub fn advance_time(
    state: &mut GameState,
    minutes: u32,
) -> Result<Vec<GameEvent>, TimeError>
```

It finds every crossed scheduled boundary in order. Do not advance one minute at a time.

Schedule event ordering:

1. Deadlines.
2. Required world-phase transitions.
3. Quest events.
4. NPC schedule updates.
5. Ambient events.

Tie ordering uses stable event ID.

## NPC schedules

A schedule slot has:

- Day filter.
- Start minute.
- End minute.
- Location.
- Arc or world-phase condition.
- Priority.

When several slots match, choose highest priority then stable definition order. NPCs outside the active location remain schedule records, not continuously simulated entities.

## Event decks

An event entry defines:

- ID.
- Weight.
- Type.
- Conditions.
- Cooldown.
- Once-per-save flag.
- Scene or encounter target.

Selection:

1. Filter legal entries.
2. Remove cooldown entries.
3. Build deterministic weighted total.
4. Roll through engine RNG.
5. Record selection.
6. Update cooldown history.

If nothing is legal, emit a quiet-travel event and continue.

## Map UI

The map renders known nodes:

```text
                    [North Tower]
                          |
[Greenhouses] -- [Main Hall] -- [Library]
                          |
                    [Rain Gate]
```

Use text labels at compact sizes. Do not rely on spatial art as the only route list. The selected location panel shows travel time, access, known danger, and next scheduled event.

## Observe and search

`Observe` is free the first time a location variant is viewed.

`Search`:

- Costs authored time.
- Uses the best relevant passive or active check.
- Records attempted discovery IDs.
- Does not allow identical retry without changed circumstances.

Changed circumstances include a tool, spell, clue, companion, time variant, or higher skill.

## Waiting

Offer fixed choices:

- 10 minutes.
- 30 minutes.
- One hour.
- Until next scheduled event.

Preview crossed deadlines. Waiting in a dangerous location may draw an event.

## Rest

Short rest:

- Default 30 minutes.
- Restores only abilities whose definitions name `ShortRest`.
- Does not restore normal spell slots or sorcery points by default.
- Companion conversation slot.

Long rest:

- Advances to campaign-defined wake time.
- Processes deadlines and night events.
- Removes temporary spell slots created through flexible casting.
- Restores normal spell slots to their level-based maxima.
- Restores sorcery points to their feature-based maximum.
- Restores HP and abilities whose definitions name `LongRest`.
- Updates injuries.

The confirmation dialog lists known consequences before dispatch.

Apply long-rest resource events in that order. Temporary slots must disappear before normal slots refresh so they cannot become permanent through save and load.

## Validator additions

- Location IDs unique.
- Route destinations exist.
- Travel minutes greater than zero.
- Event budget within campaign maximum.
- Entry location reachable.
- Required locations reachable under at least one fixture state.
- Schedule minutes valid.
- No NPC has ambiguous equal-priority slots.
- Event weights greater than zero.

## Tests

- Midnight wraps correctly.
- Crossed schedules emit in order.
- Locked route rejects travel.
- Travel interruption preserves journey.
- Arrival discovers location.
- One-time event never repeats.
- Cooldown event returns after required distance or time.
- Search retry is blocked without changed state.
- Long rest warns and then crosses a deadline after confirmation.
- Long rest removes temporary slots and restores normal slots.
- Long rest restores sorcery points without exceeding their maximum.
- Short rest leaves slots and sorcery points unchanged without a named feature.

## Acceptance check

- Three academy locations connect through validated routes.
- At least one hidden route can be discovered.
- One NPC changes location by schedule.
- One travel event triggers deterministically.
- Waiting and resting change available content.
- Saving mid-journey resumes correctly.

## Suggested commit

```powershell
git add .
git commit -m "Add location graph exploration and world time"
```
