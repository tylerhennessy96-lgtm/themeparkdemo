# Dynamica — Marketing/Demo Website

Marketing/demo website for **Dynamica**, a B2B revenue management platform.
Repurposed from a prior internal data-app demo; the design system was preserved
verbatim during the strip-back so we can build out new content on top of it.

## Stack
- Pure HTML + CSS, no framework, no build step, no JS dependencies.
- All shared styles in `styles.css`.
- Fonts: Inter + JetBrains Mono via Google Fonts (imported at the top of `styles.css`).
- Accent color: `#e8007d` (Dynamica pink/magenta).

## Local path
`/Users/tylerhennessy/logdemo/`

## Routing
Plain static files. `index.html` IS the Pricing page (no redirect). When more
pages are added later, each becomes its own `.html` file.

## Pages
- `index.html` — Pricing (placeholder; just `<h1>Pricing</h1>` for now).

## Persistent header
A dark header bar appears on every page, defined inline per-page (no shared
templating since there is no build step). Markup pattern lives in `index.html`:
- `<nav class="topnav">` styled by `.topnav` in `styles.css` (`--nav-bg`, 48px, 1px bottom border).
- Logo block on the left, made up of:
  - The pink/magenta diamond image: `engine_logo (2).png` (28px tall).
  - "DYNAMICA" wordmark: Inter 11px / 600 / uppercase / 0.18em letter-spacing / white.
  - "SmartRents" sub-label underneath: Inter 12px / 400 / 0.02em / `rgba(255,255,255,0.55)`.
- **Currently nothing else** — no nav tabs, no divider, no hamburger menu. These
  may return when the IA is decided.

The logo block uses inline styles (matches the original demo) so the rendering
is locked in regardless of any CSS changes.

## Design system (`styles.css`)
**Do not edit lightly.** This file holds all design tokens, fonts, the header
styling, and a large amount of component CSS from the previous demo. The dead
component CSS (filter bars, data tables, modals, etc.) is intentionally retained
so we can reuse pieces as we build new pages. Key tokens in `:root`:
- `--nav-bg: #1a1a2e`
- `--nav-border: #2d2d44`
- `--accent: #e8007d`
- `--accent-dim: #c2185b`
- `--body-bg: #f5f6f8`
- `--surface: #ffffff`
- `--border: #e4e6eb`
- `--text-primary / --text-secondary / --text-muted`
- `--radius: 6px`, `--radius-lg: 10px`
- `--shadow-sm / --shadow-md / --shadow-lg`

## Known quirks to revisit later
- `html, body { height: 100%; overflow: hidden }` and `.app { height: 100vh; overflow: hidden }`
  are tuned for fixed-viewport data apps. For a scrolling marketing site we will
  likely want to relax these. The current `index.html` works around it locally
  by giving `<main>` `flex:1; overflow:auto`.
- Body font-size defaults to 13px (data-app density). Marketing copy may want
  larger sizing per page or via a future override.

## Local dev
No build step. From the project directory:
```
python3 -m http.server 8000
```
Then open http://localhost:8000/

## Workflow notes
- Claude does not run git operations unless explicitly asked.
- Do not add new dependencies without discussion.
