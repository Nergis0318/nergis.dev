# AGENTS.md

## Commands
- `bun run dev` (or `npm start`): Astro dev server. Binds `0.0.0.0:4321` (astro.config.mjs), not localhost default.
- `bun run build`: outputs to `dist/`.
- `bun run preview`: serves the built `dist/`.
- `bun run astro -- <cmd>`: direct Astro CLI passthrough.

No `lint`, `test`, `typecheck`, `format`, or CI scripts exist. No vitest/jest/eslint/biome configs at repo root.

## Architecture
- Single Astro 4 app ("nergis-portfolio"). Entry: `src/pages/index.astro` (imports Layout + Nav/Hero/About/Projects/Contact).
- `src/layouts/Layout.astro` is the wiring point:
  - Imports `global.css`, `pretendard-gov` variable CSS, specific `@fontsource/jetbrains-mono` weights.
  - `html lang="ko"`.
  - Inline script walks `document.body` text nodes; any leaf whose trimmed text is pure ASCII (`/^[\x20-\x7E]+$/`) is wrapped in `<span lang="en">`. This is how mixed Korean/English text gets the right font.
- Fonts are not automatic:
  - Default body: Pretendard GOV Variable (Korean + CJK).
  - `:lang(en)` in `src/styles/global.css` forces JetBrains Mono (with `font-feature-settings: 'liga' 0, 'calt' 0`).
- Components: each has its own `<style>` (Astro scoped). Global tokens and layout in `global.css`.
- Reveal animations: `.reveal` + `.is-visible` toggled by IntersectionObserver in Layout (threshold 0.12, rootMargin bottom -40px).

## Font QA tools (root)
These are the executable sources of truth for bilingual font rendering:
- `node font-check.mjs` (and `font-glyph-check.mjs`, `font-lang-check.mjs`, `font-screenshot.mjs`, `font-survey.mjs`)
- They drive Puppeteer against `http://127.0.0.1:4321/` and assert computed `fontFamily` + sizes on specific selectors (h1.name, .kicker, .stack-items li, cards, etc.).

Run `bun run dev` first; these scripts expect the dev server.

## Config & generated
- `tsconfig.json` extends `astro/tsconfigs/strict`.
- `.astro/` and `dist/` are generated (see `.gitignore`).
- No `opencode.json`, no other root instruction files.
- `public/favicon.svg` only static asset besides fonts.

## Gotchas an agent will hit
- Editing Korean vs. ASCII text: the runtime `<span lang="en">` wrapper is inserted client-side. Pure-ASCII strings must stay ASCII-only or the font switch breaks.
- Dev server is reachable on the LAN/IP because of the 0.0.0.0 bind; `localhost` works but tests that hard-code it (like font-check) use 127.0.0.1:4321.
- No test/lint gate; changes that would normally be caught elsewhere will ship unless you run the font scripts manually.
