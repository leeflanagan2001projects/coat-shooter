# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the Game

Open `game.html` directly in a browser — no build step, no server required:
```
file:///C:/Users/leefl/ClaudeCodeTest/game.html
```

## Git Workflow

After every change: commit with a descriptive message and push to keep GitHub in sync.
- Remote: `https://github.com/leeflanagan2001projects/coat-shooter`
- Branch: `master`

```bash
git add game.html
git commit -m "brief description of what changed"
git push
```

## Architecture

Everything lives in `game.html` as a single file: `<style>` → `<canvas>` → `<script>`. There are no modules, bundlers, or external dependencies.

### Game Loop

`requestAnimationFrame` drives a fixed pipeline every frame:
```
gameLoop(timestamp) → update(dt) → render()
```
`dt` is capped at 50ms to prevent spiral-of-death on tab blur. All timers in the game use **milliseconds** (e.g. `invincTimer`, `shootCooldown`, `hitFlash`) and are decremented by `dt * 1000`.

### State Machine

`gameState` controls both `update()` and `render()` branching:

| State | Value | Behaviour |
|---|---|---|
| `MENU` | 0 | Draw menu; Space starts game |
| `PLAYING` | 1 | Full update + render |
| `LEVEL_TRANSITION` | 2 | Only decrement `levelTransTimer`; overlay drawn on top of game |
| `GAME_OVER` | 3 | Freeze update; R restarts |
| `WIN` | 4 | Freeze update; R restarts |

### Object Pools

Bullets (50) and particles (200) are pre-allocated at startup. Objects are reused by setting `active = true/false` — never push/pop from these arrays. Enemies are a plain dynamic array (`enemies[]`) that is spliced on death.

### Wave / Level Progression

`LEVEL_DATA[]` defines the first 4 levels. `getLevelData(level)` extrapolates beyond index 3 by adding 3 enemies/wave and shifting mix toward tanks. Wave completion is detected in `checkWaveCompletion()` by checking three conditions simultaneously: all enemies spawned, `enemies[]` empty, and `waveEnemiesLeft <= 0`.

### Coordinate System

Canvas is fixed at **900 × 650**. Mouse coordinates are remapped from CSS pixels to canvas pixels via the `getBoundingClientRect` scale ratio in the `mousemove` listener. All collision is simple circle-circle (`dist()`) using each enemy's `radius` property.

### Adding a New Enemy Type

1. Add a key to `ENEMY_TYPES`
2. Add a config entry in the `configs` object inside `spawnEnemy()`
3. Add a draw branch in `drawEnemies()`
4. Adjust `mix` arrays in `LEVEL_DATA` (must sum to ≤ 1.0; remainder falls to Tank)
