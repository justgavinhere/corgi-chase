# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the Games

No build step — open any HTML file directly in a browser:

```bash
open corgichase.html
open tictactoe.html
```

To launch after making changes, just re-open the file (or hard-refresh with Cmd+Shift+R).

## Repository Overview

This is a collection of single-file browser games. Each game is a self-contained `.html` file with embedded CSS and JS — no dependencies, no bundler, no server required.

| File | Description |
|---|---|
| `corgichase.html` | Pac-Man style game — corgi eats cookies, chased by humans |
| `tictactoe.html` | Two-player Tic Tac Toe with session score tracking |

## Visual Design System

All games share the same dark theme palette — keep new work consistent:

| Role | Color |
|---|---|
| Background | `#1a1a2e` |
| Primary accent (red) | `#e94560` |
| Secondary accent (teal) | `#a8dadc` |
| Card/panel | `#16213e` |
| Wall/deep bg | `#0f3460` |
| Score/highlight | `#f9c74f` |

Buttons use class `.btn` with `background: #e94560`, `border-radius: 6px`, hover darkens to `#c73652`.

## Corgi Chase Architecture (`corgichase.html`)

The entire game lives in one `<script>` tag, organized in this order:

1. **Constants** — `TILE=40`, `COLS=14`, `ROWS=14`, timing values, tile type enum `T`
2. **Level data** — `LEVELS[]` array of `{ map, enemySpeed, enemyCount }`. Maps are 14-row string arrays where `#`=wall, `.`=path+cookie, ` `=empty, `S`=enemy spawn, `P`=player start. Maps are normalized to exactly 14 chars wide at boot.
3. **Maze parsing** — `parseMaze()` produces a flat `Uint8Array mazeData[196]` and a separate `Uint8Array cookies[196]` (so geometry is immutable; only the cookie array mutates during play)
4. **BFS pathfinding** — `bfsNextDir(data, fc, fr, tc, tr)` runs on the 14×14 grid, called only when an enemy arrives at a new tile
5. **Wall cache** — `buildWallCache()` renders walls once to an `OffscreenCanvas` per level load; `render()` blits it with a single `drawImage` call
6. **Sprite functions** — `drawCorgi`, `drawHuman`, `drawCookie`, `drawPowerCookie` all use `ctx.save/translate/rotate/restore` so the same function works at any position/direction
7. **Update functions** — `updatePlayer`, `updateEnemy`, `updateAIMode`, `checkCookies`, `checkEnemyCatch`, `updateAnimFrames` — all take `dt` (milliseconds)
8. **State machine** — `setState(s)` drives `gameState` and swaps `#overlay` innerHTML. States: `MENU → PLAYING → CAUGHT / LEVEL_COMPLETE → GAME_OVER / WIN_GAME`
9. **Game loop** — `requestAnimationFrame` loop; skips all logic when `gameState !== 'PLAYING'`; caps `dt` at 50ms

**Key design decisions:**
- Movement is tile-based with pixel interpolation (`lerp` between `prevPx/prevPy` and target over `tileDur` ms). Collision uses integer tile coords; catch detection uses pixel distance `< TILE * 0.52`.
- Enemies alternate chase (10s) and scatter (4s) modes. In scatter, each enemy BFS-navigates to its assigned corner instead of the player.
- Player has a `nextDir` buffer — the queued direction is applied when the player reaches a tile center, giving the classic Pac-Man pre-turn feel.
- `player.invincible` (ms countdown) prevents catch immediately after respawn; corgi flashes while active.
- Power cookies are determined by `getPowerCookiePositions()` checking the four maze corners — no separate data; they're just visually distinguished cookies at those tiles.

## Git Workflow

This repo is at `https://github.com/justgavinhere/corgi-chase`.

**Claude must commit and push to GitHub after every meaningful unit of work** — a completed feature, a bug fix, a refactor, a new level, etc. Never leave the session with uncommitted changes. This ensures work is never lost and any state can be reverted to.

After completing any task:

```bash
git add <file>
git commit -m "concise description of what changed and why"
git push
```

Commit message rules:
- Use the imperative mood: "Add sound effects", not "Added sound effects"
- Describe *what* changed and *why*, not just *what*
- Keep the subject line under 72 characters
- One logical change per commit so history is easy to bisect and revert
