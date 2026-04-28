# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Pitch Record Checker (録画記録)** is a mobile-optimized, single-file web application for tracking baseball pitch recording status during games. Built with pure vanilla JavaScript, HTML, and CSS — no dependencies, no build tools.

## Running the App

```bash
# Open directly in browser
open index.html

# Or serve via HTTP server
python3 -m http.server 8000
# → http://localhost:8000/index.html
```

There are no build steps, no package managers, and no test suite.

## Architecture

The entire application lives in `index.html` (~550 lines). Structure within the file:

1. **`<style>`** — All CSS, including CSS variables for the design system (colors, spacing) and iOS-specific optimizations (safe area insets, `-webkit-tap-highlight-color`)
2. **`<body>`** — Static HTML shell: `#app-header`, `#inning-bar`, `#content`, `#footer-actions`, and `#modal-overlay`
3. **`<script>`** — All application logic

### State Model

```javascript
state = {
  innings: [
    {
      id: number,       // Inning number (1-based)
      batters: [
        {
          id: number,             // Batter number (1-based)
          pitches: ['', 'o', 'x'] // '' = unrecorded, 'o' = OK, 'x' = NG
        }
      ]
    }
  ],
  cur: number  // Index of currently displayed inning
}
```

### Rendering Pattern

The app uses a simple re-render-everything approach:
- All user interactions update `state` directly
- `render()` rebuilds the entire DOM from `state` on each interaction
- No virtual DOM, no framework — direct `innerHTML` assignment

### Key Functions

- `render()` — Rebuilds header stats, inning tabs, and batter cards for `state.innings[state.cur]`
- `openModal(inningIdx, batterIdx, pitchIdx)` — Shows the pitch-status modal
- `setPitch(mark)` — Updates a pitch in state and calls `render()`
- `exportCSV()` — Generates and downloads a CSV of all pitch data

### UI Components

- **Stats pills** in header: live counts of ○ (OK), × (NG), and unrecorded pitches across all innings
- **Inning tabs** (`#inning-bar`): horizontal scroll, active inning highlighted in green
- **Batter cards**: show batter number, pitch grid (44×52px cells), per-batter summary, delete button
- **Modal** (`.modal-overlay`): bottom-sheet for setting pitch status (○, ×, or —)
- **Footer**: Add Inning and CSV Export buttons

### Design System

CSS variables defined at `:root`:
- `--ok`: `#2d7a3a` (green) — successfully recorded
- `--ng`: `#c0392b` (red) — failed to record
- `--bg`: `#f5f4f0` (beige) — app background
- Font stack targets Japanese: Hiragino Sans, Apple system fonts
