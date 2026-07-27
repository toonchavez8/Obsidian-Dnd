# 07: Build character creation

## Result

The player can move through a reversible creation flow, inspect effects, confirm a valid character, and open the character sheet.

## Model creation as state

Create `storyforge-core/src/character.rs` with:

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

#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]
pub struct PlayerCharacter {
    pub name: String,
    pub pronouns: String,
    pub level: u8,
    pub experience: u32,
    pub abilities: AbilityScores,
    pub background: ContentId,
    pub affiliation: ContentId,
    pub wand: ContentId,
    pub familiar: ContentId,
    pub traits: Vec<ContentId>,
    pub skill_proficiencies: Vec<ContentId>,
    pub hp: i32,
    pub spellcasting: SpellcastingState,
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

#[derive(Debug, Clone, PartialEq, Eq, thiserror::Error)]
pub enum CharacterError {
    #[error("character name must contain between 1 and 32 visible characters")]
    InvalidName,
    #[error("ability scores must be between 3 and 20")]
    AbilityOutOfRange,
    #[error("exactly two personality traits are required")]
    TraitCount,
}
```

Use `i32` for HP because damage and healing perform signed arithmetic before clamping. Slot and sorcery-point counts are small nonnegative integers, so `u8` makes illegal negative state unrepresentable. Validate upper bounds whenever state enters the engine.

## Creation draft

Do not construct `PlayerCharacter` with temporary IDs that look valid. Use a draft:

```rust
#[derive(Debug, Clone, Default)]
pub struct CharacterDraft {
    pub name: String,
    pub pronouns: String,
    pub background: Option<ContentId>,
    pub affiliation: Option<ContentId>,
    pub wand: Option<ContentId>,
    pub familiar: Option<ContentId>,
    pub traits: Vec<ContentId>,
    pub skill_proficiencies: Vec<ContentId>,
}
```

Creation steps:

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq, Default)]
pub enum CreationStep {
    #[default]
    Name,
    Pronouns,
    Background,
    Traits,
    Affiliation,
    Wand,
    Familiar,
    Skills,
    Review,
}
```

The TUI owns `CreationStep` and text-edit focus. Core owns draft validation and final character construction.

## Name input

While a text field is focused:

- Printable characters edit the field.
- Backspace deletes one Unicode scalar value.
- Enter validates and continues.
- Esc asks whether to discard creation.
- `q`, `c`, and number keys are text, not global shortcuts.

Trim leading and trailing whitespace at confirmation. Preserve internal spaces and the player's capitalization.

## Ability method

Use a fixed, friendly array for the MVP:

```text
15, 14, 13, 12, 10, 8
```

Background and affiliation apply small bonuses after assignment. Do not add rolling during creation until the fixed flow is tested and balanced.

The review screen shows the base score, each bonus source, and final score.

## Derived statistics

Core functions:

```rust
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

Test odd scores below 10. Integer division toward zero gives the wrong result for 9 unless you use Euclidean division.

Default level-one formulas:

```text
max_hp = 8 + Constitution modifier + background bonus
defense = 10 + Dexterity modifier + robe bonus
initiative = Dexterity modifier + trait bonus
spell_attack_bonus = proficiency bonus + casting ability modifier
spell_save_dc = 8 + proficiency bonus + casting ability modifier
```

Clamp maximum HP to at least 1.

Spell slots come from a validated progression table rather than a formula. The default full-caster level-one row grants two 1st-level slots and zero slots at higher levels:

```text
slots = [2, 0, 0, 0, 0, 0, 0, 0, 0]
```

At finalization, copy each maximum into the corresponding remaining count. Temporary slots start at zero. A character feature may grant `SorceryState`; otherwise it is `None`. A sorcery feature supplies the point maximum and learned metamagic IDs for the current level.

Known cantrips are separate from known leveled spells. Cantrips are always available once learned. Prepared spells must be a subset of known leveled spells.

## Finalization

Use `TryFrom<CharacterDraft>` or a builder that consumes the draft. Return all validation problems together for the review screen:

```rust
#[derive(Debug, Clone, PartialEq, Eq)]
pub struct CharacterProblem {
    pub field: &'static str,
    pub message: String,
}
```

Only `CharacterDraft` may be incomplete. `PlayerCharacter` invariants must hold after construction.

## Creation commands and events

Add commands:

- `SetCharacterName`
- `SetPronouns`
- `ChooseBackground`
- `ChooseAffiliation`
- `ChooseWand`
- `ChooseFamiliar`
- `ToggleTrait`
- `ToggleSkill`
- `ConfirmCharacter`

Add events:

- `CharacterDraftChanged`
- `CharacterCreationRejected`
- `CharacterCreated`

The `CharacterCreated` event starts the entry scene and requests an autosave in chapter 08.

## TUI screens

Each step uses:

- Title.
- One-sentence explanation.
- Main choices.
- Focus marker.
- Mechanical preview.
- Back and continue hints.
- Progress such as `Step 4 of 9`.

The review screen must show every choice and derived value before confirmation.

## Tests

Required tests:

- Ability modifier for 8, 9, 10, 11, 12, and 18.
- Proficiency at boundaries 1, 4, 5, 8, 9, 12, 13, 16, 17, and 20.
- Empty and 33-character names fail.
- One and three traits fail.
- Missing required IDs produce named problems.
- Finalization consumes a complete draft.
- Back navigation preserves prior choices.
- Changing background recalculates derived values.

## Manual test

1. Start a new game.
2. Enter a name containing a space.
3. Move forward four steps.
4. Move backward twice.
5. Confirm the earlier choices are still present.
6. Change background.
7. Confirm the review numbers update.
8. Confirm the character.
9. Open the character sheet with `c`.

## Verify

```powershell
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features --locked -- -D warnings
cargo test --workspace --all-features --locked
cargo run -p storyforge-tui -- play --pack campaigns/academy-demo
```

## Common mistakes

- An incomplete `PlayerCharacter` uses empty strings as IDs.
- The TUI calculates HP differently from core.
- Global shortcuts consume letters while typing.
- Back navigation recreates the draft.
- Confirmation mutates the draft before all checks pass.

## Acceptance check

- Creation is reversible before confirmation.
- Invalid fields are identified on screen.
- The same inputs always create the same character.
- The character sheet uses core-derived values.
- The entry scene begins only after `CharacterCreated`.

## Suggested commit

```powershell
git add .
git commit -m "Add reversible character creation"
```
