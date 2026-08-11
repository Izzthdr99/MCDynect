# Chart Hover Interaction Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add pure-CSS hover affordances to the SuperAdmin dashboard's four visual charts (Sales Trend, Cumulative Sales, Sales By Licensee, Sales By Channel) so the dashboard reads as interactive.

**Architecture:** All four charts already have their own inner CSS class in `SuperAdminDashboard.vue`'s `<style scoped>` block (`.line-chart img`, `.bar-track`/`.bar-fill`, `.donut-chart img`). Hover rules are added directly to those existing selectors — no new elements, no JS, no new data. Scoping to the inner element (not `.chart-card`) matters because `.chart-card` is shared with the Sales By Outlet / Sales By State ranked-list tables, which must NOT get this treatment.

**Tech Stack:** Vue 3 `<script setup>`, scoped CSS only (no animation library, no new dependencies).

## Global Constraints

- No test framework exists in this repo (`package.json` has no test script/devDependency) — do not add one for this feature. Verification is manual: run `npm run dev` and check each hover in a browser, per this repo's existing convention (see `docs/superpowers/plans/2026-08-11-sidebar-collapse.md`).
- No tooltips or exact values on hover — the line/donut charts are static exported SVGs with no underlying numeric data (per `docs/superpowers/specs/2026-08-11-chart-hover-design.md`).
- No hover treatment on the ranked-list tables (Sales By Outlet, Sales By State) — out of scope per the approved spec.
- No change to underlying chart data or rendering — SVGs stay static exported assets.
- Transition timing: ~150–200ms ease, consistent with this file's existing `220ms ease` sidebar-margin transition (`.dashboard-main`, `SuperAdminDashboard.vue:313`).

---

### Task 1: Hover affordances on line charts, bar chart, and donut chart

**Files:**
- Modify: `src/pages/SuperAdminDashboard.vue` (style block only — `.line-chart img` around line 556, `.bar-track`/`.bar-fill` around line 654, `.donut-chart img` around line 687)

**Interfaces:**
- Consumes: nothing new — purely additive CSS on existing selectors.
- Produces: nothing consumed by other tasks (this is the only task in the plan).

- [ ] **Step 1: Add scale + drop-shadow hover to the line-chart images**

In `src/pages/SuperAdminDashboard.vue`, find the existing `.line-chart img` rule (around line 556):

```css
.line-chart img {
  flex: 1;
  min-height: 0;
  width: 100%;
  object-fit: fill;
}
```

Replace it with:

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

(Hovering `.line-chart`, the flex wrapper around the `<img>`, rather than the `<img>` itself, keeps the hover zone stable — `transform: scale` on the image itself would otherwise shrink/grow the very box the `:hover` is evaluated against, causing flicker at the scaled edge.)

- [ ] **Step 2: Add brighten + shadow hover to individual licensee bars**

In `src/pages/SuperAdminDashboard.vue`, find the existing `.bar-track` / `.bar-fill` rules (around line 654):

```css
.bar-track {
  width: 100%;
  height: 26px;
}
.bar-fill {
  height: 100%;
  border-radius: 4px;
}
```

Replace them with:

```css
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

- [ ] **Step 3: Add scale + drop-shadow hover to the donut chart image**

In `src/pages/SuperAdminDashboard.vue`, find the existing `.donut-chart` / `.donut-chart img` rules (around line 679):

```css
.donut-chart {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 209px;
  height: 209px;
}
.donut-chart img {
  width: 100%;
  height: 100%;
}
```

Replace them with:

```css
.donut-chart {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 209px;
  height: 209px;
}
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

- [ ] **Step 4: Manual verification in the browser**

Run: `npm run dev`

Open the SuperAdmin dashboard in a browser and verify:
- Hovering the Sales Trend chart scales it slightly with a soft shadow; moving the mouse within the chart area doesn't cause flicker.
- Hovering the Cumulative Sales chart does the same.
- Hovering each individual bar in Sales By Licensee brightens only that bar and shows a pointer cursor; other bars are unaffected.
- Hovering the Sales By Channel donut scales it slightly with a soft shadow.
- Hovering rows in Sales By Outlet or Sales By State shows no new hover effect (confirms scope stayed off the ranked-list tables).

- [ ] **Step 5: Commit**

```bash
git add src/pages/SuperAdminDashboard.vue
git commit -m "$(cat <<'EOF'
Add hover affordances to dashboard charts

Scale/shadow on line and donut charts, per-bar brighten on the
licensee bar chart. Scoped to chart-specific inner elements so the
shared .chart-card wrapper doesn't pull in the ranked-list tables.

Co-Authored-By: Claude Sonnet 5 <noreply@anthropic.com>
EOF
)"
```
