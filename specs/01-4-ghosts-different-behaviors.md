# SPEC 01 — 4 fantasmas con comportamientos diferenciados

> **Estado:** Aprobado
> **Depends on:** —
> **Date:** 2026-09-16
> **Objetivo:** Implementar 4 fantasmas con comportamientos únicos (agresivo, punto fijo, intermitente, huidizo) que salen de la pen secuencialmente cada 1.5 segundos.

---

## Scope

**In:**

- Agregar 2 posiciones adicionales de inicio en la pen (`GHOST_STARTS` → 4 entradas)
- Modificar `decideGhost()` en `game.js` para soportar 4 tipos de comportamiento: `'hunter'`, `'ambusher'`, `'shadow'`, `'coward'`
- Implementar sistema de salida secuencial de la pen (1 fantasma cada 1.5 segundos)
- Mantener los 4 colores ya definidos en `render.js`

**Fuera de alcance (specs futuros):**

- Power pellet / modo asustado
- Reencarnación de fantasmas comidos
- Velocidades variables por fantasma
- Nombres visibles en el HUD

---

## Data model

```js
// maze.js — 4 posiciones de inicio en la pen
const GHOST_STARTS = [
  { x: 12, y: 14, kind: 'hunter' },   // Blinky: sale primero
  { x: 13, y: 14, kind: 'ambusher' }, // Pinky: sale 2do
  { x: 14, y: 14, kind: 'shadow' },   // Inky: sale 3ro
  { x: 15, y: 14, kind: 'coward' },   // Clyde: sale último
];

// game.js — estado de salida de la pen
// Se agrega al objeto game:
const game = {
  // ...existing fields...
  ghostExitTimer: 0,   // tiempo acumulado desde el inicio
  ghostsReleased: 1,   // cuántos fantasmas han salido (1 = el primero ya salió)
};
```

---

## Implementation plan

1. **Actualizar `GHOST_STARTS` en `maze.js`** con 4 posiciones dentro de la pen y sus kinds correspondientes. Verificar que el laberinto tenga espacio para 4 fantasmas.

2. **Agregar estado de liberación en `createGame()` en `game.js`** (`ghostExitTimer`, `ghostsReleased`). Inicializar el primer fantasma como ya liberado.

3. **Implementar lógica de salida temporal en `moveGhost()`** — los fantasmas que no han salido permanecen estáticos en su posición de inicio. Cada 1.5 segundos (90 frames a 60fps), liberar el siguiente fantasma.

4. **Extender `decideGhost()` en `game.js`** con los 4 comportamientos:
   - `'hunter'`: persigue a Pac-Man directamente (Manhattan distance) — ya existe
   - `'ambusher'`: se dirige a un punto fijo 4 celdas ahead de Pac-Man en su dirección actual
   - `'shadow'`: persigue a Pac-Man pero con retardo (usa posición de Pac-Man de hace N frames)
   - `'coward'`: huye de Pac-Man cuando está cerca (< 8 celdas), persigue cuando está lejos

5. **Ajustar `resetPositions()`** para reiniciar también `ghostExitTimer` y `ghostsReleased`.

6. **Probar manualmente:** abrir `src/index.html`, verificar que los 4 fantasmas salen secuencialmente y cada uno tiene un patrón de movimiento distinto.

---

## Acceptance criteria

- [ ] Hay 4 fantasmas en el juego, cada uno con un color diferente
- [ ] Los 4 fantasmas arrancan dentro de la pen
- [ ] Los fantasmas salen de la pen uno por uno, cada ~1.5 segundos
- [ ] El fantasma `'hunter'` persigue a Pac-Man directamente por el camino más corto
- [ ] El fantasma `'ambusher'` se dirige a un punto fijo delante de Pac-Man
- [ ] El fantasma `'shadow'` persigue con retardo (no va directo al Pac-Man actual)
- [ ] El fantasma `'coward'` huye cuando Pac-Man está cerca y persigue cuando está lejos
- [ ] Ningún fantasma atraviesa paredes
- [ ] El juego no tiene errores en consola

---

## Decisions

- **Sí:** 4 comportamientos clásicos de Pac-Man (Blinky, Pinky, Inky, Clyde). Son probados y bien entendidos.
- **No:** Velocidades diferentes por fantasma. Se puede agregar después.
- **Sí:** Salida temporal (1.5s cada uno). Más intuitivo que por puntos.
- **No:** Power pellet / modo asustado. Queda para otro spec.
- **No:** Nombres en el HUD. Solo se distinguen por color.

---

## ¿Qué NO está en este spec?

- Power pellet / modo asustado.
- Reencarnación de fantasmas comidos.
- Velocidades variables por fantasma.
- Nombres visibles en el HUD.
