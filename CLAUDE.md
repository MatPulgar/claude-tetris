# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JS Tetris. No build step, no package manager, no dependencies — three files: `index.html`, `style.css`, `game.js`.

## Running

Open `index.html` directly in a browser, or serve statically:

```bash
python3 -m http.server 8000
# or
npx serve .
```

There is no test suite, linter, or build/bundle command. Verify changes by loading the page and playing.

## Architecture

`game.js` is a single-file game with no modules/classes — global mutable state (`board`, `current`, `next`, `score`, `lines`, `level`, ...) plus top-level functions, initialized by `init()` at the bottom of the file.

- **Board**: `ROWS × COLS` matrix (20×10); each cell is `0` (empty) or a piece color index (1–7).
- **Pieces**: `PIECES` holds the 7 tetromino shapes as square matrices. `rotateCW` rotates via transpose + row-reverse; `tryRotate` applies `rotateCW` then tries wall-kick offsets `[0, -1, 1, -2, 2]` on the x-axis, discarding the rotation if all collide.
- **Collision**: `collide(shape, ox, oy)` is the single source of truth for both movement and rotation legality — checks bounds and overlap with locked board cells.
- **Game loop**: `requestAnimationFrame`-driven `loop(ts)` accumulates elapsed time in `dropAccum`; once it exceeds `dropInterval`, the piece drops a row or locks (`lockPiece` → `merge` + `clearLines` + `spawn`).
- **Scoring/leveling**: line clears score via `LINE_SCORES` (`[0,100,300,500,800]`) × `level`; hard drop adds 2/cell, soft drop 1/row. Level = `floor(lines / 10) + 1`; `dropInterval = max(100, 1000 - (level-1)*90)`.
- **Ghost piece**: `ghostY()` projects the current piece straight down until it would collide, drawn at `globalAlpha = 0.2`.
- **Rendering**: two canvases — `#board` (main playfield, drawn each frame by `draw()`) and `#next-canvas` (next-piece preview, drawn by `drawNext()` only on spawn). `drawBlock` is the shared per-cell rasterizer used by both.

When changing `COLS`, `ROWS`, or `BLOCK` in `game.js`, also update the `#board` canvas `width`/`height` attributes in `index.html` to match (`COLS × BLOCK`, `ROWS × BLOCK`).

Comments and UI strings in this repo are in Spanish (README is fully in Spanish); match that convention for user-facing text.
