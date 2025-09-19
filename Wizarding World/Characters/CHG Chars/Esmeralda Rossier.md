---
Type: Player
Art:
Level: 1
AC: 15
Prof: "2"
HP: 14
HitDice: d10
Speed: 15
STR: 12
DEX: 13
CON: 18
INT: 11
WIS: 12
CHA: 17
House: Gryffindor
SchoolOfMagic: Adivinación
Gender: Male
Age: "13"
Location:
  - - Hogwarts
CastingStyle: Willpower
Background: Socialite
Heritage: PureBlood
Wand: Álamo Temblón,25cm ,Pluma de Fénix, Firme
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
| <font color="#ffff00">**DEX**</font> | `=this.DEX` | +`=floor((this.DEX - 10)/2)` | <input type="checkbox" unchecked> | +`=floor((this.DEX - 10)/2)` |
| <font color="#00b050">**CON**</font> | `=this.CON` | +`=floor((this.CON - 10)/2)` | <input type="checkbox" checked> | +`=floor((this.CON - 10)/2)` |
| <font color="#7030a0">**INT**</font> | `=this.INT` | +`=floor((this.INT - 10)/2)` | <input type="checkbox" unchecked> | +`=floor((this.INT - 10)/2)` |
| <font color="#245bdb">**WIS**</font> | `=this.WIS` | +`=floor((this.WIS - 10)/2)` | <input type="checkbox" unchecked> | +`=floor((this.WIS - 10)/2)` |
| <font color="#de7802">**CHA**</font> | `=this.CHA` | +`=floor((this.CHA - 10)/2)` | <input type="checkbox" checked> | +`=floor((this.CHA - 10)/2)` |
>> ### Skill Checks
| Ability               | Prof                                   | Mod |
| --------------------- | --------------------------------- | --- |
| Acrobatics (DEX)      | <input type="checkbox" unchecked> | +`=floor((this.DEX - 10)/2)` |
| Magical Creatures (WIS) | <input type="checkbox" unchecked> | +`=floor((this.WIS - 10)/2)` |
| Magical Theory (INT)  | <input type="checkbox" unchecked> | +`=floor((this.INT - 10)/2)` |
| Athletics (STR)       | <input type="checkbox" unchecked> | +`=floor((this.STR - 10)/2)` |
| Deception (CHA)       | <input type="checkbox" checked> | +`=floor((this.CHA - 10)/2 +1)` |
| Muggle Studies (INT)  | <input type="checkbox" unchecked> | +`=floor((this.INT - 10)/2)` |
| Insight (WIS)         | <input type="checkbox" unchecked> | +`=floor((this.WIS - 10)/2)` |
| Intimidation (CHA)    | <input type="checkbox" unchecked> | +`=floor((this.CHA - 10)/2+1)` |
| Investigation (INT)   | <input type="checkbox" unchecked> | +`=floor((this.INT - 10)/2)` |
| Medicine (WIS)        | <input type="checkbox" unchecked> | +`=floor((this.WIS - 10)/2)` |
| Herbology (INT)       | <input type="checkbox" unchecked> | +`=floor((this.INT - 10)/2)` |
| Perception (WIS)      | <input type="checkbox" unchecked> | +`=floor((this.WIS - 10)/2)` |
| Performance (CHA)     | <input type="checkbox" unchecked> | +`=floor((this.CHA - 10)/2)` |
| Persuasion (CHA)      | <input type="checkbox" checked> | +`=floor((this.CHA - 10)/2 +1)` |
| Sleight of Hand (DEX) | <input type="checkbox" unchecked> | +`=floor((this.DEX - 10)/2)` |
| Stealth (DEX)         | <input type="checkbox" unchecked> | +`=floor((this.DEX - 10)/2)` |
| Survival (WIS)        | <input type="checkbox" unchecked> | +`=floor((this.WIS - 10)/2)` |
| Potion Making (INT)   | <input type="checkbox" unchecked> | +`=floor((this.INT - 10)/2 + 1)` |

## Atributos

- **Corazón Valiente:** Tienes ventaja en tiradas de salvación contra el estado de miedo, excepto si es causado por un Dementor.
- - **Verdadero Gryffindor:** En momentos de extrema necesidad, la Espada de Gryffindor puede presentarse ante ti.

### Legeremantes

La _Legeremancia_ es una habilidad mucho más tangible: navegar por las múltiples capas de la mente de una persona e interpretar lo que allí se encuentra. Los muggles lo llaman “lectura de mentes”, pero los practicantes desprecian ese término por ser ingenuo. A menudo se acompaña de la práctica de _Oclumancia_, el arte de proteger la mente de la invasión de otro legeremante. Aunque existen legeremantes naturales poco comunes, se trata de un campo de estudio que requiere gran dedicación y cuidado.

**Rasgo de Trasfondo:** Rumorista  
_Tu habilidad para conectar con otros te ha abierto puertas entre quienes saben más. Cuando intentas descubrir un secreto jugoso o un rumor peligroso, tus contactos pueden ayudarte o indicarte el camino correcto._
## Feats

## Magia Salvaje

Tu conexión con la magia es volátil y poderosa, manifestándose de manera inesperada cuando más la necesitas.

**Beneficios:**

- Una vez por descanso largo, cuando lances un hechizo, el HM puede **activar un estallido de magia salvaje**.
- Este estallido otorga **ventaja en la tirada de ataque o hace que los objetivos tengan desventaja en su tirada de salvación**.
- Sin embargo, tras el lanzamiento, debes realizar una tirada de **d20**: con un resultado de 1-2, ocurre un **efecto impredecible** (decidido por el director o una tabla de magia salvaje).
  
  - **Adivino.** Has desarrollado afinidad por la astronomía, la taseografía y la cristalomancia. Obtienes un _Kit de Adivino_ y competencia para usarlo.
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
- [ ] Una Capa de Invierno 11 AC base + Modificador Dex
- [ ] Herramientas de Rompemaldiciones

**Equipo:** Un anillo con el escudo familiar u otra joya heredada, y una pluma de pavo rea
### Transfondo del personaje
Idea: Destino. Nada ni nadie puede desviarme de mi llamado superior.

Vinculo:Es mi deber liderar e inspirar a otros.

defecto:Nunca estoy satisfecho con lo que tengo; siempre quiero más.



## Magia



### Encantamientos



### Hechizos
