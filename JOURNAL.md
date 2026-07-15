# Project Journal — Dynamica SmartGate demo

Ongoing record of changes and important context for this UI demo.
**Convention: append a dated entry here for every working session** (what changed,
why, and any decisions/assumptions worth remembering). Newest entries at the top.

## Standing context

- **What this is:** a clickable, mock-data-only demo of "Dynamica SmartGate", a
  revenue-management / dynamic-pricing tool for theme parks. Pure HTML/CSS/JS,
  no build step, no dependencies. All data is generated client-side with a
  seeded RNG (`rngFor`) so every reload shows identical numbers.
- **Demo "today"** is hard-coded as **2026-07-15**; the pricing horizon is the
  following 30 days from 2026-07-16 (Autopilot shows 8 weeks). Holiday week:
  2026-07-25 → 07-31.
- **Pages:** `index.html` (Pricing — table + calendar views, alerts, RM Copilot),
  `autopilot.html` (Autopilot — auto-accept rules over Park × Week),
  `settings.html` (account + alert thresholds), `sso.html` (fake login).
  Shared shell (top nav + burger drawer) is duplicated per page; `app.js` only
  drives the drawer/logout.
- **Rules reference:** `rules.png` — the "Example customisable rules" table.
  Demand Levels DL1–DL5 come from forecast-vs-expected-demand ppt bands
  (≥+20 → DL5 … <−10 → DL1) and a price ladder per DL × days-out. The demand
  level engine in both pages implements the DL bands from this image.
- **Workflow:** work locally; push to GitHub only when Tyler says it's ready.
  Claude never runs git operations unless explicitly asked.

## 2026-07-15 (later still) — Autopilot stripped back to per-row rules

- Autopilot rewritten per Tyler's feedback: no Demand column, no Moves column,
  no status chips, no day-level rows, and the right-hand master-rules panel is
  gone. The table is now just **Park → Week | Period | rules on each row**:
  an "Auto-accept within ±X%" input and a "Review peak days" switch per row.
- Week rows inherit their park's rules (muted); editing a value creates a
  pink "custom" override with an ↺ reset back to inherited. Park rows inherit
  built-in defaults (±5%, peak-day review off). "Reset all rules" in the header
  clears every override. Same localStorage key (`dynamica.autopilot`), now just
  `{enabled, overrides}`.
- Dropping the demand-consistency rule made the old ±3% default look weak
  (only ~27% auto-accepted), so the default band is ±5% (~77% auto).
- The hard review cap was removed too: with a single band, anything outside it
  goes to review, so a second cap threshold was redundant.
- Kept: master on/off toggle (header), KPI strip, park filter.
- Pricing table now loads fully collapsed (no default Warner expansion).

## 2026-07-15 (later) — Bulk update, reason codes, per-row Autopilot rules, fixes

Second batch from Tyler's review of the first round:

- **Dates**: horizon now starts **2026-07-16** (demo today = 2026-07-15);
  holiday week moved to 25–31 Jul. Autopilot weeks now run Thu 16 Jul → 09 Sep.
- **Calendar hover text**: every cell (not just the total row) now has a title
  spelling out the metrics — "Pickup vs historic … · Units sold YoY … · Avg
  price change …" plus alert count. The ⓘ tooltip on the total row remains.
- **Removed the green/red dots**: calendar accept/reject decision dots and the
  matching Action-cell corner dots in the table are gone (they read as alert
  noise). The pink top-left alert dot stays.
- **Reason code dropdown** in each leaf row's Action cell (Autopilot /
  DL Increase / DL Decrease / Holiday), stored in `rowStates[key].reason`,
  cleared on Commit.
- **Fixed inline price editing** — a broken comparison in `enterRecEdit`
  (`Math.round(n) !== Math.round(n)`) meant cell edits never saved. Edits now
  persist as Overridden with a "Price saved: €X" toast (mock-persistence only,
  same as everything else).
- **"Forecast units" column renamed "Forecast sold".**
- **Bulk price update**: checkbox column (leaf rows; park row checkbox selects
  its channels; header checkbox = all filtered rows) + "Bulk update (n)" button
  opens a modal — percentage or € amount, live preview, Apply marks rows
  Overridden and queues them for Commit.
- **Autopilot per-row thresholds**: ⚙ button on every park and week row opens a
  popover to override the auto-accept band and hard cap for that scope. Week
  override beats park override beats global; overrides persist under
  `dynamica.autopilot.overrides` and show as a pink chip on the row. Reset to
  defaults clears them.

## 2026-07-15 — Booking channels, demand levels, recommendations, Autopilot

Batch of changes requested by Tyler:

- **Booking Channel replaces Ticket Type** as the second accordion level
  (Website / Mobile App / OTA / Gate). All prices shown are the **Adult rate**
  for that channel; other ticket types (Junior/Senior/Evening) are derived
  multipliers via the new **Rate selector** in the Pricing header (defaults to
  Adult). Rec-price editing is only allowed in Adult view — derived views are
  read-only.
  - ⚠️ Internal naming: the code still uses `tickets` / `ticket` for the
    channel arrays and keys (renaming every internal reference wasn't worth the
    risk in a 5.6k-line file). Only data + labels changed.
  - Price-inversion alert reinterpreted: OTA/Gate undercutting the direct
    Website price (seeded promo-error discounts on ~7% of OTA/Gate leaves).
- **Demand Level replaces Demand score.** Score (0–100) maps to a ppt delta
  (`(score−50)×0.6`) and then to DL1–DL5 per the rules.png bands. Rendered as a
  colored `DLn` chip + label with an explanatory tooltip. Alert tooltips and
  RM Copilot messages now speak in DLs.
- **Recommendation engine is now rules-driven:** DL5 → +4..9%, DL4 → +2..5%,
  DL3 → hold (No change), DL2 → −3..6%, DL1 → −7..12%. Because most days are
  DL3, most rows carry **no price change** (realistic). A ~4% wild-outlier seed
  still feeds the out-of-line alert.
- **New Recommendation column** (Increase / Decrease / No change chips) next to
  % change; sortable; new filter pills for **Recommendation** and **Action**
  (Pending/Accepted/Rejected/Overridden). Park rows show a `n↑ n↓` summary.
- **Calendar:** new pinned **"All parks" total row** aggregating across visible
  parks, with an ⓘ hover tooltip explaining the cell metrics (big number =
  Pickup vs historic, U = units YoY, P = avg price change, dot meanings).
- **Width optimization:** two-line column headers (via the existing `.sub`
  pattern), tighter cell padding, narrower date/alerts columns, reduced accordion
  indents, calendar sticky column 270→240px.
- **New Autopilot page** (`autopilot.html`), added to the burger drawer on all
  pages. Park → Week → Day accordion over an 8-week horizon; each channel-day
  price change is classified against configurable auto-accept rules
  (persisted in `localStorage` under `dynamica.autopilot`):
  1. auto-accept moves within ±X% (default 3),
  2. hard review cap outside ±Y% (default 10),
  3. between the bands, require DL-consistency (increase ⇒ DL4/5, decrease ⇒ DL1/2),
  4. optional manual review on weekends/holidays; plus a master on/off switch.
  KPI strip totals changes / auto-accepted / needs-review.
- `settings.html`: price-inversion alert description updated to the channel
  semantics; Autopilot drawer item added.
- CLAUDE.md rewritten — it previously described an unrelated "Dynamica
  marketing site" at `/Users/tylerhennessy/logdemo/` (stale copy from the repo
  this demo was forked from).

Known deliberate simplifications:
- Row/cell detail modals and RM Copilot always show Adult-rate prices, whatever
  the Rate selector says (only the table applies the multiplier).
- The leaf detail modal's "Channel mix" panel is hidden at leaf level (a leaf
  *is* a channel now) but kept at park level.
- Autopilot generates its own seeded dataset (same style, simplified); its
  numbers won't tie out 1:1 with the Pricing page.
