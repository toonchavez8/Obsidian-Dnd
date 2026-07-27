# 07: Build character creation as a playable shopping day

## Result

The player creates a character through a short prologue instead of filling out one long form. They establish an identity, choose how they approach magic, visit a market street, meet possible animal companions, try several wands, review the result, and enter the academy.

The public demo calls the market `Lantern Row`. A private content pack may give the location another name. The engine only knows that this is a character-creation scene sequence.

This chapter has six checkpoints. Do not implement all six before running the first test.

## Checkpoint 1: Create the permanent character types

Create `crates/storyforge-core/src/character.rs`. These types describe a finished character. Optional creation choices do not belong here.

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
pub struct SorceryState {
    pub current_points: u8,
    pub maximum_points: u8,
    pub metamagic_options: Vec<ContentId>,
}

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct SpellcastingState {
    pub casting_ability: Ability,
    pub known_cantrips: Vec<ContentId>,
    pub known_spells: Vec<ContentId>,
    pub prepared_spells: Vec<ContentId>,
    pub slots: [SpellSlotState; 9],
    pub temporary_slots: [u8; 5],
    pub sorcery: Option<SorceryState>,
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
    #[error("prepared spells must be known leveled spells")]
    UnknownPreparedSpell,
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

Use `i32` for HP because damage and healing use signed arithmetic before clamping. Spell slots and sorcery points are small nonnegative counts, so `u8` is sufficient. Chapter 13 implements slot spending, upcasting, Flexible Casting, and Metamagic.

In `crates/storyforge-core/src/lib.rs`, add:

```rust
mod character;

pub use character::{
    Ability, AbilityScores, CastingStyle, CharacterError, PlayerCharacter,
    SorceryState, SpellSlotState, SpellcastingState, ability_modifier,
    proficiency_bonus,
};
```

Create `crates/storyforge-core/tests/character_math.rs`:

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
        (1, 2),
        (4, 2),
        (5, 3),
        (8, 3),
        (9, 4),
        (12, 4),
        (13, 5),
        (16, 5),
        (17, 6),
        (20, 6),
    ];

    for (level, expected) in cases {
        assert_eq!(proficiency_bonus(level), expected, "level {level}");
    }
}
```

Run:

```powershell
cargo test -p storyforge-core --test character_math
```

Continue when both tests pass.

## Checkpoint 2: Keep unfinished choices in a draft

A half-created character is not a valid `PlayerCharacter`. Create `crates/storyforge-core/src/creation.rs`:

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
            problems.push(CharacterProblem::new(
                "background",
                "Choose a background.",
            ));
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
            (
                "study_interest",
                &self.draft.study_interest,
                "Choose a field of study.",
            ),
            (
                "affiliation",
                &self.draft.affiliation,
                "Choose an affiliation preference.",
            ),
            (
                "equipment_package",
                &self.draft.equipment_package,
                "Collect an equipment package.",
            ),
            (
                "companion",
                &self.draft.companion,
                "Choose a pet or familiar.",
            ),
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

#[derive(Debug, Clone, PartialEq, Eq)]
pub struct CreationOutcome {
    pub character: PlayerCharacter,
    pub opening_scene: ContentId,
}
```

`candidate_seed` makes the same new-game seed produce the same wand candidates. `BTreeSet` keeps saved output and debug views stable. The TUI owns text focus and cursor position, but core owns every gameplay choice.

Export the module from `lib.rs`:

```rust
mod creation;

pub use creation::{
    CharacterCreationState, CharacterDraft, CharacterProblem, CreationOutcome,
    CreationStage,
};
```

Run:

```powershell
cargo check -p storyforge-core
```

Continue when the crate compiles.

## Checkpoint 3: Define every creation command and event

In `creation.rs`, add:

```rust
#[derive(Debug, Clone, PartialEq, Eq)]
pub enum CreationCommand {
    SetIdentity {
        name: String,
        pronouns: String,
    },
    ChooseBackground {
        background: ContentId,
    },
    ToggleTrait {
        trait_id: ContentId,
    },
    ChooseCastingStyle {
        style: CastingStyle,
    },
    ChooseStudyInterest {
        interest: ContentId,
    },
    ChooseAffiliationPreference {
        affiliation: ContentId,
    },
    EnterShop {
        shop: ContentId,
    },
    ChooseEquipmentPackage {
        package: ContentId,
    },
    InspectCompanion {
        companion: ContentId,
    },
    AdoptCompanion {
        companion: ContentId,
    },
    BeginWandTrial,
    SelectWand {
        wand: ContentId,
    },
    ReturnTo {
        stage: CreationStage,
    },
    ConfirmCharacter,
}

#[derive(Debug, Clone, PartialEq, Eq)]
pub enum CreationEvent {
    IdentitySet {
        name: String,
        pronouns: String,
    },
    BackgroundChosen {
        background: ContentId,
    },
    TraitAdded {
        trait_id: ContentId,
    },
    TraitRemoved {
        trait_id: ContentId,
    },
    CastingStyleChosen {
        style: CastingStyle,
    },
    StudyInterestChosen {
        interest: ContentId,
    },
    AffiliationPreferenceChosen {
        affiliation: ContentId,
    },
    ShopEntered {
        shop: ContentId,
        next_stage: CreationStage,
    },
    EquipmentPackageChosen {
        package: ContentId,
    },
    CompanionInspected {
        companion: ContentId,
    },
    CompanionAdopted {
        companion: ContentId,
    },
    WandCandidatesGenerated {
        candidates: Vec<ContentId>,
    },
    WandChosen {
        wand: ContentId,
    },
    CreationStageChanged {
        stage: CreationStage,
    },
    CreationRejected {
        problems: Vec<CharacterProblem>,
    },
    CharacterCreated {
        outcome: Box<CreationOutcome>,
    },
    AutosaveRequested {
        reason: String,
    },
}
```

`CharacterCreated` boxes the larger payload so a vector of events is not forced to reserve that size for every small variant.

The commands have these contracts:

| Command | Validation | Events and state change |
| --- | --- | --- |
| `SetIdentity` | Trimmed name is 1 to 32 characters; pronouns are not blank | Stores both values and moves to `Background` |
| `ChooseBackground` | ID exists in the loaded background catalog | Replaces the background |
| `ToggleTrait` | ID exists; no more than two selected | Adds or removes one trait |
| `ChooseCastingStyle` | Enum is already valid | Stores the style |
| `ChooseStudyInterest` | ID exists in the study catalog | Stores the interest |
| `ChooseAffiliationPreference` | ID exists and is available at creation | Stores a preference, not a guaranteed placement |
| `EnterShop` | Shop exists and is open in this sequence | Records the visit and changes stage |
| `ChooseEquipmentPackage` | Package exists and matches the campaign budget policy | Replaces the package |
| `InspectCompanion` | Companion is offered by the active shop | Records that the player met it; no adoption yet |
| `AdoptCompanion` | Companion was inspected and is adoptable | Replaces the selected companion |
| `BeginWandTrial` | Player is in the wand shop | Generates three stable candidates once |
| `SelectWand` | Wand is in the generated candidate list | Stores the wand |
| `ReturnTo` | Target is not `Complete` | Changes the stage without deleting choices |
| `ConfirmCharacter` | Entire draft has no problems | Emits the finished character, opening scene, and autosave request |

Do not put keyboard keys in these enums. The TUI maps `Enter` to a command; tests and future mouse support dispatch the same command directly.

## Checkpoint 4: Handle and apply commands

The engine needs read-only catalogs when it validates content IDs. Add this narrow view to `creation.rs`:

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
```

Add a candidate generator. It is deterministic and does not imply that one wand is secretly the best:

```rust
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

Now add the command handler. It decides what happened but does not mutate state:

```rust
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
                CreationEvent::CreationStageChanged {
                    stage: CreationStage::Background,
                },
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
            vec![CreationEvent::EquipmentPackageChosen { package }]
        }
        CreationCommand::InspectCompanion { companion } => {
            if !contains_id(catalog.companions, &companion) {
                return rejected("companion", "That companion is not in this shop.");
            }
            vec![CreationEvent::CompanionInspected { companion }]
        }
        CreationCommand::AdoptCompanion { companion } => {
            if !state.inspected_companions.contains(&companion) {
                return rejected("companion", "Meet this companion before adopting it.");
            }
            vec![CreationEvent::CompanionAdopted { companion }]
        }
        CreationCommand::BeginWandTrial => {
            if state.stage != CreationStage::WandShop {
                return rejected("wand", "Enter the wand shop before starting a trial.");
            }
            let candidates = wand_candidates(state.candidate_seed, catalog.wands);
            if candidates.is_empty() {
                return rejected("wand", "The campaign does not define any wands.");
            }
            vec![CreationEvent::WandCandidatesGenerated { candidates }]
        }
        CreationCommand::SelectWand { wand } => {
            if !state.wand_candidates.contains(&wand) {
                return rejected("wand", "Try this wand before choosing it.");
            }
            vec![CreationEvent::WandChosen { wand }]
        }
        CreationCommand::ReturnTo { stage } => {
            if stage == CreationStage::Complete {
                return rejected("stage", "Only confirmation can complete creation.");
            }
            vec![CreationEvent::CreationStageChanged { stage }]
        }
        CreationCommand::ConfirmCharacter => {
            let problems = state.problems();
            if !problems.is_empty() {
                return vec![CreationEvent::CreationRejected { problems }];
            }

            match build_character(&state.draft, catalog.opening_scene) {
                Ok(outcome) => vec![
                    CreationEvent::CharacterCreated {
                        outcome: Box::new(outcome),
                    },
                    CreationEvent::AutosaveRequested {
                        reason: "character creation completed".to_owned(),
                    },
                ],
                Err(problems) => vec![CreationEvent::CreationRejected { problems }],
            }
        }
    }
}
```

`build_character` is the only remaining constructor. Add it below the handler:

```rust
fn build_character(
    draft: &CharacterDraft,
    opening_scene: &ContentId,
) -> Result<CreationOutcome, Vec<CharacterProblem>> {
    let required = (
        draft.background.clone(),
        draft.casting_style,
        draft.study_interest.clone(),
        draft.affiliation.clone(),
        draft.equipment_package.clone(),
        draft.wand.clone(),
        draft.companion.clone(),
    );
    let (
        Some(background),
        Some(casting_style),
        Some(study_interest),
        Some(affiliation),
        Some(equipment_package),
        Some(wand),
        Some(companion),
    ) = required
    else {
        return Err(vec![CharacterProblem::new(
            "character",
            "The character draft is incomplete.",
        )]);
    };

    let abilities = starting_abilities(casting_style);
    let constitution_modifier = i32::from(crate::ability_modifier(abilities.constitution));
    let hp = (8 + constitution_modifier).max(1);
    let cantrip = ContentId::new("academy.spell.spark")
        .map_err(|error| vec![CharacterProblem::new("content", error.to_string())])?;
    let first_spell = ContentId::new("academy.spell.guard")
        .map_err(|error| vec![CharacterProblem::new("content", error.to_string())])?;

    let character = PlayerCharacter {
        name: draft.name.trim().to_owned(),
        pronouns: draft.pronouns.trim().to_owned(),
        level: 1,
        experience: 0,
        abilities,
        background,
        casting_style,
        study_interest,
        affiliation,
        equipment_package,
        wand,
        companion,
        traits: draft.traits.clone(),
        skill_proficiencies: draft.skill_proficiencies.clone(),
        hp,
        spellcasting: crate::SpellcastingState {
            casting_ability: casting_style.casting_ability(),
            known_cantrips: vec![cantrip],
            known_spells: vec![first_spell.clone()],
            prepared_spells: vec![first_spell],
            slots: {
                let mut slots = [
                    crate::SpellSlotState {
                        remaining: 0,
                        maximum: 0,
                    };
                    9
                ];
                slots[0] = crate::SpellSlotState {
                    remaining: 2,
                    maximum: 2,
                };
                slots
            },
            temporary_slots: [0; 5],
            sorcery: None,
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

The slot array has nine entries, one for each leveled-spell tier. Index zero is the 1st-level slot pool. Cantrips do not use this array.

Add the event applier:

```rust
pub fn apply_creation_events(
    state: &mut CharacterCreationState,
    events: &[CreationEvent],
) {
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
            CreationEvent::CreationRejected { .. }
            | CreationEvent::AutosaveRequested { .. } => {}
        }
    }
}
```

Export the new types and functions from `lib.rs`, alongside the Checkpoint 2 exports:

```rust
pub use creation::{
    CreationCatalog, CreationCommand, CreationEvent, apply_creation_events,
    handle_creation_command, wand_candidates,
};
```

Run:

```powershell
cargo fmt --all
cargo check -p storyforge-core
```

Continue when the core crate compiles without warnings.

## Checkpoint 5: Make Lantern Row data-driven

Add these content records to `storyforge-content`. They are deliberately small for the first vertical slice:

```rust
#[derive(Debug, Clone, serde::Deserialize)]
pub struct CreationMarketDefinition {
    pub id: ContentId,
    pub title: String,
    pub arrival_scene: ContentId,
    pub shops: Vec<CreationShopDefinition>,
}

#[derive(Debug, Clone, serde::Deserialize)]
pub struct CreationShopDefinition {
    pub id: ContentId,
    pub title: String,
    pub description: String,
    pub kind: CreationShopKind,
    pub inventory: Vec<ContentId>,
}

#[derive(Debug, Clone, Copy, serde::Deserialize)]
pub enum CreationShopKind {
    Equipment,
    Companion,
    Wand,
}
```

Create `campaigns/academy-demo/locations/lantern-row.ron`:

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
            description: "Cages stand open while several watchful animals choose whom to approach.",
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

The public pack uses original names and descriptions. Private campaign terminology belongs in the private sibling repository created in chapter 19.

Equipment packages represent different circumstances without creating a stronger paid option:

| Policy | Narrative effect | Mechanical rule |
| --- | --- | --- |
| Assisted | School or sponsor supplied the list | Complete required kit, one visible relationship hook |
| Second-hand | Used equipment with history | Complete required kit, one cosmetic quirk |
| Standard | Ordinary new supplies | Complete required kit |
| Comfortable | Extra spending money | Complete required kit, cosmetic choice only |

A companion may be an ordinary pet or a rules-bearing familiar:

- A pet provides relationship scenes, observations, and story reactions.
- A familiar also has explicit exploration or combat abilities.
- Both have a content ID, bond state, and memory.
- The campaign definition decides which kind each candidate is.
- The player may keep only one active companion at first.

Validate the market:

1. Every shop ID is unique.
2. Exactly one equipment, companion, and wand shop exists.
3. Every inventory ID resolves.
4. At least three companions and three wands are available.
5. Every equipment policy grants the required school items.
6. No creation-only item grants an unexplained combat advantage.

Run:

```powershell
cargo run -p storyforge-tui -- validate --pack campaigns/academy-demo
```

Expected result:

```text
academy-demo is valid
```

## Checkpoint 6: Connect the state machine to the TUI

Create `crates/storyforge-tui/src/screens/creation.rs`. Do not calculate character rules in this file. Its responsibilities are:

1. Render `CharacterCreationState`.
2. Convert input into `CreationCommand`.
3. Send commands to the engine.
4. Show `CreationRejected` problems without closing the screen.
5. Begin the opening scene after `CharacterCreated`.

Use one screen layout for every stage:

```text
┌──────────────────────────────────────────────────────────────────────┐
│ LANTERN ROW                                      Shopping Day  2 / 3 │
├─────────────────────────────────────────────┬────────────────────────┤
│ MOTH & FEATHER                              │ YOUR CHARACTER         │
│                                             │ Mara Vale             │
│ The ash owl watches from an open perch.     │ Intellect             │
│ The ink cat pretends not to notice you.     │ Archive studies       │
│                                             │                        │
│ > Meet the ash owl                          │ REQUIRED               │
│   Offer the ink cat a ribbon                │ [x] Equipment          │
│   Listen beside the rain toad               │ [ ] Companion          │
│                                             │ [ ] Wand               │
├─────────────────────────────────────────────┴────────────────────────┤
│ ↑↓ choose   Enter interact   B back   C sheet preview                │
└──────────────────────────────────────────────────────────────────────┘
```

Input behavior:

| Context | Key | Result |
| --- | --- | --- |
| Text field focused | Printable key | Edits the field; global shortcuts do not run |
| Text field focused | `Enter` | Dispatches `SetIdentity` |
| Choice list | Up or `k` | Moves visual selection only |
| Choice list | Down or `j` | Moves visual selection only |
| Choice list | `Enter` | Dispatches the command attached to the selected choice |
| Any incomplete stage | `b` | Dispatches `ReturnTo` for the prior narrative stage |
| Review | `Enter` | Dispatches `ConfirmCharacter` |
| Any stage | `Esc` | Opens a discard confirmation; it does not silently erase the draft |

After each shop choice, request a lightweight creation autosave. Chapter 08 supplies the save writer. Until then, log the request:

```rust
match event {
    CreationEvent::AutosaveRequested { reason } => {
        tracing::info!(%reason, "autosave requested");
    }
    CreationEvent::CharacterCreated { outcome } => {
        app.begin_scene(outcome.opening_scene.clone());
    }
    CreationEvent::CreationRejected { problems } => {
        app.creation_problems.clone_from(problems);
    }
    _ => {}
}
```

The final review shows:

- Identity and background.
- Two traits.
- Casting style and its linked ability.
- Study interest and affiliation preference.
- Equipment package.
- Companion name and whether it is a pet or familiar.
- Wand wood, core, length, flexibility, and authored temperament.
- Starting HP, defense, skills, known cantrips, prepared spell, and two 1st-level slots.

## Focused command tests

Create `crates/storyforge-core/tests/creation_flow.rs`. Use a helper for valid IDs:

```rust
use storyforge_core::{
    CastingStyle, CharacterCreationState, ContentId, CreationCatalog,
    CreationCommand, CreationEvent, CreationStage, apply_creation_events,
    handle_creation_command, wand_candidates,
};

fn id(value: &str) -> ContentId {
    ContentId::new(value).expect("test IDs are valid")
}

#[test]
fn identity_validation_keeps_the_player_on_the_same_stage() {
    let state = CharacterCreationState::new(7);
    let opening_scene = id("academy.scene.arrival");
    let catalog = CreationCatalog {
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
        &state,
        CreationCommand::SetIdentity {
            name: " ".to_owned(),
            pronouns: "they/them".to_owned(),
        },
        &catalog,
    );

    assert!(matches!(
        events.as_slice(),
        [CreationEvent::CreationRejected { .. }]
    ));
    assert_eq!(state.stage, CreationStage::Identity);
}

#[test]
fn companion_must_be_inspected_before_adoption() {
    let state = CharacterCreationState::new(7);
    let owl = id("academy.companion.ash-owl");
    let opening_scene = id("academy.scene.arrival");
    let catalog = CreationCatalog {
        backgrounds: &[],
        traits: &[],
        studies: &[],
        affiliations: &[],
        shops: &[],
        equipment_packages: &[],
        companions: std::slice::from_ref(&owl),
        wands: &[],
        opening_scene: &opening_scene,
    };

    let rejected = handle_creation_command(
        &state,
        CreationCommand::AdoptCompanion {
            companion: owl.clone(),
        },
        &catalog,
    );
    assert!(matches!(
        rejected.as_slice(),
        [CreationEvent::CreationRejected { .. }]
    ));

    let mut inspected_state = state;
    let inspected = handle_creation_command(
        &inspected_state,
        CreationCommand::InspectCompanion {
            companion: owl.clone(),
        },
        &catalog,
    );
    apply_creation_events(&mut inspected_state, &inspected);

    let adopted = handle_creation_command(
        &inspected_state,
        CreationCommand::AdoptCompanion { companion: owl },
        &catalog,
    );
    assert!(matches!(
        adopted.as_slice(),
        [CreationEvent::CompanionAdopted { .. }]
    ));
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
    let catalog = CreationCatalog {
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
        CreationCommand::ReturnTo {
            stage: CreationStage::Identity,
        },
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

Run:

```powershell
cargo test -p storyforge-core --test creation_flow
```

## Manual playthrough

1. Start a new game with seed `19`.
2. Enter a name containing a space.
3. Choose a background, two traits, a casting style, and a study interest.
4. Enter Lantern Row.
5. Choose an equipment policy and confirm every policy grants the required kit.
6. Try to adopt a companion before meeting it; confirm the TUI explains why it failed.
7. Meet two companions, adopt one, go back, and confirm the meetings are remembered.
8. Begin the wand trial and record the three candidates.
9. Quit, resume the creation autosave, and confirm the same candidates appear.
10. Choose a wand and open the review screen.
11. Go back, change the background, and confirm derived values update.
12. Confirm the character.
13. Verify the opening academy scene begins and the character sheet opens with `c`.

## Full verification

```powershell
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features --locked -- -D warnings
cargo test --workspace --all-features --locked
cargo run -p storyforge-tui -- validate --pack campaigns/academy-demo
cargo run -p storyforge-tui -- play --pack campaigns/academy-demo --seed 19
```

## Common mistakes

- The TUI constructs `PlayerCharacter` directly.
- Going back replaces the draft with `CharacterDraft::default()`.
- The familiar is just an inventory item and cannot remember interactions.
- The wand screen reveals a mathematically best option.
- Equipment wealth creates a permanent combat advantage.
- A failed command advances the stage.
- Keyboard shortcuts consume letters while the name field has focus.
- Random candidates are regenerated after load instead of saved.

## Acceptance check

- Character creation feels like a short story sequence.
- Identity, casting style, study interest, equipment, companion, and wand are all explicit choices.
- A player meets a companion before adopting it.
- Wand candidates are deterministic and survive save and load.
- Back navigation preserves every earlier choice.
- Invalid commands explain the affected field.
- A completed character starts with cantrips and spell slots, not mana.
- `CharacterCreated` begins the opening scene and requests an autosave.

## Suggested commit

```powershell
git add .
git commit -m "Build the shopping-day character creator"
```
