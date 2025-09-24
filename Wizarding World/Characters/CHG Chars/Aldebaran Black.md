---
Type: Player
Art:
Level: 1
AC: 13
Prof: "2"
HP: 12
HitDice: d10
Speed: 15
STR: 12
DEX: 15
CON: 14
INT: 9
WIS: 15
CHA: 16
House: Slytherin
SchoolOfMagic: Jinxes, Hexes and Curses
Gender: Male
Age: "13"
Location:
  - - Hogwarts
CastingStyle: Willpower
Background: Socialite
Heritage: PureBlood
Wand: Manzano,30cm ,Fibra de corazon de dragon, Muy flexible
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
| Deception (CHA)       | <input type="checkbox" unchecked> | +`=floor((this.CHA - 10)/2+1)` |
| Muggle Studies (INT)  | <input type="checkbox" unchecked> | +`=floor((this.INT - 10)/2)` |
| Insight (WIS)         | <input type="checkbox" unchecked> | +`=floor((this.WIS - 10)/2)` |
| Intimidation (CHA)    | <input type="checkbox" unchecked> | +`=floor((this.CHA - 10)/2)` |
| Investigation (INT)   | <input type="checkbox" unchecked> | +`=floor((this.INT - 10)/2)` |
| Medicine (WIS)        | <input type="checkbox" unchecked> | +`=floor((this.WIS - 10)/2)` |
| Herbology (INT)       | <input type="checkbox" unchecked> | +`=floor((this.INT - 10)/2)` |
| Perception (WIS)      | <input type="checkbox" unchecked> | +`=floor((this.WIS - 10)/2)` |
| Performance (CHA)     | <input type="checkbox" unchecked> | +`=floor((this.CHA - 10)/2)` |
| Persuasion (CHA)      | <input type="checkbox" unchecked> | +`=floor((this.CHA - 10)/2+2)` |
| Sleight of Hand (DEX) | <input type="checkbox" unchecked> | +`=floor((this.DEX - 10)/2)` |
| Stealth (DEX)         | <input type="checkbox" unchecked> | +`=floor((this.DEX - 10)/2)` |
| Survival (WIS)        | <input type="checkbox" unchecked> | +`=floor((this.WIS - 10)/2)` |
| Potion Making (INT)   | <input type="checkbox" unchecked> | +`=floor((this.INT - 10)/2 + 1)` |

## Atributos

- **Aumento de Puntuación de Habilidad:** Tu puntuación de Destreza, Carisma y una habilidad de tu elección aumentan en 1.
- **Información Comprometedora:** Cuando usas secretos de alguien en una prueba de Carisma, obtienes competencia doble.
- **Calidad Noble:** Puedes adoptar una personalidad de importancia para mezclarte entre oficiales y magos prestigiosos. 
- Obtienes un conjunto de Herramientas de Rompemaldiciones y competencia con ellas.

## Feats


## Occlumency Training

Has desarrollado la **presencia mental** para resistir invasiones psíquicas.

**Beneficios:**

- Cuando eres objetivo de _Legilimens_, haces una **salvación de Sabiduría** para resistir sus efectos iniciales.
- Tienes **ventaja en tiradas enfrentadas de Inteligencia** contra legilimens.
- Si fallas, eres consciente inmediatamente de que están intentando leer tu mente.
- Si superas una salvación o tirada enfrentada, puedes **dejar que el hechizo continúe** revelando **información, emociones o recuerdos falsos**.
- El _Veritaserum_ no funciona en ti, a menos que lo permitas.
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

La Noble y Antiquísima Casa de Black fue una de las familias de magos de sangre pura más grandes, antiguas y ricas de Gran Bretaña, y una de las Sagradas Veintiocho. Muchas familias de magos en Gran Bretaña estaban emparentadas de forma lejana con la Casa de Black. Al igual que las familias Malfoy y Lestrange (ambas relacionadas con la familia Black), la Casa de Black era sinónimo de estatus elevado y riqueza. Tradicionalmente, los Black eran seleccionados para Slytherin en el Colegio Hogwarts de Magia y Hechicería.

> El lema de la familia, que podía encontrarse en el escudo familiar, era _Toujours Pur_, que en francés significa "Siempre Puro".

Debido a sus creencias, un gran número de miembros de la familia Black practicaban las Artes Oscuras y apoyaron a Lord Voldemort durante la Primera y la Segunda Guerra Mágica. Algunos Black incluso se convirtieron en Mortífagos, mientras que muchos otros nunca tomaron la Marca Tenebrosa, pero aun así creían en la “purificación” del mundo mágico. Walburga y Orion Black, dos miembros de la familia que vivieron durante el auge de los Mortífagos, eran conocidos por ser primos y Padres de Sirius Black y Regulus Arcturus Black.

Sin embargo, algunos miembros de la familia no estaban de acuerdo con las creencias tradicionales; Sirius Black, por ejemplo, se unió a la Orden del Fénix, al igual que Andrómeda Tonks, quien se casó con un hijo de muggles, y Nymphadora Tonks, miembro lejano de la familia Black. Ambas fueron asesinadas en batalla por Bellatrix Lestrange (nacida Black), prima de Sirius y tía de Tonks.

Regulus Arcturus Black (1961 – 1979) fue un mago inglés de sangre pura y miembro de la noble Casa de Black. Nació de Orion y Walburga Black, y fue el hermano menor de Sirius Black.

Asistió al Colegio Hogwarts de Magia y Hechicería en 1972 y fue seleccionado en la casa Slytherin. Desde joven, Regulus admiraba a Lord Voldemort y tenía la ambición de convertirse en Mortífago. Guardaba fotografías y artículos del _Profeta Diario_ sobre el Señor Tenebroso y sus seguidores, los cuales colgaban en su habitación cerca de un cuadro con el escudo familiar. Regulus recibió la Marca Tenebrosa alrededor de los dieciséis años, lo que fue aprobado por su familia, ya que veían a Voldemort como alguien que buscaba la supremacía de la sangre pura sobre los demás magos y los muggles. Aunque sus padres nunca fueron Mortífagos, compartían muchas de las creencias de Voldemort.

Tras unirse a los Mortífagos, Regulus comenzó a replantearse su lealtad, en parte porque su amo maltrató y pretendía matar al elfo doméstico de la familia Black, Kreacher, mientras preparaba las defensas para uno de sus Horrocruxes.

> Aqui podemos agregar que fue cuando concio a tu mama y tuvieron unas buenas semanas como pareja. Tu mama se podria llamar algo como Wismera Abbot. Al inicio ella tambien Slythirin tenia prejuicios pesados de ser Sangre Pura pero el como lo estaba logrando Voldemort no le gusto y tambien compartia las dudas de Regulas

Para 1979, Regulus ya tenía serias dudas sobre servir como Mortífago, pero aún era reacio a actuar directamente contra Voldemort. Un día, Voldemort pidió usar a Kreacher, y Regulus aceptó de buen grado, deseoso de complacerlo. Voldemort utilizó a Kreacher para probar las defensas que rodeaban el guardapelo Horrocrux, dejándolo morir allí. Sin embargo, Kreacher logró escapar usando la magia de los elfos y le contó a Regulus lo que había ocurrido. Regulus dedujo que el guardapelo era un Horrocrux y la clave de la inmortalidad de Voldemort.

> En este punto tu mama esta enbarazada contigo, no se han caszado y le avisa Regulgues que si no regresa que ella corre y que nunca regrese que lo que el va ser podria ser su perdicion o la liberacion de su futuro de ambos de el camino turbio que tomaron 

Ese fue el factor decisivo para su traición. Regulus creó una copia del guardapelo y colocó dentro una nota destinada a Voldemort, esperando que este revisara su Horrocrux en el futuro. Luego ordenó a Kreacher llevarlo al lugar donde estaba escondido el guardapelo real. Kreacher lo guió a través de las defensas de la cueva. En la isla con la vasija que contenía la poción esmeralda, Regulus ordenó a Kreacher tomar el guardapelo una vez que la poción estuviera consumida, reemplazarlo con la copia y luego escapar sin él, encontrando después la forma de destruirlo.

Regulus bebió la poción él mismo, y cuando intentó saciar su sed con el agua del lago, fue arrastrado a la muerte por los Inferi. Kreacher cumplió la última orden de su amo y cambió los relicarios antes de escapar. Sin embargo, a pesar de sus esfuerzos, nunca pudo destruir el Horrocrux

**Herencia Olvidada de los Black**

Tras la desaparición de **Regulus Black**, tu madre huyó de Inglaterra en un estado de desesperación y paranoia. Temiendo por tu seguridad, viajó fuera del continente, perseguida en más de una ocasión por Mortífagos. Finalmente, llegó a una aldea escondida de **duendes similares a los de Gringotts**. Desconfiados al principio, estos seres la aceptaron solo a cambio de algo invaluable: **su varita mágica**. Con miedo y resignación, ella accedió.

Con el tiempo, adoptó una nueva identidad: **Rebecca Tombs**, ocultando su linaje y viviendo bajo otra piel.

Pero los duendes, fascinados con la varita, intentaron replicar su magia. Un accidente en sus experimentos desató una **explosión mágica** que abrió un portal hacia otro mundo. Ella lo cruzó contigo, y ambos vivieron durante años en paz, lejos de las guerras de los magos, aprendiendo a sobrevivir de la tierra.

Sin embargo, recientemente otro portal se abrió… y decidiste cruzarlo, con tu madre siguiéndote, temerosa por tu vida. Ese portal se abrió en las afueras de **Londres**.

Los **Aurores** fueron los primeros en encontrarlos. Alarmados al detectar criaturas “fuera de este mundo”, los detuvieron, los interrogaron y terminaron comprendiendo la increíble historia de portales y huida. Decidieron mantenerlos bajo custodia, pero pronto comprendieron quién eras en realidad: un **descendiente de la Casa Black**.

Con esa revelación, fuiste llevado a hablar con **Harry Potter**, actual líder del mundo mágico tras la caída de Voldemort.

Harry te recibió con amabilidad y te explicó la importancia de tu apellido. Te habló de tu tío **Sirius Black**, de cómo fue injustamente perseguido y de su sacrificio. También te reveló la verdad sobre tu padre, **Regulus**, quien murió intentando derrotar a Voldemort, dando a su elfo doméstico, **Kreacher**, la orden de proteger el relicario de Slytherin.

Finalmente, Harry te entregó lo que te corresponde por herencia: **el Número 12 de Grimmauld Place**, la ancestral casa de los Black, custodiada todavía por Kreacher, el elfo que sirvió a tu padre y a tu tío.

Ideal - Poder. Los fuertes deben liderar y los débiles seguir.
Vinculo- Quiero ser famoso, cueste lo que cueste
Defecto-Soy un romántico (equivocado) y me dejo llevar por un rostro bonito.
## Magia



### Encantamientos



### Hechizos
