---
Type: Player
Art: "![[Character Image Placholder]]"
Level: 5
AC: 0
Prof: 
HP: 34
HitDice: d0
Speed: 0
STR: 11
DEX: 11
CONST: 15
INT: 16
WIS: 18
CHA: 14
House: Hufflepuff
SchoolOfMagic: Magizoology
Gender: GenderName
Age: "14"
Location: Hogwarts, England
CastingStyle: Intellect
Heritage: HalfBlood
Wand: WandName
Likes: NONE
Dislikes: NONE
Pronouns: NONE
PersonalityTrait:
  - NONE
SocialTrait:
  - NONE
MentalTrait:
  - NONE
Proficiencies:
  - NONE
Resistances:
  - NONE
Languages:
  - NONE
DmgTkn: 0
TempHP: 0
Knut: 68
Sickles: 42
Galeons: 6
Ruby: 0
---

## `=this.file.name`

>[!column|flex 2]
>> [!infobox]
>> # `=this.file.name`
>> ![[Character Image Placeholder]]
>> ###### Stats
>>  |
>> ---|---|
>> **Level** |`=this.level` |
>>  **Speed** |`=this.Speed` |
>> **Proficiency** | +`=this.Prof` |
>> **Initiative** | +`=(this.DEX - 10)/2` |
>> **AC** | `=this.AC`
>> **HP** | `=this.HP - this.DmgTkn + this.TempHP` |
>> **Hit Dice** | `=this.Level + this.HitDice`  |
>> **Passive Perception** |
>>  
>> ###### Bio
>>   |
>> ---|---|
>> **House** | `=this.House` |
>> **Sex** | `=this.gender` |
>> **Age** | `=this.age` |
>> **Sexuality** | `=this.sexuality` |
>> **School of Magic** | `=this.SchoolOfMagic` |
>> ###### Info
>>   |
>> ---|---|
>> **Casting Style** | `=this.CastingStyle` |
>> **Wand Name** | `=this.Wand` |
>> **Heritage** | `=this.Heritage` |
>> **Current Location** | `=this.Location` |
>>  ### Currency
| Knut         | Sickles         | Galeons         | Ruby         |
| -------------- | -------------- | ------------  | ------------ |
| `=this.Knut` | `=this.Sickles` | `=this.Galeons` | `=this.Ruby` |
>
>> [!infobox] Death Saves
>> ### Death Saves
| Success | <input type="checkbox" unchecked>  | <input type="checkbox" unchecked> | <input type="checkbox" unchecked> | 
| ------- | --- | --------------------------------- | --------------------------------- |
>>
| Fails | <input type="checkbox" unchecked>  | <input type="checkbox" unchecked> | <input type="checkbox" unchecked> |
| ----- | --- | --------------------------------- | --------------------------------- |
>>
>> ### Skills
| Skill | Score       | Mod                     | Prof                              | ST                                  |
| ----- | ----------- | ----------------------- | --------------------------------- | ----------------------------------- |
| <font color="#ff0000">**STR**</font>   | `=this.STR` | +`=floor((this.STR - 10)/2)`   | <input type="checkbox" unchecked> | +`=floor((this.STR - 10)/2)`               |
| <font color="#ffff00">**DEX**</font>   | `=this.DEX`  | +`=floor((this.DEX - 10)/2)`   | <input type="checkbox" unchecked> | +`=floor((this.DEX - 10)/2)`               |
| <font color="#00b050">**CON**</font>   | `=this.CONST` | +`=floor((this.CONST - 10)/2)` | <input type="checkbox" unchecked>   | +`=floor(((this.CONST - 10)/2))` |
| <font color="#7030a0">**INT**</font>   | `=this.INT`          | +`=floor((this.INT - 10)/2)`   | <input type="checkbox" unchecked>   | +`=floor(((this.INT - 10)/2))`   |
| <font color="#245bdb">**WIS**</font>   | `=this.WIS`          | +`=floor((this.WIS - 10)/2)`   | <input type="checkbox" unchecked> | +`=floor((this.WIS - 10)/2)`               |
| <font color="#de7802">**CHA**</font>   | `=this.CHA`          | +`=floor((this.CHA - 10)/2)`   | <input type="checkbox" unchecked> | +`=(this.CHA - 10)/2`               |
>> ### Skill Checks
| Ability               | Prof                                   | Mod |
| --------------------- | --------------------------------- | --- |
| Acrobatics (DEX)      | <input type="checkbox" unchecked> | +`=floor((this.DEX - 10)/2)`   |
| Magical Creatures (WIS) | <input type="checkbox" unchecked> | +`=floor((this.WIS - 10)/2)`  |
| Magical Theory (INT)          | <input type="checkbox" unchecked> | +`=floor((this.INT - 10)/2)`  |
| Athletics (STR)       | <input type="checkbox" unchecked> | +`=floor((this.STR - 10)/2)`   |
| Deception (CHA)       | <input type="checkbox" unchecked> | +`=floor((this.CHA - 10)/2)`  |
| Muggle Studies (INT)         | <input type="checkbox" unchecked> | +`=floor((this.INT - 10)/2)`  |
| Insight (WIS)         | <input type="checkbox" unchecked>   | +`=floor((this.WIS - 10)/2)`  |
| Intimidation (CHA)    | <input type="checkbox" unchecked> | +`=floor((this.CHA - 10)/2)`  |
| Investigation (INT)   | <input type="checkbox" unchecked>   | +`=floor((this.INT - 10)/2)`  |
| Medicine (WIS)        | <input type="checkbox" unchecked> | +`=floor((this.WIS - 10)/2)`  |
| Herbology (INT)          | <input type="checkbox" unchecked> | +`=floor((this.INT - 10)/2)`  |
| Perception (WIS)      | <input type="checkbox" unchecked>   | +`=floor((this.WIS - 10)/2)`  |
| Performance (CHA)     | <input type="checkbox" unchecked> | +`=floor((this.CHA - 10)/2)`  |
| Persuasion (CHA)      | <input type="checkbox" unchecked> | +`=floor((this.CHA - 10)/2)`  |
| Sleight of Hand (DEX) | <input type="checkbox" unchecked> | +`=floor((this.DEX - 10)/2)`   |
| Stealth (DEX)         | <input type="checkbox" unchecked> | +`=floor((this.DEX - 10)/2)`   |
| Survival (WIS)        | <input type="checkbox" unchecked> | +`=floor((this.WIS - 10)/2)`  |
| Potion Making (INT)          | <input type="checkbox" unchecked> | +`=floor((this.INT - 10)/2)`  |

## Atributos

- **Aumento de Puntuaciones de Característica**. Tu puntuación de Constitución, Sabiduría y una puntuación de característica adicional a tu elección aumentan en 1.

- **Lealtad Inquebrantable**. Tienes ventaja en las tiradas de salvación contra cualquier efecto que te haga atacar o actuar en contra de una criatura que normalmente considerarías un aliado.

- **Viajes a la Cocina**. Tu experiencia con los elfos domésticos de Hogwarts te ha enseñado cómo tratar y relacionarte educadamente con los seres mágicos. Ah, y puedes conseguir montones de postres. 

## Feats

