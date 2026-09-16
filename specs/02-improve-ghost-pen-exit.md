# SPEC 02 — Mejora de salida de fantasmas desde la pen

> **Estado:** Aprobado
> **Depends on:** SPEC 01
> **Date:** 2026-09-16
> **Objetivo:** Mejorar la mecánica de salida de los fantasmas desde la pen para que al ser liberados se muevan directamente hacia la puerta en lugar de quedar atrapados.

---

## Scope

**In:**

- Agregar propiedad `exitingPen` a cada fantasma en `game.js`
- Implementar lógica de movimiento directo hacia la puerta (fila 12, cols 13-14) cuando `exitingPen` es `true`
- Cambiar de `exitingPen` a `false` una vez que el fantasma cruza la puerta (y < 12)
- Modificar `moveGhost()` para usar la lógica de pen exit cuando `exitingPen` es `true`

**Fuera de alcance (specs futuros):**

- Cambiar posiciones iniciales de los fantasmas
- Ajustar tiempos de salida
- Power pellet / modo asustado

---

## Data model

```js
// game.js — se agrega a cada fantasma:
ghosts: GHOST_STARTS.map( ( g ) => ( {
  x: g.x,
  y: g.y,
  dir: 'up',
  speed: GHOST_SPEED,
  kind: g.kind,
  exitingPen: true,  // nuevo: activo hasta cruzar la puerta
} ) )
```

---

## Implementation plan

1. **Agregar `exitingPen: true`** a cada fantasma en `createGame()`.

2. **Crear función `moveGhostToDoor( game, g )`** en `game.js`:
   - Si `g.y > 12`: mover hacia arriba (dir = 'up')
   - Si `g.y <= 12` y `g.x < 13`: mover a la derecha (dir = 'right')
   - Si `g.y <= 12` y `g.x > 14`: mover a la izquierda (dir = 'left')
   - Si `g.y <= 12` y `g.x` entre 13-14: ya está en la puerta, salir

3. **Modificar `moveGhost()`** para que cuando `g.exitingPen` sea `true`, llame a `moveGhostToDoor()` en lugar de `decideGhost()`.

4. **Agregar condición de salida**: cuando `g.y <= 11` (cruzó la puerta), setear `g.exitingPen = false`.

5. **Ajustar `resetPositions()`** para reiniciar `exitingPen` a `true` en todos los fantasmas.

6. **Probar manualmente**: verificar que los 4 fantasmas salen directamente sin quedar atrapados.

---

## Acceptance criteria

- [ ] Al ser liberado, cada fantasma se mueve directamente hacia la puerta
- [ ] Ningún fantasma queda atrapado en la pen
- [ ] Una vez fuera de la puerta, el fantasma usa `decideGhost()` normalmente
- [ ] Al perder una vida, los fantasmas regresan a la pen con `exitingPen: true`
- [ ] El juego no tiene errores en consola

---

## Decisions

- **Sí:** Estado `exitingPen` por fantasma. Simple y explícito.
- **No:** Camino fijo predefinido. La lógica condicional es más flexible.
- **Sí:** Priorizar subir primero, luego alinearse horizontalmente. El camino más corto a la puerta.

---

## ¿Qué NO está en este spec?

- Cambiar posiciones iniciales de los fantasmas.
- Ajustar tiempos de salida.
- Power pellet / modo asustado.
