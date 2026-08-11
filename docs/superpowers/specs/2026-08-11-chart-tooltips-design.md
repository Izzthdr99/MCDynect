# Chart Data Tooltips

## Problem

`SuperAdminDashboard.vue`'s charts (Sales Trend, Cumulative Sales, Sales By Licensee,
Sales By Channel) got a CSS-only hover affordance in an earlier pass (see
`2026-08-11-chart-hover-design.md`), by explicit choice at the time: the line and
donut charts were static Figma-exported SVG images with no numeric data behind them,
so real tooltips weren't feasible without a rework. That constraint is now being
lifted — this spec replaces the visual-only hover with real data tooltips across all
four charts.

## Design

### Data model

Add real numeric arrays to back every chart (all mock/fabricated, consistent with
the rest of this dashboard's placeholder data — no live backend exists):

- `salesTrend`: `[{ day, value }]`, 7 entries (Jan 21–27), replacing the standalone
  `salesTrendDays` array. Fixes an existing inconsistency where `salesTrendDays` had
  only 6 labels (missing "Jan 24") while `cumulativeSalesDays` had 7 — both charts
  now cover the same 7-day range.
- `cumulativeSales`: `[{ day, value }]`, 7 entries, running-total shape.
- `salesByLicensee`: gains `name` and `amount` fields alongside the existing
  `widthPct`/`color` (currently has neither — nothing identifies which bar is
  which).
- `salesByChannel`: new `[{ label, value }]` array (Cash, Cash Equivalent), replacing
  the hardcoded `donut-percent`/`donut-caption` text ("82.4%" / "Emergency Fund" —
  the caption doesn't match the legend's "Cash"/"Cash Equivalent" labels, an existing
  copy mismatch). The center label is now computed from the data instead of
  hardcoded.

Values are shaped to roughly match each chart's current visual curve/proportions so
the rebuild doesn't visibly change the dashboard's silhouette.

### Charting library

Add `apexcharts` + `vue3-apexcharts` as dependencies. Replace all four static
`<img>` chart elements with `<apexchart>` components:

- Sales Trend & Cumulative Sales → `area` chart type (smooth line + gradient fill,
  matching current look), single series, no legend (single-series charts don't need
  one).
- Sales By Licensee → horizontal `bar` chart, one color per bar via `colors`,
  matching the existing per-licensee palette.
- Sales By Channel → `donut` chart, 2 segments, existing legend markup stays
  (already lists Cash / Cash Equivalent with color dots).

All existing colors are preserved exactly — this is a rendering-technology swap, not
a redesign:
- Line/area: current blue gradient.
- Bars: `#00a6f4`, `#ff8a4c`, `#05df72`, `#ff6467`, `#c27aff`, `#fdc700` (unchanged,
  now passed as ApexCharts' `colors` option).
- Donut: existing orange/cyan pair from the legend dots.

### Tooltips

ApexCharts' built-in tooltip is used (not a hand-built overlay), configured per
chart:

- **Line charts**: shows the day and the value formatted as currency (e.g. "Jan 23"
  / "MYR 4,200"), value styled as the strong/high-contrast element, day as
  secondary — mirroring the dataviz convention that values lead and labels follow.
  Crosshair enabled (ApexCharts default for area/line) so the pointer snaps to the
  nearest day.
- **Bar chart**: per-bar tooltip on hover showing licensee name + revenue amount,
  keyed with that bar's color.
- **Donut**: per-segment tooltip showing the segment label + value/percentage, keyed
  with that segment's color.
- Tooltip container is restyled via ApexCharts' custom-tooltip / CSS override to
  match the dashboard's card chrome (white background, `var(--border)` hairline,
  subtle shadow, existing font stack) instead of ApexCharts' default theme.

### Cleanup

- Remove the CSS-only hover rules added in the prior pass (`.line-chart:hover img`,
  `.bar-track:hover .bar-fill`/`.bar-track { cursor: pointer }`, `.donut-chart:hover
  img`) — the elements they targeted (static `<img>`s, CSS bar divs) no longer
  exist.
- Delete the now-unused static chart assets (`chart-sales-trend.svg`,
  `chart-cumulative-sales.svg`, `chart-donut-channel.svg`) and their imports.
- `.line-chart`, `.bar-chart`, `.donut-chart` wrapper CSS is adapted as needed to
  host an `<apexchart>` component instead of an `<img>`/CSS bars, keeping the same
  card dimensions (`height: 462px` chart-card, `209px` donut) so the surrounding
  layout is unaffected.

## Out of scope

- Live/backend data — everything stays mock data, matching the rest of the
  dashboard.
- Filters or date-range controls over the charts (not requested).
- Changing any existing color value (bars, line, donut) — this is a rendering swap
  only.
- Keyboard-focus tooltip parity beyond what ApexCharts provides out of the box.
