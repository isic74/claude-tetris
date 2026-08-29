# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A classic Tetris implementation in vanilla JavaScript (no frameworks, no build tools, no dependencies, no `package.json`). Three files: `index.html`, `style.css`, `game.js`.

## Running the game

No install or build step. Either open `index.html` directly in a browser, or serve it with any static file server, e.g.:

```bash
python3 -m http.server 8000
# or
npx serve .
```

There is no test suite, linter, or build/watch process in this repo.

## Architecture

Everything lives in `game.js` (a single script loaded at the end of `index.html`, no modules/bundler). Global mutable state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc.) is declared once at the top and reset in `init()`.

Key pieces to understand before making changes:

- **Board model**: `board` is a `ROWS × COLS` matrix (`createBoard()`); each cell is `0` (empty) or a color index `1–7` identifying which piece locked there.
- **Pieces**: `PIECES` holds the 7 tetrominoes as square matrices. `current`/`next` are `{ type, shape, x, y }`. Rotation is done via `rotateCW()` (transpose + row reverse), not by storing rotation states.
- **Collision**: `collide(shape, ox, oy)` is the single source of truth for "does this shape at this offset hit a wall, floor, or locked block" — reused by movement, rotation, ghost-piece projection, and spawn checking.
- **Wall kicks**: `tryRotate()` rotates then tries offsets `[0, -1, 1, -2, 2]` via `collide()` until one works, else the rotation is discarded.
- **Game loop**: `loop(ts)` runs on `requestAnimationFrame`, accumulates elapsed time in `dropAccum`, and advances the piece one row (or calls `lockPiece()`) once `dropAccum >= dropInterval`. `draw()` redraws the whole canvas every frame (grid, locked board, ghost piece, current piece) — there's no dirty-rect optimization.
- **Locking/scoring**: `lockPiece()` → `merge()` (writes the piece into `board`) → `clearLines()` (scans bottom-up, splices full rows, unshifts empty ones, updates `score`/`lines`/`level`/`dropInterval` via the classic `LINE_SCORES` table) → `spawn()` (promotes `next` to `current`, generates a new `next`, and calls `endGame()` if the new piece immediately collides).
- **Ghost piece**: `ghostY()` projects `current` straight down via repeated `collide()` checks; drawn at `globalAlpha = 0.2`.
- **Input**: a single `keydown` listener switches on `e.code` (arrows + `Space` + `KeyX` for rotate + `KeyP` for pause), guarded by `paused`/`gameOver`.

Tunable constants live at the top of `game.js`: `COLS`, `ROWS`, `BLOCK` (px per cell), `COLORS`, `LINE_SCORES`, and the initial `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, the `<canvas id="board">` `width`/`height` in `index.html` must be updated to match (`COLS × BLOCK`, `ROWS × BLOCK`).
