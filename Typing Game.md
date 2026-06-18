# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single self-contained file, `typing-game.html` — "Type Racer", a browser typing-speed
game. There is **no build system, no dependencies, no package manager, and no tests**.
All HTML, CSS, and JavaScript live inline in the one file.

## Running / verifying

Open `typing-game.html` directly in a browser (double-click or the editor's preview panel).
There is no dev server to start. Note: this Windows environment has **no working Node or
Python runtime** (the `python` on PATH is the Microsoft Store stub), so the game cannot be
run or driven headlessly here — verify changes by reading the code and/or in a real browser.

## Architecture (one IIFE in the `<script>` tag)

The whole app is a single IIFE. Key pieces, in dependency order:

- **Three "screens"** (`#startScreen`, `#gameScreen`, `#resultScreen`) are `<section>`s toggled
  by the `.active` class via `show(name)`. Only one is visible at a time.
- **`WORDS`** — three difficulty tiers (1/2/3). `pickWord()` selects the tier from
  `state.frac` (fraction of the round elapsed), so difficulty ramps within a single round.
- **`store`** — a localStorage wrapper with a try/catch + in-memory (`mem`) fallback.
  **Always read/write persistence through `store.get`/`store.set`, never raw `localStorage`** —
  sandboxed preview iframes throw `SecurityError` on `localStorage`, and a raw call at init
  time crashes the whole IIFE before event handlers are wired (this was a real bug).
- **`state`** (from `freshState()`) — per-round data: current word, typed index, correct/total
  keystrokes, combo, score, timing. Recreated on each `startGame()`.
- **`scene`** — all racing-visual state. The car's apparent speed is **derived from recent
  typing rate**: `noteType()` pushes a timestamp on each correct keystroke; `sceneTick()`
  computes chars/sec over a ~1.4s window, maps it to a `target` speed, and smooths `scene.speed`
  toward it each frame. Speed then drives background-position scroll of road/hills/clouds,
  wheel rotation, car tilt, distance, speed-lines opacity, and exhaust puffs.
- **`tick()`** — the single `requestAnimationFrame` loop (started in `startGame`, cancelled in
  `endGame`/`quitToMenu`). Computes `dt` from frame timestamps, updates the countdown and live
  stats, then calls `sceneTick(now, dt)`. WPM = `(correctChars/5) / minutes`.
- **`onKey(e)`** — per-keystroke validation. Wired via a single `document` keydown listener that
  routes to `onKey` when the game screen is active, else handles Enter-to-start. Correct keys
  advance the word; a wrong key zeroes the combo. Completing a word every 5th in a row triggers
  NITRO (`triggerNitro`, flames, particles, boost).

## Conventions / gotchas

- The `scene` object holds **both** a numeric `dist` and a DOM element reference `distEl` —
  keep DOM refs and numeric state under distinct keys (an earlier `dist`/`dist` collision was a bug).
- The car and speedometer are inline SVG. Wheels are rotated by setting the `transform`
  attribute (`translate(...) rotate(deg)`) on `#wheelRear`/`#wheelFront` each frame, drawn
  centered at 0,0 (`wheelMarkup()` + `REAR`/`FRONT` coordinates).
- Sound is generated with WebAudio oscillators (`beep()`), gated by `soundOn`; no audio assets.
- Colors are CSS custom properties on `:root` (synthwave palette: `--cyan`, `--magenta`,
  `--gold`, `--green`, `--red`, `--orange`). Reuse these rather than hardcoding hex values.
