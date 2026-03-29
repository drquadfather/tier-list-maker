# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a zero-dependency, single-file tier list maker. The entire application lives in `index.html` — there is no build system, no package manager, and no separate JS/CSS files.

**Only external dependency:** `html2canvas` v1.4.1 loaded via CDN (used for PNG export).

## Running the App

Open `index.html` directly in a browser. No server, build step, or `npm install` required.

## Architecture

Everything is in `index.html`:
- **Lines 9–283:** Embedded CSS with CSS custom properties for theming
- **Lines 286–331:** HTML structure (toolbar, tier container, item pool, modals)
- **Lines 333–844:** Vanilla JavaScript

### State Model

Two top-level state containers:
- `tiers` — array of `{ name, color, items[] }` objects
- `window._poolItems` — array of unranked items

Each item has `{ id, type, content }` where `type` is `"text"` or `"image"`.

### Rendering

State → DOM is one-directional: mutate state, then call `render()` (rebuilds tier rows) and/or `renderPool()` (rebuilds the pool). There is no reactivity layer.

### Drag and Drop

Implemented manually via mouse and touch events: `startDrag()` → `moveDrag()` → `endDrag()`. A floating clone element follows the cursor; `endDrag()` calls `moveItemToTier()` or drops back to pool based on drop target.

### Key Functions

| Function | Purpose |
|---|---|
| `init()` | Bootstrap default tiers and initial render |
| `render()` | Rebuild all tier rows from `tiers` state |
| `renderPool()` | Rebuild unranked item pool |
| `createItemEl(item)` | Factory: returns a DOM element for an item |
| `moveItemToTier(id, tierIndex)` | Move item between tiers or pool |
| `exportAsImage()` | Uses html2canvas to download a PNG |

### No Persistence

Data is in-memory only. Page refresh clears everything.
