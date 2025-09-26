---
Type: Player
Art:
Level: 1
AC: 13
Prof: "2"
HP: 12
HitDice: d10
Speed: 14
STR: 15
DEX: 12
CON: 13
INT: 12
WIS: 12
CHA: 15
House: Slytherin
SchoolOfMagic: Jinxes, Hexes and Curses
Gender: Male
Age: "13"
Location:
  - - Hogwarts
CastingStyle: Willpower
Background: Socialite
Heritage: PureBlood
Wand: Caoba,30cm ,Fibra de corazon de dragon, Muy flexible
Likes: NONE
Dislikes: NONE
Pronouns: NONE
Mascota: Jumper, Harlequin Toad
SorceryPoints: 4
MetamagicPoints: 1
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
Knut: 0
Sickles: 0
Galeons: 2
Ruby: 1
---

## `=this.file.name`

>[!column|flex 2]
>> [!infobox]
>> # `=this.file.name`
>>
>> ###### Stats
>>  |
>> ---|---|
>> **Level** | `=this.Level` |
>> **Speed** | `=this.Speed` |
>> **Proficiency** | +`=this.Prof` |
>> **Initiative** | +`=floor((this.DEX - 10)/2)` |
>> **AC** | `=this.AC + floor((this.DEX - 10)/2)` |
>> **HP** | `=this.HP - this.DmgTkn + this.TempHP` |
>> **Hit Dice** | `=this.Level + this.HitDice` |
>> **Passive Perception** | `=floor((this.WIS - 10)/2+10)` |
>> **Sorcery Points** | `=this.SorceryPoints` |
>> **Metamagic Points** | `=this.MetamagicPoints` |
>> ###### Bio
>>   |
>> ---|---|
>> **House** | `=this.House` |
>> **Sex** | `=this.Gender` |
>> **Age** | `=this.Age` |
>> **Background** | `=this.Background` |
>> **School of Magic** | `=this.SchoolOfMagic` |
>> **Mascota** | `=this.Mascota` |
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
| Success | <input type="checkbox" unchecked> | <input type="checkbox" unchecked> | <input type="checkbox" unchecked> | 
| ----- | --- | --------------------------------- | --------------------------------- |
>>
| Fails | <input type="checkbox" unchecked> | <input type="checkbox" unchecked> | <input type="checkbox" unchecked> |
| ----- | --- | --------------------------------- | --------------------------------- |
>>
>> ### Skills
| Skill | Score       | Mod                     | Prof                              | ST                                  |
| ----- | ----------- | ----------------------- | --------------------------------- | ----------------------------------- |
| <font color="#ff0000">**STR**</font> | `=this.STR` | +`=floor((this.STR - 10)/2)` | <input type="checkbox" unchecked> | +`=floor((this.STR - 10)/2)` |
| <font color="#ffff00">**DEX**</font> | `=this.DEX` | +`=floor((this.DEX - 10)/2)` | <input type="checkbox" checked> | +`=floor((this.DEX - 10)/2)` |
| <font color="#00b050">**CON**</font> | `=this.CON` | +`=floor((this.CON - 10)/2)` | <input type="checkbox" unchecked> | +`=floor((this.CON - 10)/2)` |
| <font color="#7030a0">**INT**</font> | `=this.INT` | +`=floor((this.INT - 10)/2)` | <input type="checkbox" unchecked> | +`=floor((this.INT - 10)/2)` |
| <font color="#245bdb">**WIS**</font> | `=this.WIS` | +`=floor((this.WIS - 10)/2)` | <input type="checkbox" unchecked> | +`=floor((this.WIS - 10)/2)` |
| <font color="#de7802">**CHA**</font> | `=this.CHA` | +`=floor((this.CHA - 10)/2)` | <input type="checkbox" checked> | +`=floor((this.CHA - 10)/2)` |
>> ### Skill Checks
| Ability               | Prof                                   | Mod |
| --------------------- | --------------------------------- | --- |
| Acrobatics (DEX)      | <input type="checkbox" unchecked> | +`=floor((this.DEX - 10)/2)` |
| Magical Creatures (WIS) | <input type="checkbox" unchecked> | +`=floor((this.WIS - 10)/2)` |
| Magical Theory (INT)  | <input type="checkbox" unchecked> | +`=floor((this.INT - 10)/2)` |
| Athletics (STR)       | <input type="checkbox" checked> | +`=floor((this.STR - 10)/2) +1` |
| Deception (CHA)       | <input type="checkbox" unchecked> | +`=floor((this.CHA - 10)/2+1)` |
| Muggle Studies (INT)  | <input type="checkbox" unchecked> | +`=floor((this.INT - 10)/2)` |
| Insight (WIS)         | <input type="checkbox" unchecked> | +`=floor((this.WIS - 10)/2)` |
| Intimidation (CHA)    | <input type="checkbox" checked> | +`=floor((this.CHA - 10)/2)` |
| Investigation (INT)   | <input type="checkbox" unchecked> | +`=floor((this.INT - 10)/2)` |
| Medicine (WIS)        | <input type="checkbox" unchecked> | +`=floor((this.WIS - 10)/2)` |
| Herbology (INT)       | <input type="checkbox" unchecked> | +`=floor((this.INT - 10)/2)` |
| Perception (WIS)      | <input type="checkbox" unchecked> | +`=floor((this.WIS - 10)/2)` |
| Performance (CHA)     | <input type="checkbox" unchecked> | +`=floor((this.CHA - 10)/2)` |
| Persuasion (CHA)      | <input type="checkbox" checked> | +`=floor((this.CHA - 10)/2+2)` |
| Sleight of Hand (DEX) | <input type="checkbox" unchecked> | +`=floor((this.DEX - 10)/2)` |
| Stealth (DEX)         | <input type="checkbox" unchecked> | +`=floor((this.DEX - 10)/2)` |
| Survival (WIS)        | <input type="checkbox" checked> | +`=floor((this.WIS - 10)/2)` |
| Potion Making (INT)   | <input type="checkbox" checked> | +`=floor((this.INT - 10)/2 + 1)` |

## Atributos

- **Aumento de Puntuación de Habilidad:** Tu puntuación de Destreza, Carisma y una habilidad de tu elección aumentan en 1.
- **Información Comprometedora:** Cuando usas secretos de alguien en una prueba de Carisma, obtienes competencia doble.
- **Calidad Noble:** Puedes adoptar una personalidad de importancia para mezclarte entre oficiales y magos prestigiosos. 
- Obtienes un conjunto de Herramientas de Rompemaldiciones y competencia con ellas.
**Rasgo de Trasfondo:** Campeón del Pueblo  
_Tu manera de conducirte hace que los demás confíen en que puedes ayudarlos cuando están acosados o en peligro. Mantener tu reputación puede llevarte a ser prefecto, capitán de equipo u ocupar otros puestos de autoridad menores._
## Feats
## Wandless Magic

Has aprendido a realizar pequeños actos de magia **sin varita**.

**Beneficios:**

- Puedes lanzar los siguientes hechizos sin varita ni componentes somáticos, si ya los conoces:  
    _accio, alohomora, colovaria, illegibilus, incendio, glacia, pereo, wingardium leviosa._
- No puedes usar **espacios de conjuro de nivel superior** al lanzarlos de esta forma.

- **Entrenamiento de Auror.**  
    Ya has comenzado a practicar las habilidades necesarias para convertirte en un Auror.  
    Aprendes una receta de poción común y obtienes competencia en dos de las siguientes habilidades: Investigación, Elaboración de Pociones, Sigilo o Supervivencia.
## Equipo

- [ ] Varita magica
- [ ] Una Bolsa de estudiante rico
  - [ ] Bolsa que no se rompe
  - [ ] 3 cambios de ropa negra con su nombre embordado
  - [ ] Gorrita Elegante
  - [ ] 3 rollos de 10 metros de papel
  - [ ] 5 plumillas
  - [ ] 2 bolletas de tinte negro
  - [ ] 1 botella de tinte esmeralda
  - [ ] 1 botella de tinte rojo
- [ ] Una Capa de Seda 11 AC base + Modificador Dex
- [ ] Herramientas de Rompemaldiciones

### Transfondo del personaje



Ideal - Justicia. Nadie debe recibir trato preferencial, y nadie está por encima de la justicia.
Vinculo - Protejo a quienes no pueden protegerse por sí mismos.
Defecto -	Tengo poco respeto por quien no se atreve a defender lo correcto.
## Magia



### Encantamientos



### Hechizos
