# Dashboard Sidebar Collapse

## Problem

`DashboardSidebar.vue` currently supports a "collapsed" state only via a parent-level
CSS hack (`SuperAdminDashboard.vue`'s `.sidebar-collapsed :deep(.sidebar) { display: none }`),
which abruptly hides the entire sidebar with no animation and no icon-only affordance.

## Design

- `DashboardSidebar.vue` gains a `collapsed: Boolean` prop.
- Collapsed state renders an icon-only rail (72px wide, down from 251px), animated via
  a CSS `width` transition (~220ms ease) on `.sidebar`.
- Nav item labels fade/collapse out (`opacity` + `max-width` transition) rather than
  clipping abruptly.
- The header logo (`full-logo.svg`, a wordmark with no separate icon mark in the asset
  set) fades out when collapsed; no substitute mark is introduced.
- When collapsed, hovering (or focusing) a nav button reveals its label in a floating
  tooltip positioned to the right of the icon, built in pure CSS. Each button also gets
  a native `title` attribute as a non-hover/screen-reader fallback.
- `SuperAdminDashboard.vue` passes `:collapsed="sidebarCollapsed"` to `DashboardSidebar`
  and replaces the `display: none` hack with an animated `margin-left` transition on
  `.dashboard-main` (251px ↔ 72px), matching the sidebar's timing.
- No new toggle button — the existing `DashboardTopnav.vue` icon button already emits
  `toggle-sidebar`, which `SuperAdminDashboard.vue` already wires to `sidebarCollapsed`.

## Out of scope

- Persisting collapsed state (e.g. localStorage).
- A dedicated icon-only logo mark asset.
