# COLINA DEL RESENTIMIENTO

## Resumen De la Misión

> [!hint]
> La comadrona local, una acólita de Chauntea llamada Adabra Gwynn, vive sola en un molino de piedra en la ladera de una colina a varias millas al sur de [[Phandalin]]. Como cada vez hay más avistamientos del dragón, ya no es seguro para ella seguir ahí. Convence a Adabra de que regrese a [[Phandalin]]. En cuanto esté a salvo, visita al alcalde [[Harbin Wester]] para recibir una recompensa de 25 po.

**Nivel recomendado:** Esta misión está diseñada para personajes de nivel 3 o inferior. Los personajes de niveles superiores no enfrentarán grandes peligros aquí, pero pueden visitar este sitio para comprar pociones de curación a Adabra Gwynn.

## DESCRIPCIÓN DEL LUGAR

El nombre de Colina del Resentimiento proviene de la batalla campal que libraron dos clanes rivales enanos en su cima. No se conoce la causa de tal resentimiento y los únicos recuerdos de aquello son los túmulos de los caídos en combate. 

El molino de piedra de la colina se construyó después, pero a pesar de ello cuenta con más de cien años. Aquí vive Adabra Gwynn, una comadrona y boticaria devota de Chauntea (diosa de la agricultura). Una mantícora a la que Cryovain, el dragón blanco, expulsó de las montañas ataca el molino antes de que lleguen los aventureros. 

Lee el siguiente texto descriptivo para ambientar la escena:

> [!hint]
> En la ladera de la Colina del Resentimiento hay un viejo molino de piedra rodeado por una valla de hierro. Un gran monstruo alado con una cola llena de púas intenta derribar la puerta del molino. Una mujer aparece en la ventana del segundo piso, gesticula con los brazos y grita: "¡¿Me echan una manita?!".

Los personajes pueden luchar o negociar con la mantícora, que deja de atacar y se va volando si se le ofrecen al menos 25 po o unas cuantas libras de carne. Si no se le da muerte, la mantícora podría volver con su pareja para causar más problemas en el futuro.

## OBJETIVOS DE LA MISIÓN

Adabra no quiere volver a [[Phandalin]], pero los personajes pueden completar la misión si le piden una nota para [[Harbin Wester]] en la que diga que está bien. Adabra también les da a sus rescatadores una poción de curación por librarse de la mantícora.

## LUGARES DE LA COLINA DEL RESENTIMIENTO

Los siguientes lugares se destacan en el mapa de la Colina del Resentimiento:

### C1. TÚMULOS DE LOS ENANOS

Bajo estas pilas de piedra descansan los restos de los enanos, pero have mucho tiempo que sus huesos, armaduras y armas se desintegraron. 

### C2. CASA EN RUINAS

Lo único que queda de esta vieja casa de piedra es una chimenea. 

### C3 - C5. MOLINO DE PIEDRA

- **Planta baja (zona C3)**: En la rueda de molino, Adabra muele hierbas y otros ingredientes para elaborar emplastos y pociones.
- **Segunda planta (zona C4)**: Aquí se encuentra el cuarto de Adabra.
- **Tercera planta (zona C5)**: Hay una buhardilla para los huéspedes.

**Tesoro:** Adabra vende pociones de curación por 50 po. Dale una carta de poción de curación a cualquier jugador cuyo personaje adquiera uno de estos objetos. Si se agotan las cartas, Adabra se quedará sin pociones para vendor.

## HABITANTES DE LA COLINA

### Adabra Gwynn

Adabra es una mujer mayor de aspecto afable pero determinado. Lleva el cabello gris recogido en un moño y viste ropas sencillas pero prácticas con un delantal lleno de bolsillos para sus hierbas. Su molino es tanto su hogar como su lugar de trabajo, y lo ha defendido durante décadas contra todo tipo de peligros.

A pesar de su edad, Adabra es valiente y testaruda. No tiene intención de abandonar su hogar, argumentando que el dragón no ha mostrado interés en su pequeño molino y que la gente de la región depende de sus remedios. Sin embargo, está dispuesta a escribir una nota para Harbin asegurando que está bien, entendiendo que esto ayudará a los aventureros a cobrar su recompensa.

Adabra posee un vasto conocimiento de hierbas medicinales y pociones, y puede set una valiosa aliada para cualquier grupo de aventureros que gane su confianza.

```statblock
name: Adabra Gwynn
size: Medium
type: humanoid
subtype: (humana)
alignment: Neutral Bueno
ac: 10
hp: 8
hit_dice: 2d8
speed: 30 pies
stats: [10, 10, 10, 10, 14, 11]
fage_stats: [+0, +0, +0, +0, +2, +0]
saves:
  - WIS: +4
skillsaves:
  - Medicina: +4
  - Naturaleza: +2
  - Religión: +2
senses: Percepción pasiva 12
languages: Común
cr: 0
traits:
  - name: Conocimiento Herbario
    desc: "Adabra tiene ventaja en pruebas de habilidad relacionadas con hierbas medicionales, plantas y la elaboración de pociones."
  - name: Bendición de Chauntea
    desc: "Una vez al día, Adabra puede tocar a una criatura y curar 1d4 + 2 puntos de vida."
actions:
  - name: Bastón
    desc: "Ataque con arma cuerpo a cuerpo: +2 al impactar, alcance 5 pies, un objetivo. Impacto: 2 (1d6-1) de daño contundente."
equipment:
  - "Bastón de madera"
  - "Bolsa con hierbas medicinales"
  - "3 pociones de curación"
  - "1 amuleto sagrado de Chauntea"
  - "25 po en monedas"
```

### Mantícora

Las mantícoras son asesinas letales que recorren amplias distancias en busca de presas. No son especialmente inteligentes, pero pueden conversar con presas que sí lo sean. Si ven que dejar vivir a un oponente les reporta algún beneficio, le perdonarán la vida, pero exigirán a cambio un tributo o sacrificio que compense el alimento perdido.

Esta mantícora en particular está hambrienta y furiosa después de haber sido expulsada de su territorio por Cryovain. Sin embargo, es astuta y preferiría una comida fácil que una pelea difícil.

```statblock
name: Mantícora
size: Large
type: monstruosidad
alignment: legal malvada
ac: 14
hp: 68
hit_dice: 8d10+24
speed: 30 pies, volar 50 pies
stats: [17, 16, 17, 7, 12, 8]
fage_stats: [+3, +3, +3, -2, +1, -1]
senses: visión en la oscuridad 60 pies, Percepción pasiva 11
languages: Común
cr: 3
traits:
  - name: Regenerar Púas de Cola
    desc: "La mantícora tiene veinticuatro púas en la cola. Las púas utilizadas vuelven a crecer cuando finaliza un descanso largo."
actions:
  - name: Ataque múltiple
    desc: "La mantícora realiza tres ataques: uno de mordisco y dos con sus garras, o tres con sus púas de cola."
  - name: Garra
    desc: "Ataque con arma cuerpo a cuerpo: +5 al impactar, alcance 5 pies, un objetivo. Impacto: 6 (1d6+3) de daño cortante."
  - name: Mordisco
    desc: "Ataque con arma cuerpo a cuerpo: +5 al impactar, alcance 5 pies, un objetivo. Impacto: 7 (1d8+3) de daño perforante."
  - name: Púas de cola
    desc: "Ataque con arma a distancia: +5 al impactar, alcance 100/200 pies, un objetivo. Impacto: 7 (1d8+3) de daño perforante."
```

## POSIBLES DESARROLLOS

### Negociación

Si los jugadores deciden negociar con la mantícora, podrían:
- **Ofrecer comida**: La mantícora aceptará 5+ libras de carne fresca.
- **Ofrecer dinero**: La mantícora entiende el valor de las monedas y aceptará 25+ po.
- **Indicar otro territorio de caza**: Con una prueba exitosa de Persuasión CD 15, pueden dirigirla a una zona donde pueda cazar sin molestar a los habitantes.

### Completando la Misión

Dependiendo de cómo interactúen con Adabra, los jugadores tienen varias opciones:
- **Convencerla de ir a [[Phandalin]]**: Require una prueba de Persuasión CD 18 (casi impossible).
- **Obtener una nota para Harbin**: Prueba de Persuasión CD 12.
- **Ayudarla a fortificar el molino**: Ganará su gratitud y les ofrecerá descuentos en pociones futuras.

### Recompensas

1. **Por completar la misión**: 25 po del alcalde [[Harbin Wester]].
2. **Por derrotar a la mantícora**: 700 PX para dividir entre los miembros del grupo.
3. **Por ayudar a Adabra**: Una poción de curación gratuita.
4. **Por fortificar el molino**: 10% de descuento en todas las pociones futuras.

## CONSEJOS PARA EL DM

- Si el grupo parece tener problemas con la mantícora, Adabra podría ayudar arrojando una poción que temporalmente debilita al monstruo.
- Si los jugadores están teniendo un combate muy fácil, la pareja de la mantícora podría aparecer a la mitad de la pelea.
- Adabra puede convertirse en un contacto valioso para el grupo, proporcionándoles información sobre hierbas raras o efectos de pociones especiales en futuras aventuras.
  
  
  ![[Pasted image 20250618135835.png]]