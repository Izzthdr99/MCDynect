# Chart Hover Interaction

## Problem

`SuperAdminDashboard.vue`'s charts (Sales Trend, Cumulative Sales, Sales By Licensee,
Sales By Channel) are static — nothing signals they're part of an interactive dashboard
when the cursor is over them.

## Design

Pure-CSS hover affordances, no new data and no tooltips (the line/donut charts are
static exported SVGs with no underlying numeric data to show per-point). Scoped to the
chart-specific inner elements, not the shared `.chart-card` wrapper, since that wrapper
class is also used by the Sales By Outlet / Sales By State ranked-list tables, which are
out of scope.

- **`.line-chart img`** (Sales Trend, Cumulative Sales): on hover, `transform: scale(1.02)`
  plus a soft `drop-shadow` filter, transitioning over ~180ms ease.
- **`.bar-track` / `.bar-fill`** (Sales By Licensee): on hover of an individual bar,
  `filter: brightness(1.08)` and a light `box-shadow` on that bar only; `cursor: pointer`
  on the track.
- **`.donut-chart img`** (Sales By Channel): same scale + drop-shadow treatment as the
  line charts on hover.

All transitions ~150–200ms ease, matching the timing already used for the sidebar
collapse and other card interactions in this file.

## Out of scope

- Data tooltips / exact values on hover.
- Hover treatment on the ranked-list tables (Sales By Outlet, Sales By State).
- Any change to underlying chart data or rendering (SVGs stay static exported assets).
