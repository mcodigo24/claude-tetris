# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A single-page Tetris implementation in vanilla JavaScript (HTML5 Canvas, CSS3). No dependencies, no build step, no package.json.

## Running

Open `index.html` directly in a browser, or serve it with any static server:

```bash
python3 -m http.server 8000
npx serve .
php -S localhost:8000
```

There is no test suite, linter, or build/bundle process — verify changes by loading the page in a browser and playing.

## Architecture

Three files, all logic lives in `game.js` (~300 lines, no modules/classes — flat top-level functions and module-scoped mutable state):

- `index.html` — DOM shell: `#board` canvas (300×600, the 10×20 grid at `BLOCK=30`px/cell), `#next-canvas` for the next-piece preview, HUD spans (`#score`, `#lines`, `#level`), and the pause/game-over `#overlay`.
- `style.css` — dark/retro arcade visual styling only.
- `game.js` — all game logic and rendering.

Key mechanics in `game.js`:

- **Board model**: `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or a piece color index `1–7`.
- **Pieces**: `PIECES` are square matrices; `rotateCW` rotates via transpose + row reverse. `tryRotate` attempts wall kicks at offsets `[0, -1, 1, -2, 2]` before giving up.
- **Collision**: `collide(shape, ox, oy)` is the single source of truth used by movement, rotation, ghost-piece projection, and spawn-blocking (game over) checks.
- **Game loop**: `loop(ts)` runs via `requestAnimationFrame`, accumulates elapsed time in `dropAccum`, and advances the piece one row (or locks it) once `dropAccum >= dropInterval`.
- **Locking**: `lockPiece()` → `merge()` (writes piece into `board`) → `clearLines()` → `spawn()`.
- **Line clears**: `clearLines()` scans bottom-up, splices completed rows out and unshifts empty rows at the top; scores via `LINE_SCORES = [0, 100, 300, 500, 800]` × `level`.
- **Level/speed**: level = `floor(lines / 10) + 1`; `dropInterval = max(100, 1000 - (level - 1) * 90)` ms.
- **Ghost piece**: `ghostY()` projects the current piece straight down until it would collide; drawn at `globalAlpha = 0.2`.
- **Rendering**: `draw()` clears and redraws the grid, locked board, ghost piece, and current piece every frame (no dirty-rect optimization). `drawNext()` renders the preview canvas separately.
- **Input**: single `keydown` listener switches on `e.code` (arrows, `KeyX` for rotate, `Space` for hard drop, `KeyP` for pause).

When changing `COLS`, `ROWS`, or `BLOCK`, update the `#board` canvas `width`/`height` in `index.html` to match (`COLS × BLOCK`, `ROWS × BLOCK`).
