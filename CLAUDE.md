# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
yarn dev          # Dev server at localhost:4321
yarn build        # Type-check + build + jampack optimization (outputs to ./dist)
yarn preview      # Preview production build
yarn lint         # ESLint
yarn format       # Prettier (all files)
yarn format:check # Check formatting without writing
```

> Use `yarn` — `package-lock.json` and `yarn.lock` both exist but the project uses Yarn as its primary package manager.

## Architecture

Built on the **AstroPaper** theme (Astro 4 + Tailwind CSS + React islands). The site is a personal portfolio that doubles as a blog, with `featured: true` posts surfaced as "Projects" on the homepage.

### Content system

All posts live in `src/content/blog/` as `.md` files. The Astro content collection schema is defined in `src/content/config.ts`. Key frontmatter fields:

| Field | Notes |
|-------|-------|
| `pubDatetime` | ISO 8601, required |
| `featured` | `true` → appears in "Projects" section on homepage |
| `draft` | `true` → hidden in production, visible in dev |
| `tags` | Array, defaults to `["others"]` |
| `ogImage` | Optional; auto-generated via Satori/resvg if omitted |

Posts with `draft: true` are filtered out in production by `src/utils/postFilter.ts`. Sorting uses `modDatetime ?? pubDatetime` (see `src/utils/getSortedPosts.ts`).

### Configuration

`src/config.ts` is the single source of truth for site metadata and social links. To activate a social link, set `active: true` and update `href`. Adding a new social platform requires adding its SVG icon to `src/assets/socialIcons.ts` first — the `SocialObjects` type is keyed to that object.

### Theming

Colors are CSS custom properties on `:root` / `html[data-theme]` in `src/styles/base.css`. Tailwind references them via `withOpacity()` wrappers defined in `tailwind.config.cjs`. To change the color scheme, edit the CSS variables there — don't add Tailwind color values directly.

The light/dark toggle is handled by `public/toggle-theme.js` (vanilla JS, runs before hydration to avoid flash).

### Path aliases

`tsconfig.json` defines these aliases (usable in both `.astro` and `.ts`/`.tsx` files):

| Alias | Resolves to |
|-------|-------------|
| `@config` | `src/config.ts` |
| `@assets` | `src/assets/` |
| `@layouts` | `src/layouts/` |
| `@components` | `src/components/` |
| `@utils` | `src/utils/` |

### OG image generation

`src/utils/generateOgImages.tsx` uses Satori + `@resvg/resvg-js` to render OG images at build time. `src/pages/og.png.ts` serves the site-level image; per-post images are generated in `src/layouts/PostDetails.astro`. The `@resvg/resvg-js` package is excluded from Vite's `optimizeDeps` because it ships a native binary.

### Pre-commit hooks

Husky + lint-staged run Prettier on all staged `{js,jsx,ts,tsx,md,mdx,json,astro}` files before each commit. Commits follow Conventional Commits — run `yarn cz` to use the interactive commitizen prompt.
