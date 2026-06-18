# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A workspace of **self-contained, single-file HTML apps**. Each app is one `.html` file with all
HTML, CSS, and JavaScript inline — **no build system, no dependencies, no package manager, no
tests, no framework**. Apps are independent of each other; there is no shared code.

Current apps:
- `typing-game.html` — "Type Racer", a browser typing-speed game. Documented in `Typing Game.md`.
- `meeting-action-board.html` — a Kanban board (To Do / In Progress / Done) for meeting action
  items, seeded from Granola meeting notes.

Some apps have a companion `*.md` doc (e.g. `Typing Game.md`) holding app-specific architecture
notes. When working on an app, read its companion doc first if one exists.

## Running / verifying

Open the `.html` file directly in a browser (double-click, or the editor's Launch preview panel,
which auto-refreshes on save). There is no dev server.

This Windows environment has **no working Node or Python runtime** — the `python` on PATH is the
Microsoft Store stub, and `node`/`npx` are absent. So apps **cannot be run, served, or driven
headlessly here**, and the preview-server MCP can't be started. Verify changes by reading the
code carefully and relying on the live Launch preview panel; treat behavioral testing as
something the user does in a real browser.

## Conventions across apps

- **Persistence:** state is kept in `localStorage` under a single versioned key (e.g.
  `meeting-action-board:v1`). Sandboxed preview iframes can throw `SecurityError` on
  `localStorage` — guard every access in try/catch. In `typing-game.html` this is centralized in
  a `store` wrapper with an in-memory fallback; a raw `localStorage` call at init time crashes the
  whole script before handlers wire up. Prefer the same pattern.
- **Theming:** colors are CSS custom properties on `:root`. Reuse the existing variables rather
  than hardcoding hex values; each app has its own palette.
- **No escaping shortcuts:** user/meeting text is rendered into the DOM. Set free text via
  `textContent`, and run any string interpolated into `innerHTML` through the local `esc()` helper.
- **Structure:** vanilla JS only. Logic lives in one `<script>` block (an IIFE in the typing game;
  a top-level `"use strict"` script in the Kanban app). Keep it that way — don't introduce modules,
  bundlers, or external CDNs.

## Granola integration (meeting-action-board.html)

The board's source of truth is the connected **Granola MCP**. A standalone browser page cannot
call MCP tools, so the data flow is **Claude-side**: Claude fetches meetings via the Granola MCP
(`list_meetings` / `query_granola_meetings` / `get_meetings`), extracts action items per meeting,
and regenerates the `seedBoard()` block in the HTML. One card = one meeting; action items are the
card's checklist. To "refresh from Granola", re-seed `seedBoard()` and the user clicks "Reset to
samples". If the Granola MCP returns "user has not created a Granola account yet", the board falls
back to its built-in sample data (an in-app banner explains the sign-up at granola.ai/mcp-signup).
