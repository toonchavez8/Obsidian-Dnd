---
Type: Player
Art:
Level: 1
AC: 12
Prof: "2"
HP: 10
HitDice: d8
Speed: 15
STR: 8
DEX: 11
CON: 14
INT: 12
WIS: 14
CHA: 14
House: Hufflepuff
SchoolOfMagic: Magizoology
Gender: Male
Age: "14"
Location:
  - - Hogwarts
CastingStyle: Intellect
Background: GuardaBosques
Heritage: Mestizo
Wand: Castano,34cm,Ligeramente Elastica, Pluma de Fenix
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
| <font color="#7030a0">**INT**</font> | `=this.INT` | +`=floor((this.INT - 10)/2)` | <input type="checkbox" checked> | +`=floor((this.INT - 10)/2)` |
| <font color="#245bdb">**WIS**</font> | `=this.WIS` | +`=floor((this.WIS - 10)/2)` | <input type="checkbox" unchecked> | +`=floor((this.WIS - 10)/2)` |
| <font color="#de7802">**CHA**</font> | `=this.CHA` | +`=floor((this.CHA - 10)/2)` | <input type="checkbox" unchecked> | +`=floor((this.CHA - 10)/2)` |
>> ### Skill Checks
| Ability               | Prof                                   | Mod |
| --------------------- | --------------------------------- | --- |
| Acrobatics (DEX)      | <input type="checkbox" unchecked> | +`=floor((this.DEX - 10)/2)` |
| Magical Creatures (WIS) | <input type="checkbox" checked> | +`=floor((this.WIS - 10)/2)+1` |
| Magical Theory (INT)  | <input type="checkbox" unchecked> | +`=floor((this.INT - 10)/2)` |
| Athletics (STR)       | <input type="checkbox" unchecked> | +`=floor((this.STR - 10)/2)` |
| Deception (CHA)       | <input type="checkbox" unchecked> | +`=floor((this.CHA - 10)/2)` |
| Muggle Studies (INT)  | <input type="checkbox" unchecked> | +`=floor((this.INT - 10)/2)` |
| Insight (WIS)         | <input type="checkbox" unchecked> | +`=floor((this.WIS - 10)/2)` |
| Intimidation (CHA)    | <input type="checkbox" unchecked> | +`=floor((this.CHA - 10)/2)` |
| Investigation (INT)   | <input type="checkbox" unchecked> | +`=floor((this.INT - 10)/2)` |
| Medicine (WIS)        | <input type="checkbox" unchecked> | +`=floor((this.WIS - 10)/2)` |
| Herbology (INT)       | <input type="checkbox" checked> | +`=floor((this.INT - 10)/2)` |
| Perception (WIS)      | <input type="checkbox" unchecked> | +`=floor((this.WIS - 10)/2)` |
| Performance (CHA)     | <input type="checkbox" unchecked> | +`=floor((this.CHA - 10)/2)` |
| Persuasion (CHA)      | <input type="checkbox" unchecked> | +`=floor((this.CHA - 10)/2)` |
| Sleight of Hand (DEX) | <input type="checkbox" unchecked> | +`=floor((this.DEX - 10)/2)` |
| Stealth (DEX)         | <input type="checkbox" unchecked> | +`=floor((this.DEX - 10)/2)` |
| Survival (WIS)        | <input type="checkbox" unchecked> | +`=floor((this.WIS - 10)/2)` |
| Potion Making (INT)   | <input type="checkbox" unchecked> | +`=floor((this.INT - 10)/2)` |

## Atributos

- **Aumento de Puntuaciones de Característica**. 
	Tu puntuación de Constitución, Sabiduría y una puntuación de característica adicional a tu elección aumentan en 1.

- **Lealtad Inquebrantable**. 
	Tienes ventaja en las tiradas de salvación contra cualquier efecto que te haga atacar o actuar en contra de una criatura que normalmente considerarías un aliado.

- **Viajes a la Cocina**. 
	Agregas la mitad de tu bonificación de competencia a tu Iniciativa y no puedes ser sorprendido mientras estés consciente.
  
  - **Cuidador.**  
    Tu estudio de las criaturas mágicas te ha enseñado sobre sus lesiones y fisiologías.  
    Puedes lanzar cualquier **hechizo de sanación** conocido sobre bestias.
  
  ## Magia Metamórfica

En muy contadas ocasiones, nace en una familia mágica un **metamorfomago**, con el talento particular de **cambiar cada aspecto de su apariencia humana**.

Antes de alcanzar la adultez, un metamorfomago no posee control total sobre esta habilidad, dejando que sus **emociones o el estrés** lo dominen y perdiendo el control.

- A **voluntad**, puedes transformar tu apariencia.
- Como **acción**, decides tu aspecto: altura, peso, facciones, voz, longitud y color del cabello, así como características distintivas.
- Ninguna de tus **estadísticas** cambia, no puedes parecer de un **tamaño distinto** al tuyo, y tu forma básica se mantiene (si eres bípedo, no puedes adoptar forma cuadrúpeda).
- En cualquier momento puedes usar una acción para **cambiar tu apariencia nuevamente**.
- Además, puedes adaptar tu cuerpo al **entorno acuático**, generando membranas entre tus dedos. Como acción, obtienes una **velocidad de nado igual a tu velocidad de movimiento**
- Si solo has visto a alaguien tienes un +3 sobre tu deceoption o persuation
- Si no has visto a alguien tienes un menos 1d6 menos 
- si lo conoces bien tienes un +5
- si has pasado mas de 48 hrs +10
- 

## Feats

### Lanzamiento Ritual

Tu capacidad para recordar información te permite lanzar conjuros libremente, siempre que tengas tiempo para concentrarte.

A nivel 1, puedes lanzar un conjuro como ritual si tiene la etiqueta de ritual y lo conoces. Una versión ritual de un conjuro solo tarda 1 minuto más en lanzarse que lo normal. Además, no gasta un espacio de conjuro, lo que significa que la versión ritual

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



Ideas: Naturaleza: El mundo natural es más precioso que toda la llamada civilización.

Vinculos:  	Es mi deber preservar y proteger las criaturas en peligro de extinción.

Defecto: 	Hablo sin pensar, con poca diplomacia


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
