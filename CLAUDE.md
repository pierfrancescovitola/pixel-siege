# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Pixel Siege: a retro top-down pixel shooter. The whole game is one file, `index.html`, built with HTML5 Canvas and plain JavaScript inside one IIFE. There is no build step, no package manager, no dependencies and no test suite. The only outside resource is the "Press Start 2P" font from Google Fonts, which falls back to monospace.

## Commands

- **Run:** open `index.html` in a browser (on Windows: `Start-Process index.html`). The user plays it this way. Don't publish it as an Artifact unless they ask.
- **Syntax check** (the closest thing to a test):
  ```bash
  node -e "const s=require('fs').readFileSync('index.html','utf8');new (require('vm').Script)(s.slice(s.indexOf('<script>')+8,s.lastIndexOf('</script>')));console.log('OK')"
  ```
- **Runtime check** (catches errors that a syntax check misses, such as crashes on the first frame). Run it headless, then read the screenshot:
  ```powershell
  & "C:\Program Files\Google\Chrome\Application\chrome.exe" --headless=new --disable-gpu --user-data-dir="$env:TEMP\ps-headless\profile" --enable-logging=stderr --v=0 --virtual-time-budget=4000 --screenshot="$env:TEMP\ps-headless\shot.png" --window-size=960,540 "file:///C:/Users/franc/OneDrive/Desktop/Claude%20test/shooter/index.html" 2>&1 | Select-String "CONSOLE|Uncaught"
  ```

## Git workflow (required, every session)

Commit to Git and push to GitHub regularly as you work, so no work or progress is ever lost. The user relies on this history to see where the project stands and to revert changes.

- **Commit as you go**, not only at the end of a task. Make a commit each time a logical piece of work is done and checked: a feature, a fix, a balance change, a docs update. A larger task should become several small commits.
- **Push right after each commit:** `git push` to the private repo `pierfrancescovitola-jpg/pixel-siege` (remote `origin`, branch `main`). Don't leave commits that exist only on this machine.
- **Clean commit messages:** a subject line in the imperative mood of 72 characters or less (for example "Add shotgun weapon tier"), a blank line, then a short body saying what changed and why. End with the `Co-Authored-By` trailer. One topic per commit. Don't commit broken code: run the syntax check (and the runtime check for gameplay changes) first.
- **Before you start work**, run `git status` and commit or ask about any leftover changes, so each commit stays focused.
- **Don't rewrite pushed history:** no `--force`, `reset --hard` or amending pushed commits unless the user asks. To undo something, use `git revert`.
- **Commit identity** is set at repo level to the user's GitHub no-reply address. Leave it as is.

## Architecture

`index.html` is split into sections marked by `/* ===== NAME ===== */` banners, in this order: CONFIG, SAVE, CANVAS, SPRITES, FLOORS, AUDIO, INPUT, GAME STATE, SPAWNING, COMBAT, UPDATE, BUTTONS, MAIN UPDATE, RENDER, LOOP.

- **Data-driven tuning (CONFIG):** almost every gameplay change is an edit to a table, not to logic:
  - `LEVELS`: each level has waves, and each wave is a list of `[type, count, secondsBetween]` groups that spawn at the same time.
  - `TYPES`: enemy stats.
  - `WEAPONS`: unlocked by `kills` thresholds in the run-wide `G.totalKills`.
  - `DROP_CHANCE`, `PICKUP_LIFE`, `MAX_PICKUPS`, `PLAYER_CFG`, `THEMES`: one floor palette per level index.
- **Sprites drawn in code:** sprites are arrays of strings, one character per pixel. `makeSprite(rows, palette, mirror)` draws them to offscreen canvases when the game starts. Rows can have different lengths. `'.'`, and any character not in the palette, is transparent. `whiteOf()` creates the white silhouettes used for hit flashes. The boss is drawn as a left half and mirrored. `TYPES[name].frames` and `.white` get filled in after the sprites are built. Weapon sprites are attached to `WEAPONS[i].sprite` after `GUN*` is defined, and the barrel length is the sprite's width.
- **Single state object `G`** with a string state machine: `menu → playing ⇄ paused → levelclear → playing … → victory`, and `playing → dying → gameover`. Always switch states with `setState()`, which resets `G.stateT` and the menu selection. `startGame()` resets the whole run. `startLevel(i, carryOver)` resets the level and records `levelStartScore` / `levelStartKills` so Retry can restore them.
- **Wave flow (`updateWorld`):** groups count down → `queueSpawn()` adds a telegraph marker to `G.warnings` → when the timer runs out, the enemy spawns just off-screen. A wave is cleared when `remainingEnemies()` (alive + warnings + unspawned) reaches 0. Then `nextWaveT` starts the next wave, or `endT` leads to `finishLevel()`. The boss summons more enemies through `queueSpawn`.
- **Fixed internal resolution:** the game always draws at 480×270, and CSS scales the canvas up with `image-rendering: pixelated`. Mouse positions are converted to game coordinates in `toGame()`. All positions are in game pixels.
- **Menus are drawn on the canvas:** `buildButtons()` returns the buttons for the current state each frame. `handleButtons()` handles mouse hit-testing, and `onPress()` handles the keyboard (Up/Down/Enter). Clicks are ignored for a short time after a state change (`activate()`), so a held click doesn't carry over into the next screen.
- **Loop:** `requestAnimationFrame` with `dt` limited to `[0, 0.05]`. The lower limit matters, because the first frame's timestamp can come before the script's start time. A negative `dt` once crashed the menu on its first frame.
- **Persistence:** `localStorage` key `pixel-siege-save` stores `{unlocked, best, muted}`. Every access is wrapped in try/catch.
- **Audio:** `SFX.*` functions synthesize sounds with Web Audio (`tone()`, `noise()`). The audio context is created on the first click or key press.
