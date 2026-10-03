# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a zero-dependency, single-file tier list maker. The entire application lives in `index.html` — there is no build system, no package manager, and no separate JS/CSS files.

**Only external dependency:** `html2canvas` v1.4.1 loaded via CDN (used for PNG export).

## Running the App

Open `index.html` directly in a browser. No build step or `npm install` required. `.claude/launch.json` defines a `static` preview server (`python3 -m http.server 8791`) — use it for browser testing, since IndexedDB needs a real origin.

## Architecture

Everything is in `index.html`:
- **Lines 9–326:** Embedded CSS with CSS custom properties for theming
- **Lines 328–379:** HTML structure (tier container, item pool, upload area, modals, lightbox)
- **Lines 380–1104:** Vanilla JavaScript

### State Model

Two top-level state containers:
- `tiers` — array of `{ name, color, items[] }` objects
- `window._poolItems` — array of unranked items

Each item is `{ id, src, name }` — `src` is an image data URL, `name` is the lowercased filename (used for A→Z sorting). There are no text items yet.

### Rendering

State → DOM is one-directional: mutate state, then call `render()` (rebuilds tier rows) and/or `renderPool()` (rebuilds the pool). There is no reactivity layer.

### Drag and Drop

Implemented manually via mouse and touch events: `startDrag()` → `moveDrag()` → `endDrag()`. A floating clone element follows the cursor and a `.drop-marker` shows the insertion point; `getInsertPosition()` works out the index from pointer position, and `endDrag()` calls `moveItemToTier()` with the target tier (or `'pool'`) and that index. A click without movement opens the lightbox. Drag handlers ignore presses on the `.delete-item` button.

### Key Functions

| Function | Purpose |
|---|---|
| `init()` | Load saved state (or default tiers) and initial render |
| `render()` | Rebuild all tier rows from `tiers` state |
| `renderPool()` | Rebuild unranked item pool |
| `createItemEl(item)` | Factory: returns a DOM element for an item |
| `moveItemToTier(id, target, index)` | Move item to a tier index or `'pool'`, at `index` (end if omitted) |
| `handleFiles(files)` | Read uploads, shrink via `downscaleImage()` (max 800px), add to pool in order |
| `exportAsImage()` | Uses html2canvas to download a PNG |

### Persistence

State (`tiers`, pool items, `itemIdCounter`) is saved to IndexedDB (database `tierlist`, key `current`). `render()` and `renderPool()` both call `scheduleSave()` (300ms debounce), so any mutation followed by a render is persisted automatically. Saving is disabled until `init()` finishes loading, so defaults never overwrite saved data.
