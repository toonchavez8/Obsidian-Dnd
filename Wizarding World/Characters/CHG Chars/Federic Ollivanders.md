---
Type: Player
Art:
Level: 1
AC: 12
Prof: "2"
HP: 11
HitDice: d8
Speed: 15
STR: 11
DEX: 16
CON: 16
INT: 12
WIS: 12
CHA: 11
House: Ravenclaw
SchoolOfMagic: Adivnacion
Gender: Male
Age: "11"
Location:
  - - Hogwarts
CastingStyle: Intellect
Background: Master of potions
Heritage: PureBlood
Wand: Pino,33cm,Ligeramente Elastica, fibra de corazion de dragon
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
| <font color="#00b050">**CON**</font> | `=this.CON` | +`=floor((this.CON - 10)/2)` | <input type="checkbox" unchecked> | +`=floor((this.CON - 10)/2)` |
| <font color="#7030a0">**INT**</font> | `=this.INT` | +`=floor((this.INT - 10)/2)` | <input type="checkbox" unchecked> | +`=floor((this.INT - 10)/2)` |
| <font color="#245bdb">**WIS**</font> | `=this.WIS` | +`=floor((this.WIS - 10)/2)` | <input type="checkbox" unchecked> | +`=floor((this.WIS - 10)/2)` |
| <font color="#de7802">**CHA**</font> | `=this.CHA` | +`=floor((this.CHA - 10)/2)` | <input type="checkbox" unchecked> | +`=floor((this.CHA - 10)/2)` |
>> ### Skill Checks
| Ability               | Prof                                   | Mod |
| --------------------- | --------------------------------- | --- |
| Acrobatics (DEX)      | <input type="checkbox" unchecked> | +`=floor((this.DEX - 10)/2)` |
| Magical Creatures (WIS) | <input type="checkbox" unchecked> | +`=floor((this.WIS - 10)/2)` |
| Magical Theory (INT)  | <input type="checkbox" unchecked> | +`=floor((this.INT - 10)/2)` |
| Athletics (STR)       | <input type="checkbox" unchecked> | +`=floor((this.STR - 10)/2)` |
| Deception (CHA)       | <input type="checkbox" unchecked> | +`=floor((this.CHA - 10)/2)` |
| Muggle Studies (INT)  | <input type="checkbox" unchecked> | +`=floor((this.INT - 10)/2)` |
| Insight (WIS)         | <input type="checkbox" unchecked> | +`=floor((this.WIS - 10)/2)` |
| Intimidation (CHA)    | <input type="checkbox" unchecked> | +`=floor((this.CHA - 10)/2)` |
| Investigation (INT)   | <input type="checkbox" checked> | +`=floor((this.INT - 10)/2)` |
| Medicine (WIS)        | <input type="checkbox" unchecked> | +`=floor((this.WIS - 10)/2)` |
| Herbology (INT)       | <input type="checkbox" unchecked> | +`=floor((this.INT - 10)/2)` |
| Perception (WIS)      | <input type="checkbox" unchecked> | +`=floor((this.WIS - 10)/2)` |
| Performance (CHA)     | <input type="checkbox" unchecked> | +`=floor((this.CHA - 10)/2)` |
| Persuasion (CHA)      | <input type="checkbox" unchecked> | +`=floor((this.CHA - 10)/2)` |
| Sleight of Hand (DEX) | <input type="checkbox" unchecked> | +`=floor((this.DEX - 10)/2+1)` |
| Stealth (DEX)         | <input type="checkbox" unchecked> | +`=floor((this.DEX - 10)/2)` |
| Survival (WIS)        | <input type="checkbox" unchecked> | +`=floor((this.WIS - 10)/2)` |
| Potion Making (INT)   | <input type="checkbox" checked> | +`=floor((this.INT - 10)/2)` |

## Atributos

- **Conocimiento Profundo:** Cuando realizas una prueba de Inteligencia o Sabiduría con competencia, puedes tratar una tirada de 5 o menos como si fuera un 6.
- **Biblioteca de Rowena:** Puedes investigar fácilmente cualquier tema con la ayuda de tus compañeros de casa y libros exclusivos de la sala común

## Feats

- **Sentir de peligro**.Agregas la mitad de tu bonificación de competencia a tu Iniciativa y no puedes ser sorprendido mientras estés consciente. 

## Fortuna Encantada

Algunos magos parecen contar con una suerte fuera de lo común, como si la magia misma les sonriera en los momentos más críticos.

**Beneficios:**

- Obtienes **3 puntos de suerte**.
- Cuando hagas una tirada de ataque, salvación o habilidad, puedes gastar 1 punto de suerte para **tirar un d20 adicional** y elegir cuál de los resultados usar.
- Puedes usar esta característica después de ver el resultado, pero antes de que el HM confirme si es un éxito o un fallo.
- Recuperas todos los puntos de suerte al terminar un **descanso largo**.
### Lanzamiento Ritual

Tu capacidad para recordar información te permite lanzar conjuros libremente, siempre que tengas tiempo para concentrarte.

A nivel 1, puedes lanzar un conjuro como ritual si tiene la etiqueta de ritual y lo conoces. Una versión ritual de un conjuro solo tarda 1 minuto más en lanzarse que lo normal. Además, no gasta un espacio de conjuro, lo que significa que la versión ritual de un conjuro no puede lanzarse a un nivel superior.

### Característica de Trasfondo: Aprendiz

Aunque aún no puedes crear obras originales, tienes los conocimientos y habilidades rudimentarias suficientes para **iniciar un aprendizaje bajo un mentor**, si encuentras a alguien dispuesto a enseñarte.  
Además, has practicado la **coordinación mano-ojo** y el trabajo **preciso**.
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
- [ ] Bolsa de la divinacion
  - [ ] Hojas de té
  - [ ] Taza y platillo de té
  - [ ] Tetera de viaje
  - [ ] Huesos de grindylow
  - [ ] Palitos con inscripciones rúnicas
  - [ ] Baraja de cartas de tarot
  - [ ] Bola de cristal pequeña
- [ ] a compass

### Transfondo del personaje

 Un joven mago atormentado por la verdad, Ignacio porta una varita de Lima Plateada con Cuerda de Corazón de Dragón, un instrumento que refleja su don de profecía y su alma agitada. Sus visiones del futuro, aunque fragmentadas y llenas de simbolismo, lo han convertido en un solitario, marcado por la carga de presenciar el desenlace de eventos aún por ocurrir.

Aquí hay algunos detalles adicionales sobre el personaje:

Ideal:	Personas. Quiero impactar a todos los que experimenten mi arte.
Lazos: Mi cámara/caballete/instrumento es mi posesión más preciada y me recuerda a alguien que amo.
debelidad: No hay lugar para la cautela en una vida vivida al máximo

## Magia

Conoce 5 Encantamientos y 8 Hechisos

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
