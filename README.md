# Pixel Siege

A retro top-down pixel shooter that runs in the browser. Survive 5 levels of slimes, bats, golems and eyeballs, then take down the boss.

## Play

Open `index.html` in any modern browser. No build step or install needed.

| Control | Action |
|---|---|
| Arrow keys / WASD | Move |
| Mouse | Aim |
| Click / hold | Shoot |
| P / Esc | Pause |

## Features

- 5 levels with waves of enemies spawning from every edge, plus a boss fight
- Weapon upgrades by kill count: Pistol → SMG (15) → Shotgun (45) → Laser Rifle (90)
- Hearts that enemies sometimes drop; walk over them for +1 life (up to 8)
- Pixel-art sprites drawn in code, synthesized sound effects, saved progress

## Tuning

All the gameplay numbers are near the top of the script in `index.html`: `LEVELS`, `TYPES`, `WEAPONS`, `DROP_CHANCE`, `PICKUP_LIFE`.
