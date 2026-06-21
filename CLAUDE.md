# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the Game

Open `index.html` directly in a browser — no build step or server required.

To serve via Docker:
```bash
docker build -t skull-pacman .
docker run -p 8080:80 skull-pacman
# Game available at http://localhost:8080
```

## Architecture

The entire game is a single self-contained `index.html` file with no external dependencies. It is structured in three sections: CSS styles, HTML markup, and a `<script>` block containing all game logic.

### Rendering
- HTML5 Canvas (21×23 grid, 24px cells)
- An offscreen canvas is used to draw the skull player so `destination-out` compositing can cut out eye sockets and nose holes before stamping it onto the main canvas
- Neon glow effects use `ctx.shadowBlur` / `ctx.shadowColor`

### Game Loop
`requestAnimationFrame` drives the loop. Each tick: move entities → eat dots → check collisions → draw.

### Movement Model
Entities move cell-to-cell (not pixel-by-pixel). `stepEntity()` interpolates toward the target cell center; direction is chosen on arrival. Tunnel wrapping is handled in `tryMove()`.

### Ghost AI
Each ghost has a state machine: `house → exiting → scatter → chase`. Chase mode uses greedy distance minimization toward the player; scatter targets a fixed corner. U-turns are forbidden except as a last resort. Power pellets flip ghosts to `frightened` (random movement) and reverse their current direction.

### Game State Machine
`title → playing` and then cycles through `dead`, `gameover`, and `levelup` states. Space/Enter drives all transitions.

### Power-ups
Periodically a power-up spawns on a random walkable cell (max 2 at once). Either the player or a ghost picks it up by entering its cell, and the effect applies automatically to that entity (self-buff, so it's symmetric). Three types: **Speed Boost** (`⚡`, +40% move speed ~4s via `boostTimer`), **Phase Dash** (`✦`, one-shot teleport 2 cells forward through a wall), and **Ghost-Pass** (`◈`, walk through walls ~3s via `passTimer`; entity renders semi-transparent). Wall-passing is handled by `canMoveE()` (entity-aware variant of `canMove()`) and a relaxed filter in the ghost AI.

### Audio
All sound is synthesized at runtime via the Web Audio API (the `SFX` module) — no asset files, keeping the single-file design. Square/triangle/sawtooth oscillators with quick gain envelopes produce 8-bit SFX (chomp, power, ghost-eaten, death, level-up). Background music is a low-volume ambient arpeggio looped on a `setInterval` scheduler. The `AudioContext` is created on the first Space/Enter press (browsers block audio before user interaction). `M` toggles mute.

### Map Format
`BASE_MAP` is a 2D array of integers: `0`=empty, `1`=wall, `2`=dot, `3`=power pellet, `4`=ghost house (passable by ghosts only).
