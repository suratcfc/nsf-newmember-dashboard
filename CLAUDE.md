# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single self-contained HTML file (`index.html`) implementing a Thai-language dashboard that tracks progress of the National Savings Fund's (กอช. — Government Pension Fund / National Savings Fund) "New Member Drive" campaign for July–December 2026 (target: 146,000 new members across 7 programs). There is no build step, no dependencies, and no package manager — open `index.html` directly in a browser to view/develop it.

## Commands

There is no build, lint, or test tooling in this repo. To develop:

```bash
open index.html   # or double-click in Finder — works directly, no server needed
```

There's nothing to install, compile, or run beyond opening the file.

## Architecture

Everything lives in `index.html`, structured top to bottom as:

1. **`<style>` block** — all CSS, using custom properties defined in `:root` (colors prefixed `--p1`…`--p7` per-project, `--ink`/`--paper`/`--gold` for the theme). Font is Kanit (Google Fonts, Thai-friendly).
2. **`<body>`** — static markup with empty containers (`#project-grid`, `#bars`, `#trendchart`, `#weekly-wrap`, etc.) that JS fills in on load. Text elements use `data-i18n`/`data-i18n-html` attributes for the TH/EN language toggle rather than being hardcoded.
3. **Two inline `data:image/png;base64,...` logos** embedded directly in `<img>` tags (these produce the very long lines ~188 and ~329 — expected, not corruption).
4. **`<script>` block at the bottom** — all logic, no external JS libraries. Charts (`paintBars`, `paintTrend`) are hand-built inline SVG strings, not a charting library.

### The one section meant to be edited regularly

Near the top of the `<script>` block is a clearly delimited data block (comment: `บล็อกข้อมูล — แก้ตรงนี้จุดเดียวพอ` / "edit only here"):

- **`CONFIG`** — `totalTarget`, campaign `startDate`/`endDate`, and `asOf` (the as-of date driving pace/gap calculations).
- **`PROJECTS`** — the 7 fixed programs (`id`, `target`, `color`, and `th`/`en` name+description pairs). Structural changes to programs go here.
- **`WEEKLY`** — the actual data-entry log. One object per week: `{ week, end, v:{ p1:…, p2:…, …, p7:… } }`, where each `v[pid]` is the **incremental** count added that week (not cumulative). This array starts empty and is appended to as real data comes in — this is the primary ongoing maintenance task in this repo.

Everything below that block (`I18N` strings, date helpers, derived totals, and the `paint*` render functions) is display logic driven by the three structures above and shouldn't need to change for routine data updates.

### Rendering flow

`applyLang()` (called once on load, and again on TH/EN toggle) drives the whole render: it swaps `data-i18n` text, then calls `paintHero`, `paintCards`, `paintBars`, `paintTrend`, `paintTable` in sequence. All derived numbers (`actualBy`, `actualTotal`, `paceNow`, `gap`, `daysElapsed`, etc.) are computed once at script top-level from `CONFIG`/`PROJECTS`/`WEEKLY` and reused across the paint functions — if you add a new derived metric, compute it up there rather than inside a paint function.
