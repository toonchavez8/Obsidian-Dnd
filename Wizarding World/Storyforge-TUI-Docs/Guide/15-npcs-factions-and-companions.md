# 15: Add NPCs, factions, and companions

## Result

NPCs remember specific actions, factions react through named reputation ranks, and up to two companions can join exploration and combat with configurable control.

## Actor definitions

An actor contains:

- Stable content ID.
- Display name and pronouns.
- Description variants.
- Affiliation memberships.
- Relationship axes used.
- Schedule.
- Dialogue entry points.
- Secrets.
- Combat definition.
- Companion definition, when recruitable.
- Arc variants.

Separate immutable definitions from runtime state:

```rust
pub struct ActorState {
    pub relationship: HashMap<RelationshipAxis, i8>,
    pub memories: Vec<Memory>,
    pub schedule_override: Option<ScheduleOverride>,
    pub conditions: Vec<ConditionInstance>,
    pub available: bool,
}
```

## Memories

```rust
pub struct Memory {
    pub id: ContentId,
    pub source_event: u64,
    pub summary: String,
    pub emotion: MemoryEmotion,
    pub arc: ContentId,
    pub discussed: bool,
    pub expires: Option<GameClock>,
}
```

Memory IDs identify the meaning, while `source_event` identifies the occurrence. A repeatable memory may appear more than once only when its definition allows it.

Dialogue conditions can check:

- Has memory.
- Memory count.
- Memory discussed.
- Memory emotion.
- Relationship rank.

Marking a memory discussed is an effect and emits an event.

## Faction definitions

```rust
pub struct FactionDefinition {
    pub id: ContentId,
    pub name: String,
    pub subgroups: Vec<ContentId>,
    pub ranks: Vec<ReputationRank>,
    pub arc_agendas: Vec<ArcAgenda>,
    pub allies: Vec<ContentId>,
    pub rivals: Vec<ContentId>,
}

pub struct ReputationRank {
    pub id: String,
    pub minimum: i16,
    pub display_name: String,
}
```

Reputation clamps to -100 through +100. Ranks must cover the entire range without overlaps or gaps.

## Reputation events

Each change stores:

- Faction.
- Requested amount.
- Applied amount.
- Reason ID.
- Source event.
- New rank.

A rank change may unlock a scene, price policy, location, service, or quest. It does not directly grant unrelated character power.

## Internal faction groups

Do not model political conflict as one species or community with one opinion. Use a parent faction for public reputation and subgroups for policy, leadership, and quest routes.

For example, a labor coalition can contain negotiators, mutual-aid organizers, hardliners, and unaffiliated civilians. Player action can change which subgroup leads without declaring the whole population good or evil.

## Companion state

```rust
pub struct CompanionState {
    pub actor: ContentId,
    pub loyalty: i8,
    pub stress: u8,
    pub personal_quest: Option<ContentId>,
    pub tactic: CompanionTactic,
    pub control: CompanionControl,
    pub inventory: Inventory,
    pub equipment: Equipment,
    pub spellcasting: SpellcastingState,
}

pub enum CompanionControl {
    Direct,
    Suggested,
    Autonomous,
}

pub enum CompanionTactic {
    ProtectPlayer,
    FocusTarget,
    ConserveSpellSlots,
    ReserveSorceryPoints,
    Support,
    AvoidLethalForce,
    PursueObjective,
}
```

Loyalty uses -5 through +5. Stress uses 0 through 100.

## Recruitment

`RecruitCompanion` validates:

- Recruit conditions.
- Current availability.
- Party capacity.
- No departure state.
- Campaign rule compatibility.

Default active party:

- Player.
- Companion one.
- Companion two.

Other recruited companions remain at a hub or follow their schedules.

## Companion approval

Approval changes need an authored reason and normally create a memory. Repeated small actions use cooldowns so buying the same gift cannot farm loyalty.

Disapproval does not automatically cause departure. Departure checks explicit loyalty, boundary memories, personal-quest state, and arc conditions.

## Companion conversations

Conversation opportunities occur:

- During safe short rests.
- At hub locations.
- After major events.
- Before arc transitions.
- When a personal quest updates.

The UI marks a new conversation without revealing its subject.

## Combat control

Direct mode opens the same action menu as the player.

Suggested mode:

1. AI scores legal actions.
2. UI shows proposal and reason.
3. Player confirms or selects another action.

Autonomous mode dispatches the highest-scored action, with deterministic RNG tie-breaking.

Changing control mode is free outside combat and restricted to the player's turn during combat.

## Secrets

Secrets are content IDs with reveal conditions. An NPC can know, suspect, deny, or have forgotten a secret.

Information-gathering checks reveal authored confidence:

- Rumor.
- Partial.
- Credible.
- Confirmed.

The journal records source and confidence. It does not convert every rumor into truth.

## Faction dashboard

Show:

- Current rank.
- Reputation meter.
- Known goals.
- Known subgroups.
- Recent reasons.
- Active faction quests.
- Known allies and rivals.

Unknown motives stay hidden.

## Validator additions

- Relationship starting values in range.
- Memory IDs valid and repeat policy defined.
- Faction ranks fully cover -100 through +100.
- Subgroup references exist.
- Ally and rival relationships are consistent or explicitly one-sided.
- Companion tactics reference supported capabilities.
- Personal quest exists.
- Departure scenes exist.
- Secret reveal sources exist.

## Tests

- Relationship clamping emits applied amount.
- Memory cooldown blocks duplicate gain.
- Discussed memory persists.
- Reputation rank changes at exact threshold.
- Recruitment respects party capacity.
- Companion departure cannot duplicate.
- Suggested action is legal.
- Direct and autonomous modes reach equivalent reducer paths.
- Save/resume preserves memories, rank, loyalty, stress, and equipment.

## Acceptance check

- Two NPCs remember different MVP choices.
- Two factions expose ranks and recent reasons.
- One companion can recruit and leave the active party.
- Suggested combat control explains its action.
- One personal quest condition reacts to a memory.
- A faction leadership flag changes later content without rewriting reputation history.

## Suggested commit

```powershell
git add .
git commit -m "Add persistent NPC faction and companion state"
```
