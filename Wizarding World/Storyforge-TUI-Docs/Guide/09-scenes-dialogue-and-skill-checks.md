# 09: Add scenes, dialogue, and visible checks

## Result

Campaign data can present prose and choices, test conditions, apply effects, roll a visible check, and move to another scene.

## Build this chapter in four checkpoints

| Checkpoint | Destination | Proof before continuing |
| --- | --- | --- |
| 1 | `storyforge-content/src/model/scene.rs` | Valid and invalid scene fixtures |
| 2 | `storyforge-core/src/scene.rs` | Visibility, transition, and effect tests |
| 3 | `storyforge-core/src/check.rs` | Normal, advantage, disadvantage, and modifier tests |
| 4 | `storyforge-tui/src/screens/story.rs` | Choice, roll reveal, success, and failure playthrough |

## Scene commands and events

Create `storyforge-core/src/scene_command.rs`:

```rust
use crate::ContentId;

#[derive(Debug, Clone, PartialEq, Eq)]
pub enum SceneCommand {
    EnterScene {
        scene: ContentId,
    },
    ChooseSceneOption {
        scene: ContentId,
        choice: String,
    },
    ContinueAfterCheck {
        scene: ContentId,
        choice: String,
    },
}

#[derive(
    Debug, Clone, PartialEq, Eq, serde::Serialize, serde::Deserialize,
)]
pub enum SceneEvent {
    SceneEntered {
        previous: ContentId,
        current: ContentId,
    },
    ChoiceSelected {
        scene: ContentId,
        choice: String,
    },
    CheckRolled {
        skill: ContentId,
        dice: Vec<u8>,
        kept_die: u8,
        modifier: i16,
        total: i16,
        target: i16,
        success: bool,
    },
    EffectApplied {
        effect_index: usize,
        source_scene: ContentId,
    },
    SceneCommandRejected {
        reason: String,
    },
    AutosaveRequested {
        reason: String,
    },
}
```

Add `Scene(SceneCommand),` to `GameCommand` and `Scene(SceneEvent),` to `GameEvent`.

| Command | Validation | Successful behavior |
| --- | --- | --- |
| `EnterScene` | Scene exists and its entry conditions pass | Records scene history, resets choice selection, renders current scene |
| `ChooseSceneOption` | Scene is still active; choice exists, is visible, and is enabled | Emits selection, effects, optional check, transition, and autosave events |
| `ContinueAfterCheck` | Matching unresolved check result is active | Applies only the stored success or failure branch; never rerolls |

The engine stores a pending check result before the roll animation begins. The TUI reveals those stored dice over several frames. Skipping the animation changes only presentation and cannot reroll the check.

## Extend scene content

Replace the chapter 06 `ChoiceDefinition` in `storyforge-content/src/model.rs` with:

```rust
#[derive(Debug, Clone, serde::Deserialize)]
pub struct ChoiceDefinition {
    pub id: String,
    pub label: String,
    pub conditions: Vec<ConditionDefinition>,
    pub outcome: ChoiceOutcome,
}

#[derive(Debug, Clone, serde::Deserialize)]
pub enum ChoiceOutcome {
    Transition {
        target: ContentId,
        effects: Vec<EffectDefinition>,
    },
    Check {
        check: CheckDefinition,
        success: BranchDefinition,
        failure: BranchDefinition,
    },
}
```

Start with a limited condition set:

```rust
#[derive(
    Debug, Clone, PartialEq, Eq, serde::Serialize, serde::Deserialize,
)]
pub enum ConditionDefinition {
    FlagSet(ContentId),
    FlagNotSet(ContentId),
    HasItem(ContentId),
    SkillProficient(ContentId),
    RelationshipAtLeast {
        actor: ContentId,
        axis: RelationshipAxis,
        value: i8,
    },
}
```

Initial effects:

```rust
#[derive(
    Debug, Clone, PartialEq, Eq, serde::Serialize, serde::Deserialize,
)]
pub enum EffectDefinition {
    SetFlag(ContentId),
    ClearFlag(ContentId),
    GiveItem(ContentId),
    RemoveItem(ContentId),
    ChangeRelationship {
        actor: ContentId,
        axis: RelationshipAxis,
        amount: i8,
    },
    StartQuest(ContentId),
    AdvanceTime { minutes: u32 },
}
```

Conditions only read state. Effects only describe changes. Core performs validation, clamping, and event emission.

## Check definition

```rust
#[derive(
    Debug, Clone, PartialEq, Eq, serde::Serialize, serde::Deserialize,
)]
pub struct CheckDefinition {
    pub skill: ContentId,
    pub ability: Ability,
    pub target: i16,
    pub advantage: AdvantageRule,
    pub critical_success: Option<i16>,
    pub critical_failure: Option<i16>,
}

#[derive(
    Debug, Clone, Copy, PartialEq, Eq, serde::Serialize, serde::Deserialize,
)]
pub enum AdvantageRule {
    Normal,
    Advantage,
    Disadvantage,
}
```

Situational advantage from state is combined with the definition at runtime. One advantage and one disadvantage source cancel.

## Dice result

Core emits a complete result:

```rust
#[derive(Debug, Clone, PartialEq, Eq, serde::Serialize, serde::Deserialize)]
pub struct CheckResult {
    pub skill: ContentId,
    pub rolls: Vec<u8>,
    pub kept_roll: u8,
    pub ability_modifier: i8,
    pub proficiency_modifier: i8,
    pub situational_modifier: i8,
    pub total: i16,
    pub target: i16,
    pub degree: OutcomeDegree,
    pub reasons: Vec<String>,
}
```

`reasons` contains player-facing causes such as companion help or injury. It must not contain terminal formatting.

## Roll order

When confirming a check choice:

1. Verify the choice is visible and enabled.
2. Build the check context from current state.
3. Determine advantage state.
4. Roll through engine RNG.
5. Calculate total and degree.
6. Emit `CheckRolled`.
7. Apply success or failure effects.
8. Emit each resulting state event.
9. Transition scene.
10. Request autosave if the destination is safe.

No effect applies before the roll event.

## Roll reveal UI

Show a short staged reveal when motion is enabled:

```text
Investigation

d20              17
Intelligence     +2
Proficiency      +2
Burned hands     -1
Total            20
Target           15

SUCCESS
```

Reduced-motion mode shows the complete result immediately. Enter skips the animation.

The log receives a concise line:

```text
Investigation 20 vs 15: success.
```

## Choice visibility

Each choice has a presentation status:

- Visible and enabled.
- Visible and disabled with a known reason.
- Hidden.

A failed condition does not automatically mean hidden. Content authors choose reveal policy so the player can learn that an option requires a spell, item, relationship, or clue.

## Relationship state

Use bounded values:

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash, serde::Serialize, serde::Deserialize)]
pub enum RelationshipAxis {
    Trust,
    Respect,
    Fear,
    Affection,
    Rivalry,
    Obligation,
}
```

Store only axes used by the actor. Apply changes with `clamp(-5, 5)` and emit both requested and applied amounts if clamping changed the result.

## Scene history

Record:

- Entered scene ID.
- Chosen choice ID.
- Check result.
- Important effects.
- Transition target.

This is separate from prose scrollback. It supports debugging, achievements, ending conditions, and the character-history tab.

## Validator additions

Validate:

- Skills exist.
- Targets use reasonable campaign limits.
- Success and failure targets exist.
- Referenced items, actors, quests, and flags are valid.
- Relationship values stay between -5 and +5.
- A choice cannot both hide and display a requirement.
- A scene does not create an unconditional loop unless marked repeatable.

## Tests

Required cases:

- The same seed rolls the same result.
- Proficiency applies once.
- Advantage keeps the higher roll.
- Disadvantage keeps the lower roll.
- Advantage and disadvantage cancel.
- Success follows the success branch.
- Failure follows the failure branch.
- Hidden choices cannot be selected by ID.
- Disabled choices emit rejection.
- Relationship effects clamp and report applied amount.
- Scene history records choice and result.

## Demo scene

Create a gate search:

- Investigation target 12.
- Success discovers a side passage, gives Iren trust, and advances ten minutes.
- Failure opens the route but rings a warning bell, sets an alarm flag, and advances twenty minutes.

Both outcomes continue. The route differs later.

## Verify

```powershell
cargo run -p storyforge-tui -- validate --pack campaigns/academy-demo --strict
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features --locked -- -D warnings
cargo test --workspace --all-features --locked
cargo run -p storyforge-tui -- play --pack campaigns/academy-demo
```

## Common mistakes

- The TUI decides whether a check succeeded.
- Effects mutate state without events.
- A disabled choice can be selected through a number key.
- Failure ends the scene with no new lead.
- Roll animation consumes another random number.
- Relationship values are unbounded.

## Acceptance check

- One demo choice performs a visible deterministic check.
- Success and failure both continue through different state.
- Conditions control choice presentation.
- Every effect emits an event.
- Save and reload preserve the result and history.

## Suggested commit

```powershell
git add .
git commit -m "Add branching scenes and visible skill checks"
```
