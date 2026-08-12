# Finance Dashboard

## Problem

The workspace selector's "Finance" module card (`McDynectWorkspace.vue`) currently just
`console.log`s the selection — there is no Finance dashboard to route to, unlike
`super-admin` and `licensing` which already have landing pages. Figma has a Finance
dashboard design (node 1115-23370) that needs implementing.

## Design

Add `src/pages/FinanceDashboard.vue`, following the exact shell pattern already
established by `SuperAdminDashboard.vue` / `LicensingDashboard.vue`:

- `DashboardSidebar` + `DashboardTopnav` reused as-is, no component changes.
- `DashboardTopnav` gets `:show-workspace-switcher="false"` (existing prop) to hide the
  "Super Admin ▾" role-switcher pill, matching what `LicensingDashboard.vue` already does.
- Sidebar collapse is already built into `DashboardSidebar`/`DashboardTopnav` (toggle
  button, width transition, hover tooltips) — wired via a local `sidebarCollapsed` ref,
  same as the other two dashboards. No changes to the sidebar/topnav components.
- Sidebar nav items (flat list, no chevrons, per Figma): Overview, Item Category,
  Company, Order History, Order Request, Outlet Sales Analysis, Outlet, Vendor
  Registration Review, PR Approvals, Purchase Order Deliveries, Trade In, Redemption
  History, Licensee Fees, SQL Data, Invoice Entities. 7 reuse existing icons
  (`sidebar/licensing/icon-{grid,tag,cart,bar-chart,store-alt,file-lines,arrows-repeat}.svg`);
  8 are new, downloaded from Figma into `src/assets/dashboard/sidebar/finance/`.
- Header section ("Hello, Izzthdr." / Filter dropdown / Layers button) reused verbatim
  from the existing pattern — same markup/behavior as Licensing/SuperAdmin.
- 3 stat cards (Total Expenses, Other Expenses, Net Profit), same `stat-card` markup as
  `LicensingDashboard.vue`'s stat row. 3 new icon assets for the colored icon squares.
- 2 chart widgets in a `widget-grid` row:
  - "Sales vs. Expenses" — grouped bar chart (Mon–Sun, 2 series: Sales/Expenses),
    reusing the ApexCharts grouped-bar config pattern from `LicensingDashboard.vue`'s
    "State Restock vs. Sales" widget.
  - "Expense Breakdown By Category" — donut chart (Office/Travel/Marketing/Utilities/
    Salaries, center label showing the largest segment), reusing the ApexCharts donut
    config pattern from `SuperAdminDashboard.vue`'s "Sales By Channel" widget, extended
    from 2 to 5 segments.
- All numeric data (stat values, chart series, donut percentages) is fabricated
  placeholder data sized to match the Figma screenshot, same as every existing stat/chart
  in `SuperAdminDashboard.vue` and `LicensingDashboard.vue` — there is no backend yet.
- `App.vue`'s `handleModuleSelect('finance')` routes to `FinanceDashboard`, same as
  `'licensing'`/`'super-admin'` do today. `back-to-workspace`/`sign-out` emits wired the
  same way.

## Out of scope

- Any other module card in `McDynectWorkspace.vue` (procurement, production, etc.) —
  still no-ops.
- Real data/API wiring for Finance figures.
- Sub-navigation for any sidebar item (none have chevrons in this design).
