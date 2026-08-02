# 07: Build character creation as a playable shopping day

## Result

Character creation becomes a playable prologue instead of one large form. The player starts with identity, chooses early character direction, enters Lantern Row, collects required supplies, meets a companion, tries wands, reviews the draft, and then creates a finished `PlayerCharacter`.

This guide does not implement the full end-goal creator. It builds the smallest useful state machine:

```text
CharacterCreationState draft
    -> CreationCommand
    -> CreationEvent
    -> apply_creation_events
    -> optional PlayerCharacter at confirmation
```

The important rule is this: an unfinished character is not a `PlayerCharacter`. It is a draft. Only confirmation can produce the permanent character.

## Flow To Prove

The flow uses these functions and types:

```text
CreationCommand
    -> handle_creation_command
    -> CreationEvent list
    -> apply_creation_events
    -> CharacterCreationState
    -> complete_character
    -> CreationOutcome
```

First prove data reaches `handle_creation_command`. Then log or inspect the event it returns. Then apply the event to the draft. Then validate the draft. Only after that should confirmation build a finished character.

---

File: `crates/storyforge-core/src/character.rs`

## Step 1: Add permanent character types

### Current state

`storyforge-core` has engine state from Guide 05, but no character model. The TUI still displays temporary spell-slot values from `App`.

### Why this step matters

A finished character needs stable saved fields: name, abilities, background, casting style, equipment, wand, companion, HP, and spellcasting state. These belong in core because they are gameplay facts, not UI widgets.

### Before

Create:

```text
crates/storyforge-core/src/character.rs
```

### What to change

Add permanent character types only. Do not put optional draft choices here.

### Temporary MVP / debug behavior

Start with enough to render a future character sheet and prove spell-slot style casting. Do not implement leveling, inventory, combat, or full derived stats yet.

### After

```rust
use serde::{Deserialize, Serialize};

use crate::ContentId;

#[derive(Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize)]
pub enum Ability {
    Strength,
    Dexterity,
    Constitution,
    Intelligence,
    Wisdom,
    Charisma,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize)]
pub enum CastingStyle {
    Willpower,
    Technique,
    Intellect,
}

impl CastingStyle {
    #[must_use]
    pub const fn casting_ability(self) -> Ability {
        match self {
            Self::Willpower => Ability::Charisma,
            Self::Technique => Ability::Wisdom,
            Self::Intellect => Ability::Intelligence,
        }
    }
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct AbilityScores {
    pub strength: u8,
    pub dexterity: u8,
    pub constitution: u8,
    pub intelligence: u8,
    pub wisdom: u8,
    pub charisma: u8,
}

impl AbilityScores {
    /// Checks the normal supported range for a player ability.
    ///
    /// # Errors
    ///
    /// Returns [`CharacterError::AbilityOutOfRange`] when any score is lower
    /// than 3 or greater than 20.
    pub fn validate(&self) -> Result<(), CharacterError> {
        let scores = [
            self.strength,
            self.dexterity,
            self.constitution,
            self.intelligence,
            self.wisdom,
            self.charisma,
        ];

        if scores.into_iter().all(|score| (3..=20).contains(&score)) {
            Ok(())
        } else {
            Err(CharacterError::AbilityOutOfRange)
        }
    }
}

#[derive(Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize)]
pub struct SpellSlotState {
    pub remaining: u8,
    pub maximum: u8,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct SpellcastingState {
    pub casting_ability: Ability,
    pub known_cantrips: Vec<ContentId>,
    pub known_spells: Vec<ContentId>,
    pub prepared_spells: Vec<ContentId>,
    pub slots: [SpellSlotState; 9],
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct PlayerCharacter {
    pub name: String,
    pub pronouns: String,
    pub level: u8,
    pub experience: u32,
    pub abilities: AbilityScores,
    pub background: ContentId,
    pub casting_style: CastingStyle,
    pub study_interest: ContentId,
    pub affiliation: ContentId,
    pub equipment_package: ContentId,
    pub wand: ContentId,
    pub companion: ContentId,
    pub traits: Vec<ContentId>,
    pub skill_proficiencies: Vec<ContentId>,
    pub hp: i32,
    pub spellcasting: SpellcastingState,
}

#[derive(Debug, Clone, PartialEq, Eq, thiserror::Error)]
pub enum CharacterError {
    #[error("character name must contain between 1 and 32 visible characters")]
    InvalidName,
    #[error("ability scores must be between 3 and 20")]
    AbilityOutOfRange,
    #[error("exactly two personality traits are required")]
    TraitCount,
    #[error("draft is missing required field `{0}`")]
    MissingField(&'static str),
}

#[must_use]
pub const fn ability_modifier(score: u8) -> i8 {
    (i16::from(score).saturating_sub(10).div_euclid(2)) as i8
}

#[must_use]
pub const fn proficiency_bonus(level: u8) -> i8 {
    match level {
        1..=4 => 2,
        5..=8 => 3,
        9..=12 => 4,
        13..=16 => 5,
        _ => 6,
    }
}
```

### Learning checkpoint

You should understand that spell slots are counts, not mana. The array has nine entries, one for each leveled-spell tier. Index `0` is first-level slots.

### How to verify

Compilation waits until `lib.rs` exports this module.

### Next connection

Add tests for the math before building the creation draft.

---

File: `crates/storyforge-core/tests/character_math.rs`

## Step 2: Test ability and proficiency math

### Current state

No character tests exist yet.

### Why this step matters

Small math rules should be tested directly. If the character sheet later displays a wrong modifier, you want a core test to catch it.

### Before

Create:

```text
crates/storyforge-core/tests/character_math.rs
```

### What to change

Add tests for ability modifiers and proficiency boundaries.

### Temporary MVP / debug behavior

Test only pure math first. Do not build a whole character just to test `ability_modifier`.

### After

```rust
use storyforge_core::{ability_modifier, proficiency_bonus};

#[test]
fn ability_modifiers_round_down_for_odd_scores() {
    let cases = [(8, -1), (9, -1), (10, 0), (11, 0), (12, 1), (18, 4)];

    for (score, expected) in cases {
        assert_eq!(ability_modifier(score), expected, "score {score}");
    }
}

#[test]
fn proficiency_changes_at_level_boundaries() {
    let cases = [
        (1, 2), (4, 2), (5, 3), (8, 3), (9, 4),
        (12, 4), (13, 5), (16, 5), (17, 6), (20, 6),
    ];

    for (level, expected) in cases {
        assert_eq!(proficiency_bonus(level), expected, "level {level}");
    }
}
```

### Learning checkpoint

You should understand that these functions are pure: same input, same output, no state, no RNG, no content files.

### How to verify

After exporting from `lib.rs`, run:

```powershell
cargo test -p storyforge-core --test character_math
```

### Next connection

Now add the draft state that holds unfinished choices.

---

File: `crates/storyforge-core/src/creation.rs`

## Step 3: Keep unfinished choices in a draft

### Current state

There is no creation state. A half-created character would be impossible to represent without misusing `PlayerCharacter`.

### Why this step matters

The game sheet says character creation can be suspended and resumed. That requires a serializable draft with missing fields allowed.

### Before

Create:

```text
crates/storyforge-core/src/creation.rs
```

### What to change

Add `CreationStage`, `CharacterDraft`, `CharacterCreationState`, `CharacterProblem`, and `CreationOutcome`.

### Temporary MVP / debug behavior

Use `CharacterCreationState::problems()` to print or inspect what is missing before confirmation. That gives you a debug checklist before implementing the full UI.

### After

```rust
use std::collections::BTreeSet;

use serde::{Deserialize, Serialize};

use crate::{CastingStyle, ContentId, PlayerCharacter};

#[derive(Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize)]
pub enum CreationStage {
    Identity,
    Background,
    CastingStyle,
    MarketArrival,
    EquipmentShop,
    CompanionShop,
    WandShop,
    Review,
    Complete,
}

impl Default for CreationStage {
    fn default() -> Self {
        Self::Identity
    }
}

#[derive(Debug, Clone, Default, PartialEq, Eq, Serialize, Deserialize)]
pub struct CharacterDraft {
    pub name: String,
    pub pronouns: String,
    pub background: Option<ContentId>,
    pub casting_style: Option<CastingStyle>,
    pub study_interest: Option<ContentId>,
    pub affiliation: Option<ContentId>,
    pub equipment_package: Option<ContentId>,
    pub companion: Option<ContentId>,
    pub wand: Option<ContentId>,
    pub traits: Vec<ContentId>,
    pub skill_proficiencies: Vec<ContentId>,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct CharacterCreationState {
    pub stage: CreationStage,
    pub draft: CharacterDraft,
    pub visited_shops: BTreeSet<ContentId>,
    pub inspected_companions: BTreeSet<ContentId>,
    pub wand_candidates: Vec<ContentId>,
    pub candidate_seed: u64,
}
```

Continue `creation.rs` with the constructor and problem checker:

```rust
impl CharacterCreationState {
    #[must_use]
    pub fn new(candidate_seed: u64) -> Self {
        Self {
            stage: CreationStage::Identity,
            draft: CharacterDraft::default(),
            visited_shops: BTreeSet::new(),
            inspected_companions: BTreeSet::new(),
            wand_candidates: Vec::new(),
            candidate_seed,
        }
    }

    #[must_use]
    pub fn problems(&self) -> Vec<CharacterProblem> {
        let mut problems = Vec::new();
        let visible_name_length = self.draft.name.trim().chars().count();

        if !(1..=32).contains(&visible_name_length) {
            problems.push(CharacterProblem::new(
                "name",
                "Enter a name containing 1 to 32 characters.",
            ));
        }
        if self.draft.pronouns.trim().is_empty() {
            problems.push(CharacterProblem::new(
                "pronouns",
                "Choose or enter the character's pronouns.",
            ));
        }
        if self.draft.background.is_none() {
            problems.push(CharacterProblem::new("background", "Choose a background."));
        }
        if self.draft.traits.len() != 2 {
            problems.push(CharacterProblem::new(
                "traits",
                "Choose exactly two personality traits.",
            ));
        }
        if self.draft.casting_style.is_none() {
            problems.push(CharacterProblem::new(
                "casting_style",
                "Choose Willpower, Technique, or Intellect.",
            ));
        }
        for (field, value, message) in [
            ("study_interest", &self.draft.study_interest, "Choose a field of study."),
            ("affiliation", &self.draft.affiliation, "Choose an affiliation preference."),
            ("equipment_package", &self.draft.equipment_package, "Collect an equipment package."),
            ("companion", &self.draft.companion, "Choose a pet or familiar."),
            ("wand", &self.draft.wand, "Complete a wand trial."),
        ] {
            if value.is_none() {
                problems.push(CharacterProblem::new(field, message));
            }
        }

        problems
    }
}

#[derive(Debug, Clone, PartialEq, Eq)]
pub struct CharacterProblem {
    pub field: &'static str,
    pub message: String,
}

impl CharacterProblem {
    #[must_use]
    pub fn new(field: &'static str, message: impl Into<String>) -> Self {
        Self {
            field,
            message: message.into(),
        }
    }
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct CreationOutcome {
    pub character: PlayerCharacter,
    pub opening_scene: ContentId,
}
```

### Learning checkpoint

You should understand why `BTreeSet` is used instead of `HashSet`: stable ordering makes debug output, saves, and tests easier to compare.

### How to verify

Compilation waits until `lib.rs` exports the module.

### Next connection

Expose the character and creation modules.

---

File: `crates/storyforge-core/src/lib.rs`

## Step 4: Export character and creation types

### Current state

`lib.rs` exports the engine API from Guide 05. It does not know about `character.rs` or `creation.rs`.

### Why this step matters

Tests, the TUI, and future content validation need access to the public character and creation types.

### Before

You should already have module declarations similar to:

```rust
mod command;
mod engine;
mod id;
mod state;
```

### What to change

Add `mod character;` and `mod creation;`, then export the public types and functions.

### Temporary MVP / debug behavior

Export the complete public surface for this guide. Keep helper functions private until a test or another crate needs them.

### After

```rust
mod character;
mod creation;
mod command;
mod engine;
mod id;
mod state;

pub use character::{
    Ability, AbilityScores, CastingStyle, CharacterError, PlayerCharacter, SpellSlotState,
    SpellcastingState, ability_modifier, proficiency_bonus,
};
pub use creation::{
    CharacterCreationState, CharacterDraft, CharacterProblem, CreationOutcome, CreationStage,
};
pub use command::GameCommand;
pub use engine::{GameEngine, apply_events, handle_command};
pub use id::{ContentId, IdError};
pub use state::{GameEvent, GameState};
```

Keep `engine_name()` if it still exists.

### Learning checkpoint

You should understand that adding a source file is not enough in Rust. The crate root decides which modules exist and which items are public.

### How to verify

Run:

```powershell
cargo test -p storyforge-core --test character_math
cargo check -p storyforge-core
```

### Next connection

Now define creation commands and events.

---

File: `crates/storyforge-core/src/creation.rs`

## Step 5: Define creation commands and events

### Current state

`CharacterCreationState` can store a draft, but nothing can change it yet.

### Why this step matters

Commands say what the player is trying to do. Events say what actually happened. This mirrors the engine pattern from Guide 05.

### Before

Add these types below `CreationOutcome`.

### What to change

Add `CreationCommand` and `CreationEvent`.

### Temporary MVP / debug behavior

Do not connect keyboard input yet. In tests, manually construct commands and inspect returned events.

### After

```rust
#[derive(Debug, Clone, PartialEq, Eq)]
pub enum CreationCommand {
    SetIdentity { name: String, pronouns: String },
    ChooseBackground { background: ContentId },
    ToggleTrait { trait_id: ContentId },
    ChooseCastingStyle { style: CastingStyle },
    ChooseStudyInterest { interest: ContentId },
    ChooseAffiliationPreference { affiliation: ContentId },
    EnterShop { shop: ContentId },
    ChooseEquipmentPackage { package: ContentId },
    InspectCompanion { companion: ContentId },
    AdoptCompanion { companion: ContentId },
    BeginWandTrial,
    SelectWand { wand: ContentId },
    ReturnTo { stage: CreationStage },
    ConfirmCharacter,
}

#[derive(Debug, Clone, PartialEq, Eq)]
pub enum CreationEvent {
    IdentitySet { name: String, pronouns: String },
    BackgroundChosen { background: ContentId },
    TraitAdded { trait_id: ContentId },
    TraitRemoved { trait_id: ContentId },
    CastingStyleChosen { style: CastingStyle },
    StudyInterestChosen { interest: ContentId },
    AffiliationPreferenceChosen { affiliation: ContentId },
    ShopEntered { shop: ContentId, next_stage: CreationStage },
    EquipmentPackageChosen { package: ContentId },
    CompanionInspected { companion: ContentId },
    CompanionAdopted { companion: ContentId },
    WandCandidatesGenerated { candidates: Vec<ContentId> },
    WandChosen { wand: ContentId },
    CreationStageChanged { stage: CreationStage },
    CreationRejected { problems: Vec<CharacterProblem> },
    CharacterCreated { outcome: Box<CreationOutcome> },
    AutosaveRequested { reason: String },
}
```

### Learning checkpoint

You should understand that no keyboard keys appear in `CreationCommand`. The TUI can map Enter, buttons, or mouse clicks to these same commands later.

### How to verify

Run:

```powershell
cargo check -p storyforge-core
```

This may fail until you export the new types in Step 10. If so, continue and export them after the handler exists.

### Next connection

Add a narrow catalog so the handler can validate content IDs.

---

File: `crates/storyforge-core/src/creation.rs`

## Step 6: Add creation catalogs and helpers

### Current state

Commands may contain IDs, but the handler needs to know which IDs are valid for this campaign.

### Why this step matters

Core can validate choices without knowing how files were loaded. The TUI or content layer passes a narrow catalog view into the command handler.

### Before

Add these helpers below the event enum.

### What to change

Add `CreationCatalog`, `contains_id`, `rejected`, and `wand_candidates`.

### Temporary MVP / debug behavior

Use simple slices of `ContentId`. Do not build a full registry yet.

### After

```rust
#[derive(Debug)]
pub struct CreationCatalog<'a> {
    pub backgrounds: &'a [ContentId],
    pub traits: &'a [ContentId],
    pub studies: &'a [ContentId],
    pub affiliations: &'a [ContentId],
    pub shops: &'a [ContentId],
    pub equipment_packages: &'a [ContentId],
    pub companions: &'a [ContentId],
    pub wands: &'a [ContentId],
    pub opening_scene: &'a ContentId,
}

fn contains_id(catalog: &[ContentId], requested: &ContentId) -> bool {
    catalog.iter().any(|candidate| candidate == requested)
}

fn rejected(field: &'static str, message: impl Into<String>) -> Vec<CreationEvent> {
    vec![CreationEvent::CreationRejected {
        problems: vec![CharacterProblem::new(field, message)],
    }]
}

#[must_use]
pub fn wand_candidates(seed: u64, wands: &[ContentId]) -> Vec<ContentId> {
    if wands.is_empty() {
        return Vec::new();
    }

    let start = usize::try_from(seed % wands.len() as u64).unwrap_or(0);
    (0..wands.len().min(3))
        .map(|offset| wands[(start + offset) % wands.len()].clone())
        .collect()
}
```

### Learning checkpoint

You should understand that `CreationCatalog<'a>` borrows slices. It does not own or clone the whole content registry just to validate one command.

### How to verify

Run:

```powershell
cargo check -p storyforge-core
```

### Next connection

Now handle commands by returning events.

---

File: `crates/storyforge-core/src/creation.rs`

## Step 7: Handle creation commands without mutating state

### Current state

Commands and catalogs exist, but no function turns commands into events.

### Why this step matters

This is the creation version of Guide 05's reducer. It lets you test rules before the TUI exists.

### Before

Add the handler below the helpers.

### What to change

Implement `handle_creation_command` for the MVP commands.

### Temporary MVP / debug behavior

For each command, first inspect the returned events in a test. Only after the event looks right should you apply it to state.

### After

```rust
#[must_use]
pub fn handle_creation_command(
    state: &CharacterCreationState,
    command: CreationCommand,
    catalog: &CreationCatalog<'_>,
) -> Vec<CreationEvent> {
    match command {
        CreationCommand::SetIdentity { name, pronouns } => {
            let name = name.trim().to_owned();
            let pronouns = pronouns.trim().to_owned();
            if !(1..=32).contains(&name.chars().count()) {
                return rejected("name", "Enter a name containing 1 to 32 characters.");
            }
            if pronouns.is_empty() {
                return rejected("pronouns", "Pronouns cannot be blank.");
            }
            vec![
                CreationEvent::IdentitySet { name, pronouns },
                CreationEvent::CreationStageChanged { stage: CreationStage::Background },
            ]
        }
        CreationCommand::ChooseBackground { background } => {
            if !contains_id(catalog.backgrounds, &background) {
                return rejected("background", "That background is not available.");
            }
            vec![CreationEvent::BackgroundChosen { background }]
        }
        CreationCommand::ToggleTrait { trait_id } => {
            if !contains_id(catalog.traits, &trait_id) {
                return rejected("traits", "That trait is not available.");
            }
            if state.draft.traits.contains(&trait_id) {
                vec![CreationEvent::TraitRemoved { trait_id }]
            } else if state.draft.traits.len() >= 2 {
                rejected("traits", "Remove one trait before choosing another.")
            } else {
                vec![CreationEvent::TraitAdded { trait_id }]
            }
        }
```
        CreationCommand::ChooseCastingStyle { style } => {
            vec![CreationEvent::CastingStyleChosen { style }]
        }
        CreationCommand::ChooseStudyInterest { interest } => {
            if !contains_id(catalog.studies, &interest) {
                return rejected("study_interest", "That field of study is unavailable.");
            }
            vec![CreationEvent::StudyInterestChosen { interest }]
        }
        CreationCommand::ChooseAffiliationPreference { affiliation } => {
            if !contains_id(catalog.affiliations, &affiliation) {
                return rejected("affiliation", "That affiliation is unavailable.");
            }
            vec![CreationEvent::AffiliationPreferenceChosen { affiliation }]
        }
        CreationCommand::EnterShop { shop } => {
            if !contains_id(catalog.shops, &shop) {
                return rejected("shop", "That shop is not part of this market.");
            }
            let next_stage = if shop.as_str().contains("equipment") {
                CreationStage::EquipmentShop
            } else if shop.as_str().contains("companion") {
                CreationStage::CompanionShop
            } else if shop.as_str().contains("wand") {
                CreationStage::WandShop
            } else {
                return rejected("shop", "The shop has no creation-stage role.");
            };
            vec![CreationEvent::ShopEntered { shop, next_stage }]
        }
        CreationCommand::ChooseEquipmentPackage { package } => {
            if !contains_id(catalog.equipment_packages, &package) {
                return rejected("equipment_package", "That package is unavailable.");
            }
            vec![
                CreationEvent::EquipmentPackageChosen { package },
                CreationEvent::AutosaveRequested { reason: "equipment chosen".to_owned() },
            ]
        }
        CreationCommand::InspectCompanion { companion } => {
            if !contains_id(catalog.companions, &companion) {
                return rejected("companion", "That companion is unavailable.");
            }
            vec![CreationEvent::CompanionInspected { companion }]
        }
        CreationCommand::AdoptCompanion { companion } => {
            if !state.inspected_companions.contains(&companion) {
                return rejected("companion", "Meet the companion before adopting it.");
            }
            vec![
                CreationEvent::CompanionAdopted { companion },
                CreationEvent::AutosaveRequested { reason: "companion adopted".to_owned() },
            ]
        }
        CreationCommand::BeginWandTrial => {
            if state.stage != CreationStage::WandShop {
                return rejected("wand", "Begin the wand trial from the wand shop.");
            }
            let candidates = wand_candidates(state.candidate_seed, catalog.wands);
            if candidates.is_empty() {
                return rejected("wand", "No wands are available.");
            }
            vec![CreationEvent::WandCandidatesGenerated { candidates }]
        }
        CreationCommand::SelectWand { wand } => {
            if !state.wand_candidates.contains(&wand) {
                return rejected("wand", "Choose one of the trial wands.");
            }
            vec![
                CreationEvent::WandChosen { wand },
                CreationEvent::AutosaveRequested { reason: "wand chosen".to_owned() },
            ]
        }
        CreationCommand::ReturnTo { stage } => {
            if stage == CreationStage::Complete {
                return rejected("stage", "Cannot return directly to Complete.");
            }
            vec![CreationEvent::CreationStageChanged { stage }]
        }
        CreationCommand::ConfirmCharacter => {
            let problems = state.problems();
            if !problems.is_empty() {
                return vec![CreationEvent::CreationRejected { problems }];
            }
            match complete_character(&state.draft, catalog.opening_scene) {
                Ok(outcome) => vec![
                    CreationEvent::CharacterCreated { outcome: Box::new(outcome) },
                    CreationEvent::AutosaveRequested { reason: "character created".to_owned() },
                ],
                Err(error) => rejected("character", error.to_string()),
            }
        }
    }
}
```

### Learning checkpoint

You should understand that this function does not change `state`. It reads the current draft and returns events. That makes it easy to test rejected commands without side effects.

### How to verify

This references `complete_character`, which you add in Step 8.

### Next connection

Add the function that turns a valid draft into a permanent character.

---

File: `crates/storyforge-core/src/creation.rs`

## Step 8: Complete a valid draft into a character

### Current state

`ConfirmCharacter` needs `complete_character`, but it does not exist yet.

### Why this step matters

This is the boundary between draft data and permanent character data. All required fields must be present before this function succeeds.

### Before

Add this below `handle_creation_command`.

### What to change

Add helper functions to unwrap required draft fields safely and construct a level-1 character.

### Temporary MVP / debug behavior

Use simple starting abilities and spell slots. Print or inspect the resulting `PlayerCharacter` in a test before connecting the TUI.

### After

```rust
fn require(value: Option<ContentId>, field: &'static str) -> Result<ContentId, crate::CharacterError> {
    value.ok_or(crate::CharacterError::MissingField(field))
}

fn complete_character(
    draft: &CharacterDraft,
    opening_scene: &ContentId,
) -> Result<CreationOutcome, crate::CharacterError> {
    let name = draft.name.trim().to_owned();
    if !(1..=32).contains(&name.chars().count()) {
        return Err(crate::CharacterError::InvalidName);
    }
    if draft.traits.len() != 2 {
        return Err(crate::CharacterError::TraitCount);
    }

    let casting_style = draft
        .casting_style
        .ok_or(crate::CharacterError::MissingField("casting_style"))?;
    let abilities = starting_abilities(casting_style);
    abilities.validate()?;

    let mut slots = [crate::SpellSlotState { remaining: 0, maximum: 0 }; 9];
    slots[0] = crate::SpellSlotState { remaining: 2, maximum: 2 };

    let character = crate::PlayerCharacter {
        name,
        pronouns: draft.pronouns.trim().to_owned(),
        level: 1,
        experience: 0,
        abilities,
        background: require(draft.background.clone(), "background")?,
        casting_style,
        study_interest: require(draft.study_interest.clone(), "study_interest")?,
        affiliation: require(draft.affiliation.clone(), "affiliation")?,
        equipment_package: require(draft.equipment_package.clone(), "equipment_package")?,
        wand: require(draft.wand.clone(), "wand")?,
        companion: require(draft.companion.clone(), "companion")?,
        traits: draft.traits.clone(),
        skill_proficiencies: draft.skill_proficiencies.clone(),
        hp: 8,
        spellcasting: crate::SpellcastingState {
            casting_ability: casting_style.casting_ability(),
            known_cantrips: Vec::new(),
            known_spells: Vec::new(),
            prepared_spells: Vec::new(),
            slots,
        },
    };

    Ok(CreationOutcome {
        character,
        opening_scene: opening_scene.clone(),
    })
}

#[must_use]
fn starting_abilities(style: CastingStyle) -> crate::AbilityScores {
    let mut scores = crate::AbilityScores {
        strength: 8,
        dexterity: 12,
        constitution: 13,
        intelligence: 10,
        wisdom: 10,
        charisma: 10,
    };
    match style {
        CastingStyle::Willpower => scores.charisma = 15,
        CastingStyle::Technique => scores.wisdom = 15,
        CastingStyle::Intellect => scores.intelligence = 15,
    }
    scores
}
```

### Learning checkpoint

You should understand why `require` consumes an `Option<ContentId>` and returns a `Result<ContentId, CharacterError>`. A complete character cannot store `None` for required fields.

### How to verify

Compilation waits until the event applier and exports are complete.

### Next connection

Apply creation events to mutate the draft state.

---

File: `crates/storyforge-core/src/creation.rs`

## Step 9: Apply creation events to the draft

### Current state

The handler returns events, but no function applies them to `CharacterCreationState`.

### Why this step matters

This mirrors `apply_events` from Guide 05. It keeps mutation centralized and makes command handling easier to test.

### Before

Add this below the completion helpers.

### What to change

Implement `apply_creation_events`.

### Temporary MVP / debug behavior

For every command test, follow the same learning rhythm:

1. Call `handle_creation_command`.
2. Inspect the events.
3. Call `apply_creation_events`.
4. Inspect the changed draft.

### After

```rust
pub fn apply_creation_events(state: &mut CharacterCreationState, events: &[CreationEvent]) {
    for event in events {
        match event {
            CreationEvent::IdentitySet { name, pronouns } => {
                state.draft.name.clone_from(name);
                state.draft.pronouns.clone_from(pronouns);
            }
            CreationEvent::BackgroundChosen { background } => {
                state.draft.background = Some(background.clone());
            }
            CreationEvent::TraitAdded { trait_id } => {
                state.draft.traits.push(trait_id.clone());
            }
            CreationEvent::TraitRemoved { trait_id } => {
                state.draft.traits.retain(|selected| selected != trait_id);
            }
            CreationEvent::CastingStyleChosen { style } => {
                state.draft.casting_style = Some(*style);
            }
            CreationEvent::StudyInterestChosen { interest } => {
                state.draft.study_interest = Some(interest.clone());
            }
            CreationEvent::AffiliationPreferenceChosen { affiliation } => {
                state.draft.affiliation = Some(affiliation.clone());
            }
            CreationEvent::ShopEntered { shop, next_stage } => {
                state.visited_shops.insert(shop.clone());
                state.stage = *next_stage;
            }
            CreationEvent::EquipmentPackageChosen { package } => {
                state.draft.equipment_package = Some(package.clone());
            }
            CreationEvent::CompanionInspected { companion } => {
                state.inspected_companions.insert(companion.clone());
            }
            CreationEvent::CompanionAdopted { companion } => {
                state.draft.companion = Some(companion.clone());
            }
            CreationEvent::WandCandidatesGenerated { candidates } => {
                state.wand_candidates.clone_from(candidates);
            }
            CreationEvent::WandChosen { wand } => {
                state.draft.wand = Some(wand.clone());
            }
            CreationEvent::CreationStageChanged { stage } => {
                state.stage = *stage;
            }
            CreationEvent::CharacterCreated { .. } => {
                state.stage = CreationStage::Complete;
            }
            CreationEvent::CreationRejected { .. } | CreationEvent::AutosaveRequested { .. } => {}
        }
    }
}
```

### Learning checkpoint

You should understand that rejected commands do not mutate the draft. Autosave requests are notifications, not state changes.

### How to verify

Export the new API in Step 10, then run core tests.

### Next connection

Update `lib.rs` exports again.

---

File: `crates/storyforge-core/src/lib.rs`

## Step 10: Export creation commands, events, and helpers

### Current state

Step 4 exported only the draft and permanent character types.

### Why this step matters

Tests and the TUI need to dispatch creation commands and apply creation events.

### Before

Find the existing `pub use creation::{ ... };` block.

### What to change

Add the command/event API to that export list.

### Temporary MVP / debug behavior

Do not export `complete_character` yet. Keep confirmation going through `ConfirmCharacter` so the event path stays consistent.

### After

```rust
pub use creation::{
    CharacterCreationState, CharacterDraft, CharacterProblem, CreationCatalog, CreationCommand,
    CreationEvent, CreationOutcome, CreationStage, apply_creation_events,
    handle_creation_command, wand_candidates,
};
```

### Learning checkpoint

You should understand that keeping `complete_character` private prevents other code from bypassing `ConfirmCharacter` and its event output.

### How to verify

Run:

```powershell
cargo check -p storyforge-core
```

### Next connection

Now add focused tests for creation behavior.

---

File: `crates/storyforge-core/tests/creation_flow.rs`

## Step 11: Test the creation flow in small pieces

### Current state

Creation commands exist, but no tests prove the flow.

### Why this step matters

Character creation has many small rules. Focused tests prevent the TUI from hiding state-machine bugs.

### Before

Create:

```text
crates/storyforge-core/tests/creation_flow.rs
```

### What to change

Add tests for identity validation, companion inspection before adoption, stable wand candidates, back navigation, and casting ability.

### Temporary MVP / debug behavior

Each test should use explicit IDs. Do not rely on campaign files yet.

### After

```rust
use storyforge_core::{
    CastingStyle, CharacterCreationState, ContentId, CreationCatalog, CreationCommand,
    CreationEvent, CreationStage, apply_creation_events, handle_creation_command,
    wand_candidates,
};

fn id(value: &str) -> ContentId {
    ContentId::new(value).expect("test IDs are valid")
}

fn empty_catalog(opening_scene: &ContentId) -> CreationCatalog<'_> {
    CreationCatalog {
        backgrounds: &[],
        traits: &[],
        studies: &[],
        affiliations: &[],
        shops: &[],
        equipment_packages: &[],
        companions: &[],
        wands: &[],
        opening_scene,
    }
}

#[test]
fn identity_validation_keeps_the_player_on_the_same_stage() {
    let state = CharacterCreationState::new(7);
    let opening_scene = id("academy.scene.arrival");
    let catalog = empty_catalog(&opening_scene);

    let events = handle_creation_command(
        &state,
        CreationCommand::SetIdentity {
            name: " ".to_owned(),
            pronouns: "they/them".to_owned(),
        },
        &catalog,
    );

    assert!(matches!(events.as_slice(), [CreationEvent::CreationRejected { .. }]));
    assert_eq!(state.stage, CreationStage::Identity);
}

#[test]
fn companion_must_be_inspected_before_adoption() {
    let state = CharacterCreationState::new(7);
    let owl = id("academy.companion.ash-owl");
    let opening_scene = id("academy.scene.arrival");
    let companions = [owl.clone()];
    let catalog = CreationCatalog {
        companions: &companions,
        ..empty_catalog(&opening_scene)
    };

    let rejected = handle_creation_command(
        &state,
        CreationCommand::AdoptCompanion { companion: owl.clone() },
        &catalog,
    );
    assert!(matches!(rejected.as_slice(), [CreationEvent::CreationRejected { .. }]));

    let mut inspected_state = state;
    let inspected = handle_creation_command(
        &inspected_state,
        CreationCommand::InspectCompanion { companion: owl.clone() },
        &catalog,
    );
    apply_creation_events(&mut inspected_state, &inspected);

    let adopted = handle_creation_command(
        &inspected_state,
        CreationCommand::AdoptCompanion { companion: owl },
        &catalog,
    );
    assert!(matches!(adopted.as_slice(), [CreationEvent::CompanionAdopted { .. }, ..]));
}

#[test]
fn wand_candidates_are_stable_for_the_same_seed() {
    let wands = [
        id("academy.wand.a"),
        id("academy.wand.b"),
        id("academy.wand.c"),
        id("academy.wand.d"),
    ];

    assert_eq!(wand_candidates(19, &wands), wand_candidates(19, &wands));
    assert_eq!(wand_candidates(19, &wands).len(), 3);
}

#[test]
fn back_navigation_does_not_clear_the_draft() {
    let mut state = CharacterCreationState::new(7);
    let opening_scene = id("academy.scene.arrival");
    let catalog = empty_catalog(&opening_scene);

    let identity = handle_creation_command(
        &state,
        CreationCommand::SetIdentity {
            name: "Mara Vale".to_owned(),
            pronouns: "she/her".to_owned(),
        },
        &catalog,
    );
    apply_creation_events(&mut state, &identity);

    let back = handle_creation_command(
        &state,
        CreationCommand::ReturnTo { stage: CreationStage::Identity },
        &catalog,
    );
    apply_creation_events(&mut state, &back);

    assert_eq!(state.draft.name, "Mara Vale");
    assert_eq!(state.stage, CreationStage::Identity);
}

#[test]
fn casting_styles_choose_different_spellcasting_abilities() {
    assert_ne!(
        CastingStyle::Willpower.casting_ability(),
        CastingStyle::Intellect.casting_ability()
    );
}
```

### Learning checkpoint

You should understand why these tests call the handler directly. The creation rules should work without keyboard input, terminal rendering, or content files.

### How to verify

Run:

```powershell
cargo test -p storyforge-core --test creation_flow
```

### Next connection

Now add the first data-driven Lantern Row record to Guide 06's content pack.

---

File: `crates/storyforge-content/src/model.rs`

## Step 12: Add a small creation market model

### Current state

Guide 06 added manifest and scene models only. Character creation needs a market record for Lantern Row.

### Why this step matters

The end-goal creator is a playable shopping day. Content should define which shops and candidates exist, while core defines what commands do with those IDs.

### Before

Open `model.rs`, below the scene structs.

### What to change

Add market, shop, and shop-kind definitions.

### Temporary MVP / debug behavior

Model only the market shape first. Do not validate every item type yet.

### After

```rust
#[derive(Debug, Clone, Deserialize)]
pub struct CreationMarketDefinition {
    pub id: ContentId,
    pub title: String,
    pub arrival_scene: ContentId,
    pub shops: Vec<CreationShopDefinition>,
}

#[derive(Debug, Clone, Deserialize)]
pub struct CreationShopDefinition {
    pub id: ContentId,
    pub title: String,
    pub description: String,
    pub kind: CreationShopKind,
    pub inventory: Vec<ContentId>,
}

#[derive(Debug, Clone, Copy, Deserialize, PartialEq, Eq)]
pub enum CreationShopKind {
    Equipment,
    Companion,
    Wand,
}
```

Export these from `storyforge-content/src/lib.rs`:

```rust
pub use model::{
    CampaignManifest, ChoiceDefinition, CreationMarketDefinition, CreationShopDefinition,
    CreationShopKind, SceneDefinition,
};
```

### Learning checkpoint

You should understand that a shop inventory stores IDs. The actual item, companion, and wand records can be added later without changing the creation command path.

### How to verify

Run:

```powershell
cargo check -p storyforge-content
```

### Next connection

Create the Lantern Row content file.

---

File: `campaigns/academy-demo/locations/lantern-row.ron`

## Step 13: Add Lantern Row as public demo content

### Current state

Guide 06 created scenes, but no creation market data.

### Why this step matters

This moves the shopping-day structure into content. The public demo uses original names, not private or licensed setting terms.

### Before

Create this directory if needed:

```text
campaigns/academy-demo/locations/
```

### What to change

Create `lantern-row.ron`.

### Temporary MVP / debug behavior

The inventory IDs do not all need full records yet. Use them to build the creation catalog slices for command tests and early UI.

### After

```ron
(
    id: "academy.market.lantern-row",
    title: "Lantern Row",
    arrival_scene: "academy.scene.market-arrival",
    shops: [
        (
            id: "academy.shop.equipment",
            title: "The Brass Trunk",
            description: "Uniforms, notebooks, and travel trunks fill the narrow room.",
            kind: Equipment,
            inventory: [
                "academy.equipment.standard",
                "academy.equipment.second-hand",
            ],
        ),
        (
            id: "academy.shop.companions",
            title: "Moth & Feather",
            description: "Open perches and quiet cages line the warm back room.",
            kind: Companion,
            inventory: [
                "academy.companion.ash-owl",
                "academy.companion.ink-cat",
                "academy.companion.rain-toad",
            ],
        ),
        (
            id: "academy.shop.wands",
            title: "The Quiet Bough",
            description: "Long drawers cover every wall, each marked with wood, core, and temperament.",
            kind: Wand,
            inventory: [
                "academy.wand.rowan-thread",
                "academy.wand.hazel-scale",
                "academy.wand.ash-glass",
                "academy.wand.willow-feather",
            ],
        ),
    ],
)
```

### Learning checkpoint

You should understand the privacy boundary: this public pack uses original content. Private campaign names and references belong in the separate private pack.

### How to verify

After loader support is expanded for locations, run:

```powershell
cargo run -p storyforge-tui -- validate --pack campaigns/academy-demo
```

### Next connection

Finally, connect the state machine to a TUI screen in a minimal way.

---

File: `crates/storyforge-tui/src/app.rs`

## Step 14: Add creation state to the app

### Current state

`App` owns a `GameEngine` and optional loaded campaign after Guide 06, but it has no character-creation draft.

### Why this step matters

The TUI needs to display the draft and send creation commands, but it should not construct `PlayerCharacter` directly.

### Before

`Screen` currently has:

```rust
pub enum Screen {
    Story,
    Character,
    Journal,
    Map,
    Help,
}
```

### What to change

Add a `Creation` screen and a `CharacterCreationState` field.

### Temporary MVP / debug behavior

Start creation automatically on app startup or behind a temporary key. The full New Game menu can come later.

### After

Add import:

```rust
use storyforge_core::CharacterCreationState;
```

Add screen variant:

```rust
Creation,
```

Add field:

```rust
pub(crate) creation: CharacterCreationState,
```

Initialize it:

```rust
creation: CharacterCreationState::new(19),
```

For the temporary MVP, you can start on creation:

```rust
screen: Screen::Creation,
```

Or keep Story as default and add a temporary shortcut later.

### Learning checkpoint

You should understand that `App` can own draft UI flow state, but core still owns the rules for changing that draft.

### How to verify

Run:

```powershell
cargo check -p storyforge-tui
```

### Next connection

Add a small creation renderer that shows the current stage and problems.

---

File: `crates/storyforge-tui/src/ui.rs`

## Step 15: Render the creation draft before making it interactive

### Current state

No creation screen exists in the renderer.

### Why this step matters

Before adding input, prove the draft reaches the screen. This follows the MVP flow: display data first, then mutate it.

### Before

`render_story_panel` matches the existing screens but not `Screen::Creation`.

### What to change

Add a basic creation branch that shows current stage and missing problems.

### Temporary MVP / debug behavior

Do not build the full shopping layout yet. Show debug lines such as stage, name, pronouns, and missing fields.

### After

Inside the match for main text, add:

```rust
Screen::Creation => {
    let problems = app.creation.problems();
    let mut lines = vec![format!("Stage: {:?}", app.creation.stage)];
    lines.push(format!("Name: {}", app.creation.draft.name));
    lines.push(format!("Pronouns: {}", app.creation.draft.pronouns));
    lines.push(String::new());
    lines.push("Missing:".to_owned());
    for problem in problems {
        lines.push(format!("- {}: {}", problem.field, problem.message));
    }
    lines.join("\n")
}
```

### Learning checkpoint

You should understand that this is not the final UI. It is a debug screen that proves `CharacterCreationState` reaches rendering.

### How to verify

Run:

```powershell
cargo run -p storyforge-tui
```

Open or start the Creation screen. You should see `Stage: Identity` and missing-field messages.

### Next connection

Now connect one command: identity submission.

---

File: `crates/storyforge-tui/src/app.rs`

## Step 16: Dispatch one creation command from the TUI

### Current state

The creation draft displays but cannot change from input.

### Why this step matters

This is the smallest interactive proof of character creation:

```text
temporary input -> CreationCommand::SetIdentity -> CreationEvent -> draft update -> render
```

### Before

`App::update` handles `UiAction` but has no creation-specific branch.

### What to change

For a temporary MVP, make `Confirm` on the Creation screen dispatch a hard-coded identity. This avoids building text fields before the state machine is proven.

### Temporary MVP / debug behavior

Use a debug identity first:

- Name: `Mara Vale`
- Pronouns: `she/her`

After this works, replace the hard-coded values with real text fields.

### After

Add imports:

```rust
use storyforge_core::{CreationCommand, apply_creation_events, handle_creation_command};
```

In `App::update`, add this arm before the general `Confirm` arm:

```rust
UiAction::Confirm if self.screen == Screen::Creation => {
    let opening_scene = self.engine.state().active_scene.clone();
    let catalog = storyforge_core::CreationCatalog {
        backgrounds: &[],
        traits: &[],
        studies: &[],
        affiliations: &[],
        shops: &[],
        equipment_packages: &[],
        companions: &[],
        wands: &[],
        opening_scene: &opening_scene,
    };
    let events = handle_creation_command(
        &self.creation,
        CreationCommand::SetIdentity {
            name: "Mara Vale".to_owned(),
            pronouns: "she/her".to_owned(),
        },
        &catalog,
    );
    apply_creation_events(&mut self.creation, &events);
}
```

The `opening_scene` local variable is important. `CreationCatalog` borrows it, so the value must live until `handle_creation_command` finishes.

### Learning checkpoint

You should understand the Rust borrowing issue here. `CreationCatalog` borrows `opening_scene`, so the borrowed value must live long enough for the handler call. Building it inline keeps that lifetime obvious.

### How to verify

Run:

```powershell
cargo run -p storyforge-tui
```

On the Creation screen, press `Enter`. The screen should show name `Mara Vale`, pronouns `she/her`, and stage `Background`.

### Next connection

Replace hard-coded identity with real text input later, then repeat this same command/event/apply pattern for background, traits, shops, companion, wand, and confirmation.

## Manual Implementation Order After This Guide

1. Replace the hard-coded identity with text fields.
2. Build a small selectable list for backgrounds.
3. Dispatch `ChooseBackground` and verify the draft changes.
4. Add trait toggling and prove a third trait is rejected.
5. Add casting style and study interest selections.
6. Add Lantern Row shop navigation.
7. Inspect a companion before adopting it.
8. Generate wand candidates once and display the same candidates after reload once saves exist.
9. Add a review screen that calls `ConfirmCharacter`.
10. On `CharacterCreated`, leave creation and begin the opening scene.

## Full Verification

Run:

```powershell
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features --locked -- -D warnings
cargo test --workspace --all-features --locked
cargo run -p storyforge-tui -- validate --pack campaigns/academy-demo
cargo run -p storyforge-tui
```

Manual checks:

1. Creation state starts at `Identity`.
2. The debug screen lists missing required fields.
3. Pressing `Enter` dispatches `SetIdentity`.
4. The draft stores the name and pronouns.
5. The stage advances to `Background`.
6. Core tests prove companion adoption requires inspection.
7. Core tests prove wand candidates are stable for the same seed.

## Common Mistakes

- Building `PlayerCharacter` before confirmation.
- Letting the TUI mutate draft fields directly instead of dispatching commands.
- Clearing the draft when the player goes back.
- Treating a companion as a plain inventory item with no remembered meeting.
- Regenerating wand candidates every render.
- Using random time in the TUI instead of stored deterministic seed data.
- Letting failed commands advance the creation stage.

## Acceptance Check

- Permanent character types exist in core.
- Draft creation state can represent incomplete choices.
- `problems()` explains missing fields.
- Creation commands and events follow the Guide 05 reducer pattern.
- Rejected commands do not mutate the draft.
- Companion adoption requires prior inspection.
- Wand candidates are deterministic.
- The TUI can display creation state and dispatch one MVP creation command.



