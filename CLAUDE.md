# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

No build step or dependencies. Open directly or serve locally:

```bash
# Python
python3 -m http.server 8000

# Node
npx serve .
```

Then open `http://localhost:8000`.

## Architecture

Three files, no framework:

- **`index.html`** — DOM structure: `<canvas id="board">` (300×600 px) for the playfield, `<canvas id="next-canvas">` (120×120 px) for the piece preview, and `#overlay` for pause/game-over states.
- **`style.css`** — Dark/retro aesthetic; `backdrop-filter` on overlays, monospace HUD.
- **`game.js`** — All game logic (~305 lines, `'use strict'`, no modules).

### Key data model (`game.js`)

- `board`: `ROWS × COLS` (20×10) 2-D array; each cell is `0` (empty) or a color index `1–7`.
- `current` / `next`: piece objects `{ type, shape, x, y }` where `shape` is a square matrix of color indices.
- `PIECES` / `COLORS`: parallel arrays indexed 1–7 (index 0 is `null`).

### Game loop

`requestAnimationFrame` drives `loop(ts)`. It accumulates elapsed time in `dropAccum`; when it exceeds `dropInterval` the active piece moves down one row or locks. After each frame, `draw()` renders grid → locked blocks → ghost piece → current piece.

### Core functions

| Function | Purpose |
|---|---|
| `collide(shape, ox, oy)` | Boundary + overlap check |
| `rotateCW(shape)` | Transpose + reverse-rows rotation |
| `tryRotate()` | Rotation with ±1/±2 column wall kicks |
| `clearLines()` | Removes full rows, updates score/level/speed |
| `ghostY()` | Projects piece to landing row for ghost rendering |
| `lockPiece()` | `merge → clearLines → spawn` |
| `init()` | Full reset; called on load and restart |

### Tunable constants

If you change `COLS`, `ROWS`, or `BLOCK`, update the `width`/`height` attributes on `<canvas id="board">` in `index.html` to match (`COLS × BLOCK` and `ROWS × BLOCK`).

| Constant | Default |
|---|---|
| `COLS` / `ROWS` | 10 / 20 |
| `BLOCK` | 30 px |
| `LINE_SCORES` | `[0, 100, 300, 500, 800]` |
| Initial `dropInterval` | 1000 ms |
| Speed formula | `max(100, 1000 − (level−1) × 90)` ms |
