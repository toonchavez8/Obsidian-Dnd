# 17: Build the three-arc campaign foundation

## Result

The engine will carry state from levels 1 through 20 across Reconstruction, Fracture, and Convergence without hardcoding a plot. Campaign data controls entry gates, world changes, planar packs, and endings.

## Build this chapter in five checkpoints

| Checkpoint | Destination | Proof before continuing |
| --- | --- | --- |
| 1 | `storyforge-core/src/campaign_progress.rs` | Initial arc, decision, pressure, and phase tests |
| 2 | `storyforge-core/src/arc_transition.rs` | Preview, cancel, confirm, and exactly-once tests |
| 3 | `storyforge-core/src/ending.rs` | Eligibility, priority, tie, and epilogue fallback tests |
| 4 | `storyforge-content/src/model/` | Arc graph, thresholds, location packs, and endings validate |
| 5 | `storyforge-content/tests/three_arc_playthrough.rs` | One fixture carries decisions through all three arcs and an ending |

This chapter proves the engine shape with fixtures. It does not require writing the full private story.

## Campaign commands and events

Create `storyforge-core/src/campaign_command.rs`:

```rust
use crate::ContentId;

#[derive(Debug, Clone, PartialEq, Eq)]
pub enum CampaignCommand {
    RecordDecision {
        decision: ContentId,
        summary: String,
        tags: Vec<String>,
    },
    ChangePressure {
        pressure: ContentId,
        amount: i16,
        reason: ContentId,
    },
    BeginArcTransition {
        destination: ContentId,
    },
    ConfirmArcTransition {
        destination: ContentId,
    },
    CancelArcTransition,
    EvaluateEndings,
    ChooseEnding {
        ending: ContentId,
    },
}

#[derive(
    Debug, Clone, PartialEq, Eq, serde::Serialize, serde::Deserialize,
)]
pub enum CampaignEvent {
    DecisionRecorded {
        decision: DecisionRecord,
    },
    PressureChanged {
        pressure: ContentId,
        requested: i16,
        applied: i16,
        current: i16,
        reason: ContentId,
    },
    PressureThresholdExited {
        pressure: ContentId,
        threshold: String,
    },
    PressureThresholdEntered {
        pressure: ContentId,
        threshold: String,
    },
    ArcTransitionPreviewed {
        source: ContentId,
        destination: ContentId,
        blockers: Vec<String>,
        companion_conversations: Vec<ContentId>,
        packs_to_unlock: Vec<ContentId>,
    },
    ArcCompleted {
        arc: ContentId,
    },
    ActiveArcChanged {
        previous: ContentId,
        current: ContentId,
    },
    WorldPhaseChanged {
        previous: ContentId,
        current: ContentId,
    },
    LocationPackUnlocked {
        pack: ContentId,
    },
    EndingCandidatesFound {
        endings: Vec<ContentId>,
    },
    EndingChosen {
        ending: ContentId,
        final_scene: ContentId,
    },
    CampaignCommandRejected {
        reason: String,
    },
    ProtectedAutosaveRequested {
        reason: String,
    },
}
```

Add `Campaign(CampaignCommand),` to the root `GameCommand` and `Campaign(CampaignEvent),` to `GameEvent`.

| Command | Validation | Successful behavior |
| --- | --- | --- |
| `RecordDecision` | Definition exists; occurrence policy permits it | Appends a decision with source event and active arc |
| `ChangePressure` | Meter and reason exist; arithmetic is safe | Clamps requested change and emits exit then enter threshold events |
| `BeginArcTransition` | Destination is the defined next arc and current scene is safe | Emits preview with every blocker and consequence; changes no state |
| `ConfirmArcTransition` | Matching preview exists and blockers are empty | Completes old arc, changes phase, unlocks packs, enters new scene, protected autosave |
| `CancelArcTransition` | Preview exists | Clears preview only |
| `EvaluateEndings` | Campaign is in an ending-eligible phase | Filters conditions, resolves priority by family, emits stable candidates |
| `ChooseEnding` | ID is in the latest candidate set | Stores the ending and enters its final scene |

`BeginArcTransition` and `ConfirmArcTransition` are separate commands so the TUI cannot bypass companion warnings, unfinished required quests, pack compatibility, or the protected backup.

## Arc definitions

```rust
pub struct CampaignArcDefinition {
    pub id: ContentId,
    pub name: String,
    pub level_range: InclusiveLevelRange,
    pub enter_when: Vec<ConditionDefinition>,
    pub entry_scene: ContentId,
    pub location_packs: Vec<ContentId>,
    pub event_decks: Vec<ContentId>,
    pub world_rules: Vec<WorldRuleDefinition>,
    pub complete_when: Vec<ConditionDefinition>,
    pub transition_scene: Option<ContentId>,
}

pub struct InclusiveLevelRange {
    pub minimum: u8,
    pub maximum: u8,
}
```

Default original campaign arcs:

```text
academy.arc.reconstruction  levels 1-10
academy.arc.fracture        levels 11-16
academy.arc.convergence     levels 17-20
```

Level is necessary but not sufficient for transition. Main-quest and world conditions must also pass.

## Runtime campaign progress

```rust
pub struct CampaignProgress {
    pub active_arc: ContentId,
    pub completed_arcs: Vec<ContentId>,
    pub world_phase: ContentId,
    pub pressures: HashMap<ContentId, i16>,
    pub irreversible_decisions: Vec<DecisionRecord>,
    pub unlocked_location_packs: HashSet<ContentId>,
}

#[derive(Debug, Clone, PartialEq, Eq, serde::Serialize, serde::Deserialize)]
pub struct DecisionRecord {
    pub id: ContentId,
    pub source_event: u64,
    pub arc: ContentId,
    pub summary: String,
    pub tags: Vec<String>,
}
```

Decision records are append-only. A later reconciliation adds another decision; it does not delete the original betrayal or promise.

## World pressure

Generic engine state supports named meters. The original campaign can define:

- Institutional stability.
- Civil tension.
- Planar instability.
- Public fear.
- Coalition strength.

Each pressure clamps to its definition's range and has thresholds:

```rust
pub struct PressureThreshold {
    pub id: String,
    pub minimum: i16,
    pub maximum: i16,
    pub enter_effects: Vec<EffectDefinition>,
    pub exit_effects: Vec<EffectDefinition>,
}
```

Crossing a threshold emits an event. Re-entering can fire again only when the threshold definition permits it.

## World phases

A world phase is a named bundle of variants:

- Location descriptions.
- Ambient events.
- NPC schedules.
- Shop scarcity.
- Travel danger.
- Faction agenda.
- Available quests.
- Music and art.

Pressure may make a phase eligible, but a scene or quest confirms the transition. That gives the story room to show the change.

## Location packs and planes

Arc I loads the school hub pack.

Arc II can unlock:

- Regional settlements.
- Ministry or civic center.
- Banking or trade district.
- Wilderness routes.
- First planar destination.

Arc III can unlock:

- Convergence variants of existing locations.
- Several planar packs.
- Endgame arenas.
- Epilogue locations.

A save records every active pack and fingerprint. Missing packs produce a compatibility screen rather than silently removing locations.

## Faction evolution

Faction definitions include arc agendas:

```rust
pub struct ArcAgenda {
    pub arc: ContentId,
    pub leadership_variants: Vec<LeadershipVariant>,
    pub goals: Vec<FactionGoal>,
    pub default_event_deck: ContentId,
}
```

Leadership and goals are runtime state. Reputation remains a ledger across changes. Supporting one leader does not reset prior treatment of the faction.

## Companion continuity

Before an arc transition:

1. Resolve required companion conversations.
2. Evaluate departure and loyalty gates.
3. Record unresolved personal quests.
4. Choose time-skip variants.
5. Migrate equipment rules if the new arc changes limits.
6. Enter the transition scene.
7. Apply the new arc.
8. Autosave to a protected transition slot.

Keep a backup from before the transition so migration bugs are recoverable.

## Arc transition command

Only dispatch `BeginArcTransition` from an eligible safe scene.

Core:

1. Confirms active arc completion.
2. Confirms destination is the defined next arc.
3. Evaluates blockers.
4. Emits a preview event.
5. Requires confirmation.
6. Records completed arc.
7. Applies transition effects.
8. Changes active arc and world phase.
9. Unlocks packs.
10. Moves to entry scene.

## Ending rules

```rust
pub struct EndingDefinition {
    pub id: ContentId,
    pub family: ContentId,
    pub priority: i16,
    pub eligible_when: Vec<ConditionDefinition>,
    pub final_scene: ContentId,
    pub world_epilogue: ContentId,
    pub companion_epilogues: Vec<CompanionEpilogueRule>,
    pub faction_epilogues: Vec<FactionEpilogueRule>,
    pub unlocks: Vec<EffectDefinition>,
}
```

Evaluation:

1. Filter eligible endings.
2. Group by family.
3. Sort by descending priority then stable ID.
4. Reject unresolved equal-priority conflicts during validation.
5. Present an authored final decision when required.
6. Record chosen ending.
7. Resolve epilogues from state.

The final menu cannot overwrite accumulated state. It selects among outcomes already made eligible.

## Three-arc capability matrix

| Capability | Reconstruction | Fracture | Convergence |
| --- | --- | --- | --- |
| Locations | School and grounds | Regions and first planes | Convergence and endgame packs |
| Factions | Introduction and internal groups | Alliances and leadership conflict | Final coalition state |
| Companions | Recruit and establish bonds | Loyalty and departure | Final commitment and epilogue |
| Pressure | Early warning | Open world changes | Ending eligibility |
| Combat | Duels and local threats | Group objectives and war events | Multi-stage endgame encounters |
| Choices | Personal and local | Political and strategic | World-defining |

Put this matrix in the code repository README as the long-term roadmap.

## Save compatibility

Add save fixtures at:

- End of Arc I.
- Start of Arc II.
- End of Arc II.
- Start of Arc III.
- Before ending evaluation.
- After each ending family.

Every schema migration runs against every fixture.

Do not wait until Arc II content exists to add `active_arc`, pressures, decisions, and pack lists to saves. Add empty or initial values before public schema 1 freezes.

## Validator additions

- Arc level ranges valid and ordered.
- Entry and transition scenes exist.
- Arc graph contains no accidental cycle.
- Required location packs exist.
- World pressure thresholds cover their range.
- Ending priorities have no ambiguous tie.
- Every ending family has an eligible fixture.
- Companion epilogue has a fallback.
- Main campaign has at least one complete path through all arcs.

## Tests

- Level alone cannot force transition.
- Transition applies once.
- Decision records remain append-only.
- Pressure threshold emits enter and exit events correctly.
- Missing pack blocks load safely.
- Faction reputation survives leadership change.
- Companion departure persists into next arc.
- Ending selection is deterministic.
- Epilogue fallback covers an untracked NPC state.
- Arc save fixtures migrate to current schema.

## Acceptance check

- Initial saves contain Arc I progress fields.
- A test fixture transitions through all three arcs.
- One Arc I decision changes an Arc II faction variant.
- One Arc II alliance changes Arc III ending eligibility.
- At least three ending families resolve through fixtures.
- README documents the three-arc roadmap.

## Suggested commit

```powershell
git add .
git commit -m "Add three arc world progression and endings"
```
