# AGENTS.md

## Project Overview

**Asteroids** - A canvas-based arcade game clone (pure HTML5/JS, no frameworks).

**Working directory naming is misleading** - this is NOT a weather app; it's a classic Asteroids clone.

## Architecture

- **Single HTML entry point**: `index.html` loads `game.js`
- **Game logic concentrated**: All game mechanics live in `game.js` (~423 lines)
- **Canvas rendering**: `<canvas id="canvas" width="800" height="600"></canvas>`
- **No build tools, no bundlers**: Direct browser execution
- **Modular classes**: `Bullet`, `Asteroid`, `Ship`, `Particle` with clear responsibilities

## Running the Game

```bash
# Direct browser opening
open index.html  # macOS
# or double-click index.html in a file explorer

# Via local server (recommended)
npx serve .
# Then visit http://localhost:3000
```

**No install steps, no dependencies, no dev server required.**

## Core Game Loop

```javascript
function loop(ts) {
  const dt = lastTime === null ? 0 : Math.min((ts - lastTime) / 1000, 0.05);
  lastTime = ts;
  update(dt);
  draw();
  requestAnimationFrame(loop);
}
```

Uses `requestAnimationFrame` with delta time (`dt`) for smooth frame-rate independent physics.

## Key Game Constants

Defined in `game.js`:

| Entity | Constants | Values |
|--------|-----------|--------|
| Ship | `ROT` (rotation speed) | 3.5 rad/s |
| Ship | `THRUST` (acceleration) | 260 px/s² |
| Ship | `DRAG` | 0.987 (velocity decay) |
| Ship | `SHOOT_COOLDOWN` | 0.2s |
| Bullet | `SPEED` | 520 px/s |
| Bullet | `TTL` (lifetime) | 1.1s |
| Asteroid | `RADIOS` (radii) | [0, 16, 30, 50] |
| Asteroid | `SPEEDS` (base speed) | [0, 85, 55, 32] |
| Asteroid | `POINTS` | [0, 100, 50, 20] |

## Important Code Patterns

### Input Handling
- Uses `keys` object for active key states
- Uses `justPressed` object to detect key-down events once per press
- Prevents default behavior for game keys (Arrow keys, Space)

### Space Wrapping
All coordinates use modulo wrapping (`wrap()`) for toroidal space effect:
```javascript
const wrap = (v, max) => ((v % max) + max) % max;
this.x = wrap(this.x + this.vx * dt, W);
```

### Collision Detection
- Bullet-to-asteroid: `dist(b, a) < a.radius`
- Ship-to-asteroid: `dist(ship, a) < ship.radius + a.radius * 0.82` (slightly forgiving)

### Level Progression
Each level adds more asteroids: `spawnAsteroids(3 + level)` on `nextLevel()`

## Extending the Game

To add features without breaking the game loop:

1. Add new properties to existing classes in `game.js`
2. Add new classes for complex entities (power-ups, enemies, etc.)
3. Add state variables (e.g., `powerUps`, `enemyScoreMultiplier`)
4. Update `update(dt)` to handle new logic
5. Add new properties to `Ship` if needed (e.g., power-up state)
6. Update `draw()` and HUD rendering as needed

**Do not alter the main loop structure**—keep the same `requestAnimationFrame` pattern.

## Testing Strategy

No automated tests exist. Manual verification:

- Open `index.html` in browser
- Play through game states: playing → dead → playing → game over → restart
- Verify:
  - Space wrapping works at all edges
  - Asteroid splitting (large → medium → small)
  - Invincibility frames (ship blinks when respawning)
  - Score updates correctly based on asteroid size
  - Game over screen shows final score and restart instructions

## Known Design Decisions

- **No audio**: No sound effects or music implemented
- **No power-ups**: README mentions power-ups as feature ideas, none exist
- **No difficulty curve beyond level count**: Asteroid count increases, speed is fixed
- **No high score persistence**: Scores reset on page reload
- **Single canvas**: No separate UI layers or HTML overlays