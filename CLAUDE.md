# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page Astro site: a graduation invitation for Juan Diego Gallego Tachack, meant to be shared over WhatsApp. It contains real personal data (name, event address, phone number for RSVP) — treat content edits as changes to a real invitation, not placeholder copy.

## Commands

```sh
npm install
npm run dev       # local dev server
npm run build     # outputs to dist/
npm run preview   # serve the production build locally
```

There is no lint or test setup — the project is intentionally minimal (Astro core only, no UI framework, no Tailwind, no CI checks beyond the deploy build).

## Architecture

- **`src/pages/index.astro`** — the entire page. The frontmatter builds all dynamic URLs as constants: `WHATSAPP_URL` (wa.me link with a pre-filled, URL-encoded RSVP message), `GOOGLE_CAL_URL` / `OUTLOOK_URL` (calendar deep links), and `ICS_URL` (points at the static `.ics` file in `public/`). Markup, per-section `<style>`, and a small vanilla `<script>` (calendar dropdown toggle, scroll-cue dismiss-on-scroll) all live in this one file — there's no component split.
- **`src/layouts/Layout.astro`** — HTML shell: meta tags, OG/Twitter card tags (title/description come in as props from `index.astro`), favicon. `og:image` is built from `Astro.site` + `BASE_URL`.
- **`src/styles/global.css`** — design tokens as CSS custom properties (navy/gold palette, `--font-display`/`--font-body`/`--font-label`), loaded once via `@import` from Google Fonts. No Tailwind.
- **`public/grado-juan-diego.ics`** — a hand-authored static ICS file, not generated at build time. If the event date/time changes, it must be updated **in three places**: this file, the frontmatter constants in `index.astro` (`DTSTART`/`DTEND` and the calendar URL builders), and the visible date/time text in the hero and "Detalles" section.

## Base path gotcha

`astro.config.mjs` sets `base: '/grado-juan/'` (trailing slash matters) because this deploys to a GitHub Pages *project* site (`gabotachak.github.io/grado-juan/`), not a custom domain. Any internal asset/link reference must be prefixed with `import.meta.env.BASE_URL` (see `TOGA_IMG`, `MEDICO_IMG`, `ICS_URL` in `index.astro`) — a root-absolute path like `/juan-toga.jpg` will 404 in production.

## Deployment

`.github/workflows/deploy.yml` builds and deploys to GitHub Pages via `actions/upload-pages-artifact` + `actions/deploy-pages` on every push to `main`. Two non-obvious failure modes seen in practice:

- The repo's `github-pages` **environment** (Settings → Environments) can have a "Deployment branches and tags" rule that only allows a specific branch. If it doesn't include `main`, the deploy job fails instantly with no runner assigned and no logs — check that setting first if deploys mysteriously fail.
- GitHub Pages occasionally returns a transient `Deployment failed, try again later` at the `Getting Pages deployment status` step even though the build succeeded. Re-running the job (or the whole workflow) resolves it; this is not a config problem.
