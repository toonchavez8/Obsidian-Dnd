# Huerto Edermath

## Información Básica

| Campo | Descripción |
| --- | --- |
| Tipo | Huerto |
| Ubicación | Phandalin, Costa de la Espada Norte |
| Propietario | Daran Edermath |

## Descripción

El huerto se encuentra en el lado noroeste de Phandalin, y es un punto de referencia dominante para cualquiera que llegue a Phandalin por el camino principal. Los árboles rodean un pequeño grupo de edificios donde se llevan a cabo las actividades comerciales del huerto, y la casa del propietario, Daran Edermath, está cerca.

## Servicios

El huerto ofrece principalmente frutas frescas y algunos productos derivados como sidra y mermeladas artesanales. Daran Edermath es un ex-aventurero que ahora vive una vida tranquila como horticultor, pero aún tiene contactos con organizaciones como la Orden del Guantelete.

## Elementos de Interés (Para Comprar o Robar)

| Ítem | Descripción | Valor (PO) | Rareza | Notas |
| --- | --- | --- | --- | --- |
| Manzanas Edermath | Manzanas rojas y dulces, famosas en la región | 5 cp por unidad | Común | Ingrediente para pociones básicas de salud (+1d4 PV) |
| Sidra de Manzana Añeja | Botella de sidra fermentada especial | 2 po | Poco común | Al beber otorga +1 temporal a Carisma por 1 hora |
| Semillas de Manzano Mágico | Semillas que crecen rápidamente | 25 po | Rara | Si se planta, crece en 1d4 días hasta ser un árbol pequeño |
| Herramientas de Jardinería | Set de herramientas bien cuidadas | 5 po | Común | Ventaja en pruebas de Naturaleza relacionadas con plantas |
| Diario de Aventuras de Daran | Libro de cuero con historias del pasado de Daran | 50 po | Rara | Contiene pistas sobre tesoros olvidados y ruinas en la región |
| Escudo antiguo | Reliquia de los días de aventurero de Daran | 75 po | Poco común | Escudo +1, escondido detrás de un panel falso en su chimenea |
| Llave de plata | Pequeña llave ornamentada | 15 po | Poco común | Abre un cofre en la bodega (contiene 50 po y un pergamino de hechizo) |
| Mapa del huerto | Pergamino con marcas extrañas | 5 po | Común | Revela la ubicación de un pequeño tesoro enterrado (25 po) |
| Amuleto de la Orden del Guantelete | Insignia de membresía de Daran | 25 po | Rara | Permite identificarse como aliado de la Orden |
| Vino de frutas del bosque | Botella polvorienta en la bodega | 30 po | Poco común | Recupera 2d4 puntos de golpe al beber |

## Historia y Secretos

Daran Edermath fue un aventurero de renombre en su juventud y miembro de la Orden del Guantelete. Aunque ahora vive en retiro, sigue vigilante ante posibles amenazas en la región. El huerto tiene más de 50 años y algunas de las variedades de manzanos son únicas en toda la [[Costa de la Espada]].

Rumores sugieren que Daran escondió parte de su botín de aventurero en algún lugar del huerto, pero nadie ha podido encontrarlo aún. También se dice que a veces realiza reuniones secretas con otros ex-aventureros en su casa durante altas horas de la noche.

```statblock
name: Daran Edermath
size: Medium
type: humanoid
subtype: (humano)
alignment: Legal Bueno
ac: 15
hp: 27
hit_dice: 3d8+9
speed: 30ft
stats: [14, 12, 16, 11, 14, 15]
fage_stats: [+2, +1, +3, +0, +2, +2]
saves:
  - WIS: +4
  - CHA: +4
skillsaves:
  - Athletics: +4
  - History: +2
  - Insight: +4
  - Persuasion: +4
  - Religion: +2
  - Nature: +2
senses: Passive Perception 14
languages: Común, Enano
cr: 1
traits:
  - name: Miembro Retirado de la Orden del Guantelete
    desc: "Daran puede identificarse ante otros miembros de la Orden y solicitar ayuda de ellos."
  - name: Conocimiento del Horticultor
    desc: "Daran tiene ventaja en pruebas de Naturaleza relacionadas con plantas y cultivos."
  - name: Defensor Experimentado
    desc: "Cuando un enemigo visible ataca a un aliado a 5 pies de Daran, puede usar su reacción para imponer desventaja al tirada de ataque."
actions:
  - name: Martillo de Guerra
    desc: "Ataque con arma cuerpo a cuerpo: +4 al golpear, alcance 5 pies, un objetivo. Impacto: 6 (1d8+2) de daño contundente."
  - name: Ballesta Ligera
    desc: "Ataque con arma a distancia: +3 al golpear, alcance 80/320 pies, un objetivo. Impacto: 5 (1d8+1) de daño perforante."
  - name: Luz Sagrada (2/día)
    desc: "Daran canaliza energía divina para crear un resplandor sagrado. Como acción, una criatura que él pueda ver a 60 pies debe superar una tirada de salvación de Destreza CD 12 o sufre 2d6 de daño radiante. Si el objetivo es un muerto viviente o un infernal, también queda deslumbrado hasta el final de su próximo turno."
  - name: Toque Restaurador (1/día)
    desc: "Daran toca a una criatura y neutraliza una enfermedad o un veneno que le afecta, o elimina una condición entre aturdido, cegado, ensordecido o paralizado."
reactions:
  - name: Proteger Aliado
    desc: "Cuando un enemigo visible ataca a un aliado a 5 pies de Daran, puede usar su reacción para conceder +2 a la CA del objetivo contra ese ataque."
equipment:
  - "Martillo de guerra, ballesta ligera y 20 virotes"
  - "Armadura de cuero tachonado"
  - "Escudo (+1, oculto en su casa)"
  - "Amuleto de la Orden del Guantelete"
  - "Bolsa con 35 po y 12 pp"
```
