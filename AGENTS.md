# AGENTS.md

## What this is

Pac-Man clone: vanilla JS + HTML Canvas, no build system, no package manager.
Open `src/index.html` in a browser to run. No dev server required.

## Architecture

JS files are loaded as globals via `<script>` tags. Load order matters:

1. `maze.js` — maze data, constants (`MAZE`, `PACMAN_START`, `GHOST_STARTS`)
2. `game.js` — state + rules (`createGame`, `update`)
3. `render.js` — canvas drawing (`draw`)
4. `main.js` — game loop, keyboard input, UI overlay

All public APIs are attached to `window`. No ES modules, no bundler.

## Maze encoding

28x31 grid, defined as strings in `maze.js`:
- `#` = wall (1), `.` = dot (2), `-` = pen door (3), ` ` = empty (0)
- Tunnel row is 14 (edges open for wrap-around)
- Grid is copied per game session; the original `MAZE` is never mutated

## Code conventions

- No linter, no formatter, no type checker
- Code style uses spaces inside parentheses: `func( arg )`, `[ a, b ]`
- Spanish variable names in UI strings; English in code logic
- Ghost types: `'hunter'` (chases) and `'random'` (random turns)

## Workflow

This project uses spec-driven development. Skills are installed:
- `spec` — design/spec phase (`.agents/skills/spec/`)
- `spec-impl` — implementation phase (`.agents/skills/spec-impl/`)

## Gotchas

- Canvas size is hardcoded: 28 tiles * 20px = 560px wide, 31 tiles * 20px = 620px tall
- `PACMAN_SPEED` (1/8 cell/frame) and `GHOST_SPEED` (1/10 cell/frame) are tuned for alignment
- Ghosts can pass through pen door (`-`), Pac-Man cannot
