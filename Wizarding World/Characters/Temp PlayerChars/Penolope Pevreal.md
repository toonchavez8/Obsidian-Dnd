---
Type: Player
Art: "![[Character Image Placholder]]"
Level: 5
AC: 12
Prof: "3"
HP: 40
HitDice: d10
Speed: 30
STR: 11
DEX: 12
CONST: 13
INT: 12
WIS: 16
CHA: 17
House: Gryffindor
SchoolOfMagic: Jinxes,Hexes and Curses
Gender: Female
Age: "15"
Location: [[Hogwarts]] England
CastingStyle: Willpower
Background: Socialite
Heritage: PureBlood
Wand: Apple, Dragon Heartstring- 13" swishy
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
>>  |
>> ---|---|
>> **Level** |`=this.level` |
>>  **Speed** |`=this.Speed` |
>> **Proficiency** | +`=this.Prof` |
>> **Initiative** | +`=floor((this.DEX - 10)/2)` |
>> **AC** | `=this.AC`
>> **HP** | `=this.HP - this.DmgTkn + this.TempHP` |
>> **Hit Dice** | `=this.Level + this.HitDice`  |
>> **Passive Perception** | `=floor((this.WIS - 10)/2+10)` |
>>  **Sorcery Points** | `=this.SorceryPoints` |
>>  **Metamagic Points** | `=this.MetamagicPoints` |
>> ###### Bio
>>   |
>> ---|---|
>> **House** | `=this.House` |
>> **Sex** | `=this.gender` |
>> **Age** | `=this.age` |
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
| Success | <input type="checkbox" unchecked>  | <input type="checkbox" unchecked> | <input type="checkbox" unchecked> | 
| ----- | --- | --------------------------------- | --------------------------------- |
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
| <font color="#de7802">**CHA**</font>   | `=this.CHA`          | +`=floor((this.CHA - 10)/2)`   | <input type="checkbox" unchecked> | +`=floor((this.CHA - 10)/2)`               |
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
## Atributos - House

**Corazón Valiente**: Tienes ventaja en las tiradas de salvación contra el miedo proveniente de cualquier fuente, excepto de un Dementor.

**Verdadera Gryffindor**: En momentos de extrema necesidad, la Espada de Gryffindor puede presentarse ante ti.

<<<<<<< HEAD:Characters/Temp PlayerChars/Penolope Pevreal.md
- **Encanto de Veela**: Como descendiente de veela, tienes ventaja en una habilidad de Carisma a elección. Puedes intentar encantar a una criatura humana dentro de 9 metros, obligándola a hacer una tirada de salvación de Sabiduría. Si falla, estará encantada por una hora o hasta que tú o tus compañeros le hagan daño.
## Feats - Class
=======
- **Viajes a la Cocina**. Tu experiencia con los elfos domésticos de [[Hogwarts]] te ha enseñado cómo tratar y relacionarte educadamente con los seres mágicos. Ah, y puedes conseguir montones de postres.

## Feats

- **Cuidador**. Tu estudio de las criaturas mágicas te ha enseñado sobre sus lesiones y fisiología. Puedes lanzar cualquier hechizo de Curación conocido sobre bestias.
- **Folio Bruti**. Tienes tu propio cuaderno personal de bestias donde registras tus hallazgos. Cada vez que añadas la competencia de Criaturas Mágicas a una prueba de Habilidad, añade tu modificador de Inteligencia como bonificación también.
>>>>>>> origin/main:Wizarding World/Characters/Temp PlayerChars/Penolope Pevreal.md

**Rompehechizos**: Tu curiosidad por desmantelar hechizos y encantamientos ha encontrado una salida. Obtienes un set de herramientas de rompehechizos y eres competente en su uso.
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
- [ ] Una Capa de inverno 11 AC base + Modificador Dex (max 2)
- [ ] Juego de herramientas de jardineras
  - [ ] Una paleta de jardinería (trowel)
  - [ ] Un cultivador de mano (hand cultivator)
  - [ ] Tijeras de podar (pruning shears)
  - [ ] Guantes de piel de dragón (dragon-hide gloves) +2 AC cuando los trai puesto
  - [ ] Hilo de Bramante (twine)
  - [ ] Sacos pequeños de arpillera (small burlap sacks)
  - [ ] Unas cuantas macetas pequeñas (a few small pots)
  - [ ] Un par de orejeras (earmuffs)
- [ ] a compass

### Transfondo del personaje

Penélope Pevreal es un imán social, cautivando a las multitudes con su carisma y encanto. Tras su sonrisa encantadora se esconde una mente aguda y una ambición ardiente. Su varita de madera de manzano, símbolo de liderazgo e idealismo, la impulsa a inspirar a otros. Sin embargo, su romanticismo desmedido y su susceptibilidad a la adulación podrían convertirla en un peón o desviarla de su verdadero propósito. ¿Podrá Penélope superar sus debilidades y alcanzar su máximo potencial?

Aquí hay algunos detalles adicionales sobre el personaje:

Núcleo (Ideal): Cuerda de Corazón de Dragón - Poder. Los fuertes están destinados a liderar y los débiles a seguir.

Longitud (Vínculo): 13 pulgadas - Es mi deber liderar e inspirar a otros.

Flexibilidad (Defecto): Ondeada - Soy una romántica (equivocada) y me enamoro fácilmente de una cara bonita.

## Magia

Conoce 5 Encantamientos y 8 Hechisos


### Encantamientos

- Bombarda -Dark- 2D10 de explosion pequena, rango 30m -accion
- Devicto - 2d6 de daño de fuerza, un ardor sobre la creatura - accion
- Furnunculus - Casusa 2D6 daño psicico y cara de granos - accion 
- Locomoter Wibbly - cause que se caiga un enmigo al piso - accion
- Incendio Glacia - Convoca flamas azules en tu mano o varita una hora - accion


### Hechizos 8

#### Primer circulo 4

- Episkey - Sana 2d4 + competencia dentro de un rango de 5m - Bonus
- Petrificus Totalus - paraliza una creatura 1 minuto dentro de rango de 30m
- Protego - Accion o Reacion genera una barrera protectora +5 a AC
- Colloshoo - Ata a al piso a un enemigo, Spell save Dc si falla por 5 o mas se cai al piso y tiene que usar STR cada turno para safarse


#### Segundo circulo 3

- muffliato - Puedes silenciaar una zona de 15m, concentracion - 1 hora
- Ariania Exummai - en un cono de 15m cualquier arrana hace on Con save y falla recibe 4d6 daño radiante y es empujado hacia atras 5m, si pasa medio dano
- Ventus - genera un torbellino de viento moviendo creaturas 5 m

#### Tercer circulo 2

- Confringo - Dark - Explosion de 5m de radio hasta 45m de rango genere 8d6 en un dex save daño de fuero y lumbre 


