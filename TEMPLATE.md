# Dynamica Demo Template

A reusable template for building B2B revenue-management demos. The current
instance is the logistics version (lanes / freight pricing). The same skeleton
and design system can be adapted to hotels, airlines, car rentals, retail, or
any industry where someone reviews and acts on per-unit, per-period price
recommendations.

---

## 1. What this project is

A single-tenant demo of a B2B revenue-management platform. The user lands on a
data-dense Pricing page that lists every entity-period combination the system
has a recommendation for, with bookings/forecast/price metrics arranged in
grouped column bands. They can switch to a Calendar view (heatmap of YoY
revenue change), filter by category, drill into any cell or row for a detail
modal (booking curve + price curve + recommendation panel), and act on each
recommendation (Accept / Reject / Override / Note). A bulk **Accept all** and a
**Commit** flow push the pending decisions live. **Autopilot** auto-accepts
recommendations within user-configured tolerance windows on page load. Inline
**Alerts** flag rows that hit configurable thresholds (utilisation,
price-drop, demand-spike, etc.). All state is in-memory plus a small amount of
`localStorage` for user preferences — there is no backend.

---

## 2. Design system reference

**Philosophy.** Soft pastel data colors (greens for positive, reds/oranges for
negative) so a dense table reads at a glance without shouting. A single strong
pink/magenta accent for primary actions, focus rings, and active states. Dark
surfaces (`#1a1a2e` / `#1e2028`) for *transient* UI — top nav, drawer header,
date-range popovers, tooltips — so they read as floating layers against the
light app body. Inter for everything; 13px body font sizing for data-app
density.

**Copy-pasteable token block** (already in `styles.css`):

```css
:root {
  /* Dark surfaces (nav, popovers, tooltips) */
  --nav-bg:         #1a1a2e;
  --nav-border:     #2d2d44;
  --pill-bg:        #1e2028;

  /* Primary accent (Dynamica pink/magenta) */
  --accent:         #e8007d;
  --accent-dim:     #c2185b;
  --accent-text:    #e8007d;

  /* Status accents */
  --warn:           #f5a623;
  --danger:         #e05252;

  /* Light app body */
  --body-bg:        #f5f6f8;
  --surface:        #ffffff;
  --border:         #e4e6eb;
  --border-mid:     #d0d3da;

  /* Text scale */
  --text-primary:   #1a1d23;
  --text-secondary: #6b7280;
  --text-muted:     #9ca3af;

  /* Row interactions */
  --row-hover:      #f0f4ff;
  --row-selected:   #eef2ff;

  /* Radius scale */
  --radius:         6px;
  --radius-lg:      10px;

  /* Shadow scale */
  --shadow-sm:      0 1px 3px rgba(0,0,0,.08);
  --shadow-md:      0 4px 16px rgba(0,0,0,.12);
  --shadow-lg:      0 8px 32px rgba(0,0,0,.18);
}
```

**Typography.** Inter (400/500/600/700) and JetBrains Mono (400/500) from
Google Fonts, imported at the top of `styles.css`. Body text 13px. Section
titles 14–22px / 600. Table column labels 11px / 600 uppercase with a 10px
sub-label below. Action badges 9.5px / 700 uppercase.

**Conditional cell scheme — table (List view).**
Two-state cell tint, applied per cell on YoY columns and the Rec. price cell.

```css
table.srt tbody td.cell-pos { background: #dcfce7; color: #166534; } /* green */
table.srt tbody td.cell-neg { background: #fee2e2; color: #991b1b; } /* red   */
table.srt tbody tr:hover td.cell-pos { background-color: #c9f0d4; }
table.srt tbody tr:hover td.cell-neg { background-color: #fbcfcf; }
```

**Conditional cell scheme — Calendar heatmap.**
Five-tier scale on YoY revenue change (`tier()` in `index.html`):

```css
td.tier-pos2 { background: #bbf7d0; }  /* ≥ +10%  */
td.tier-pos1 { background: #dcfce7; }  /* ≥  +3%  */
td.tier-zero { background: var(--surface); } /* ±3%   */
td.tier-neg1 { background: #fee2e2; }  /* ≤  -3%  */
td.tier-neg2 { background: #fecaca; }  /* ≤ -10%  */
```

**Row-state tints.** Layered *under* cell-pos/cell-neg so YoY colors still
read. Use `:not(.cell-pos):not(.cell-neg)` when tinting a row:

```css
tr.row-accepted   td:not(.cell-pos):not(.cell-neg) { background: #f1faf3; }
tr.row-rejected   td:not(.cell-pos):not(.cell-neg) { background: #fdf3f3; }
tr.row-overridden td:not(.cell-pos):not(.cell-neg) { background: #f3f4ff; }
```

**Action badges (chip palette).**
```css
.action-badge.b-accepted   { background: #dcfce7; color: #166534; }
.action-badge.b-rejected   { background: #fee2e2; color: #991b1b; }
.action-badge.b-overridden { background: #e0e7ff; color: #3730a3; }
.action-badge.b-autopilot  { background: #fce7f3; color: var(--accent); }
```

**Alert chip palette** (4 single-letter chips in Lane cell):
- Low utilisation `L` — `#ea580c`
- High utilisation `H` — `#2563eb`
- Price drop `P` — `#dc2626`
- Demand spike `D` — `#16a34a`

**Dark popover pattern** (date picker, tooltip, range picker).
Background `var(--nav-bg)` (`#1a1a2e`) or `#1e2028`, border `var(--nav-border)`
(`#2d2d44`), white-ish text (`rgba(255,255,255,0.85)`), `border-radius: 10–12px`,
`box-shadow: var(--shadow-lg)`. **Positioned `fixed` and portaled to
`document.body`** so no ancestor's `overflow:hidden` or stacking context can
trap it. `z-index: 200` for date popover, `1100` for the range picker (above
modals at 500).

---

## 3. The page architecture

- **`index.html`** — the main data page. Holds the Pricing header row
  (heading + filters + Accept all + Commit + List/Calendar toggle), the List
  view (one wide sortable table with grouped column bands and an action
  column), the Calendar view (route × period heatmap with monthly/daily
  toggle), and inline `<script>` and `<style>` blocks for everything page-
  specific (rows data, sort, filters, modal, note modal, autopilot, alerts).
  Rename the page heading per industry; keep the structure.
- **`settings.html`** — user settings. Sidebar with **Account**, **Autopilot**,
  **Alerts** panes. Persists to `localStorage` keys `dynamica.autopilot` and
  `dynamica.alerts`. Supports `#autopilot` / `#alerts` URL-hash deep links.
- **`sso.html`** — fake corporate sign-in (e.g. SAML/Google SSO). Pure cosmetic
  — click any provider to redirect into the app. Used by `app.js` "Log out".
- **`app.js`** — shared drawer logic (open/close, ESC, backdrop, focus
  management) and the "Log out → fade overlay → SSO" handoff. Loaded by every
  page.
- **`styles.css`** — design tokens (`:root`), shared component styles (top nav,
  drawer, settings, multi-select popover, range picker, toast, switch,
  alert-config rows). Most page-specific CSS lives inline inside `index.html`;
  shared scaffolding lives here.
- **`engine_logo (2).png`** — the pink/magenta diamond logo mark. Swap or
  re-tint per industry but keep the visual placement.
- **`CLAUDE.md`** — project-specific notes for future Claude conversations.

No build step. Open with `python3 -m http.server 8000` and visit `/`.

---

## 4. The core domain model — abstracted

Every demo built from this template fits the same shape:

| Concept | Definition |
|---|---|
| **Entity** | The thing being priced. Has a name (display string), an identifier, and 0–N category attributes used for filtering. |
| **Period** | The time dimension. Has a label (e.g. "10/24", "Mar-26") and a chronological position. |
| **Metrics (per Entity × Period)** | Three groups: **actuals to date** (booked/sold/occupied), **forecast** (projected for the period), and **price recommendation** (current price, last-year price, recommended price, recommended action). Each metric may have a YoY comparison field used for `cell-pos`/`cell-neg` coloring. |
| **User actions (per Entity × Period)** | Accept, Reject, Override price, Add note (with expiry date). Composed into a single `rowStates[key]` object. |

**Current project (logistics):**
- Entity = **Lane** = `Origin → Destination` (with `origin`, `dest`, `cap` attributes)
- Period = **Date** (`10/24`, `Fri`)
- Metrics = capacity (LDM), units booked, utilisation %, price/LDM, revenue + YoY deltas for each
- Filter categories = Origin, Destination, Lane
- Actions = Accept / Reject / Override / Note

**Hotel mapping:**
- Entity = **Room type** = `Standard King`, `Deluxe Suite`, `Ocean View Queen` (with `room_class`, `bed_type`, `view` attributes)
- Period = **Stay date** (`Sat 03/14`)
- Metrics = available rooms, on the books (OTB), occupancy %, ADR ($), RevPAR ($), revenue + YoY
- Filter categories = Property, Room class, Channel
- Actions = unchanged

**Airline mapping:**
- Entity = **Flight** = `AA1234 LAX→JFK` *or* `Fare class Y` on a flight (with `route`, `equipment`, `cabin` attributes)
- Period = **Departure date** (or days-before-departure bucket)
- Metrics = seats available, seats sold, load factor %, yield ($), revenue + YoY
- Filter categories = Origin, Destination, Cabin
- Actions = unchanged

**Car rental mapping:**
- Entity = **Vehicle class** at a **location** (Economy at LAX, Full-size at JFK)
- Period = **Pickup date**
- Metrics = fleet available, on rent, utilisation %, daily rate ($), revenue + YoY
- Actions = unchanged

**Retail mapping:**
- Entity = **SKU** (with `category`, `brand`, `store` attributes)
- Period = **Week** (or day)
- Metrics = on-hand units, units sold, sell-through %, price ($), revenue + YoY
- Actions = unchanged

What changes per industry: entity name, period label, metric labels, sample
data values, filter dimensions. What stays constant: the shape (Entity ×
Period grid), the action model, the four metric groups (actuals / forecast /
price / YoY), and the autopilot + alerts behaviour.

---

## 5. Feature inventory

### Data display
- **List view** — wide sortable table; grouped header bands (Product Details /
  Bookings to date / Forecast / Price recommendations / Action); per-cell
  conditional colors on YoY columns and the Rec. price cell; click any
  sortable column header to cycle asc → desc → off; closed/inactive rows
  render greyed-out with dashes.
- **Calendar view** — Entity × Period heatmap; row = entity, column =
  period; 5-tier YoY revenue colour scale; toggle between Monthly and Daily
  granularity; sortable Route header; respects current filters and date
  range.

### Filtering
- **Multi-select searchable filters** — pill triggers in the header, click
  opens a dark popover with a search field and checkbox list. Selections
  apply immediately (no Apply button). Trigger shows the selection count.
- **Dual-month dark date-range popover** — `.dp-popover`, portaled to
  `document.body`, click first date to start range, second to end. Hover
  preview. Pre-set quick ranges in the footer.
- **Clear filters** — appears when any filter is active.

### Actions
- **Action column** — Accept (✓), Reject (✗), Notes (📝) buttons per row;
  Accept/Reject toggle, mutually-dim the other when one is active.
- **Inline price editing** — click the Rec. price cell, type a new number,
  Enter or blur to commit; sets state to `overridden`.
- **Notes modal with expiry** — textarea + optional expiry date; auto-saves
  on blur; expired notes show an "Expired" badge in the action column.

### Detail view
- **Detail modal** — opens on row click (List view) *and* calendar-cell
  click (Calendar view). Same modal, same data shape. Contains a
  booking-curve chart + price-curve chart (with hover crosshair and
  tooltip), a comparison table (Current / Forecast / Last year), and a
  recommendation panel with override input. Chart data is deterministically
  seeded from `entity|period` so reopening the same cell shows identical
  numbers.

### Bulk operations
- **Accept all (N)** — outlined pink button beside Commit. Accepts every
  visible Pending row (respects current filters + sort). Disabled when no
  eligible rows. Shows a toast.
- **Commit (N)** — primary pink button. Spinner during a fake 1.2s commit,
  then resets pending rowStates to a clean slate, re-renders, and shows a
  toast.

### Automation
- **Autopilot** — in Settings; on Pricing page load, auto-accepts any Pending
  row whose `(rec − current) / current * 100` falls inside the
  +increase / −decrease tolerance windows. Tagged with an `Autopilot` badge.
  User-driven Accept/Reject/Override clears the auto flag.

### Monitoring
- **Inline row alerts** — small coloured letter chips (L / H / P / D) in the
  Lane cell when an alert fires. Four types: low utilisation, high
  utilisation, price drop, demand spike. Each has a per-type toggle and `%`
  threshold in Settings > Alerts. Native `title` tooltip explains the alert.

### Navigation
- **Burger drawer** — left slide-out (in `app.js`); Pricing / Settings / Log
  out menu items. Closes on backdrop click or ESC.
- **Settings page** — sidebar with Account / Autopilot / Alerts panes; URL
  hash deep links (`#autopilot`, `#alerts`); per-pane Save button shows a
  toast.
- **SSO flow** — clicking Log out fades to an overlay and redirects to
  `sso.html`; SSO landing has provider buttons that redirect back to
  `index.html`.

### State model
- **`rowStates`** — single in-memory object keyed by `${entity}|${period}`
  (e.g. `Chicago, IL → Atlanta, GA|10/24`). Value: `{ state?:
  'accepted'|'rejected'|'overridden', overrideValue?: number, note?: string,
  noteExpiry?: ISO date, autoAccepted?: boolean }`. The List view and
  Calendar view both read from and write to this same object — there is no
  parallel state per view.
- **`localStorage`** — `dynamica.autopilot` (`{ enabled, increaseTolerance,
  decreaseTolerance }`) and `dynamica.alerts` (`{ lowUtil, highUtil,
  priceDrop, demandSpike }`, each `{ enabled, threshold }`).

---

## 6. Build order recipe

The order this project was built in. Each step is roughly one short prompt to
Claude Code.

1. **Skeleton.** Strip the previous demo to: top nav with logo block, blank
   `<main>`, file structure (`index.html`, `settings.html`, `sso.html`,
   `app.js`, `styles.css`), the `:root` design tokens, Inter font import.
   Drawer wiring in `app.js`.
2. **Main page table.** Build the List view: grouped column bands (Product
   details / Bookings / Forecast / Price recommendations / Action), per-row
   data, `.cell-pos` / `.cell-neg` conditional colouring on YoY columns and
   the Rec. price cell. Hard-code a small `ROWS` array.
3. **Calendar view.** Add the toggle + a route × period heatmap with the
   five-tier `tier()` colour scale and a Monthly/Daily granularity switch.
4. **Filters.** Add multi-select searchable filter pills for the entity's
   category attributes. Selections apply immediately. Show selection counts.
5. **Date range picker.** Dual-month dark `.dp-popover`, portaled to body,
   position:fixed.
6. **Column sorting.** Click a header to cycle asc/desc/off. Single-column
   sort. Closed/placeholder rows sort to the bottom.
7. **Burger drawer + Settings + SSO.** Drawer in `app.js`, a one-pane
   Settings page (Account only), and a fake SSO landing that redirects back.
8. **Detail modal — calendar.** Click any calendar cell → modal with booking
   curve chart, price curve chart, comparison table, recommendation panel.
   Seed chart data deterministically from `entity|period`.
9. **Detail modal — list rows.** Reuse the same modal when clicking a List
   row.
10. **Action column + row state model.** Accept/Reject/Override buttons,
    `rowStates` keyed by `${entity}|${period}`. Both views read/write the
    same object.
11. **Notes modal with expiry.** Textarea + optional ISO date input,
    auto-save on blur, "Expired" badge when past the date.
12. **Chart hover tooltips.** Crosshair on hover, value pill that follows
    the cursor on both charts in the detail modal.
13. **Commit button.** Counts committable rows, spinner, 1.2s fake
    persistence, clears pending states, toast.
14. **Accept all button.** Outlined pink button beside Commit; only accepts
    visible Pending rows.
15. **Autopilot.** New Settings pane, `localStorage` config, apply on Pricing
    load, `Autopilot` badge on auto-accepted rows.
16. **Inline row alerts.** New Settings pane, `localStorage` config, four
    alert types, colored letter chips in the entity-name cell.

---

## 7. Things to rename per industry

| Thing | Logistics (current) | Replace with |
|---|---|---|
| Brand name | **Dynamica** | (keep — it's the platform brand) |
| Product name | **SmartRates** | SmartStay (hotels) / SmartFares (airlines) / SmartFleet (rental) / SmartShelf (retail) |
| Entity name | Lane | Room type / Flight / Vehicle class / SKU |
| Entity composition | Origin → Destination | Property + Room class / Origin–Destination + Cabin / Location + Class / Store + SKU |
| Filter categories | Origin / Destination / Lane | (the entity's category attributes) |
| Period name | Date | Stay date / Departure date / Pickup date / Week |
| Pricing unit | **LDM** (loading meter) | Per night / Per seat / Per day / Per unit |
| Capacity label | Capacity (LDM) | Rooms available / Seats / Fleet / On hand |
| Actuals label | Units booked | Rooms on the books / Seats sold / Cars on rent / Units sold |
| Demand metric | Utilisation % | Occupancy % / Load factor % / Utilisation % / Sell-through % |
| Price label | Price per LDM ($) | ADR ($) / Yield ($) / Daily rate ($) / Unit price ($) |
| Sample routes | Chicago → DFW etc. | (industry-appropriate sample entities) |
| Sample period | 30 dates in March | (industry-appropriate horizon) |
| Page title | Pricing | Pricing / Yield / Revenue (industry term) |

---

## 8. Things to keep constant across industries

- **Visual identity** — the pink/magenta `#e8007d` accent, the dark `#1a1a2e`
  nav and popovers, soft pastel data colors, Inter font, 13px body, the
  diamond logo mark.
- **Interaction patterns** — modal on row/cell click, multi-select filters
  that apply immediately, dual-month dark date picker, sortable column
  headers cycling asc/desc/off, calendar heatmap with monthly/daily toggle,
  Accept/Reject/Override action column.
- **State model** — single `rowStates` object keyed by `${entity}|${period}`
  shared between List and Calendar views; `localStorage` only for user
  preferences (autopilot, alerts).
- **Commit flow** — pending changes accumulate, Commit button reflects the
  count and runs a fake 1.2s persistence with spinner + toast.
- **Settings structure** — Account / Autopilot / Alerts in that order, each
  with its own Save button and toast.
- **File structure** — `index.html`, `settings.html`, `sso.html`, `app.js`,
  `styles.css`, plus the logo PNG. No build step. No frameworks. No
  dependencies beyond Google Fonts.
- **The four-group metric layout** — actuals to date, forecast, price
  recommendations, action. Even when industry terms change, the *bands* and
  their order stay the same.

---

## 9. Hard-won lessons / gotchas

- **Sticky header band + wide table.** The grouped header bands (`tr.bands`)
  and column-label row (`tr.cols`) need `position: sticky` with `top: 0` and
  `top: 30px` respectively, in a scroll container that scrolls *the table*,
  not the page. Don't put the header band in a separate fixed element — when
  the table scrolls horizontally the band must scroll with it.
- **Conditional cell colors and row tints must compose.** When you tint a
  row by state (`row-accepted`, `row-rejected`, etc.) use
  `td:not(.cell-pos):not(.cell-neg)` so the YoY cell colors still read.
  Otherwise the tint clobbers the data signal.
- **Sticky z-index ordering.** Header bands `z-index: 10–11`, popovers
  `200`, modals `500`, range picker `1100`, drawer above modals when open,
  toast `2500`. Get this wrong and a popover hides behind the header, or a
  modal hides behind the drawer.
- **Notes auto-save on blur.** Don't save the textarea on every keystroke —
  it's noisy and triggers re-renders. Save on blur. Same pattern for the
  inline Rec. price input.
- **Multi-select popovers apply immediately.** No Apply button. The
  filter-pill count updates as boxes are checked; closing the popover does
  not require confirmation. This is much faster for the demo flow and users
  expect it.
- **Deterministic chart data.** When the detail modal opens, seed the chart
  generator with a hash of `${entity}|${period}` (see `hashStr` +
  `mulberry32` in `index.html`). Otherwise reopening the same cell shows
  different numbers and the demo looks broken.
- **Single `rowStates` object across views.** Calendar cell state and List
  row state must be the same object, keyed identically. Two parallel state
  models will drift the first time you toggle a cell in Calendar and look
  for the badge in List.
- **Portal popovers to `document.body`.** Date picker, multi-select popover,
  range picker — all use `position: fixed` and are appended to
  `document.body` so no parent's `overflow: hidden` or transform can trap
  them in a stacking context. Then position by getting the trigger's
  `getBoundingClientRect()`.
- **`html, body { height: 100%; overflow: hidden }`** is set globally. The
  app shell scrolls inside `<main>` (`flex: 1; overflow: auto`). Don't try
  to let the page scroll — the sticky bits will detach.
- **Filter dropdowns vs table columns are independent.** Removing a column
  from the table doesn't remove its filter pill; they're driven by separate
  lists (`buildFilterOptions()` reads from `ROWS`).
- **Clear autopilot flag on user action.** When the user explicitly
  Accepts/Rejects/Overrides a row, `delete st.autoAccepted` — otherwise the
  "Autopilot" badge sticks around on a manually-touched row.
- **Empty-state colspan must match column count.** Update it whenever
  columns are added or removed. The current table has 24 columns; the
  "No matching rows" cell uses `colspan="24"`.

---

## 10. Reusable opening prompt

Paste this into a fresh Claude Code conversation (after copying TEMPLATE.md
from this project into the new project's root):

> I want to build a **[INDUSTRY]** revenue-management demo using the
> template described in `TEMPLATE.md`. Read it first for full context.
>
> The product is called **Dynamica [PRODUCT_NAME]** (e.g. SmartStay for
> hotels, SmartFares for airlines). Keep the Dynamica visual identity exactly
> as documented in Section 2 (pink `#e8007d` accent, dark `#1a1a2e` nav and
> popovers, soft pastel data colors, Inter font, 13px body, diamond logo
> mark). Keep the file structure exactly as documented in Section 3
> (`index.html`, `settings.html`, `sso.html`, `app.js`, `styles.css`, no
> build step, no dependencies beyond Google Fonts).
>
> The domain mapping for this industry is:
> - **Entity** = [e.g. Room type with property + room class + bed type]
> - **Period** = [e.g. Stay date]
> - **Metrics** = [actuals, forecast, price] with the labels from Section 7
> - **Filter categories** = [list 2–3]
>
> Build the skeleton first (Build order step 1 in Section 6): top nav with
> the dark Dynamica header, an empty `<main>`, the design tokens in
> `styles.css`, the burger drawer in `app.js`, a placeholder `settings.html`
> with only the Account pane, and the `sso.html` landing. Use the
> "DYNAMICA" wordmark with "[PRODUCT_NAME]" as the sub-label exactly like
> the current logo block.
>
> Stop after the skeleton and show me a screenshot of the main page so I can
> confirm the look before we move on to step 2 (main table).
