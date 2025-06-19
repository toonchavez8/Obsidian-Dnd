---
Type: Player
Art: "![[Character Image Placholder]]"
Level: 5
AC: 13
Prof: "3"
HP: 32
HitDice: d6
Speed: 30
STR: 11
DEX: 17
CONST: 15
INT: 13
WIS: 13
CHA: 12
House: Slytherin
SchoolOfMagic: Healing
Gender: Male
Age: "14"
Location: [[Hogwarts]], England
CastingStyle: Technique
Background: Troublemaker
Heritage: PureBlood
Wand: Dogwood, Unicorn Hair - 11.5" stiff
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
>> ![[Rebbeca-Weasly.jpg]]
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

- **Aumento de Puntuaciones de Característica**. Tu puntuación de Constitución, Sabiduría y una puntuación de característica adicional a tu elección aumentan en 1.

- **Lealtad Inquebrantable**. Tienes ventaja en las tiradas de salvación contra cualquier efecto que te haga atacar o actuar en contra de una criatura que normalmente considerarías un aliado.

- **Viajes a la Cocina**. Tu experiencia con los elfos domésticos de [[Hogwarts]] te ha enseñado cómo tratar y relacionarte educadamente con los seres mágicos. Ah, y puedes conseguir montones de postres.

## Feats

- **Cuidador**. Tu estudio de las criaturas mágicas te ha enseñado sobre sus lesiones y fisiología. Puedes lanzar cualquier hechizo de Curación conocido sobre bestias.
- **Folio Bruti**. Tienes tu propio cuaderno personal de bestias donde registras tus hallazgos. Cada vez que añadas la competencia de Criaturas Mágicas a una prueba de Habilidad, añade tu modificador de Inteligencia como bonificación también.

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

Rebeca Weasley es un torbellino de problemas ingeniosos. Su curiosidad innata y su inclinación por las bromas la meten en aprietos con regularidad. Pero Rebeca no es una alborotadora sin cerebro; ha aprendido a ser cautelosa, evitando trampas obvias mientras perfecciona el arte de la coartada ingeniosa. Su lema es "trabaja con inteligencia, no con esfuerzo," y su habilidad para encontrar atajos, tanto literales como figurativos, la ha sacado de más de un lío.

Su varita de Fresno, ruidosa y traviesa, refleja su naturaleza juguetona, pero también es sorprendentemente capaz de magia poderosa en manos adecuadas. El núcleo de pelo de unicornio alimenta su ambición de demostrar su valía y superar a su odiada rival.  Sin embargo, la rigidez de su varita también delata un defecto: una vez que alguien cuestiona su coraje, Rebeca se lanza de cabeza al peligro, sin importar las consecuencias. ¿Podrá Rebeca canalizar su ingenio y su valentía para lograr sus objetivos sin sucumbir a la temeridad?

Aquí hay algunos detalles adicionales sobre el personaje:

Núcleo (Ideal): Pelo de Unicornio - Aspiración. Voy a demostrar que soy digna de una vida mejor.

Longitud (Vínculo): 11.5 pulgadas - Haré cualquier cosa para demostrar que soy superior a mi odiado rival.

Flexibilidad (Defecto): Rígida - Una vez que alguien cuestiona mi coraje, nunca retrocedo sin importar cuán peligrosa sea la situación.
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
