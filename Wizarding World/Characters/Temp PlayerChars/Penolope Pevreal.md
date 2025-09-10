---
Type: Player
Art: "![[Character Image Placholder]]"
Level: 5
AC: 12
Prof: 3
HP: 40
HitDice: d10
Speed: 30
STR: 11
DEX: 12
CON: 13
INT: 12
WIS: 16
CHA: 13
House: Gryffindor
SchoolOfMagic: Jinxes, Hexes and Curses
Gender: Female
Age: 15
Location: [[Hogwarts]]
CastingStyle: Willpower
Background: Socialite
Heritage: PureBlood
Wand: Apple, Dragon Heartstring – 13" swishy
Likes: []
Dislikes: []
Pronouns: []
Mascota: Jumper, Harlequin Toad
SorceryPoints: 4
MetamagicPoints: 1
PersonalityTrait: []
SocialTrait: []
MentalTrait: []
Proficiencies: []
Resistances: []
Languages: []
DmgTkn: 0
TempHP: 0
Knut: 30
Sickles: 15
Galeons: 3
Ruby: 0
---

## `=this.file.name`

>[!column|flex 2]
>> [!infobox]
>> # `=this.file.name`
>> ![[Penlope-pevreal.jpg]]
>> ###### Stats
>> | | |
>> |---|---|
>> **Level** | `=this.Level` |
>> **Speed** | `=this.Speed` |
>> **Proficiency** | +`=this.Prof` |
>> **Initiative** | +`=floor((this.DEX-10)/2)` |
>> **AC** | `=this.AC` |
>> **HP** | `=this.HP - this.DmgTkn + this.TempHP` |
>> **Hit Dice** | `=this.Level & this.HitDice` |
>> **Passive Perception** | `=floor((this.WIS-10)/2)+10` |
>> **Sorcery Points** | `=this.SorceryPoints` |
>> **Metamagic Points** | `=this.MetamagicPoints` |
>>
>> ###### Bio
>> | | |
>> |---|---|
>> **House** | `=this.House` |
>> **Sex** | `=this.Gender` |
>> **Age** | `=this.Age` |
>> **Background** | `=this.Background` |
>> **School of Magic** | `=this.SchoolOfMagic` |
>> **Mascota** | `=this.Mascota` |
>>
>> ###### Info
>> | | |
>> |---|---|
>> **Casting Style** | `=this.CastingStyle` |
>> **Wand** | `=this.Wand` |
>> **Heritage** | `=this.Heritage` |
>> **Current Location** | `=this.Location` |
>>
>> ### Currency
>> | Knut | Sickles | Galeons | Ruby |
>> |---|---|---|---|
>> `=this.Knut` | `=this.Sickles` | `=this.Galeons` | `=this.Ruby` |

>> [!infobox] Death Saves
>> ### Death Saves
>> | Success | <input type="checkbox" unchecked> | <input type="checkbox" unchecked> | <input type="checkbox" unchecked> |
>> |---|---|---|---|
>> | Fails | <input type="checkbox" unchecked> | <input type="checkbox" unchecked> | <input type="checkbox" unchecked> |

>> ### Ability Scores
>> | Stat | Score | Mod | Prof | ST |
>> |---|---|---|---|---|
>> STR | `=this.STR` | +`=floor((this.STR-10)/2)` | <input type="checkbox" unchecked> | +`=floor((this.STR-10)/2)` |
>> DEX | `=this.DEX` | +`=floor((this.DEX-10)/2)` | <input type="checkbox" unchecked> | +`=floor((this.DEX-10)/2)` |
>> CON | `=this.CON` | +`=floor((this.CON-10)/2)` | <input type="checkbox" unchecked> | +`=floor((this.CON-10)/2)` |
>> INT | `=this.INT` | +`=floor((this.INT-10)/2)` | <input type="checkbox" unchecked> | +`=floor((this.INT-10)/2)` |
>> WIS | `=this.WIS` | +`=floor((this.WIS-10)/2)` | <input type="checkbox" unchecked> | +`=floor((this.WIS-10)/2)` |
>> CHA | `=this.CHA` | +`=floor((this.CHA-10)/2)` | <input type="checkbox" unchecked> | +`=floor((this.CHA-10)/2)` |

>> ### Skills
>> | Skill | Prof | Mod |
>> |---|---|---|
>> Acrobatics (DEX) | <input type="checkbox" unchecked> | +`=floor((this.DEX-10)/2)` |
>> Magical Creatures (WIS) | <input type="checkbox" unchecked> | +`=floor((this.WIS-10)/2)` |
>> Magical Theory (INT) | <input type="checkbox" unchecked> | +`=floor((this.INT-10)/2)` |
>> Athletics (STR) | <input type="checkbox" unchecked> | +`=floor((this.STR-10)/2)` |
>> Deception (CHA) | <input type="checkbox" unchecked> | +`=floor((this.CHA-10)/2)` |
>> Muggle Studies (INT) | <input type="checkbox" unchecked> | +`=floor((this.INT-10)/2)` |
>> Insight (WIS) | <input type="checkbox" unchecked> | +`=floor((this.WIS-10)/2)` |
>> Intimidation (CHA) | <input type="checkbox" unchecked> | +`=floor((this.CHA-10)/2)` |
>> Investigation (INT) | <input type="checkbox" unchecked> | +`=floor((this.INT-10)/2)` |
>> Medicine (WIS) | <input type="checkbox" unchecked> | +`=floor((this.WIS-10)/2)` |
>> Herbology (INT) | <input type="checkbox" unchecked> | +`=floor((this.INT-10)/2)` |
>> Perception (WIS) | <input type="checkbox" unchecked> | +`=floor((this.WIS-10)/2)` |
>> Performance (CHA) | <input type="checkbox" unchecked> | +`=floor((this.CHA-10)/2)` |
>> Persuasion (CHA) | <input type="checkbox" unchecked> | +`=floor((this.CHA-10)/2)` |
>> Sleight of Hand (DEX) | <input type="checkbox" unchecked> | +`=floor((this.DEX-10)/2)` |
> Stealth (DEX) | <input type="checkbox" unchecked> | +`=floor((this.DEX-10)/2)` |
>Survival (WIS) | <input type="checkbox" unchecked> | +`=floor((this.WIS-10)/2)` |
> Potion Making (INT) | <input type="checkbox" unchecked> | +`=floor((this.INT-10)/2)` |



