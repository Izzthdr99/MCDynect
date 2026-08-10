# MCDynect

Vite + Vue 3 + Tailwind CSS v4 + flowbite-vue scaffold for the MC Dynect login page.

## Getting started

```bash
npm install
npm run dev
```

Then open the printed local URL in your browser. `npm run build` produces a production
bundle in `dist/`.

## Structure

- `src/App.vue` — mounts the login page and handles the `submit` event.
- `src/components/McDynectLogin.vue` — the login page component (see
  [its README](src/components/README-McDynectLogin.md) for design-token notes and the
  asset-URL warning).
- `src/style.css` — Tailwind v4 + flowbite entry point.

## Known follow-up

The image assets in `McDynectLogin.vue` still point at temporary Figma CDN URLs that
expire ~7 days after export — see the component's README for how to swap them for local
files before shipping.
