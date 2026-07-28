# 13: Add magic, inventory, shops, and crafting

## Result

The player can learn and prepare spells, equip items, buy and sell through integer currency, collect loot, and brew one potion recipe.

## Build this chapter in six checkpoints

Do not put every system in one `magic.rs` file:

| Checkpoint | File | Narrow proof |
| --- | --- | --- |
| 1 | `storyforge-core/src/inventory.rs` | Add, remove, equip, and capacity tests |
| 2 | `storyforge-core/src/economy.rs` | Integer price and stale-quote tests |
| 3 | `storyforge-core/src/magic.rs` | Learn, prepare, slot progression, and upcast tests |
| 4 | `storyforge-core/src/crafting.rs` | Ingredient reservation and result tests |
| 5 | `storyforge-content/src/model/` | Item, shop, spell, loot, and recipe validation |
| 6 | `storyforge-tui/src/screens/` | Inventory, shop, spellbook, and craft panels |

Run `cargo test -p storyforge-core` after each of the first four checkpoints. Run pack validation after checkpoint 5. Play the shop and recipe after checkpoint 6.

## Commands and events

Create `storyforge-core/src/item_command.rs`:

```rust
use crate::{ContentId, ItemInstanceId, Money};

#[derive(Debug, Clone, PartialEq, Eq)]
pub enum ItemCommand {
    EquipItem {
        instance: ItemInstanceId,
    },
    UnequipItem {
        instance: ItemInstanceId,
    },
    BuyItem {
        shop: ContentId,
        item: ContentId,
        quantity: u32,
        quoted_total: Money,
    },
    SellItem {
        shop: ContentId,
        instance: ItemInstanceId,
        quoted_total: Money,
    },
    TakeLoot {
        loot: ContentId,
        entries: Vec<ContentId>,
    },
    PrepareSpells {
        spells: Vec<ContentId>,
    },
    LearnSpell {
        spell: ContentId,
        source: ContentId,
    },
    BeginCraft {
        recipe: ContentId,
    },
    ConfirmCraft {
        recipe: ContentId,
    },
}

#[derive(
    Debug, Clone, PartialEq, Eq, serde::Serialize, serde::Deserialize,
)]
pub enum ItemEvent {
    ItemEquipped {
        instance: ItemInstanceId,
    },
    ItemUnequipped {
        instance: ItemInstanceId,
    },
    StackAdded {
        item: ContentId,
        quantity: u32,
    },
    StackRemoved {
        item: ContentId,
        quantity: u32,
    },
    UniqueItemAdded {
        instance: ItemInstanceId,
        definition: ContentId,
    },
    UniqueItemRemoved {
        instance: ItemInstanceId,
    },
    MoneyChanged {
        previous: Money,
        current: Money,
        reason: ContentId,
    },
    LootCollected {
        loot: ContentId,
        entries: Vec<ContentId>,
    },
    PreparedSpellsChanged {
        previous: Vec<ContentId>,
        current: Vec<ContentId>,
    },
    SpellLearned {
        spell: ContentId,
        source: ContentId,
    },
    CraftPreviewed {
        recipe: ContentId,
        minutes: u32,
    },
    CraftCompleted {
        recipe: ContentId,
        result: ContentId,
        created_instances: Vec<ItemInstanceId>,
    },
    ItemCommandRejected {
        reason: String,
    },
    DerivedStatisticsRecalculated,
    AutosaveRequested {
        reason: String,
    },
}
```

Extend the chapter 05 top-level command and event enums with wrapper variants:

```rust
Item(ItemCommand),
```

Add that variant to `GameCommand`, and add `Item(ItemEvent),` to `GameEvent`. The wrapper keeps each root enum readable while all item payloads stay typed.

| Command | Validation before mutation | Successful events |
| --- | --- | --- |
| `EquipItem` | Instance is owned; slot, level, tags, and attunement pass | Prior item unequipped, requested item equipped, derived statistics recalculated |
| `UnequipItem` | Instance is equipped and not cursed or locked | Item unequipped, derived statistics recalculated |
| `BuyItem` | Shop and stock exist; quote is current; funds and capacity suffice | Money decreases, items enter inventory, stock changes, autosave |
| `SellItem` | Shop accepts category; instance is owned and sellable; quote is current | Item leaves inventory, money increases, stock changes, autosave |
| `TakeLoot` | Loot container is active; entries exist; capacity permits them | Exact stacks and instances enter inventory; loot is marked collected |
| `PrepareSpells` | Safe preparation context; every spell is known and leveled; capacity passes | Prepared list is replaced atomically |
| `LearnSpell` | Source grants the spell and it is not already known | Spell enters known cantrip or leveled list |
| `BeginCraft` | Recipe is known; location, tool, ingredients, and time pass | Preview shows requirements and crossed deadlines |
| `ConfirmCraft` | Preview is still current | Ingredients removed once, check resolved, result created, time advanced, autosave |

Every failure emits only `ItemCommandRejected`. Validate the whole transaction before emitting removal or money events. The event applier must never leave half a purchase or half a recipe in state.

## Inventory model

Use item stacks for stackable items and unique instances for equipment:

```rust
#[derive(
    Debug, Clone, Copy, PartialEq, Eq, Hash, serde::Serialize, serde::Deserialize,
)]
pub struct ItemInstanceId(pub u64);

pub struct Inventory {
    pub stacks: HashMap<ContentId, u32>,
    pub unique_items: HashMap<ItemInstanceId, ItemInstance>,
    pub quest_items: HashSet<ContentId>,
    pub capacity_slots: u16,
}

pub struct ItemInstance {
    pub instance_id: ItemInstanceId,
    pub definition_id: ContentId,
    pub charges: Option<u16>,
    pub quality: ItemQuality,
    pub custom_name: Option<String>,
}
```

Instance IDs are generated by deterministic state counters, not random UUIDs.

## Item definitions

Each item declares:

- Category.
- Slot cost.
- Base price in minor currency units.
- Stack limit.
- Tags.
- Use action.
- Equipment slot.
- Modifiers.
- Charges.
- Sell policy.
- Drop policy.
- Quest status.

Quest items cannot be sold, dropped, consumed, or overwritten through generic commands.

## Equipment

Runtime equipment:

```rust
pub struct Equipment {
    pub wand: Option<ItemInstanceId>,
    pub robe: Option<ItemInstanceId>,
    pub accessories: [Option<ItemInstanceId>; 2],
    pub utility: Option<ItemInstanceId>,
    pub familiar_gear: Option<ItemInstanceId>,
}
```

`EquipItem` validates ownership, slot, level, tags, and attunement. It emits unequip and equip events, then one derived-statistics update event.

Do not store equipment bonuses permanently in the character. Recalculate from definitions and equipped instances.

## Currency

Core stores:

```rust
#[derive(
    Debug, Clone, Copy, PartialEq, Eq, PartialOrd, Ord, serde::Serialize,
    serde::Deserialize,
)]
pub struct Money(pub i64);
```

Use the smallest campaign unit. Reject arithmetic overflow and negative balances.

Campaign display config can format several denominations:

```toml
[[currency.denominations]]
name = "crown"
symbol = "c"
minor-units = 100

[[currency.denominations]]
name = "spark"
symbol = "s"
minor-units = 1
```

## Shops

A shop definition contains:

- Merchant actor.
- Stock entries.
- Restock rule.
- Buy and sell categories.
- Base markup.
- Relationship modifiers.
- Faction modifiers.
- Unique stock conditions.
- Service entries.

Transaction command:

```rust
BuyItem {
    shop: ContentId,
    item: ContentId,
    quantity: u32,
    quoted_price: Money,
}
```

Core recalculates current price and rejects stale quotes. This prevents a UI price from becoming authority after state changes.

## Price order

1. Item base price.
2. Shop markup.
3. Scarcity or world-phase modifier.
4. Faction modifier.
5. Relationship modifier.
6. Difficulty setting.
7. Clamp to minimum legal price.

Use integer rational arithmetic or basis points. Do not use floating-point currency.

## Spells

Store:

- Known cantrip IDs.
- Known leveled-spell IDs.
- Prepared leveled-spell IDs.
- Upgrade choice IDs.
- Remaining and maximum normal slots for levels 1 through 9.
- Temporary slot counts for levels 1 through 5.
- Optional current and maximum sorcery points.
- Learned metamagic IDs.

Known cantrips are always available and do not use prepared capacity. Prepare leveled spells only at allowed locations or rest moments. Validate prepared capacity and known status.

Exploration spell actions use the same effect definitions as scenes. Combat spells convert into validated combat actions.

### Spell definition

Use a structured definition:

```rust
pub struct SpellDefinition {
    pub id: ContentId,
    pub name: String,
    pub level: u8,
    pub school: ContentId,
    pub casting_time: CastingTime,
    pub components: SpellComponents,
    pub range: SpellRange,
    pub targeting: TargetRule,
    pub concentration: bool,
    pub ritual: bool,
    pub effects: Vec<EffectDefinition>,
    pub cantrip_scaling: Vec<CantripScale>,
    pub upcast: Vec<UpcastStep>,
}

pub struct SpellComponents {
    pub verbal: bool,
    pub somatic: bool,
    pub focus: bool,
    pub consumed_material: Option<ContentId>,
}

pub struct CantripScale {
    pub character_level: u8,
    pub additional_effects: Vec<EffectDefinition>,
}

pub struct UpcastStep {
    pub slot_level: u8,
    pub additional_effects: Vec<EffectDefinition>,
}
```

Validation requires spell levels from 0 through 9. Level 0 means cantrip. Cantrips cannot declare slot consumption or ritual casting. Leveled spells cannot define cantrip scaling. Each upcast step must be higher than the spell's base level and sorted by slot level.

### Default spell-slot progression

Keep progression in rules-profile data. The default full-caster profile uses:

| Character level | 1st | 2nd | 3rd | 4th | 5th | 6th | 7th | 8th | 9th |
| ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 1 | 2 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 2 | 3 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 3 | 4 | 2 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 4 | 4 | 3 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 5 | 4 | 3 | 2 | 0 | 0 | 0 | 0 | 0 | 0 |
| 6 | 4 | 3 | 3 | 0 | 0 | 0 | 0 | 0 | 0 |
| 7 | 4 | 3 | 3 | 1 | 0 | 0 | 0 | 0 | 0 |
| 8 | 4 | 3 | 3 | 2 | 0 | 0 | 0 | 0 | 0 |
| 9 | 4 | 3 | 3 | 3 | 1 | 0 | 0 | 0 | 0 |
| 10 | 4 | 3 | 3 | 3 | 2 | 0 | 0 | 0 | 0 |
| 11 | 4 | 3 | 3 | 3 | 2 | 1 | 0 | 0 | 0 |
| 12 | 4 | 3 | 3 | 3 | 2 | 1 | 0 | 0 | 0 |
| 13 | 4 | 3 | 3 | 3 | 2 | 1 | 1 | 0 | 0 |
| 14 | 4 | 3 | 3 | 3 | 2 | 1 | 1 | 0 | 0 |
| 15 | 4 | 3 | 3 | 3 | 2 | 1 | 1 | 1 | 0 |
| 16 | 4 | 3 | 3 | 3 | 2 | 1 | 1 | 1 | 0 |
| 17 | 4 | 3 | 3 | 3 | 2 | 1 | 1 | 1 | 1 |
| 18 | 4 | 3 | 3 | 3 | 3 | 1 | 1 | 1 | 1 |
| 19 | 4 | 3 | 3 | 3 | 3 | 2 | 1 | 1 | 1 |
| 20 | 4 | 3 | 3 | 3 | 3 | 2 | 2 | 1 | 1 |

When a character gains a level, recalculate maximum slots from the active progression. Preserve spent slots by adding only the increase in maximum capacity. Do not refill every existing slot during leveling unless campaign rules explicitly combine leveling with a long rest.

### Cantrips and upcasting

A cantrip cast sends `SpellPower::Cantrip` and spends no slot. Apply all cantrip-scaling rows whose character-level threshold has been reached.

A leveled cast chooses a slot equal to or higher than the base spell level. Apply every upcast step whose threshold is no greater than the chosen slot. The UI must show the final effect before confirmation.

Ritual casting is an authored feature. It takes extra world time and spends no slot, but remains subject to components, access, interruption, and quest deadlines.

### Sorcery progression

Sorcery is a feature, not a default resource for every character. Its default maximum is zero at feature level 1, two at level 2, and then equal to feature level through level 20.

Store the feature level separately when it can differ from character level:

```rust
pub struct SorceryState {
    pub feature_level: u8,
    pub current_points: u8,
    pub maximum_points: u8,
    pub metamagic_options: Vec<ContentId>,
}
```

Long rest restores current points to maximum. No command may raise current points above maximum.

Flexible-casting costs are rules-profile data:

```ron
(
    create_slot_costs: [
        (slot_level: 1, points: 2),
        (slot_level: 2, points: 3),
        (slot_level: 3, points: 5),
        (slot_level: 4, points: 6),
        (slot_level: 5, points: 7),
    ],
    points_from_slot: SlotLevel,
)
```

Created slots are temporary and vanish at the end of a long rest. Converting a slot grants points equal to the expended slot's level. Both conversions use a bonus action and are rejected before mutation when their resource, level, or action requirements fail.

### Metamagic definitions

Register eight default options:

```text
Careful Spell    1 point
Distant Spell    1 point
Empowered Spell  1 point
Extended Spell   1 point
Heightened Spell 3 points
Quickened Spell  2 points
Subtle Spell     1 point
Twinned Spell    spell level, minimum 1 point
```

The option definition declares its point-cost rule, compatibility tags, targeting restrictions, and effect transformer. Keep the effect transformers in Rust because they change command legality and dice resolution. Packs choose which options a character can learn and may rename their display text.

Empowered Spell carries an `allows-second-metamagic` tag. Every other pair is rejected unless a future rules profile explicitly declares compatibility.

### Recovery items

Ordinary potions do not restore spell slots or sorcery points. A rare item may restore one when its definition names the exact level or amount and the validator enforces a campaign maximum. Never use a generic "restore all magic" effect because it becomes difficult to balance and migrate.

## Loot

An authored reward may grant exact items. A loot table uses deterministic weighted selection.

The event records:

- Table ID.
- Rolled entry.
- Quantity.
- Created item instance IDs.

The pickup screen lets the player compare slot capacity. Required quest rewards bypass normal capacity and enter quest inventory.

## Crafting

Recipe:

```rust
pub struct RecipeDefinition {
    pub id: ContentId,
    pub ingredients: Vec<IngredientRequirement>,
    pub tool_tag: String,
    pub minutes: u32,
    pub check: CheckDefinition,
    pub flawed: CraftResult,
    pub standard: CraftResult,
    pub exceptional: CraftResult,
}
```

Craft order:

1. Verify recipe knowledge.
2. Verify location and tool.
3. Verify ingredients.
4. Preview time and deadlines.
5. Confirm.
6. Reserve ingredients.
7. Roll check.
8. Remove ingredients.
9. Create result.
10. Advance time.
11. Emit result and autosave.

If an unexpected internal error occurs after reservation, roll back the whole command. Reducers should build changes before committing them.

## First recipe

Create an original `Mending Tonic`:

- Two common ingredients.
- Brewing kit.
- 30 minutes.
- Potion Making target 11.
- Flawed: heals `1d4` and applies Nauseous for one scene.
- Standard: heals `1d6+1`.
- Exceptional: two standard doses.

## UI

Inventory tabs:

```text
All | Equipment | Consumables | Ingredients | Quest
```

Item panel shows effect, price, slot cost, ownership, comparison, and allowed commands.

Shop panel shows wallet, stock, quantity, current total, and reason for price modifiers.

Craft panel shows known requirements and deadline preview.

## Validator additions

- Item stack and price limits.
- Equipment slots valid.
- Spell dice and effect references valid.
- Spell levels and upcast thresholds valid.
- Cantrip scaling thresholds are ordered and unique.
- Slot-progression rows cover every supported character level.
- Normal remaining slots cannot exceed their normal maxima.
- Temporary slots are stored separately and exist only at levels 1 through 5.
- Flexible-casting costs cover only levels 1 through 5.
- Metamagic point costs and compatibility tags valid.
- Shop stock references exist.
- Restock rule is supported.
- Recipe ingredients and outputs exist.
- Recipe time greater than zero.
- No quest item has a sell or drop action.

## Tests

- Buying exact balance reaches zero.
- Buying without funds changes nothing.
- Stale quote is rejected.
- Selling quest item is rejected.
- Equipment bonuses disappear after unequip.
- Prepared spells are a subset of known spells.
- Known cantrips never appear in prepared leveled spells.
- Cantrips spend no slots.
- Higher slots apply each eligible upcast step once.
- Created slots disappear on long rest.
- Slot conversion respects the sorcery-point maximum.
- Only Empowered Spell combines with another default metamagic.
- Craft failure consumes the defined ingredients once.
- Exceptional recipe creates two stable instances.
- Currency calculations do not use float.

## Acceptance check

- One shop supports buy and sell.
- One equipment change updates the sheet.
- Three cantrips can be known, and three leveled spells can be known and prepared.
- The sheet shows normal slots, temporary slots, and optional sorcery points separately.
- Flexible casting and the eight metamagic definitions validate from rules-profile data.
- Loot uses deterministic results.
- Mending Tonic supports flawed, standard, and exceptional outcomes.
- Save and reload preserve instances, shop stock, and recipe knowledge.

## Suggested commit

```powershell
git add .
git commit -m "Add magic inventory economy and crafting"
```
