# Dynamica SmartGate — Theme-park Revenue Management demo

Clickable UI demo of **Dynamica SmartGate**, a revenue-management / dynamic-pricing
platform for theme parks. Everything runs on **mock data generated client-side**
with a seeded RNG — deterministic across reloads, no backend.

> **Keep `JOURNAL.md` up to date.** It is the ongoing record of changes,
> decisions, and assumptions. Append a dated entry every working session.

## Stack
- Pure HTML + CSS + vanilla JS, no framework, no build step, no dependencies.
- Shared design system in `styles.css` (tokens, top nav, drawer, settings, switch).
  Page-specific CSS lives in an inline `<style>` block per page.
- Fonts: Inter + JetBrains Mono via Google Fonts (imported in `styles.css`).
- Accent color: `#e8007d` (Dynamica pink/magenta).

## Pages (plain static files)
- `index.html` — **Pricing** (the main page, ~6k lines, all logic inline):
  Park → Booking Channel accordion table, calendar heatmap view (with an
  "All parks" total row + per-cell metric hover text), filters, sorting, alerts,
  demand levels (DL1–DL5), rules-driven price recommendations,
  accept/reject/override + reason codes + commit flow, row-select checkboxes
  with a bulk price-update modal (% or €), detail modals, notes, RM Copilot mock.
- `autopilot.html` — **Autopilot**: Park → Week → Day accordion over 8 weeks;
  configurable auto-accept rules (thresholds, DL-consistency, peak-day review)
  plus per-park/per-week threshold overrides (⚙ on each row), all persisted in
  `localStorage` (`dynamica.autopilot`).
- `settings.html` — account + alert thresholds (`localStorage` `dynamica.alerts`,
  read by index.html).
- `sso.html` — fake login page (logout redirects here).
- `app.js` — shared burger-drawer/logout wiring only.

## Domain model (mock data)
- 4 parks (Warner Madrid, Movie Park, Mirabilandia, Tropical Islands).
- Second accordion level = **booking channels** (Website / Mobile App / OTA / Gate).
  ⚠️ Internally still named `tickets`/`ticket` — only data + labels were changed.
- All prices are the **Adult rate**; other ticket types are derived via the Rate
  selector multipliers (`RATE_TYPES` in index.html).
- Demo "today" = **2026-07-15**; 30-day pricing horizon from 07-16; holiday week
  2026-07-25 → 07-31. Tropical Islands is indoor (weather effects inverted).
- **Demand Levels** per `rules.png`: forecast-vs-expected ppt → DL1..DL5;
  DL drives the price recommendation (DL3 = no change, most common).
- Data is seeded per key (`rngFor('...' + id + iso)`) — change a seed string and
  every number downstream changes; keep seeds stable unless intentional.

## Key quirks
- `html, body { height:100%; overflow:hidden }` — each page's `<main>` manages
  its own scroll containers (fixed-viewport data-app layout).
- Body font-size defaults to 13px (data-app density); tables use 11.5px.
- The shell (top nav + drawer) is duplicated per page — adding a menu item means
  editing every page's drawer.
- `styles.css` retains dead component CSS from an earlier demo for reuse —
  don't prune it casually.

## Local dev
No build step:
```
python3 -m http.server 8000
```
Then open http://localhost:8000/ (or use the `.claude/launch.json` preview).

## Workflow notes
- Work locally; push to GitHub only when Tyler explicitly says it's ready.
- Claude does not run git operations unless explicitly asked.
- Do not add dependencies without discussion.
- Reference for pricing rules: `rules.png` (demand-level bands + price ladder).
