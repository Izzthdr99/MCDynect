# Sidebar Collapse Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Turn the SuperAdmin dashboard sidebar's collapse toggle from an abrupt `display: none` hack into an animated icon-only rail with hover tooltips.

**Architecture:** `DashboardSidebar.vue` gains a `collapsed` prop that drives CSS-only transitions (width, label opacity/max-width, logo opacity, tooltip reveal). `SuperAdminDashboard.vue` passes its existing `sidebarCollapsed` ref into the new prop and animates `.dashboard-main`'s `margin-left` to match, instead of hiding the sidebar outright. No new toggle button — `DashboardTopnav.vue` already emits `toggle-sidebar`, already wired to `sidebarCollapsed`.

**Tech Stack:** Vue 3 `<script setup>`, scoped CSS transitions (no animation library, no new dependencies).

## Global Constraints

- No test framework exists in this repo (`package.json` has no test script/devDependency) — do not add one for this feature. Verification is manual: run `npm run dev` and check the behavior in a browser, per this repo's existing convention (no other component in `src/` has tests).
- Collapsed rail width: `72px`. Expanded width: `251px` (unchanged, already in `.sidebar`).
- Transition timing: `220ms ease` for width/margin, `160ms ease` for label/logo opacity — keep both files' timings identical so the sidebar and content shift in sync.
- Do not persist collapsed state (e.g. localStorage) — out of scope per the approved spec (`docs/superpowers/specs/2026-08-11-sidebar-collapse-design.md`).
- Do not introduce a new icon-only logo mark asset — `full-logo.svg` is a wordmark only; it fades out when collapsed instead of being replaced.

---

### Task 1: Animated collapse rail on DashboardSidebar, wired into SuperAdminDashboard

**Files:**
- Modify: `src/components/DashboardSidebar.vue`
- Modify: `src/pages/SuperAdminDashboard.vue:93-94` (template), `src/pages/SuperAdminDashboard.vue:299-314` (style)

**Interfaces:**
- Consumes: nothing new — `DashboardTopnav.vue`'s existing `toggle-sidebar` emit and `SuperAdminDashboard.vue`'s existing `sidebarCollapsed` ref (`src/pages/SuperAdminDashboard.vue:33`) are unchanged.
- Produces: `DashboardSidebar` prop `collapsed: Boolean` (default `false`). Any future page embedding `DashboardSidebar` can pass `:collapsed="..."` to get the same rail behavior.

- [ ] **Step 1: Add the `collapsed` prop to `DashboardSidebar.vue`**

In `src/components/DashboardSidebar.vue`, replace the `defineProps` call:

```js
defineProps({
  activeItem: {
    type: String,
    default: 'dashboard',
  },
  collapsed: {
    type: Boolean,
    default: false,
  },
})
```

- [ ] **Step 2: Bind the collapsed class and label/tooltip markup in the template**

In `src/components/DashboardSidebar.vue`, replace the full `<template>` block with:

```html
<template>
  <aside class="sidebar" :class="{ collapsed }">
    <div class="sidebar-header">
      <img class="logo" :src="iconFullLogo" alt="MC Dynect" />
    </div>

    <nav class="sidebar-nav">
      <button
        v-for="item in navItems"
        :key="item.key"
        type="button"
        class="nav-item"
        :class="{ active: activeItem === item.key }"
        :title="collapsed ? item.label : undefined"
        @click="emit('navigate', item.key)"
      >
        <span class="nav-icon-slot">
          <img :src="item.icon" :style="{ width: item.width + 'px', height: item.height + 'px' }" alt="" />
        </span>
        <span class="nav-label">{{ item.label }}</span>
        <span class="nav-tooltip" role="tooltip">{{ item.label }}</span>
      </button>
    </nav>

    <div class="sidebar-footer">
      <button
        type="button"
        class="nav-item"
        :title="collapsed ? 'Settings' : undefined"
        @click="emit('navigate', 'settings')"
      >
        <span class="nav-icon-slot">
          <img
            :src="settingsIcon.icon"
            :style="{ width: settingsIcon.width + 'px', height: settingsIcon.height + 'px' }"
            alt=""
          />
        </span>
        <span class="nav-label">Settings</span>
        <span class="nav-tooltip" role="tooltip">Settings</span>
      </button>
    </div>
  </aside>
</template>
```

- [ ] **Step 3: Add collapse transitions to `DashboardSidebar.vue`'s `<style>`**

In `src/components/DashboardSidebar.vue`, in the `<style scoped>` block:

Replace the `.sidebar { ... }` rule with:

```css
.sidebar {
  --nav-active-bg: #f9dfe3;
  --nav-active-text: #54101a;
  --nav-text: #4a5565;
  --border: #e5e7eb;

  display: flex;
  flex-direction: column;
  position: fixed;
  top: 0;
  left: 0;
  width: 251px;
  height: 100vh;
  background: #ffffff;
  border-right: 1px solid var(--border);
  overflow-y: auto;
  overflow-x: hidden;
  transition: width 220ms ease;
  z-index: 20;
}
.sidebar.collapsed {
  width: 72px;
}
```

(This adds `overflow-x: hidden` — needed so the tooltip and shrinking label don't cause horizontal scroll — and the `transition`/`.collapsed` rule. Everything else in the rule is unchanged from today.)

Replace the `.logo { ... }` rule with:

```css
.logo {
  height: 20px;
  width: auto;
  max-width: 130px;
  opacity: 1;
  transition: opacity 160ms ease, max-width 220ms ease;
}
.sidebar.collapsed .logo {
  opacity: 0;
  max-width: 0;
}
```

Replace the `.nav-item { ... }` rule with:

```css
.nav-item {
  position: relative;
  display: flex;
  align-items: center;
  gap: 10px;
  width: 100%;
  padding: 10px 12px;
  border: none;
  border-radius: 8px;
  background: transparent;
  font-family: inherit;
  font-size: 14px;
  font-weight: 500;
  color: var(--nav-text);
  text-align: left;
  cursor: pointer;
  transition: background 0.15s ease, color 0.15s ease;
}
```

(Only addition: `position: relative`, so the tooltip below can anchor to it.)

After the existing `.nav-icon-slot { ... }` rule, add:

```css
.nav-label {
  overflow: hidden;
  white-space: nowrap;
  max-width: 160px;
  opacity: 1;
  transition: opacity 160ms ease, max-width 220ms ease;
}
.sidebar.collapsed .nav-label {
  opacity: 0;
  max-width: 0;
}

.sidebar.collapsed .nav-item {
  justify-content: center;
  padding: 10px;
}

.nav-tooltip {
  position: absolute;
  left: calc(100% + 12px);
  top: 50%;
  padding: 6px 10px;
  border-radius: 6px;
  background: #101828;
  color: #ffffff;
  font-size: 12px;
  font-weight: 500;
  white-space: nowrap;
  opacity: 0;
  visibility: hidden;
  pointer-events: none;
  transform: translate(-6px, -50%);
  transition: opacity 160ms ease, transform 160ms ease, visibility 160ms;
  z-index: 30;
}
.sidebar.collapsed .nav-item:hover .nav-tooltip,
.sidebar.collapsed .nav-item:focus-visible .nav-tooltip {
  opacity: 1;
  visibility: visible;
  transform: translate(0, -50%);
}
```

- [ ] **Step 4: Wire `sidebarCollapsed` into `DashboardSidebar` from `SuperAdminDashboard.vue`**

In `src/pages/SuperAdminDashboard.vue:94`, change:

```html
    <DashboardSidebar :active-item="activeNavItem" @navigate="handleNavigate" />
```

to:

```html
    <DashboardSidebar :active-item="activeNavItem" :collapsed="sidebarCollapsed" @navigate="handleNavigate" />
```

- [ ] **Step 5: Replace the display:none hack with an animated margin in `SuperAdminDashboard.vue`**

In `src/pages/SuperAdminDashboard.vue`, replace this block (around line 299):

```css
.dashboard.sidebar-collapsed :deep(.sidebar) {
  display: none;
}
.dashboard.sidebar-collapsed .dashboard-main {
  margin-left: 0;
}
```

with:

```css
.dashboard.sidebar-collapsed .dashboard-main {
  margin-left: 72px;
}
```

Then find the `.dashboard-main { ... }` rule (around line 306) and add a `transition`:

```css
.dashboard-main {
  display: flex;
  flex-direction: column;
  flex: 1;
  min-width: 0;
  /* Sidebar is position: fixed (out of flex flow), so its width has to be
     reserved here manually instead of the flexbox row sizing it. */
  margin-left: 251px;
  transition: margin-left 220ms ease;
}
```

- [ ] **Step 6: Manually verify in the browser**

Run:

```bash
npm run dev
```

Open the dashboard in a browser (navigate to the SuperAdmin dashboard view). Confirm:
1. Clicking the sidebar toggle icon in the topnav smoothly animates the sidebar from 251px to 72px (no snapping/flashing), and the main content shifts in sync.
2. At 72px, only icons are visible — logo and nav labels have faded out, not clipped abruptly.
3. Hovering (or tab-focusing) a nav icon while collapsed reveals a dark tooltip with the item's label to the right of the icon.
4. Clicking the toggle again smoothly expands back to 251px with labels and logo fading back in.
5. No horizontal scrollbar appears on the sidebar at any point during the transition.
6. Clicking a nav item still calls `handleNavigate` correctly (active state highlight still works) in both collapsed and expanded states.

Stop the dev server once confirmed (Ctrl+C).

- [ ] **Step 7: Commit**

```bash
git add src/components/DashboardSidebar.vue src/pages/SuperAdminDashboard.vue
git commit -m "Add animated collapse rail to dashboard sidebar"
```

---

## Self-Review Notes

- **Spec coverage:** icon-only rail ✅ (Step 3), animation ✅ (width/opacity/margin transitions), hover tooltip ✅ (Step 3 `.nav-tooltip`), logo fade (no new mark) ✅ (Step 3 `.logo`), reuse existing topnav toggle ✅ (no new button added), parent wiring replaces `display:none` hack ✅ (Step 5), no persistence / no new logo asset ✅ (not implemented, per Global Constraints).
- **Placeholder scan:** none found — every step has literal code.
- **Type consistency:** `collapsed` prop name matches between `defineProps` (Step 1), template binding (Step 2), CSS selector `.sidebar.collapsed` (Step 3), and the parent's `:collapsed="sidebarCollapsed"` binding (Step 4).
