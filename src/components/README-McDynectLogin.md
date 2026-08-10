# McDynectLogin.vue

Vue 3 (Composition API, `<script setup>`) implementation of the [Figma login frame](https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=756-15010), built on [flowbite-vue](https://flowbite-vue.com).

## Setup

This project scaffold already has this wired up (see `vite.config.js` and `src/style.css`
at the project root). For reference, flowbite-vue 0.4.x runs on **Tailwind CSS v4**
(CSS-based config, no `tailwind.config.js`):

```bash
npm install flowbite flowbite-vue
npm install -D tailwindcss @tailwindcss/vite
```

`vite.config.js`:

```js
import vue from '@vitejs/plugin-vue'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [vue(), tailwindcss()],
})
```

Main CSS entry (`src/style.css`):

```css
@import "tailwindcss";
@import "flowbite/src/themes/default";
@plugin "flowbite/plugin";
@source "../node_modules/flowbite";
@source "../node_modules/flowbite-vue";
```

## Usage

```vue
<script setup>
import McDynectLogin from './components/McDynectLogin.vue'
</script>

<template>
  <McDynectLogin @submit="({ email, password, rememberMe }) => login(email, password, rememberMe)" />
</template>
```

## Design tokens matched

Pulled directly from Figma variables — hardcoded as arbitrary Tailwind values since no project design-token system was available to map onto:

| Token | Value |
|---|---|
| Brand background / CTA | `#911b2d` |
| Heading (Welcome!) | `#54101a` |
| Danger asterisk | `#c70036` |
| Body text | `#4a5565` |
| Placeholder / subtle text | `#6a7282` |
| Input border | `#e5e7eb` |
| Input background | `#f9fafb` |
| Email field radius | `12px` (rounded-base) |
| Password field radius | `8px` (rounded) — matches a genuine inconsistency in the source design |
| Checkbox radius | `4px` |
| CTA button radius | `12px` |
| Heading size | `36px` / `Geist SemiBold` |
| Body font | `Geist` (Regular/Medium/SemiBold/Bold as used) |

## Orbit icon animation

The right panel's 8 orbit icons and the heading/subtext loop a staggered pop-in animation every 2s (extracted from Figma's motion data — originally authored with Framer Motion / `motion.div`). Reimplemented here as two shared CSS `@keyframes` (`orbit-pop`, `fade-up`) with per-element `animation-delay` to reproduce the same stagger, avoiding a `motion`/`framer-motion` runtime dependency.

## ⚠️ Asset URLs expire in ~7 days

The `img*` constants at the top of the component point at Figma's temporary CDN (`figma.com/api/mcp/asset/...`). This session's sandbox couldn't reach `figma.com` to download the bytes locally, so the temporary URLs were kept as-is (per the design-to-code fidelity rule: never fake/placeholder an icon).

**Before shipping**, re-export the same layers from Figma (select the frame → Dev Mode → Assets, or use the Figma MCP `download_assets` tool from an environment with network access) and swap each constant for a local import, e.g.:

```js
// before
const imgLogoMark = 'https://www.figma.com/api/mcp/asset/c42434bd-.../....svg'
// after
import imgLogoMark from './assets/icons/logo-mark.svg'
```

The node IDs for each asset are visible in the Figma dev-mode inspector if you need to re-match them; sizing/positioning classes around each `<img>` won't need to change.
