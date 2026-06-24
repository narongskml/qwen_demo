# Qwen — Notes & Documentation

## Overview

**Qwen** — Alibaba's large language model family (Qwen, Qwen2, Qwen2.5, Qwen3, etc.).

## Contents

- `Readme.MD` — placeholder for project notes (currently empty)
- `platformer.html` — HTML5 2D platformer game ("Super Qwen Bros")

## Project Type

Mixed: documentation / research-notes directory plus an HTML5 game project.

---

## Game Prompt (for `platformer.html`)

> You are an expert HTML5 Game Developer. Write a complete, single-file HTML5 game (HTML, CSS, JS in one file) that plays like a 2D platformer similar to Super Mario Bros.
>
> **Requirements:**
>
> - Use HTML5 Canvas for rendering.
> - Player character: A red square. Can move Left/Right with Arrow keys, Jump with Spacebar. Implement gravity and jumping physics.
> - Environment: A ground platform, and some floating brick platforms.
> - Enemies: Blue squares that walk back and forth on platforms.
> - Collision & Mechanics: Player dies if touching enemy from the side. Player kills enemy if jumping on top of it. Player can collect yellow coins.
> - Camera: Side-scrolling camera that follows the player.
> - Win condition: Reach a green flag at the end of the stage.
>
> Constraints: clean, object-oriented JavaScript. No external libraries, just pure Vanilla JS.

## Implementation Notes (`platformer.html`)

- **Canvas:** 800×480, sky-blue gradient background with parallax clouds & hills.
- **Player (`Player` class):** 26×32 red square with a cap and eyes; Arrow Left/Right to move, Space/Up to jump; gravity (0.55), jump force (-11), terminal velocity (12); proper X-then-Y axis collision resolution.
- **Platforms (`Platform` class):** Ground sections with pit gaps + 27 floating brick platforms at varying heights; textured with simple line patterns.
- **Enemies (`Enemy` class):** 28×28 blue squares with animated eyes; patrol between set bounds on ground and platforms; 12 total.
- **Collision logic:** Side touch with enemy → player dies; landing on top of enemy (feet above ~60% of enemy height) → squish enemy, bounce player (+100 pts).
- **Coins (`Coin` class):** Yellow coins with bob animation; 60+ placed throughout the level (+50 pts each).
- **Flag (`Flag` class):** Green triangular flag on a pole at tile x=160; reaching it ends the level.
- **Camera:** Smooth lerp follow (`0.1` interpolation), clamped to level bounds (level is 200 tiles wide × 15 tiles tall).
- **HUD:** Score + coin counter overlay, plus a `🔊 Music` / `🔇 Music` toggle button in the top-right.
- **Overlays:** "GAME OVER" / "YOU WIN!" screens; press **R** to restart.
- **Architecture:** 7 OOP classes (`Platform`, `Coin`, `Enemy`, `Flag`, `Player`, `Particle`, `Game`) plus a `SoundManager` for audio; no external dependencies; single-file, pure Vanilla JS.

## Game Juice (visual feedback)

Lightweight "juice" layer on top of the simulation — decorative only, does **not** affect collision or physics.

- **`Particle` class:** tiny colored square with per-particle velocity, gravity (0.28), air drag (0.97×), and a linear alpha fade from 1 → 0 over its lifetime. Drawn in world-space (offset by camera) *after* the player so they sit on top of the scene.
- **Screen shake:** two fields on `Game` — `shakeIntensity` (decays by ×0.85 each idle frame) and `shakeTimer` (holds intensity constant for N frames). Applied as a `ctx.translate(randX, randY)` *after* the sky fill but *before* the world draw, wrapped in `save/restore` so the HUD stays rock-steady.
- **Squash & stretch:** `Player.scaleX / scaleY`, lerped back to 1.0 each frame (`(1 - scale) * 0.18`). The sprite is drawn around its own center using these scales so its feet stay roughly planted.

### Triggers

| Event | Particles | Shake | Squash / Stretch |
|---|---|---|---|
| Jump | — | — | `scaleX 0.82, scaleY 1.22` (tall & thin) |
| Land (airborne → grounded) | — | — | `scaleX 1.25, scaleY 0.75` (wide & short) |
| Stomp enemy | 10 blue/white particles from enemy center | intensity 8, timer 8 | — |
| Player die | 14 red/white particles from player center | intensity 14, timer 18 | — |
| Collect coin | 8 yellow/white particles from coin center | — | — |

## Sound System (`SoundManager`)

All audio is synthesized at runtime with the **Web Audio API** — **no external `.mp3`/`.wav` files**.

### SFX (oscillator one-shots)

| Sound   | Trigger | Synthesis |
|---------|---------|-----------|
| `jump`  | Space / ↑ while grounded | Square wave, 330 → 660 Hz sweep, 140 ms |
| `coin`  | Picking up a coin | Square wave: B5 (988 Hz) 60 ms → E6 (1319 Hz) 140 ms |
| `stomp` | Landing on an enemy | Sine wave 200 → 60 Hz pop, 120 ms |
| `death` | Touching an enemy side-on | Square wave 440 → 80 Hz descending, 550 ms |
| `win`   | Touching the flag | Square-wave arpeggio C5–E5–G5–C6, 4 × 140 ms |

### Background Music (BGM)

- **Rendered once** at startup into an `AudioBuffer` via `OfflineAudioContext`, then looped with `AudioBufferSourceNode.loop = true` — no per-frame scheduling cost and a **truly seamless** loop.
- **Loop length:** 8 bars at 140 BPM (~13.7 s).
- **Melody (square wave):** 2-bar hook `C5 C5 G5 G5 C6 B5 | A5 G5 F5 E5`, repeated 4× to fill 8 bars.
- **Bassline (triangle wave):** root notes for a `I–IV–V–I` progression (C → F → G → C), one note every half-bar, with a 5th-step turnaround on the final bar.
- **Routing:** `BGM source → _bgmGain → masterGain → destination`. SFX route directly to `masterGain`, so they keep playing even when the BGM is muted.

### Lifecycle

- BGM **starts** whenever a `Game` is constructed (including on restart).
- BGM **stops** once on the transition `playing → dead` or `playing → win` (detected via `prevState` tracking in `update()`); the matching SFX still plays on top.
- **Mute toggle** (`setBGMMuted(bool)`) fades `_bgmGain` via `setTargetAtTime` (50 ms ramp). The source keeps running underneath, so toggling back on resumes at exactly the same position — no re-syncing glitches.
- **Autoplay policy:** `AudioContext.resume()` is bound to `keydown` + `mousedown`, so the first user gesture unlocks audio.

## Running the Game

Just open `platformer.html` in any modern browser — no server, build step, or dependencies required.

## Controls

| Key / control | Action |
|---------------|--------|
| ← / → | Move left / right |
| Space / ↑ | Jump |
| R | Restart (after win or death) |
| 🔊/🔇 Music button (top-right) | Toggle BGM (SFX continue to play) |
