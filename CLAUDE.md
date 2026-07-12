# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Classic Tetris implemented in vanilla JavaScript, HTML5 Canvas, and CSS. No dependencies, no build step, no package.json.

## Running the game

Open `index.html` directly in a browser, or serve it with any static server:

```bash
python3 -m http.server 8000
npx serve .
php -S localhost:8000
```

There is no build, lint, or test tooling in this project — it's just open-and-play.

## Architecture

Three files cooperate, with all game logic in `game.js` (~300 lines):

- **`index.html`** — DOM structure: a `300×600` `<canvas id="board">` for the game, a `<canvas id="next-canvas">` (120×120) for the next-piece preview, HUD elements (`#score`, `#lines`, `#level`), and an `#overlay` used for both PAUSE and GAME OVER states.
- **`style.css`** — dark/retro arcade visual theme.
- **`game.js`** — all game state and logic, driven by module-level `let` variables (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc.) rather than a class/object.

### Core model

- `board` is a `ROWS × COLS` matrix (20×10) of `0` (empty) or a color index `1–7` identifying a locked piece.
- Pieces (`PIECES`) are defined as square matrices; `COLORS[index]` maps a piece's color index to a hex color.
- `current` / `next` pieces are `{ type, shape, x, y }` objects produced by `randomPiece()`.

### Key functions and game flow

- `rotateCW(shape)` rotates a piece matrix via transpose + row reversal. `tryRotate()` applies this and attempts wall kicks (`kicks = [0,-1,1,-2,2]` columns) before giving up on a rotation.
- `collide(shape, ox, oy)` checks bounds and overlap against `board` — the single source of truth for all movement/rotation legality.
- `lockPiece()` → `merge()` (writes piece into `board`) → `clearLines()` (scans bottom-up, splices full rows, unshifts empty ones at top, updates score/level/dropInterval) → `spawn()` (promotes `next` to `current`, generates a new `next`; if the new `current` immediately collides, calls `endGame()`).
- `ghostY()` projects where the current piece would land; used both for drawing the ghost piece (alpha 0.2) and for `hardDrop()` scoring.
- `loop(ts)` is the `requestAnimationFrame` game loop: accumulates elapsed time in `dropAccum`, drops the piece one row (or locks it) once `dropAccum >= dropInterval`, then redraws.
- Scoring: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop = 2 pts/row descended, soft drop = 1 pt/row. Level increases every 10 lines; `dropInterval = max(100, 1000 - (level-1)*90)`.
- Input handling is a single `keydown` listener switching on `e.code` (arrows, `KeyX` for rotate, `Space` for hard drop, `KeyP` for pause).

### Tunable constants (in `game.js`)

`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, update the `<canvas id="board">` `width`/`height` in `index.html` to match (`COLS × BLOCK` and `ROWS × BLOCK`).
