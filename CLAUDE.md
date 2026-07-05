# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Classic Tetris implemented in vanilla JavaScript with HTML5 Canvas and CSS. No dependencies, no build process, no package.json — just three files: `index.html`, `style.css`, `game.js`.

## Running the game

There is no build/lint/test tooling. Open directly or serve statically:

```bash
open index.html                # macOS, or just open in a browser
python3 -m http.server 8000    # or: npx serve .
```

To verify a change works, reload the page in a browser and play; there are no automated tests.

## Architecture (`game.js`, ~300 lines, single file)

Global mutable state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `dropAccum`, `lastTime`, `animId`) is declared once and reset in `init()`. There is no module system, no classes — everything is top-level functions operating on these globals.

- **Board model**: `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or an index `1–7` into `COLORS`/`PIECES` identifying the piece that occupies it.
- **Pieces**: the 7 tetrominoes are defined as fixed square matrices in `PIECES`. Rotation (`rotateCW`) is a generic transpose+reverse on the shape matrix — pieces don't carry per-type rotation states (no SRS kick tables).
- **Collision** (`collide`): the single source of truth for whether a shape can occupy a given offset — used by movement, rotation, ghost-piece projection, and spawn-blocking (game over check).
- **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` columns and keeps the first that doesn't collide. This is a simplified approximation of SRS wall kicks, not the official table.
- **Game loop** (`loop`): driven by `requestAnimationFrame`; accumulates elapsed time in `dropAccum` and advances the piece one row (or locks it) once `dropAccum >= dropInterval`. Rendering (`draw`) happens every frame regardless of drop timing.
- **Locking** (`lockPiece` → `merge` + `clearLines` + `spawn`): merges the current piece into `board`, clears full rows (scanning bottom-up, splicing cleared rows and unshifting empty ones — note the `r++` compensation in the loop after a splice), then spawns the next piece.
- **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop adds 2 points per row dropped, soft drop adds 1 point per row.
- **Leveling/speed**: level = `floor(lines / 10) + 1`; `dropInterval = max(100, 1000 - (level - 1) * 90)` ms.
- **Ghost piece**: `ghostY()` projects the current piece straight down via repeated `collide` checks; drawn at `globalAlpha = 0.2` in `draw()`.
- **Rendering**: two canvases — `#board` (main play field, drawn by `ctx`) and `#next-canvas` (next-piece preview, drawn by `nextCtx` via `drawNext()`). Both share the `drawBlock` helper.

### Control flow

```
init() → createBoard(), next = randomPiece(), spawn(), requestAnimationFrame(loop)
loop(ts) → accumulate dt → maybe advance/lock piece → draw() → requestAnimationFrame(loop)
keydown → move / tryRotate / softDrop / hardDrop / togglePause (ignored while paused/gameOver, except P)
```

`spawn()` promotes `next` to `current` and generates a new `next`; if the newly spawned piece immediately collides, `endGame()` fires and the overlay shows GAME OVER. `restartBtn` re-runs `init()`.

## Tunable constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK`, `COLORS`, `PIECES`, `LINE_SCORES`, and the initial `dropInterval` in `init()`. If `COLS`, `ROWS`, or `BLOCK` change, the `#board` canvas `width`/`height` in `index.html` must be updated to match (`COLS × BLOCK` and `ROWS × BLOCK`).

## Language note

The README and in-game UI copy (labels, overlay text) are in Spanish; keep new user-facing strings consistent with that unless told otherwise.
