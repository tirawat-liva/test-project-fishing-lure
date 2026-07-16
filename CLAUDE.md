# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A single-file static website (`index.html`) that visualizes the underwater action of 4 fishing lure types (Plug, Popping, Speed Jigging, Slow Jigging) with self-contained canvas animations. No build system, no dependencies, no package manager — all CSS and JavaScript live inline in `index.html`.

## Development Commands

There is no build, lint, or test step. To verify changes render correctly, take a headless screenshot with the pre-installed Chromium:

```bash
/opt/pw-browsers/chromium --headless=new --no-sandbox --disable-gpu \
  --virtual-time-budget=2200 --window-size=1280,1350 \
  --screenshot=/tmp/preview.png "file:///home/user/test-project-fishing-lure/index.html"
```

Append `?lang=en` to the URL to verify the English mode.

## Architecture of index.html

The page is bilingual (Thai default / English) and driven by one IIFE in the `<script>` block, organized as:

- **i18n**: an `I18N` dictionary (`th` / `en`) keyed by strings like `plug.desc`. Elements carry `data-i18n` attributes; `applyLang()` swaps `innerHTML` from the dictionary. The `#lang-toggle` button flips `uiLang`, and `?lang=en` in the URL preselects English. Values may contain HTML (`<b>`). **Any new visible text needs both a `data-i18n` key in the markup and entries in both dictionary languages.** Canvas-drawn text (e.g. the popper's "ป๊อก!"/"POP!") is translated inline via `uiLang` checks, not the dictionary.
- **Panel engine**: `makePanel(canvasId, sim)` registers each card's canvas into `panels[]` with its own clock (`panel.t`), pause flag (click toggles), particle list, and `mem` scratch object. A single `requestAnimationFrame` loop clears each canvas and calls its `sim` function every frame. DPI scaling and resize are handled in `panel.resize()`.
- **Sim functions** (`simPlug`, `simPop`, `simSpeed`, `simSlow`): one per lure, each draws the scene (`drawWater`), computes the lure's position/angle from `panel.t` with phase-based cycles (`t % P`), draws the fishing line (`drawFishingLine`) and the lure shape (`drawPlugLure` / `drawPopper` / `drawJig`), and spawns particles (`spawn`) for bubbles/splashes/glints.
- **Lure shapes** are drawn in local coordinates with the line-tie point on the negative-x side, then translated/rotated — sims compute the line endpoint from the lure's angle using that offset.

## Conventions

- The footer's "Last updated" line is baked into the HTML and must match the commit time. When changing `index.html`, update it to the current UTC time in the format `Last updated : By Tirawat Liva, 16 July 2026 - 09:28:44 (UTC)` and set `GIT_AUTHOR_DATE`/`GIT_COMMITTER_DATE` to the same instant when committing.
- Keep the page fully self-contained: no external scripts, stylesheets, fonts, or images.
