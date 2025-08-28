---
Type: Player
Art: "![[Character Image Placholder]]"
Level: 5
AC: 13
Prof: "3"
HP: 23
HitDice: d8
Speed: 30
STR: 8
DEX: 16
CONST: 8
INT: 17
WIS: 16
CHA: 13
House: Ravenclaw
SchoolOfMagic: Charms
Gender: Female
Age: "14"
Location: [[[[[[[[[[[[[[Hogwarts]]]]]]]]]]]]]], England
CastingStyle: Intellect
Background: Groundskeeper
Heritage: HalfBlood
Wand: Vine, Dragon HeartString - 10" Unyealding
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
>> ![[OscarOrtiz_Img_PlaceHolder]]
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

## Atributos

- **Conocimiento Profundo**: Siempre que realices una prueba de habilidad de Inteligencia o Sabiduría en la que añadas tu bono de competencia, puedes tratar un lanzamiento de d20 de 5 o menos como un 6.

- **Biblioteca de Rowena**: Puedes investigar fácilmente un tema deseado al pedir ayuda a tus compañeros de casa y consultar libros exclusivos de la sala común de Ravenclaw.

<<<<<<< HEAD:Characters/Temp PlayerChars/Valentina Voraz.md
- **Detectar Rastros**: Has aprendido a sentir la magia y reconocer los estilos de hechizos y maldiciones. Usando concentración, puedes detectar la presencia de magia en un radio de 30 pies de ti. Si detectas magia de esta manera, puedes usar tu acción para ver una tenue aura alrededor de cualquier criatura u objeto visible en el área que posea magia, y aprender la escuela de magia asociada al hechizo, si la hay. Esta habilidad puede penetrar la mayoría de barreras, pero se ve bloqueada por 1 pie de piedra, 1 pulgada de metal común, o 3 pies de madera o tierra.
## Feats - class
=======
- **Viajes a la Cocina**. Tu experiencia con los elfos domésticos de [[Hogwarts]] te ha enseñado cómo tratar y relacionarte educadamente con los seres mágicos. Ah, y puedes conseguir montones de postres.
>>>>>>> origin/main:Wizarding World/Characters/Temp PlayerChars/Valentina Voraz.md

- **Disparo Preciso**: Has perfeccionado tu puntería para golpear específicamente con tus Hechizos de Duelo. Puedes apuntar a objetos o partes del cuerpo y restringir el efecto del hechizo a esa área.

School of Magic Spells Spell Level Spell 2nd muffliato 4th capacious extremis 5th obliviate, piertotum locomotor
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

Creyente en la redención, esta bruja o mago posee una varita de álamo con un núcleo de pelo de unicornio. Este núcleo simboliza su creencia inquebrantable de que incluso los corazones más oscuros albergan un destello de bondad. Su vínculo con su varita, de apenas 10,5 pulgadas, revela una profunda conexión con el mundo natural. Cualquier daño a la naturaleza es una herida personal, lo que refleja su feroz protección hacia el medio ambiente.

Aquí hay algunos detalles adicionales sobre el personaje:

Ideales: **Conocimiento**. El camino hacia el poder y la superación personal es a través del conocimiento..
Lazos: Quiero que mi obra de vida sean libros de texto mágicos relacionados con un campo específico. 
Desventaja: Descubrir conocimiento vale cualquier precio, incluso una parte de mi propia humanidad.

## Magia

Conoce 8 Encantamientos y 16 Hechisos

### Encantamientos

- Accio - Atrai objetos no mayor a 20kg de peso
- Alohomora - Puede puertas/ventanas pero hace ruido
- Lumos/Nox - Genera Luz en la punta de la varita  
- Wingadiam leviosa - Puedes Levantar hasta 50k por 1 min hasta 10 m de alto
- Rennervate - Despierta una creatura inconsciente
- Genu Recurvatum - Falsea la rodilla de una creatura reduce velocidad x 1 min
- Aguamenti - Convoca agua hasta 1 min en un cono de 10 m
- Scurgify - limpia un objeto o creatura no mayor a 2.5 m cuadros

### Hechizos

#### Primer circulo 4

- Arresto momentum - Reduce la velocidad de caida de hasta 5 creaturas a 30M/ps Puede ser reacion
- Diffindo - Corta una creatura por 4d4
- Protego - Accion o Reacion genera una barrera protectora +5 a AC
- Riddikules - Transforma un Boggart a una version chistosa
- Flipendo - Voltea una creatura generando 1d10 de dano
- Rictusempra - Incapacita a una creatura de risa un turno y reduce movilidad
- Specialis Revelio - Revela los secretos de un objeto, area. *Leer hechizo en manual*

#### Segundo circulo 3

- Episkey - Sana 2d4 + modifcador de hechizos una creatura dentro de 10m
- Reparo - Repara objetos dañados
- Stupefy - una creatura dentro de 20m se caí inconsciente
- Immobulus - Imobaliza en un rango de 7.5m a creaturas de 7d10 de puntos de vida
- Expelliarmus - Una creatura de 30m avienta un articulo y cai 5m de distancia

#### Tercer circulo 2

- Herbiviscus - Puedes Crecer plantas de 60 pies cuadrados causando terreno dificil
- Expluso - avientas a una creatura 10 pies con un trueno 4d8 dmg, se esucha el trueno a 100 pies
- Expecto Patronum - Te protege de criaturas obsuras como dementos, banshees, inferi y obscuri dependiendo de lo agil
