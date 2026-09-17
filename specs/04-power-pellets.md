# SPEC 04 — Power pellets y modo asustado

> **Estado:** aprobado
> **Depends on:** SPEC 03
> **Date:** 2026-09-17
> **Objetivo:** Añadir 4 power pellets (uno por cuadrante) que al ser comidos vuelven azules a los fantasmas activos durante ~6 s, permitiendo comérselos por puntos escalados.

---

## Scope

**In:**

- 4 power pellets en las celdas (1,1), (26,1), (1,29), (26,29), representados como nuevo valor de celda `4` en el grid
- Comer el pellet suma 50 puntos y cuenta para la condición de victoria (`dotsRemaining`)
- Modo asustado: los fantasmas liberados se vuelven azules 360 frames, revierten la dirección y se mueven al azar
- Durante el modo asustado, chocar con un fantasma lo come: 200/400/800/1600 puntos según cadena del mismo pellet
- Un fantasma comido vuelve a su spawn en la pen con `exitingPen: true` y re-sale normal
- Perder una vida cancela el modo asustado
- Render de power pellets (más grandes que un dot) y de fantasmas azules

**Fuera de alcance (specs futuros):**

- Parpadeo final del modo asustado (blinking blanco/azul)
- Ojos dibujados (`<eyes>`) para el fantasma comido; se simula con respawn en la pen
- Más de 4 pellets, niveles con distinta duración
- Velocidad de fantasmas asustados distinta
- Refactorizar el código existente al estilo de legibilidad para juniors (hoy `game.js`/`render.js` usan abreviaturas como `g`, `p`). Se decide como spec propia si este estilo nos convence al implementar este spec

---

## Data model

```js
// maze.js — parseTile agrega:
if ( ch === 'o' ) return 4;

// game.js — createGame agrega al estado return:
fright: {
  active: false,
  timer: 0,     // frames restantes; FRIGHT_DURATION = 360
  chain: 0,     // fantasmas comidos en el pellet actual
},

// game.js — cada fantasma agrega:
scared: false,
```

Celdas cambiadas en `MAZE_STR` de `maze.js` (de `'.'` a `'o'`):

- (1,1) y (26,1) → fila 1 `#o...........##...........o#`
- (1,29) y (26,29) → fila 29 `#o..........................o#`

Las 4 celdas son dots hoy, así que cada power pellet reemplaza un dot de 10 puntos.

---

## Convenciones de código (legibilidad para juniors)

Todo el código nuevo de este spec debe ser legible para un dev junior:

- **Nombres completos y descriptivos.** Nada de abreviaturas de una letra ni renombrados crípticos como `ghost → g`, `pacman → p` o expresiones tipo `g < r`. Si necesitas el fantasma, llámalo `ghost`; si es la puntuación, `score`; el temporizador, `frightenedTimer`.
- **Funciones pequeñas con nombre intencional** (`applyFrightenedMode`, `respawnEatenGhost`, `startFrightenedMode`) y sin parámetros de una letra: `function eatGhost( game, ghost, ghostIndex )`.
- **Respetar el estilo del repo**: espacios dentro de paréntesis `func( arg )`, mayúsculas solo donde marca la convención existente.
- **Comentarios breves en español** solo cuando añaden contexto que no se deduce del nombre (p.ej. la celda donde vive cada power pellet).
- No se refactorizará el código existente (usa `g`, `p`); la regla aplica al código nuevo de este spec.

---

## Implementation plan

1. **`maze.js`:** en `parseTile`, añadir `if ( ch === 'o' ) return 4;`. Reemplazar los 4 dots en `MAZE_STR` por `'o'`. Las celdas son transables (`isWall` no las bloquea).

2. **`game.js` — conteo y comer:** en `createGame`, contar `v === 2 || v === 4` en `dotsRemaining`. En `movePacman`, añadir rama `else if ( grid[ p.y ][ p.x ] === 4 )`: limpiar celda a 0, `score += 50`, `dotsRemaining--`, y llamar a `triggerFrightened( game )`.

3. **`game.js` — estado asustado:** definir `const FRIGHT_DURATION = 360;`. Crear `triggerFrightened( game )`: poner `active = true`, `timer = FRIGHT_DURATION`, `chain = 0`; para cada fantasma con `i < ghostsReleased` y `!g.exitingPen`: `scared = true` y `g.dir = OPPOSITE[ g.dir ]` (reversión). En `update()`, si `fright.active`: decrementar `timer` y al llegar a 0 poner `active = false` y `scared = false` en todos los fantasmas.

4. **`game.js` — movimiento y colisión:** en `decideGhost()`, si `ghost.scared` elegir dirección aleatoria entre las válidas (sin giro 180°) en cada alineamiento. En `update()`, cambiar el bucle de colisión a `forEach( ( ghost, ghostIndex ) => ... )`: si `ghost.scared` → comer: `score += 200 * 2^chain`, `chain++`, respawn del fantasma en `GHOST_STARTS[ ghostIndex ]` con `exitingPen = true`, `scared = false`, `dir = 'up'`; si no → perder vida como hoy. En `resetPositions()`, además de lo actual, poner `fright.active = false`, `timer = 0`, `chain = 0` y `scared = false` en todos los fantasmas.

5. **`render.js`:** crear `drawPowerPellets( ctx, grid )` con radio ~6 en `DOT_COLOR`, solo celdas `=== 4`, y llamarla tras `drawDots`. En `drawGhost()`, si `ghost.scared` pintar cuerpo en azul `#0000ff` (las paredes usan `#2121ff`, evitamos confusión) manteniendo los ojos.

6. **`index.html`:** actualizar el texto del overlay de inicio a: "Flechas para moverte. Come todos los puntos. Los power pellets vuelven azules a los fantasmas."

7. **Probar manualmente:** verificar todos los puntos de los criterios de aceptación.

---

## Acceptance criteria

- [ ] El juego carga sin errores en consola
- [ ] Se ven 4 power pellets grandes en (1,1), (26,1), (1,29), (26,29) y ya no hay dots en esas celdas
- [ ] Comer un power pellet suma 50 puntos (sin sumar también los 10 del dot)
- [ ] Al comerlo, los fantasmas ya liberados y fuera de la pen se vuelven azules y revierten su dirección
- [ ] Los fantasmas aún en la pen o sin liberar no se vuelven azules
- [ ] El modo asustado dura ~360 frames; se puede volver azul de nuevo comiendo otro pellet (reinicia el temporizador)
- [ ] Durante el modo asustado, chocar con un fantasma azul lo come: no se pierde vida
- [ ] Los puntos por comer fantasmas asustados escalan 200/400/800/1600 y se reinician con cada pellet
- [ ] Un fantasma comido reaparece en la pen y re-sale normal (con la lógica de `moveGhostToDoor` de SPEC 03)
- [ ] Al terminar el temporizador los fantasmas vuelven a la normalidad
- [ ] Perder una vida cancela el modo asustado y reinicia posiciones
- [ ] Los 4 power pellets cuentan en `dotsRemaining`: hay que comérselos para ganar
- [ ] Nombres de variables y funciones son descriptivos (sin abreviaturas de una letra)

---

## Decisions

- **Sí:** celda `4` en el laberinto. Consistente con dot `2` y lectura directa del grid; `isWall` ya la trata como transitable.
- **No:** lista constante `POWER_PELLETS` con estado aparte. Doble fuente de verdad y más código de sincronía.
- **Sí:** 50 puntos y conteo para victoria. Fiel al original.
- **Sí:** cadena de puntos 200/400/800/1600. Coste bajo (contador `chain`) y sabor clásico.
- **No:** punto fijo por fantasma asustado. Pierde la tensión del original.
- **Sí:** 360 frames (~6 s). Valor fiel al juego original.
- **No:** parpadeo final. Out of scope; se decide si llega en otra spec.
- **Sí:** reversión de dirección al activarse. Comportamiento clásico.
- **No:** ojos con sprite dedicado. Se simula con respawn en la pen; más barato y ya hay infraestructura de pen exit.
- **Sí:** perder vida cancela el modo asustado. Fiel al clásico y evita estados raros en el reset.
- **Sí:** azul `#0000ff` para asustados. Distintivo de las paredes (`#2121ff`).
- **Sí:** código nuevo con nombres legibles y descriptivos (sin abreviaturas tipo `g`, `p`).
- **Sí:** esquinas de pantalla (1,1)(26,1)(1,29)(26,29). El usuario cambió la posición tras aprobar: se priorizan las esquinas físicas sobre la disposición clásica del juego.

---

## ¿Qué NO está en este spec?

- Parpadeo final del modo asustado.
- Sprite de ojos para el fantasma comido.
- Más de 4 power pellets o niveles con duraciones distintas.
- Fantasmas asustados con velocidad distinta.
- Refactorizar el código existente. Si el estilo de código de SPEC 04 gusta, se hará una refactorización global en una spec futura.