# Sun Surfer — Game Specification

## Overview

Single-player browser arcade game. Surfer hovers above a massive star, collecting solar flares for upward boosts while gravity pulls down. Death by burning (too low) or drifting into space (too high).

## Terminology

| Term           | Meaning                                       |
| -------------- | --------------------------------------------- |
| **Surfer**     | Player character                              |
| **Flare**      | Collectible erupting upward from star surface |
| **Burned**     | Game over: fell into star                     |
| **Lost**       | Game over: drifted off top of screen          |
| **Altitude**   | Surfer distance (px) above star surface       |
| **Sweet spot** | ~45% altitude zone awarding bonus score       |

## Project Constraints

Single `index.html` — all HTML/CSS/JS, no dependencies. HTML5 Canvas rendering. Vanilla ES6+. `requestAnimationFrame` loop.

## Game States

| State            | Enters when                | Behavior                                    |
| ---------------- | -------------------------- | ------------------------------------------- |
| **Start Screen** | App launch                 | Title + animated star. Any input → Playing. |
| **Playing**      | Start / restart            | Active gameplay.                            |
| **Burned**       | Altitude < `BURN_ALTITUDE` | Game over message + restart.                |
| **Lost**         | `surfer.y ≤ 0`             | Game over message + restart.                |

## Controls

| Input                    | Action                           |
| ------------------------ | -------------------------------- |
| ←/→ or A/D               | Horizontal acceleration          |
| Touch left/right half    | Horizontal acceleration (mobile) |
| Space / "Ride Again" btn | Restart (when not playing)       |
| Any input (start screen) | Begin game                       |

**Core rule:** No direct vertical control. Vertical movement = gravity (down) + flare boosts (up). This is the central tension.

## Physics

Per-frame. See **Constants Reference** for values.

1. Input → add `±SURFER_SPEED` to `vx`
2. Gravity → add `GRAVITY` to `vy`
3. Drag → `vx *= DRAG_X`, `vy *= DRAG_Y`
4. Clamp → `|vx| ≤ 7`, `vy` in `[-10, 6]`
5. Integrate → update position

## World & Camera

- Horizontal scroll only; surfer locked to screen center X.
- `cameraX = surfer.x` (no smoothing).
- No vertical scrolling; Y is absolute.
- Star surface at `y = H - SURFACE_Y_OFFSET`.
- Star radius = `W × STAR_RADIUS_MULT` (surface appears flat).

## Star

Massive radial-gradient circle (white-yellow core → orange-red edge).

- **Surface**: 15 animated bright spots, parallax-scrolled.
- **Corona**: 20 upward gradient rays with parallax + oscillation.
- **Glow**: Pulsing vertical gradient above surface (sine wave).
- **Face**: Eyes (blink every ~180 frames), rosy cheeks, smile. Lags behind camera at rate `0.03`.

## Surfer

### Appearance

~18px radius character: cyan surfboard (tilts with `vx × 0.08`), peach body + blue suit accent, helmet visor, 3 orange hair tufts (wave in solar wind), waving arms.

- **Bob**: `sin(phase) × 3px` vertical oscillation.
- **Glow**: Pulsing blue radial aura.
- **Heat glow**: Orange aura within 200px of surface, intensity ∝ proximity.

### Expressions

| Expression | Trigger                         | Duration      | Visual                                           |
| ---------- | ------------------------------- | ------------- | ------------------------------------------------ |
| Happy      | Default                         | Persistent    | Dot eyes + highlights, smile arc                 |
| Cool       | In sweet spot (no active timer) | While in zone | Dark sunglasses with blue lens shine, smirk      |
| Excited    | Flare collected, `force ≤ 3`    | 20 frames     | Large eyes + star highlights, big smile          |
| Woah       | Flare collected, `force > 3`    | 40 frames     | Hollow circle eyes, "O" mouth                    |
| Scared     | Altitude < `BURN_ALTITUDE + 50` | 30 frames     | Wide eyes + white pupils, wavy mouth, sweat drop |
| Worried    | `surfer.y < 60`                 | 30 frames     | Dot eyes, flat slanted mouth                     |

### Trail

Up to 25 cyan circles fading at `0.04` life/frame.

## Solar Flares

### Spawning

Timer-based: one flare per expiry, reset to `[FLARE_SPAWN_INTERVAL_MIN, FLARE_SPAWN_INTERVAL_MAX]` frames. Upper bound compresses up to 40% as score → 500. Spawn at `surfaceY - 5`, random X within `±1.5W` of camera.

**Starting flare**: On game start, one guaranteed medium flare spawns within ±10% screen width of the surfer's starting X. This prevents unwinnable opening situations.

### Types

| Type   | Prob | Size (px) | Color (R,G,B) | Speed ↑ | Force | Boost mult |
| ------ | ---- | --------- | ------------- | ------- | ----- | ---------- |
| Small  | 50%  | 10–16     | 255,230,100   | 2.0–3.0 | 5.0   | 0.7        |
| Medium | 32%  | 18–28     | 255,180,50    | 1.5–2.3 | 9.0   | 0.7        |
| Large  | 18%  | 30–44     | 255,100,30    | 1.2–1.8 | 16.0  | 0.595      |

### Behavior

- Rise at assigned speed + slight horizontal drift (`±0.3`).
- Size pulsates ±20% (sine wave).
- Large flares (size > 24px) show radiating sparkle lines.
- Rendered: outer glow (2× radius) + inner core gradient.

### Collection

- **Hit test**: distance(surfer, flare) < `flare.size + SURFER_SIZE`.
- **Boost**: `vy -= Force × Boost mult`.
- **Score**: `force × 10` points (→ 50 / 90 / 160).
- Flare counter +1. Burst of 12 particles in flare color.

### Removal

Collected, or off-screen: `screenX < -1.5W`, `screenX > 2.5W`, `y < -80`, or `y > surfaceY + 20`.

## Scoring

- **Altitude bonus** (per frame): `max(0, 1 - |altNorm - 0.45| × 6) × 0.3` where `altNorm = altitude / surfaceY`.
- **Flare bonus**: `force × 10` on collection.

## HUD

Top-center: `FLARES: N | SCORE: N`

- Golden text (#ffe8a0), letter-spacing, warm text-shadow.

## Game Over

| Condition | Title             | Effect                         |
| --------- | ----------------- | ------------------------------ |
| Burned    | ☀️ Burned Up!     | 40 yellow/gold burst particles |
| Lost      | 🌌 Lost in Space! | (no extra particles)           |

Both show: sub-text "You rode N flares and scored N points", orange gradient "Ride Again" button with hover scale.

## Visual Effects

- **Background stars**: 200 stars in upper 75% of screen. Twinkle (sine alpha). Parallax at 5–20% camera speed, wrap horizontally.
- **Danger zone (low)**: Warm red gradient, `BURN_ALTITUDE + 50` px tall, fading upward from surface.
- **Danger zone (high)**: Cool blue gradient near `surfaceY - H × MAX_ALTITUDE`.
- **Sweet spot aurora**: 5 horizontal bands spanning the sweet spot zone (`altNorm` 0.117–0.783). Each band uses a shifting HSL hue (green→cyan range), fading to transparent at edges. Bands animate via sine wave for a gentle shimmer.
- **Guide lines**: Dashed lines at burn/max altitude boundaries (5% opacity).
- **Particles**: Position + velocity (0.98 drag) + size + color + life (decays 0.015–0.04/frame). Render as circles, alpha = life.

## Scoreboard

### Persistence

- Top 20 highest-scoring games stored in `localStorage` key `"sunSurferScoreboard"`.
- Each entry: `{ score, date (ISO string), playTimeSeconds }`.
- Sorted descending by score. Entries beyond 20 are discarded.

### Real-Time Rank

- During gameplay, the player's **current rank** is displayed to the right of the score in the HUD (e.g. `#4`).
- Rank = position the current score would occupy if inserted into the scoreboard now (1 = best).
- If the score is below all 20 entries and the board is full, rank shows `—`.

### Rank-Up Animation

When the player's live rank improves (passes a stored score):

1. A **green up-arrow** (▲) launches upward from the surfer toward the rank display.
2. The arrow leaves a **faint trail of rainbow sparkles** that fade quickly.
3. When the arrow reaches the rank text, the rank text **wobbles** (oscillating horizontal stretch) for ~20 frames.
4. After the wobble, the rank digit(s) **roll over** like a mechanical counter (old number scrolls up, new number scrolls in from below) and settle on the new rank.

### Game Over

- On game over, the final score is inserted into the scoreboard (if it qualifies for top 20).
- The game-over overlay shows the player's final rank on the board (or "unranked" if outside top 20).
- Below the "Ride Again" button, the full top-20 scoreboard is displayed as an arcade-styled table.
  - Columns: **Rank** (🥇🥈🥉 for top 3, then numerals), **Score** (comma-formatted), **Date** (MM/DD/YY), **Time** (e.g. `1m 23s`).
  - The current run's row is highlighted with an amber glow.
  - The table scrolls vertically (max ~42vh) and auto-scrolls to the current run's row.
  - Style: monospace font, orange header gradient, dark background, low-opacity row dividers.

## Responsive Design

Full-viewport canvas. Dynamic resize on `window.resize`. Touch controls for mobile. Dark background (`#0a0a1a`), flexbox centered, overflow hidden.

## Constants Reference

| Constant                   | Value | Description                                 |
| -------------------------- | ----- | ------------------------------------------- |
| `STAR_RADIUS_MULT`         | 8     | Star radius as multiple of screen width     |
| `GRAVITY`                  | 0.06  | Downward acceleration / frame               |
| `DRAG_X`                   | 0.97  | Horizontal velocity multiplier / frame      |
| `DRAG_Y`                   | 0.995 | Vertical velocity multiplier / frame        |
| `SURFER_SIZE`              | 18    | Surfer collision radius (px)                |
| `SURFACE_Y_OFFSET`         | 100   | Star surface offset from screen bottom (px) |
| `FLARE_SPAWN_INTERVAL_MIN` | 3     | Min frames between flare spawns             |
| `FLARE_SPAWN_INTERVAL_MAX` | 8     | Max frames between flare spawns             |
| `SURFER_SPEED`             | 0.45  | Horizontal acceleration / frame             |
| `MAX_ALTITUDE`             | 0.92  | Screen-height fraction for altitude warning |
| `BURN_ALTITUDE`            | 30    | Px above surface → death                    |
