# UI Component Library — Phase 1 Design

## Goal

Turn the design tokens already centralized in `src/style.css` (pulled from
the **MC'D Design System II** Figma file) into a small set of reusable Vue
components, so future screens don't have to re-derive styling by hand the
way `McDynectLogin.vue` currently does.

## Scope

The Figma file is a full enterprise dashboard kit (a "Dynectus"-style
template) with 100+ pre-built pages and dozens of component sets — Button,
Input field, Checkbox/Radio form, Select form, Dropdown menu, Table cell,
Pagination, Toast, Modal footer, Bottom navigation, and more. Porting all
of it is a multi-phase effort, not a single build. This spec covers
**Phase 1 only**: the 8 components below. Everything else (icon set, brand
logo assets, remaining component sets, full page layouts) is backlog —
each becomes its own spec when it's actually needed.

**Phase 1 catalog:** Button, Input, Checkbox, Badge, Avatar, Alert,
NavigationListItem, Sidebar.

## Sources

Two tiers of fidelity, stated plainly:

- **Sidebar, Avatar, NavigationListItem, Alert, Badge** — pulled directly
  via `get_design_context` on the Sidebar frame (`fileKey
  RuMycmM9hWuGC9Ku7Zw203`, node `1057:2041`) in the MC'D Design System II
  file. Colors, spacing, radii, and states below are the real resolved
  values from that call.
- **Button, Input, Checkbox** — the design system has dedicated
  component sets for these (`Button`, `Input field`, `Checkbox/Radio
  form`), but they live in a separate published library file this
  session doesn't have direct node access to. Instead, these three
  generalize the styling already verified against Figma in
  `McDynectLogin.vue` (itself generated from a Figma design-to-code
  pass), extended with secondary/danger variants and states using the
  same token system. If the dedicated Button/Input/Checkbox frames are
  needed pixel-exact later, that's a follow-up: share the specific node
  link and the component gets rebuilt against real variant data.

## Architecture

- New directory: `src/components/ui/`
- One Vue 3 `<script setup>` SFC per component, PascalCase filenames
  matching the component name (`Button.vue`, `Input.vue`, `Checkbox.vue`,
  `Badge.vue`, `Avatar.vue`, `Alert.vue`, `NavigationListItem.vue`,
  `Sidebar.vue`)
- No new runtime dependency. Built with Tailwind utility classes bound to
  the tokens already in `style.css` (`brand-700`, `brand-900`,
  `danger-600`, `body`, `muted`, `line`, `surface`). Where a component
  needs a color not yet tokenized (see **New tokens** below), it's added
  to `style.css`'s `@theme` block rather than hardcoded in the component.
- These components replace flowbite-vue's `FwbButton` / `FwbInput` /
  `FwbCheckbox` for **new** usage. `McDynectLogin.vue` is left as-is in
  this phase — migrating it onto the new components is a separate,
  explicit follow-up, not a silent side effect of this work.
- Components are presentational only: props in, events out, no internal
  data fetching or routing.

## New tokens required

A few colors show up in the Figma data that aren't in `style.css` yet.
Adding them to the `@theme` block as part of this work:

| Token | Value | Used by |
|---|---|---|
| `--color-ink` | `#101828` (already added, currently unused) | Alert title, NavigationListItem active state |
| `--color-surface-dark` | `#1e2939` | Alert CTA button |
| `--color-danger-soft` | `#fef0f2` | Badge background |
| `--color-danger-subtle` | `#ffccd3` | Badge border |
| `--color-danger-strong` | `#8b0836` | Badge text |
| `--color-success` | `#00a63e` | Avatar status dot |
| `--color-surface-tertiary` | `#f3f4f6` | Avatar remove-button background |

## Component specs

### 1. Button

Props: `variant: 'primary' \| 'secondary' \| 'danger'` (default `primary`),
`pill: boolean` (default `false`), `loading: boolean`, `disabled: boolean`,
`type: 'button' \| 'submit'` (default `button`).

- `primary`: `bg-brand-700 text-white`, hover darkens slightly
  (`hover:bg-brand-900`)
- `secondary`: `bg-surface text-body border border-line`
- `danger`: `bg-danger-600 text-white`
- Shared: `rounded-xl px-4 py-2.5 text-base font-medium leading-6
  disabled:opacity-50 disabled:cursor-not-allowed`, `pill` swaps
  `rounded-xl` for `rounded-full`
- `loading` shows a small inline spinner before the slot content and sets
  `disabled`
- Slot-based content (not a hardcoded `label` prop), matching how
  `McDynectLogin.vue` already calls its button

### 2. Input

Props: `modelValue: string`, `label: string`, `required: boolean`,
`type: string` (default `text`), `placeholder: string`, `error: string`
(optional — presence puts the field in error state), `id` (auto-generated
if absent, for label association).

- Label: `text-sm font-medium leading-5`, brand-colored
  (`text-brand-700`) to match the existing Email/Password labels, with a
  `text-danger-600` asterisk when `required`
- Field: `rounded-xl border border-line bg-surface px-3 py-2.5 text-sm
  text-muted placeholder:text-muted shadow-[0px_1px_0.25px_0px_rgba(29,41,61,0.02)]
  focus:border-brand-700 focus:ring-brand-700`
- Error state: border/ring swap to `border-danger-600
  focus:ring-danger-600`, error message rendered below in
  `text-xs text-danger-600`
- Emits `update:modelValue` (v-model compatible)

### 3. Checkbox

Props: `modelValue: boolean`, `label: string`, `disabled: boolean`.

- Input: `size-4 rounded-[4px] border border-line bg-surface
  text-brand-700 focus:ring-brand-700`
- Label: `text-sm font-medium leading-4 text-body`
- Emits `update:modelValue`

### 4. Badge

Props: `variant: 'danger' \| 'neutral'` (default `danger`), slot content
(typically a short number/word).

- `danger`: `bg-danger-soft border border-danger-subtle text-danger-strong`
- `neutral`: `bg-surface border border-line text-body`
- Shared: `rounded-full size-5 flex items-center justify-center text-xs
  font-medium`

### 5. Avatar

Props: `src: string`, `alt: string`, `size: number` (default `32`),
`showStatusDot: boolean`, `showRemoveButton: boolean`. Emits `remove`.

- Root: `relative rounded-full` sized via inline `style` from the `size`
  prop (matches Figma's `size-[var(--spacing/8,32px)]` pattern)
- Image: `absolute inset-0 rounded-full object-cover size-full`
- Status dot: `bg-success size-2.5 rounded-full absolute right-0 top-0`
- Remove button: `bg-surface-tertiary rounded-full size-4 absolute
  left-[22px] top-0`, renders a close icon, emits `remove` on click

### 6. Alert

Props: `title: string`, `description: string`, `actionLabel: string`
(optional), `dismissible: boolean` (default `true`). Emits `dismiss`,
`action`.

- Container: `bg-surface border border-line rounded-xl p-4 flex flex-col
  gap-4`
- Title row: icon + `text-ink text-base font-medium leading-4` title +
  optional close button (visible when `dismissible`, emits `dismiss`)
- Description: `text-ink text-sm leading-5`
- Action button (rendered when `actionLabel` given): `bg-surface-dark
  text-white text-xs font-medium rounded-xl px-3 py-1.5`, emits `action`

### 7. NavigationListItem

Props: `label: string`, `icon` (component or slot), `active: boolean`,
`disabled: boolean`, `badge: string \| number` (optional), `expandable:
boolean`, `expanded: boolean`, `indent: boolean` (for nested items under
an expanded parent).

- Row: `flex items-center justify-between px-2 py-1.5 rounded-xl w-full`,
  `indent` adds `pl-8`
- `active`: `bg-surface`, label text `text-brand-700`
- default: label text `text-body`
- `disabled`: `opacity-50 pointer-events-none`
- Right side: `Badge` (if `badge` given) or a chevron (if `expandable`,
  rotated based on `expanded`)

### 8. Sidebar

Props: `type: 'default' \| 'with-alert' \| 'double'` (default `default`),
`contracted: boolean`, `items: NavItem[]` (each `{ label, icon, badge?,
children? }`), `footerItems: NavItem[]`, `user?: { name, email,
avatarSrc }`.

- Composes `NavigationListItem` for the main list and footer list,
  `Avatar` for the `with-alert`/`double` user row, `Alert` for the
  dashboard-announcement card in the default (non-contracted) variant
- Container: `bg-white flex flex-col h-full w-72 p-5 gap-5` (`contracted`
  narrows to `w-16`, centers icon-only buttons, drops labels)
- `double`: renders a fixed 64px icon rail (`Left sidebar`) alongside the
  full 288px panel (`Right sidebar`), matching the Figma variant's two-pane
  layout
- This is the most complex component in the set — it's the one place in
  Phase 1 that composes everything else, which is intentional: it proves
  the primitives work together before any real page is built on top of
  them

## Testing approach

No page in this repo currently mounts these components (the login screen
stays on flowbite-vue in this phase). Verification is a temporary preview:
a scratch route/file that renders one instance of each component with
representative props, screenshotted via the existing headless-Chromium
Playwright setup (same approach used to verify the `style.css` token
change), then deleted once visually confirmed against the Figma
screenshots already captured for Sidebar/Avatar/Alert/Badge. Button/Input/
Checkbox are checked against the already-approved login page rendering
instead, since that's their source of truth.

## Explicitly out of scope (backlog for future phases)

- Icon set (Icons page) — dozens of individual SVGs, needs its own
  Icon-component spec (sprite vs. per-icon components)
- Remaining component sets: Select, Dropdown menu, Table cell, Pagination,
  Toast, Modal footer, Drawer footer, Bottom navigation, Search form,
  Rich text editor input, and others surfaced in the design-system search
- Full pre-built page layouts (dashboard, ecommerce, settings, pricing,
  etc.)
- Migrating `McDynectLogin.vue` onto the new `Button`/`Input`/`Checkbox`
  components
