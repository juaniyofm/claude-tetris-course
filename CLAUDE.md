# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Tetris implemented in vanilla JavaScript, HTML5 Canvas, and CSS. No dependencies, no build step, no package.json.

## Running

Open `index.html` directly, or serve it statically:

```bash
python3 -m http.server 8000
# or
npx serve .
```

There is no build, lint, or test tooling in this repo.

## Architecture

Three files, no modules:

- `index.html` — DOM shell: `<canvas id="board">` (300×600, 10×20 grid at `BLOCK=30`px), a `<canvas id="next-canvas">` for the preview piece, HUD spans (`#score`, `#lines`, `#level`), and a shared `#overlay` used for both PAUSE and GAME OVER.
- `style.css` — dark/retro arcade visual theme.
- `game.js` — all game logic, as top-level functions operating on module-level `let` state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropAccum`, `dropInterval`, `animId`). There are no classes and no state encapsulation — any function can read/mutate global state directly.

### Core mechanics (game.js)

- **Board**: `ROWS × COLS` matrix; each cell is `0` (empty) or a piece color index `1–7`.
- **Pieces**: `PIECES` are square matrices; `rotateCW` transposes + reverses rows to rotate. `tryRotate` applies wall kicks by trying x-offsets `[0, -1, 1, -2, 2]` until a non-colliding position is found.
- **Collision**: `collide(shape, ox, oy)` is the single source of truth for both movement and rotation legality, checking bounds and existing board cells.
- **Game loop**: `loop(ts)` runs via `requestAnimationFrame`, accumulates `dt` into `dropAccum`, and advances the piece one row (or locks it) once `dropAccum >= dropInterval`.
- **Locking/scoring**: `lockPiece` → `merge` (writes piece into `board`) → `clearLines` (scans bottom-up, splices full rows, unshifts empty rows at top, awards `LINE_SCORES[cleared] * level`) → `spawn` (promotes `next` to `current`, generates new `next`; if the new piece immediately collides, calls `endGame`).
- **Level/speed**: level = `floor(lines / 10) + 1`; `dropInterval = max(100, 1000 - (level - 1) * 90)`.
- **Ghost piece**: `ghostY()` projects `current` straight down via repeated `collide` checks; drawn at `globalAlpha = 0.2` in `draw()`.
- **Input**: single `keydown` listener does bounds/collision checks inline before mutating `current.x/y` or calling `tryRotate`/`softDrop`/`hardDrop`; `KeyP` toggles pause independent of the `paused`/`gameOver` guard that blocks other input.

When changing board dimensions or block size (`COLS`, `ROWS`, `BLOCK` in `game.js`), also update the `#board` canvas `width`/`height` attributes in `index.html` to match (`COLS × BLOCK`, `ROWS × BLOCK`).
