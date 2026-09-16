# SPEC 03 — Corrección del pathfinding de salida de pen

> **Estado:** Borrador
> **Depends on:** SPEC 02
> **Date:** 2026-09-16
> **Objetivo:** Corregir la lógica de salida de pen para que los fantasmas usen pathfinding real hacia la puerta (row 12, cols 13-14) en lugar de depender únicamente de la distancia Manhattan a Pac-Man, permitiendo movimiento lateral cuando es necesario.

---

## Scope

**In:**

- Reimplementar la función de pathfinding para fantasmas dentro de la pen
- Usar BFS precalculado desde la puerta para calcular distancias reales a cada celda de la pen
- Los fantasmas se mueven celda por celda hacia la dirección con menor distancia BFS
- Cambiar `exitingPen` a `false` cuando el fantasma cruza la puerta (y < 12)

**Fuera de alcance (specs futuros):**

- Cambiar posiciones iniciales de los fantasmas
- Ajustar tiempos de salida
- Power pellet / modo asustado
- Velocidades diferentes por fantasma

---

## Data model

No se introducen nuevas estructuras de datos. Se reutiliza el modelo de SPEC 02:

```js
// game.js — se mantiene la propiedad exitingPen en cada fantasma
ghosts: GHOST_STARTS.map( ( g ) => ( {
  x: g.x,
  y: g.y,
  dir: 'up',
  speed: GHOST_SPEED,
  kind: g.kind,
  exitingPen: true,  // de SPEC 02
} ) )
```

---

## Implementation plan

1. **Crear función `penDistanceBFS( grid )`** en `game.js`:
   - Ejecuta BFS desde la puerta (cells [13,12] y [14,12])
   - Retorna un Map con clave `"x,y"` y valor distancia (número)
   - Solo calcula distancias para celdas transitables por fantasmas (ignora paredes)
   - La puerta (value 3) es transitable para fantasmas

2. **Crear función `moveGhostToDoor( game, g )`** en `game.js`:
   - Usa el mapa de distancias BFS precalculado
   - Cuando `g` está alineado (aligned en ambas coordenadas):
     - Filtra direcciones opuestas a la actual (no giros de 180)
     - Filtra direcciones que no son transitables (`canMove` con actor `'ghost'`)
     - De las opciones válidas, elige la dirección con menor distancia BFS
     - Si no hay opciones válidas, permite giro de 180
   - Aplica la dirección elegida a `g.dir`

3. **Modificar `moveGhost()`** para que cuando `g.exitingPen` sea `true`, llame a `moveGhostToDoor()` en lugar de `decideGhost()`.

4. **Agregar condición de salida**: cuando `g.y < 12` (cruzó la puerta), setear `g.exitingPen = false`.

5. **Ajustar `resetPositions()`** para reiniciar `exitingPen` a `true` en todos los fantasmas (verificar que SPEC 02 ya lo hizo).

6. **Probar manualmente**: verificar que los 4 fantasmas salen de la pen sin quedar atrapados y que el movimiento lateral funciona cuando la dirección directa está bloqueada.

---

## Acceptance criteria

- [ ] Al ser liberado, cada fantasma navega celda por celda hacia la puerta usando pathfinding BFS
- [ ] Cuando la dirección hacia la puerta está bloqueada por una pared, el fantasma se mueve lateralmente para绕过 el obstáculo
- [ ] Ningún fantasma queda atrapado en la pen
- [ ] Una vez fuera de la puerta (y < 12), el fantasma usa `decideGhost()` normalmente
- [ ] Al perder una vida, los fantasmas regresan a la pen con `exitingPen: true`
- [ ] El BFS cubre todas las celdas transitables de la pen (rows 13-15, cols 12-15)
- [ ] El juego no tiene errores en consola

---

## Decisions

- **Sí:** BFS precalculado desde la puerta. Calcula distancias reales considerando paredes, no solo Manhattan distance.
- **No:** Manhattan distance simple. No account for walls inside the pen, causes ghosts to move in blocked directions.
- **Sí:** Precalcular BFS una vez por sesión (no cada frame). El laberinto no cambia, así que las distancias son constantes.
- **No:** pathfinding dinámico con A*. BFS es suficiente para la pen (área pequeña y estática).
- **Sí:** Mantener `exitingPen` como en SPEC 02. El mecanismo de estado es correcto; solo la lógica de pathfinding necesita corrección.

---

## ¿Qué NO está en este spec?

- Cambiar posiciones iniciales de los fantasmas.
- Ajustar tiempos de salida.
- Power pellet / modo asustado.
- Velocidades diferentes por fantasma.
