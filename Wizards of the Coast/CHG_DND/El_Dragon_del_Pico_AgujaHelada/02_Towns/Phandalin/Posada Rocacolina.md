
# Posada Rocacolina

## Información Básica

| Campo | Descripción |
| --- | --- |
| Tipo | Posada |
| Pisos | 2 |
| Ubicación | Phandalin, Costa de la Espada Norte |
| Propietario | Toblen Stonehill |
| Personal | Camarera (Elsa) |
| Servicios | Alojamiento, comida y bebida |


## Ubicación

La modesta posada está ubicada cerca del centro del pueblo. Es uno de los edificios más grandes de la localidad.

Construida recientemente en el siglo XV [DR](https://forgottenrealms.fandom.com//wiki/DR "DR"), la posada es un gran edificio de dos pisos hecho de rocas sin refinar y madera cortada rudimentariamente.

## Ambiente

Los habitantes de Phandalin, desde mineros hasta granjeros, disfrutan relajándose en la sala común, que se ha convertido en un buen lugar para escuchar chismes y historias locales. La posada se mantiene limpia y ordenada, y generalmente es un lugar agradable para visitar.

La Posada Rocacolina es propiedad de [Toblen Stonehill](https://forgottenrealms.fandom.com//wiki/Toblen_Stonehill "Toblen Stonehill"), quien la gestiona junto con su esposa [Trilena Stonehill](https://forgottenrealms.fandom.com//wiki/Trilena_Stonehill "Trilena Stonehill") y con la ayuda de su hijo [Pip Stonehill](https://forgottenrealms.fandom.com//wiki/Pip_Stonehill "Pip Stonehill").

## Servicios

La posada ofrece seis habitaciones para alquilar en el piso superior, además de bebidas alcohólicas y comida. Una cama cuesta cinco [piezas de plata](https://forgottenrealms.fandom.com//wiki/Currency "Moneda") por noche, y los precios son bastante estándar para comida, [cerveza](https://forgottenrealms.fandom.com//wiki/Ale "Cerveza") y [sidra](https://forgottenrealms.fandom.com//wiki/Cider "Sidra"), con comidas que vienen a costar aproximadamente 1 [pieza de plata](https://forgottenrealms.fandom.com//wiki/Currency "Moneda"). Una camarera llamada [Elsa](https://forgottenrealms.fandom.com//wiki/Elsa "Elsa") sirve bebidas en la barra.

## Elementos de Interés

| Ítem                           | Descripción                              | Valor (PO)     | Rareza     | Notas                                                                  |
| ------------------------------ | ---------------------------------------- | -------------- | ---------- | ---------------------------------------------------------------------- |
| Cerveza especial de Rocacolina | Cerveza oscura con sabor intenso         | 5 pp por jarra | Común      | Ventaja en tiradas de Carisma durante 1 hora                           |
| Diario de huéspedes            | Libro donde se registran visitantes      | 25 po          | Poco común | Contiene información sobre viajeros recientes                          |
| Llave maestra                  | Abre todas las habitaciones de la posada | 50 po          | Raro       | Guardada en una caja fuerte tras la barra                              |
| Pastel de frutas de Trilena    | Postre casero famoso en la región        | 3 pp           | Común      | Recupera 1d4 puntos de vida                                            |
| Colección de chismes           | Información recopilada por Toblen        | 10 po          | Común      | Ventaja en pruebas de Investigación locales                            |
| Licor de bayas del bosque      | Bebida fuerte destilada en secreto       | 1 po           | Poco común | Con sabor intenso, +1 a tiradas de salvación de CON por 1 hora         |
| Talismán de viajero            | Amuleto de madera tallado                | 15 po          | Poco común | +1 a las tiradas de salvación contra el frío, oculto en una habitación |
| Mapa de la región              | Detalla caminos y puntos de interés      | 5 po           | Común      | Ventaja en pruebas de Supervivencia en la zona                         |
| Juego de dados encantados      | Dados ligeramente trucados               | 20 po          | Raro       | Ventaja en juegos de azar 1 vez al día                                 |
| Nota cifrada                   | Mensaje misterioso dejado por un viajero | 5 po           | Común      | Podría contener información sobre un tesoro                            |

```statblock
name: Toblen Stonehill
size: Medium
type: humanoid
subtype: (humano)
alignment: Neutral Bueno
ac: 12
hp: 15
hit_dice: 2d8+6
speed: 30ft
stats: [12, 14, 16, 13, 12, 16]
fage_stats: [+1, +2, +3, +1, +1, +3]
saves:
  - CON: +5
  - CHA: +5
skillsaves:
  - Percepción: +3
  - Perspicacia: +3
  - Persuasión: +5
  - Historia: +3
senses: Passive Perception 13
languages: Común
cr: 1/2
traits:
  - name: Hospitalidad del Posadero
    desc: "Toblen tiene ventaja en tiradas de Carisma relacionadas con el servicio y la hospitalidad."
  - name: Conocedor Local
    desc: "Toblen conoce todos los rumores importantes y personajes de Phandalin, lo que le otorga ventaja en pruebas de Investigación relacionadas con la zona."
actions:
  - name: Garrote
    desc: "Ataque con arma cuerpo a cuerpo: +3 al golpear, alcance 5 pies, un objetivo. Impacto: 4 (1d6+1) de daño contundente."
  - name: Daga
    desc: "Ataque con arma cuerpo a cuerpo o a distancia: +4 al golpear, alcance 5 pies o alcance 20/60 pies, un objetivo. Impacto: 4 (1d4+2) de daño perforante."
  - name: Palabra Tranquilizadora (1/día)
    desc: "Toblen utiliza su carisma natural para apaciguar una situación tensa. Todas las criaturas hostiles en un radio de 10 pies deben realizar una tirada de salvación de Sabiduría CD 13 o desisten de atacar durante 1 minuto a menos que sean atacadas."
equipment:
  - "Delantal de posadero (funciona como armadura ligera)"
  - "Daga, garrote y una bolsa con 15 po y 35 pp"
  - "Llave maestra de la posada"
  - "Libro de cuentas y registro de huéspedes"
```

```statblock
name: Elsa
size: Medium
type: humanoid
subtype: (humana)
alignment: Neutral
ac: 11
hp: 9
hit_dice: 1d8+1
speed: 30ft
stats: [10, 12, 12, 11, 10, 14]
fage_stats: [+0, +1, +1, +0, +0, +2]
saves:
  - DEX: +3
  - CHA: +4
skillsaves:
  - Percepción: +2
  - Perspicacia: +2
  - Persuasión: +4
  - Entretenimiento: +4
senses: Passive Perception 12
languages: Común
cr: 1/8
traits:
  - name: Oído Atento
    desc: "Elsa tiene ventaja en pruebas de Percepción que impliquen oír conversaciones en la posada."
  - name: Sonrisa Encantadora
    desc: "Elsa tiene ventaja en pruebas de Persuasión cuando intenta obtener información de los clientes."
actions:
  - name: Jarra
    desc: "Ataque con arma cuerpo a cuerpo: +2 al golpear, alcance 5 pies, un objetivo. Impacto: 2 (1d4) de daño contundente."
  - name: Daga
    desc: "Ataque con arma cuerpo a cuerpo: +3 al golpear, alcance 5 pies, un objetivo. Impacto: 3 (1d4+1) de daño perforante."
  - name: Distracción (1/día)
    desc: "Elsa utiliza sus encantos o una distracción repentina para confundir a un objetivo. Una criatura debe realizar una tirada de salvación de Sabiduría CD 12 o queda distraída durante 1 ronda, teniendo desventaja en su siguiente tirada de ataque o prueba de habilidad."
equipment:
  - "Ropas de trabajo"
  - "Delantal con varios bolsillos (contiene 5 pp y 12 pc)"
  - "Daga oculta"
  - "Colección de chismes y rumores"
```