# Chart Data Tooltips Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the SuperAdmin dashboard's 4 static/CSS-only charts (Sales Trend, Cumulative Sales, Sales By Licensee, Sales By Channel) with ApexCharts-rendered charts backed by real mock data, each showing a data tooltip on hover.

**Architecture:** Add `apexcharts` + `vue3-apexcharts` as dependencies. In `SuperAdminDashboard.vue`, import `VueApexCharts` and use it directly in the template (script-setup auto-exposes imports; no global registration needed, consistent with this file's existing per-component imports like `DashboardSidebar`). Each chart gets its own small `chartOptions`/`series` pair built from a new mock-data array. Tooltips use ApexCharts' `tooltip.custom` hook to render markup styled to match the dashboard's card chrome via a shared `.chart-tooltip` class (pierced into ApexCharts' unscoped injected DOM via `:deep()`). One task per chart keeps each conversion independently reviewable and testable; a final cleanup task removes the now-orphaned static-image hover CSS and unused SVG assets once all four charts have moved off them.

**Tech Stack:** Vue 3 `<script setup>`, ApexCharts 6.x via `vue3-apexcharts` 1.x, scoped CSS with `:deep()` for ApexCharts' injected tooltip DOM.

## Global Constraints

- Preserve every existing color exactly — this is a rendering-technology swap, not a redesign (per `docs/superpowers/specs/2026-08-11-chart-tooltips-design.md`):
  - Sales Trend: line stroke `#155DFC`, area gradient fill `#1447E6` (0.4 opacity → 0 opacity).
  - Cumulative Sales: line stroke `#51A2FF`, **no area fill** (the original SVG is a bare stroked line, not an area chart — confirmed by reading `src/assets/dashboard/charts/chart-cumulative-sales.svg`, which has no gradient/fill, only `stroke="#51A2FF"`).
  - Sales By Licensee bars: `#00a6f4`, `#ff8a4c`, `#05df72`, `#ff6467`, `#c27aff`, `#fdc700`, in that order, one per bar.
  - Sales By Channel donut: Cash `#ff8a4c`, Cash Equivalent `#00bcff`.
- No test framework exists in this repo (`package.json` has no test script/devDependency) — do not add one. Verification is `npm run build` (syntax/bundle sanity) plus a manual Playwright browser pass per task (pattern already used and documented working in this repo — see the browser-verification step of `docs/superpowers/plans/2026-08-11-chart-hover.md`'s execution).
- Card dimensions must not change: `.chart-card` stays `height: 462px`, `.donut-chart` stays `209px` × `209px`. The point is a rendering swap with an unchanged layout.
- ApexCharts injects its tooltip DOM directly into the chart's own container at runtime, **not** through Vue's compiler — scoped `<style>` rules do not apply to it without `:deep()`. Any CSS targeting `.chart-tooltip*` or `.apexcharts-*` must use `:deep()`.
- Currency values are formatted with a single shared helper: `formatMyr(value)` → `` `MYR ${Math.round(value).toLocaleString('en-MY')}` ``. Defined once (Task 1), reused by later tasks — do not redefine it.
- All chart data is mock/fabricated (no live backend exists anywhere in this repo) — this is expected and matches every other number already on this dashboard.
- Delete a static SVG asset only once its last `import`/usage is gone from the file (this happens for all three in Task 5, after Tasks 1–4 land) — never leave a dangling import.
- Axis-chart tooltips (area/line/bar — anything using `xaxis.categories`) must read the category label via `w.globals.categoryLabels[dataPointIndex]`, **not** `w.globals.labels[dataPointIndex]`. Verified during Task 1: against installed `apexcharts@6.8.0`, `w.globals.labels` holds numeric indices for this config shape while `w.globals.categoryLabels` holds the real category strings — confirmed both against library source (`Data.js`/`Globals.js`/`KeyboardNavigation.js`) and empirically in-browser (see `task-1-report.md`). This does not apply to the donut/pie tooltip (Task 4), which correctly uses `w.globals.labels[seriesIndex]` — donut/pie is a non-axis chart with a different code path (confirmed against `Labels.js`'s `j === null` branch), unaffected by this bug.

---

### Task 1: Add ApexCharts dependency; convert Sales Trend to a data-driven area chart with tooltip

**Files:**
- Modify: `package.json`, `package-lock.json` (via `npm install`)
- Modify: `src/pages/SuperAdminDashboard.vue`

**Interfaces:**
- Consumes: nothing from other tasks (this is the first task).
- Produces (for Tasks 2–4 to reuse, do not redefine):
  - `import VueApexCharts from 'vue3-apexcharts'` at the top of `<script setup>`, used in templates as `<VueApexCharts>`.
  - `function formatMyr(value)` → returns e.g. `"MYR 22,400"` (defined near the top of `<script setup>`, after the imports).
  - Shared tooltip CSS classes `.chart-tooltip`, `.chart-tooltip-value`, `.chart-tooltip-label`, added once in this task's `:deep()` block; later tasks' tooltip HTML reuses these same class names without adding new CSS for them.

- [ ] **Step 1: Install the dependency**

```bash
npm install apexcharts vue3-apexcharts
```

Confirm `package.json`'s `dependencies` now includes `apexcharts` and `vue3-apexcharts` (npm adds them automatically; no manual edit needed).

- [ ] **Step 2: Add the import, `formatMyr` helper, and `salesTrend` data array**

In `src/pages/SuperAdminDashboard.vue`, find this line near the top of `<script setup>` (currently line 10):

```js
import { ref } from 'vue'
```

Add the ApexCharts import directly after it:

```js
import { ref } from 'vue'
import VueApexCharts from 'vue3-apexcharts'
```

Find this line (currently line 26):

```js
import chartSalesTrend from '../assets/dashboard/charts/chart-sales-trend.svg'
```

Delete it — Sales Trend no longer uses a static image. Leave the other two chart-SVG imports (`chartCumulativeSales`, `chartDonutChannel`) untouched; they're removed in Tasks 2 and 4 respectively.

Find this line (currently line 52, immediately followed by line 53's `cumulativeSalesDays`):

```js
const salesTrendDays = ['Jan 21', 'Jan 22', 'Jan 23', 'Jan 25', 'Jan 26', 'Jan 27']
```

Replace ONLY this line (leave `cumulativeSalesDays` on the next line untouched — Task 2 handles it) with:

```js
function formatMyr(value) {
  return `MYR ${Math.round(value).toLocaleString('en-MY')}`
}

const salesTrend = [
  { day: 'Jan 21', value: 18400 },
  { day: 'Jan 22', value: 21900 },
  { day: 'Jan 23', value: 19600 },
  { day: 'Jan 24', value: 27300 },
  { day: 'Jan 25', value: 20100 },
  { day: 'Jan 26', value: 25800 },
  { day: 'Jan 27', value: 22400 },
]

const salesTrendChartOptions = {
  chart: {
    type: 'area',
    toolbar: { show: false },
    zoom: { enabled: false },
    animations: { enabled: false },
  },
  colors: ['#155DFC'],
  fill: {
    type: 'gradient',
    gradient: {
      shadeIntensity: 1,
      opacityFrom: 0.4,
      opacityTo: 0,
      stops: [0, 100],
    },
  },
  stroke: { curve: 'straight', width: 2 },
  dataLabels: { enabled: false },
  grid: { show: false },
  xaxis: {
    categories: salesTrend.map((point) => point.day),
    labels: { show: false },
    axisTicks: { show: false },
    axisBorder: { show: false },
  },
  yaxis: { show: false },
  tooltip: {
    custom: ({ series, seriesIndex, dataPointIndex, w }) => `
      <div class="chart-tooltip">
        <div class="chart-tooltip-value">${formatMyr(series[seriesIndex][dataPointIndex])}</div>
        <div class="chart-tooltip-label">${w.globals.categoryLabels[dataPointIndex]}</div>
      </div>
    `,
  },
}

const salesTrendSeries = [{ name: 'Sales', data: salesTrend.map((point) => point.value) }]
```

This fixes an existing inconsistency: the old `salesTrendDays` array had only 6 entries (missing "Jan 24") while `cumulativeSalesDays` had 7 — `salesTrend` now has all 7 days, matching `cumulativeSalesDays`.

- [ ] **Step 3: Replace the Sales Trend template markup**

Find this block (currently lines 142–147):

```html
            <div class="line-chart">
              <img :src="chartSalesTrend" alt="Daily sales trend chart" />
              <div class="chart-x-axis">
                <span v-for="day in salesTrendDays" :key="day">{{ day }}</span>
              </div>
            </div>
```

Replace it with:

```html
            <div class="line-chart">
              <div class="line-chart-canvas">
                <VueApexCharts
                  type="area"
                  width="100%"
                  height="100%"
                  :options="salesTrendChartOptions"
                  :series="salesTrendSeries"
                />
              </div>
              <div class="chart-x-axis">
                <span v-for="point in salesTrend" :key="point.day">{{ point.day }}</span>
              </div>
            </div>
```

- [ ] **Step 4: Add CSS for the new chart canvas wrapper and the shared tooltip styling**

Find this block in `<style scoped>` (currently lines 548–566):

```css
/* Line charts */
.line-chart {
  display: flex;
  flex: 1;
  flex-direction: column;
  justify-content: flex-end;
  min-height: 0;
}
.line-chart img {
  flex: 1;
  min-height: 0;
  width: 100%;
  object-fit: fill;
  transition: transform 180ms ease, filter 180ms ease;
}
.line-chart:hover img {
  transform: scale(1.02);
  filter: drop-shadow(0 4px 10px rgba(16, 24, 40, 0.12));
}
```

Leave `.line-chart img` and `.line-chart:hover img` in place for now — Cumulative Sales still uses an `<img>` until Task 2. Add a new rule immediately after this block for the ApexCharts wrapper, plus the shared tooltip styles:

```css
.line-chart-canvas {
  flex: 1;
  min-height: 0;
  width: 100%;
}

/* Shared ApexCharts tooltip styling. ApexCharts injects this markup directly
   into the chart's DOM at runtime, bypassing Vue's compiler — :deep() is
   required for scoped styles to reach it. */
:deep(.apexcharts-tooltip) {
  border: none !important;
  box-shadow: none !important;
  background: transparent !important;
}
:deep(.chart-tooltip) {
  padding: 8px 12px;
  background: #ffffff;
  border: 1px solid var(--border);
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(16, 24, 40, 0.12);
}
:deep(.chart-tooltip-value) {
  font-size: 14px;
  font-weight: 600;
  color: var(--heading);
}
:deep(.chart-tooltip-label) {
  margin-top: 2px;
  font-size: 12px;
  color: var(--body-text);
}
```

- [ ] **Step 5: Verify the build**

```bash
npm run build
```

Expected: builds successfully with no errors. Clean up the `dist/` output afterward (`rm -rf dist`) — it's a build artifact, not something to commit.

- [ ] **Step 6: Manual browser verification**

No test framework exists in this repo, so verification is a real browser pass. Start the dev server and drive it with Playwright (already proven to work in this environment — Chromium is cached at `~/Library/Caches/ms-playwright/`):

```bash
lsof -ti:5173 -sTCP:LISTEN | xargs -r kill 2>/dev/null
nohup npm run dev > /tmp/vite-dev.log 2>&1 &
disown
for i in $(seq 1 30); do curl -sf http://localhost:5173 >/dev/null && break; sleep 1; done
```

Then run a Playwright script (adjust the path to your own scratch directory) that logs in, navigates to the Super Admin dashboard, and hovers the Sales Trend chart:

```js
import { chromium } from 'playwright'

const browser = await chromium.launch({ args: ['--no-sandbox'] })
const page = await browser.newPage({ viewport: { width: 1600, height: 1200 } })
const consoleErrors = []
page.on('console', (msg) => { if (msg.type() === 'error') consoleErrors.push(msg.text()) })
page.on('pageerror', (err) => consoleErrors.push(String(err)))

await page.goto('http://localhost:5173', { waitUntil: 'networkidle' })
await page.fill('#email', 'test@example.com')
await page.fill('#password', 'password123')
await page.click('button[type=submit]')
await page.waitForSelector('.module-card')
await page.click('.module-card')
await page.waitForSelector('text=Sales Trend')

await page.hover('.line-chart-canvas')
await page.waitForTimeout(300)
await page.screenshot({ path: '/tmp/sales-trend-hover.png', fullPage: true })
console.log('CONSOLE_ERRORS:', JSON.stringify(consoleErrors))
await browser.close()
```

Expected: `CONSOLE_ERRORS` is `[]`, and the screenshot shows the Sales Trend chart rendered as a blue gradient-filled area chart with a tooltip visible near the cursor showing a day and a "MYR …" value. Look at the screenshot — a blank chart area is a failure to render.

Stop the dev server when done: `lsof -ti:5173 -sTCP:LISTEN | xargs -r kill`.

- [ ] **Step 7: Commit**

```bash
git add package.json package-lock.json src/pages/SuperAdminDashboard.vue
git commit -m "$(cat <<'EOF'
Convert Sales Trend to a data-driven ApexCharts area chart with tooltip

Co-Authored-By: Claude Sonnet 5 <noreply@anthropic.com>
EOF
)"
```

---

### Task 2: Convert Cumulative Sales to a data-driven line chart with tooltip

**Files:**
- Modify: `src/pages/SuperAdminDashboard.vue`

**Interfaces:**
- Consumes: `VueApexCharts` import, `formatMyr(value)` helper, `.chart-tooltip`/`.chart-tooltip-value`/`.chart-tooltip-label` CSS classes, and the `.line-chart-canvas` CSS rule — all from Task 1. Do not redefine any of them.
- Produces: nothing new consumed by later tasks (Tasks 3 and 4 are independent chart conversions).

- [ ] **Step 1: Add the `cumulativeSales` data array and chart config**

Find this line (currently line 53, right after where Task 1 left `salesTrendChartOptions`/`salesTrendSeries`):

```js
const cumulativeSalesDays = ['Jan 21', 'Jan 22', 'Jan 23', 'Jan 24', 'Jan 25', 'Jan 26', 'Jan 27']
```

Replace it with:

```js
const cumulativeSales = [
  { day: 'Jan 21', value: 15200 },
  { day: 'Jan 22', value: 19800 },
  { day: 'Jan 23', value: 17500 },
  { day: 'Jan 24', value: 21300 },
  { day: 'Jan 25', value: 16900 },
  { day: 'Jan 26', value: 24600 },
  { day: 'Jan 27', value: 23100 },
]

const cumulativeSalesChartOptions = {
  chart: {
    type: 'line',
    toolbar: { show: false },
    zoom: { enabled: false },
    animations: { enabled: false },
  },
  colors: ['#51A2FF'],
  stroke: { curve: 'smooth', width: 2 },
  dataLabels: { enabled: false },
  grid: { show: false },
  xaxis: {
    categories: cumulativeSales.map((point) => point.day),
    labels: { show: false },
    axisTicks: { show: false },
    axisBorder: { show: false },
  },
  yaxis: { show: false },
  tooltip: {
    custom: ({ series, seriesIndex, dataPointIndex, w }) => `
      <div class="chart-tooltip">
        <div class="chart-tooltip-value">${formatMyr(series[seriesIndex][dataPointIndex])}</div>
        <div class="chart-tooltip-label">${w.globals.categoryLabels[dataPointIndex]}</div>
      </div>
    `,
  },
}

const cumulativeSalesSeries = [{ name: 'Sales', data: cumulativeSales.map((point) => point.value) }]
```

Note this chart is `type: 'line'` with **no `fill`/gradient option** — the original `chart-cumulative-sales.svg` is a bare stroked line with no area fill underneath (confirmed by inspecting the SVG source; it has no gradient or fill, unlike Sales Trend's SVG). Do not add a gradient fill here — that would visually diverge from the current design.

- [ ] **Step 2: Remove the now-unused `chartCumulativeSales` import**

Find this line (currently line 27):

```js
import chartCumulativeSales from '../assets/dashboard/charts/chart-cumulative-sales.svg'
```

Delete it.

- [ ] **Step 3: Replace the Cumulative Sales template markup**

Find this block (currently lines 193–198):

```html
            <div class="line-chart">
              <img :src="chartCumulativeSales" alt="Cumulative sales chart" />
              <div class="chart-x-axis">
                <span v-for="day in cumulativeSalesDays" :key="day">{{ day }}</span>
              </div>
            </div>
```

Replace it with:

```html
            <div class="line-chart">
              <div class="line-chart-canvas">
                <VueApexCharts
                  type="line"
                  width="100%"
                  height="100%"
                  :options="cumulativeSalesChartOptions"
                  :series="cumulativeSalesSeries"
                />
              </div>
              <div class="chart-x-axis">
                <span v-for="point in cumulativeSales" :key="point.day">{{ point.day }}</span>
              </div>
            </div>
```

- [ ] **Step 4: Remove the now-orphaned `.line-chart img` CSS**

Both line charts now use `.line-chart-canvas` instead of `<img>`. Find this block in `<style scoped>` (added complete in Task 1, currently around lines 556–566):

```css
.line-chart img {
  flex: 1;
  min-height: 0;
  width: 100%;
  object-fit: fill;
  transition: transform 180ms ease, filter 180ms ease;
}
.line-chart:hover img {
  transform: scale(1.02);
  filter: drop-shadow(0 4px 10px rgba(16, 24, 40, 0.12));
}
```

Delete this entire block — no `<img>` remains under `.line-chart` after this task. Leave `.line-chart-canvas` (added by Task 1) and everything else untouched.

Also find this line in the `@media (max-width: 1100px)` responsive block near the bottom of the file:

```css
  .line-chart img {
    height: 220px;
  }
```

Replace it with:

```css
  .line-chart-canvas {
    height: 220px;
  }
```

- [ ] **Step 5: Verify the build**

```bash
npm run build
```

Expected: builds successfully with no errors. `rm -rf dist` afterward.

- [ ] **Step 6: Manual browser verification**

Same dev-server-start pattern as Task 1, Step 6. Extend the Playwright script (or write a fresh one) to also hover the second `.line-chart-canvas`:

```js
// after the Task 1 steps (login, navigate to dashboard)...
const lineCharts = page.locator('.line-chart-canvas')
await lineCharts.nth(1).scrollIntoViewIfNeeded()
await lineCharts.nth(1).hover()
await page.waitForTimeout(300)
await page.screenshot({ path: '/tmp/cumulative-sales-hover.png', fullPage: true })
```

Expected: no console errors, and the screenshot shows the Cumulative Sales chart as a plain light-blue line (no area fill under it) with a tooltip showing a day and "MYR …" value near the cursor.

Stop the dev server when done.

- [ ] **Step 7: Commit**

```bash
git add src/pages/SuperAdminDashboard.vue
git commit -m "$(cat <<'EOF'
Convert Cumulative Sales to a data-driven ApexCharts line chart with tooltip

Co-Authored-By: Claude Sonnet 5 <noreply@anthropic.com>
EOF
)"
```

---

### Task 3: Convert Sales By Licensee to a data-driven bar chart with tooltip

**Files:**
- Modify: `src/pages/SuperAdminDashboard.vue`

**Interfaces:**
- Consumes: `VueApexCharts` import, `formatMyr(value)` helper, `.chart-tooltip`/`.chart-tooltip-value`/`.chart-tooltip-label` CSS classes — all from Task 1. Do not redefine any of them.
- Produces: nothing consumed by other tasks.

- [ ] **Step 1: Replace the `salesByLicensee` data array with named, valued entries and add the chart config**

Find this block (currently lines 63–70):

```js
const salesByLicensee = [
  { key: 'licensee-1', widthPct: 43.92, color: '#00a6f4' },
  { key: 'licensee-2', widthPct: 87.25, color: '#ff8a4c' },
  { key: 'licensee-3', widthPct: 64.9, color: '#05df72' },
  { key: 'licensee-4', widthPct: 19.61, color: '#ff6467' },
  { key: 'licensee-5', widthPct: 36.47, color: '#c27aff' },
  { key: 'licensee-6', widthPct: 57.65, color: '#fdc700' },
]
```

Replace it with:

```js
const salesByLicensee = [
  { key: 'licensee-1', name: 'Mr Churros Klang Valley', amount: 76500, color: '#00a6f4' },
  { key: 'licensee-2', name: 'Mr Churros Johor Bahru', amount: 152000, color: '#ff8a4c' },
  { key: 'licensee-3', name: 'Mr Churros Penang', amount: 113000, color: '#05df72' },
  { key: 'licensee-4', name: 'Mr Churros Ipoh', amount: 34000, color: '#ff6467' },
  { key: 'licensee-5', name: 'Mr Churros Melaka', amount: 63500, color: '#c27aff' },
  { key: 'licensee-6', name: 'Mr Churros Kuching', amount: 100500, color: '#fdc700' },
]

const salesByLicenseeChartOptions = {
  chart: {
    type: 'bar',
    toolbar: { show: false },
    animations: { enabled: false },
  },
  plotOptions: {
    bar: {
      horizontal: true,
      distributed: true,
      borderRadius: 4,
      barHeight: '60%',
    },
  },
  colors: salesByLicensee.map((licensee) => licensee.color),
  dataLabels: { enabled: false },
  legend: { show: false },
  grid: { show: false },
  xaxis: {
    categories: salesByLicensee.map((licensee) => licensee.name),
    labels: { show: false },
    axisTicks: { show: false },
    axisBorder: { show: false },
  },
  yaxis: { labels: { show: false } },
  tooltip: {
    custom: ({ series, seriesIndex, dataPointIndex, w }) => `
      <div class="chart-tooltip">
        <div class="chart-tooltip-value">${formatMyr(series[seriesIndex][dataPointIndex])}</div>
        <div class="chart-tooltip-label">${w.globals.categoryLabels[dataPointIndex]}</div>
      </div>
    `,
  },
}

const salesByLicenseeSeries = [{ name: 'Revenue', data: salesByLicensee.map((licensee) => licensee.amount) }]
```

`distributed: true` makes ApexCharts color each bar individually from the `colors` array (matching the current per-bar distinct-color look) instead of coloring the whole series one color. The `amount` values preserve the original bars' relative proportions (the old `widthPct` values, scaled up to plausible currency figures) — bar length is now computed by ApexCharts directly from `amount`, so `widthPct` is fully superseded and intentionally removed.

- [ ] **Step 2: Replace the Sales By Licensee template markup**

Find this block (currently lines 213–221):

```html
            <div class="bar-chart">
              <div
                v-for="bar in salesByLicensee"
                :key="bar.key"
                class="bar-track"
              >
                <div class="bar-fill" :style="{ width: bar.widthPct + '%', background: bar.color }"></div>
              </div>
            </div>
```

Replace it with:

```html
            <div class="bar-chart">
              <VueApexCharts
                type="bar"
                width="100%"
                height="100%"
                :options="salesByLicenseeChartOptions"
                :series="salesByLicenseeSeries"
              />
            </div>
```

- [ ] **Step 3: Remove the now-orphaned `.bar-track`/`.bar-fill` CSS**

Find this block in `<style scoped>` (currently lines 651–672):

```css
/* Bar chart (licensee) */
.bar-chart {
  display: flex;
  flex: 1;
  flex-direction: column;
  justify-content: space-evenly;
  min-height: 0;
}
.bar-track {
  width: 100%;
  height: 26px;
  cursor: pointer;
}
.bar-fill {
  height: 100%;
  border-radius: 4px;
  transition: filter 180ms ease, box-shadow 180ms ease;
}
.bar-track:hover .bar-fill {
  filter: brightness(1.08);
  box-shadow: 0 2px 8px rgba(16, 24, 40, 0.15);
}
```

Replace it with just the `.bar-chart` rule (still needed — it's the ApexCharts host container's sizing) — delete `.bar-track`, `.bar-fill`, and `.bar-track:hover .bar-fill` entirely:

```css
/* Bar chart (licensee) */
.bar-chart {
  display: flex;
  flex: 1;
  flex-direction: column;
  justify-content: space-evenly;
  min-height: 0;
}
```

- [ ] **Step 4: Verify the build**

```bash
npm run build
```

Expected: builds successfully with no errors. `rm -rf dist` afterward.

- [ ] **Step 5: Manual browser verification**

Same dev-server pattern as Task 1/2. Extend the Playwright script to hover a bar in the licensee chart:

```js
// after login + navigate to dashboard...
await page.locator('text=Sales By Licensee').scrollIntoViewIfNeeded()
await page.waitForTimeout(200)
const bars = page.locator('.apexcharts-bar-area')
console.log('bar count:', await bars.count())
await bars.nth(1).hover()
await page.waitForTimeout(300)
await page.screenshot({ path: '/tmp/licensee-bar-hover.png', fullPage: true })
```

Expected: `bar count` is `6`, no console errors, and the screenshot shows 6 horizontal bars in the original colors with a tooltip near the cursor showing a licensee name and "MYR …" value.

Stop the dev server when done.

- [ ] **Step 6: Commit**

```bash
git add src/pages/SuperAdminDashboard.vue
git commit -m "$(cat <<'EOF'
Convert Sales By Licensee to a data-driven ApexCharts bar chart with tooltip

Co-Authored-By: Claude Sonnet 5 <noreply@anthropic.com>
EOF
)"
```

---

### Task 4: Convert Sales By Channel to a data-driven donut chart with tooltip

**Files:**
- Modify: `src/pages/SuperAdminDashboard.vue`

**Interfaces:**
- Consumes: `VueApexCharts` import from Task 1. Does not use `formatMyr` (this chart's tooltip shows a percentage, not currency).
- Produces: nothing consumed by other tasks.

- [ ] **Step 1: Add the `salesByChannel` data array and chart config**

Find this line (currently line 72, right after `salesByLicensee`'s closing `]` — note Task 3 already modified `salesByLicensee` above this point, but this insertion point is unaffected by that):

```js
const salesByState = [
```

Insert a new `salesByChannel` array and its chart config directly before this line:

```js
const salesByChannel = [
  { label: 'Cash', value: 82.4, color: '#ff8a4c' },
  { label: 'Cash Equivalent', value: 17.6, color: '#00bcff' },
]

const donutChartOptions = {
  chart: {
    type: 'donut',
    toolbar: { show: false },
    animations: { enabled: false },
  },
  labels: salesByChannel.map((channel) => channel.label),
  colors: salesByChannel.map((channel) => channel.color),
  dataLabels: { enabled: false },
  legend: { show: false },
  stroke: { show: false },
  plotOptions: {
    pie: {
      donut: {
        size: '75%',
        labels: { show: false },
      },
    },
  },
  tooltip: {
    custom: ({ series, seriesIndex, w }) => `
      <div class="chart-tooltip">
        <div class="chart-tooltip-value">${series[seriesIndex]}%</div>
        <div class="chart-tooltip-label">${w.globals.labels[seriesIndex]}</div>
      </div>
    `,
  },
}

const donutSeries = salesByChannel.map((channel) => channel.value)

const salesByState = [
```

(The rest of the `salesByState` array body and its closing `]` stay exactly as they are — only the line shown above is used as an anchor for where to insert.)

- [ ] **Step 2: Remove the now-unused `chartDonutChannel` import**

Find this line (currently line 28):

```js
import chartDonutChannel from '../assets/dashboard/charts/chart-donut-channel.svg'
```

Delete it.

- [ ] **Step 3: Replace the donut chart, center label, and legend template markup**

Find this block (currently lines 238–254):

```html
            <div class="donut-chart">
              <img :src="chartDonutChannel" alt="Sales by channel donut chart" />
              <div class="donut-label">
                <p class="donut-percent">82.4%</p>
                <p class="donut-caption">Emergency Fund</p>
              </div>
            </div>
            <div class="donut-legend">
              <div class="legend-item">
                <span class="legend-dot" style="background: #ff8a4c"></span>
                <span>Cash</span>
              </div>
              <div class="legend-item">
                <span class="legend-dot" style="background: #00bcff"></span>
                <span>Cash Equivalent</span>
              </div>
            </div>
```

Replace it with:

```html
            <div class="donut-chart">
              <VueApexCharts
                type="donut"
                width="100%"
                height="100%"
                :options="donutChartOptions"
                :series="donutSeries"
              />
              <div class="donut-label">
                <p class="donut-percent">{{ salesByChannel[0].value }}%</p>
                <p class="donut-caption">{{ salesByChannel[0].label }}</p>
              </div>
            </div>
            <div class="donut-legend">
              <div v-for="channel in salesByChannel" :key="channel.label" class="legend-item">
                <span class="legend-dot" :style="{ background: channel.color }"></span>
                <span>{{ channel.label }}</span>
              </div>
            </div>
```

This also fixes an existing copy bug: the center label previously read "Emergency Fund" (a leftover placeholder that didn't match the legend's "Cash"/"Cash Equivalent" labels below it). It's now computed from `salesByChannel[0]`, so the center label and the legend can never drift out of sync again.

- [ ] **Step 4: Remove the now-orphaned `.donut-chart img` CSS**

Find this block in `<style scoped>` (currently lines 698–706):

```css
.donut-chart img {
  width: 100%;
  height: 100%;
  transition: transform 180ms ease, filter 180ms ease;
}
.donut-chart:hover img {
  transform: scale(1.02);
  filter: drop-shadow(0 4px 10px rgba(16, 24, 40, 0.12));
}
```

Delete this entire block — no `<img>` remains under `.donut-chart` after this task. Leave `.donut-chart` itself (the sizing container, currently lines 690–697) untouched.

- [ ] **Step 5: Verify the build**

```bash
npm run build
```

Expected: builds successfully with no errors. `rm -rf dist` afterward.

- [ ] **Step 6: Manual browser verification**

Same dev-server pattern as prior tasks. Extend the Playwright script to hover the donut:

```js
// after login + navigate to dashboard...
await page.locator('text=Sales By Channel').scrollIntoViewIfNeeded()
await page.waitForTimeout(200)
await page.hover('.donut-chart')
await page.waitForTimeout(300)
await page.screenshot({ path: '/tmp/donut-hover.png', fullPage: true })
```

Expected: no console errors, the donut renders in the original orange/cyan colors, the center label reads "82.4%" / "Cash", and a tooltip near the cursor shows a segment's label and percentage on hover.

Stop the dev server when done.

- [ ] **Step 7: Commit**

```bash
git add src/pages/SuperAdminDashboard.vue
git commit -m "$(cat <<'EOF'
Convert Sales By Channel to a data-driven ApexCharts donut chart with tooltip

Co-Authored-By: Claude Sonnet 5 <noreply@anthropic.com>
EOF
)"
```

---

### Task 5: Delete unused static chart assets and do a full cross-chart verification pass

**Files:**
- Delete: `src/assets/dashboard/charts/chart-sales-trend.svg`
- Delete: `src/assets/dashboard/charts/chart-cumulative-sales.svg`
- Delete: `src/assets/dashboard/charts/chart-donut-channel.svg`
- Modify: `src/pages/SuperAdminDashboard.vue` (verification only — no code changes expected; this task exists to catch anything the per-chart tasks missed)

**Interfaces:**
- Consumes: the completed state of Tasks 1–4 (all four charts converted, all `<img>`-based hover CSS removed).
- Produces: nothing — this is the final task in the plan.

- [ ] **Step 1: Confirm no remaining references to the three SVG assets**

```bash
grep -rn "chart-sales-trend.svg\|chart-cumulative-sales.svg\|chart-donut-channel.svg" src/
```

Expected: no output. If anything matches, stop — one of Tasks 1–4 left a dangling import; that's a bug in this task's prerequisites, not something to silently patch around. Report BLOCKED with what you found.

- [ ] **Step 2: Delete the three unused SVG files**

```bash
rm src/assets/dashboard/charts/chart-sales-trend.svg
rm src/assets/dashboard/charts/chart-cumulative-sales.svg
rm src/assets/dashboard/charts/chart-donut-channel.svg
```

- [ ] **Step 3: Confirm no orphaned hover-CSS selectors remain**

```bash
grep -n "\.bar-track\|\.bar-fill\|\.line-chart img\|\.line-chart:hover img\|\.donut-chart img\|\.donut-chart:hover img" src/pages/SuperAdminDashboard.vue
```

Expected: no output (Tasks 2, 3, and 4 each removed their own chart type's orphaned rules). If anything matches, remove it — it's dead CSS targeting elements that no longer exist.

- [ ] **Step 4: Verify the build**

```bash
npm run build
```

Expected: builds successfully with no errors (confirms nothing still imports the deleted SVGs). `rm -rf dist` afterward.

- [ ] **Step 5: Full cross-chart manual browser verification**

Start the dev server (same pattern as prior tasks) and run one Playwright script that exercises all four charts plus the two untouched ranked-list tables in a single pass:

```js
import { chromium } from 'playwright'

const browser = await chromium.launch({ args: ['--no-sandbox'] })
const page = await browser.newPage({ viewport: { width: 1600, height: 1200 } })
const consoleErrors = []
page.on('console', (msg) => { if (msg.type() === 'error') consoleErrors.push(msg.text()) })
page.on('pageerror', (err) => consoleErrors.push(String(err)))

await page.goto('http://localhost:5173', { waitUntil: 'networkidle' })
await page.fill('#email', 'test@example.com')
await page.fill('#password', 'password123')
await page.click('button[type=submit]')
await page.waitForSelector('.module-card')
await page.click('.module-card')
await page.waitForSelector('text=Sales Trend')

// Sales Trend
await page.hover('.line-chart-canvas')
await page.waitForTimeout(300)
await page.screenshot({ path: '/tmp/final-sales-trend.png' })
await page.mouse.move(10, 10)

// Cumulative Sales
const lineCharts = page.locator('.line-chart-canvas')
await lineCharts.nth(1).scrollIntoViewIfNeeded()
await lineCharts.nth(1).hover()
await page.waitForTimeout(300)
await page.screenshot({ path: '/tmp/final-cumulative.png' })
await page.mouse.move(10, 10)

// Sales By Licensee
const bars = page.locator('.apexcharts-bar-area')
await bars.nth(2).scrollIntoViewIfNeeded()
await bars.nth(2).hover()
await page.waitForTimeout(300)
await page.screenshot({ path: '/tmp/final-licensee.png' })
await page.mouse.move(10, 10)

// Sales By Channel donut
await page.locator('text=Sales By Channel').scrollIntoViewIfNeeded()
await page.hover('.donut-chart')
await page.waitForTimeout(300)
await page.screenshot({ path: '/tmp/final-donut.png' })
await page.mouse.move(10, 10)

// Sanity: ranked-list rows (Sales By Outlet) are untouched by this feature
await page.locator('text=Sales By Outlet').scrollIntoViewIfNeeded()
await page.screenshot({ path: '/tmp/final-ranked-list.png' })

console.log('CONSOLE_ERRORS:', JSON.stringify(consoleErrors))
await browser.close()
```

Expected: `CONSOLE_ERRORS` is `[]`. Look at all five screenshots — each of the four chart screenshots shows its tooltip visible with the correct content (day+MYR value for the two line charts, licensee name+MYR value for the bar, segment label+percentage for the donut), in the original colors, and the ranked-list screenshot shows Sales By Outlet rendering normally (unaffected by this feature).

Stop the dev server when done: `lsof -ti:5173 -sTCP:LISTEN | xargs -r kill`.

- [ ] **Step 6: Commit**

```bash
git add -A
git commit -m "$(cat <<'EOF'
Remove unused static chart assets and orphaned hover CSS

Co-Authored-By: Claude Sonnet 5 <noreply@anthropic.com>
EOF
)"
```
