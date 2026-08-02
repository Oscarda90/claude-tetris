# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla-JS Tetris. 3 files: `index.html`, `style.css`, `game.js`. No `package.json`, no build, no bundler, no tests, no lint config, no dependencies.

## Running

Open `index.html` directly (`start index.html` on Windows), or serve statically:

```bash
python3 -m http.server 8000   # or: npx serve .
```

There is no test command. Verification = load the page and play.

## Architecture (`game.js`)

Classic `<script>` (not a module), loaded at end of `<body>`, so:

- All DOM refs are resolved at top level at load time — moving the `<script>` tag earlier breaks it.
- All state lives in module-level `let` globals (`board`, `current`, `next`, `score`, …). Functions mutate them directly; there is no state object to pass around. Keep new code in that style rather than introducing classes/modules.

Core model:

- `board` is `ROWS × COLS` of `0` (empty) or `1–7`, where the number is both the piece type and the index into `COLORS` / `PIECES`. This dual meaning is load-bearing — piece cells store their own type, which is what makes `merge()` and `drawBlock()` one-liners.
- Pieces are square matrices; rotation is transpose+reverse (`rotateCW`), with wall kicks tried at offsets `[0,-1,1,-2,2]` in `tryRotate`.
- `collide(shape, ox, oy)` is the single source of truth for legality — movement, rotation, ghost projection, and game-over all go through it.
- `loop()` (rAF) accumulates `dt` into `dropAccum` and drops one row per `dropInterval`. `lockPiece()` = `merge` → `clearLines` → `spawn`.
- `init()` is both first-start and restart; it resets every global and is wired to the restart button.

## Constraints when editing

- Canvas sizes are hardcoded in `index.html` and must stay in sync with `game.js`: `#board` = `COLS*BLOCK × ROWS*BLOCK` (300×600), `#next-canvas` = 120×120 because `drawNext` centers the shape in a fixed 4×4 grid at 30px.
- `clearLines` splices rows and compensates with `r++` inside a descending loop — required to re-check the shifted row.
- `endGame()` calls `cancelAnimationFrame`, but when it fires from inside `loop` (via `lockPiece`→`spawn`) the current frame still finishes and re-queues itself; guard on `gameOver` if adding logic after the drop step.
- UI strings and README are in Spanish; keep new user-facing text in Spanish.
