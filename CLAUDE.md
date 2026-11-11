# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Vanilla JavaScript Tetris on HTML5 Canvas. No dependencies, no `package.json`, no build step, no tests, no linter. User-facing text (UI labels, overlay messages, README) is in Spanish; keep it that way.

## Running

Open `index.html` directly, or serve the folder statically (preferred):

```bash
python3 -m http.server 8000   # then open http://localhost:8000
```

## Architecture

Three files: `index.html` (DOM: `#board` canvas, side panel with HUD and `#next-canvas`, `#overlay` for pause/game over), `style.css`, and `game.js` (all logic, global script with `'use strict'`, no modules).

Key points in `game.js`:

- **State** is module-level `let` globals (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `animId`, ...). `init()` resets all of it and is also the restart handler.
- **Cell values double as color indices.** `board` is a `ROWS × COLS` matrix of `0` (empty) or `1–7`; each `PIECES[type]` matrix stores its own type number in filled cells, and `merge()` copies those values straight into `board`. `COLORS[i]` maps the index to a color. Adding a piece type requires extending both arrays in the same index.
- **Piece flow:** `spawn()` promotes `next` to `current` and generates a new `next`; a collision right at spawn calls `endGame()`. `lockPiece()` = `merge()` → `clearLines()` → `spawn()`. Rotation is `rotateCW` (transpose + reverse) with simple horizontal kicks `[0, -1, 1, -2, 2]` in `tryRotate` (not SRS).
- **Game loop:** `requestAnimationFrame` accumulates `dt` into `dropAccum` and drops one row when it exceeds `dropInterval`. Pause/game over work by cancelling `animId`; resuming calls `loop()` again with a fresh `lastTime`.
- **Scoring/levels** live in `clearLines()`: `LINE_SCORES[cleared] * level`, level = `floor(lines / 10) + 1`, `dropInterval = max(100, 1000 - (level - 1) * 90)`. Soft drop +1 per row, hard drop +2 per row.
- **Rendering** redraws the whole board every frame (`draw()`: grid, locked cells, ghost at alpha 0.2, current piece). `drawNext()` only runs on spawn.

## Coupled values

Canvas sizes in `index.html` are hardcoded and must match `game.js` constants: `#board` is `COLS × BLOCK` by `ROWS × BLOCK` (300×600), and `#next-canvas` (120×120) assumes a 4×4 grid at the `NB = 30` block size in `drawNext()`.
