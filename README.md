# Wekaflow Automation — Portfolio & GEO Case Study Site

A minimal [Astro](https://astro.build) static site built to showcase "Generative Engine Optimization" (GEO) case studies — technical write-ups on making content retrievable and citable by AI answer engines (ChatGPT, Perplexity, Gemini) rather than traditional search engines.

> **Note on this document:** This README was written after a direct audit of the repository's source files (`package.json`, `astro.config.mjs`, `src/`). It documents what is actually implemented, not aspirational features. Sections that describe project gaps say so explicitly.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Problem It Solves](#problem-it-solves)
- [Features Actually Implemented](#features-actually-implemented)
- [Tech Stack](#tech-stack)
- [Architecture Overview](#architecture-overview)
- [Folder Structure](#folder-structure)
- [Installation](#installation)
- [Environment Variables](#environment-variables)
- [Running Locally](#running-locally)
- [Deployment](#deployment)
- [Screenshots](#screenshots)
- [Roadmap](#roadmap)
- [Challenges & Lessons Learned](#challenges--lessons-learned)

---

## Project Overview

**Wekaflow Automation** is a single-purpose Astro site consisting of:

1. A homepage that lists blog-style "technical case studies."
2. A dynamic blog route that renders individual Markdown posts.
3. A shared layout that injects `schema.org` JSON-LD structured data (`Article`/`WebPage`) into every page for machine-readability by AI crawlers and search engines.

It is currently a **content/portfolio site**, not an application with user accounts, a backend, or a database.

## Problem It Solves

The site's stated thesis (visible in its homepage copy and its one published post) is that traditional SEO — built around backlinks, keyword density, and domain age — does not guarantee visibility inside Retrieval-Augmented Generation (RAG) pipelines used by AI answer engines. The project positions itself as a demonstration/portfolio piece for **"Generative Engine Optimization" (GEO)**: structuring page content and metadata (headings, JSON-LD, semantic entity data) so that LLM-based search products can parse, retrieve, and cite it.

In short: it's a marketing/portfolio site for a one-person "AI Search Visibility" consulting service, and it uses its own architecture (structured data, clean markup) as a proof of concept for the service it's promoting.

## Features Actually Implemented

Based on the code in this repository, the following are implemented:

- **Static homepage** (`src/pages/index.astro`) that pulls all entries from the `blog` content collection and renders them as cards linking to individual posts.
- **Dynamic blog routing** (`src/pages/blog/[...slug].astro`) using Astro's Content Collections API (`getCollection`, `getStaticPaths`, `render`) to statically generate one page per Markdown file.
- **Content Collections schema** (`src/content.config.ts`) validating blog frontmatter (`title`, `description`, `publishDate`, optional `author`) via Zod, loaded from `src/content/blog/**/*.md` using Astro's `glob` loader.
- **JSON-LD structured data injection** (`src/components/Schema.astro`) — every page emits a `<script type="application/ld+json">` block describing itself as a `TechArticle` (if it has a `publishDate`) or `WebPage`, with hardcoded `author` and `publisher` metadata.
- **Shared page layout** (`src/layouts/MainLayout.astro`) providing `<title>`, meta description, canonical URL, and a simple nav bar. All styling is **inline CSS** on individual elements — there is no external stylesheet or CSS framework.
- **One published blog post** (`src/content/blog/llm-retrieval-blind-spot.md`) demonstrating the site's own content strategy.

**Not implemented** (explicitly absent from the codebase, so listed here rather than assumed):

- No test suite of any kind (no `test`/`vitest`/`jest` scripts or config).
- No CI/CD configuration (no `.github/workflows`, no other pipeline config).
- No linting or formatting configuration (no ESLint/Prettier config files).
- No environment variables or `.env` usage anywhere in the code.
- No backend, API routes, database, or CMS integration.
- No image assets, logo, or `public/` directory — the JSON-LD schema references a logo URL (`/assets/logo.png`) that does not exist in the repo.
- No `LICENSE` file.
- No `.gitignore` — as a result, `node_modules/` currently appears to be committed to version control in this repo, which should be fixed (see [Roadmap](#roadmap)).

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | [Astro](https://astro.build) `^6.4.7` (static site generation) |
| Language | TypeScript `^6.0.3` (strict config, via `astro/tsconfigs/strict`) |
| Content | Markdown + Astro Content Collections, validated with Zod (bundled with Astro) |
| Styling | Inline CSS only — no Tailwind, CSS Modules, or Sass |
| Package manager | npm (`package-lock.json` present) |
| Hosting target | Vercel (inferred from `astro.config.mjs` `site` value: `https://wekaflow.vercel.app`) |

No UI framework (React/Vue/Svelte) is integrated — this is plain `.astro` components only.

## Architecture Overview

The site follows Astro's default static-first architecture: everything is prerendered to HTML at build time, with no client-side JavaScript framework in use.

```
Request for "/"                     Request for "/blog/{slug}"
      │                                       │
      ▼                                       ▼
index.astro                       [...slug].astro
  - getCollection('blog')           - getStaticPaths() enumerates
  - renders post list                 all blog entries at build time
      │                              - render(entry) compiles the
      ▼                                Markdown body to HTML
MainLayout.astro  ◄────────────────────────┘
  - <head>: title, meta description, canonical URL
  - <Schema /> injects JSON-LD
  - <nav> + <slot /> for page body
      │
      ▼
Schema.astro
  - Builds a schema.org JSON-LD object
    (TechArticle or WebPage) and
    inlines it as a <script> tag
```

Content flows one way: Markdown files in `src/content/blog/` → validated against the Zod schema in `content.config.ts` → queried via `getCollection`/`getStaticPaths` → rendered into `.astro` page templates → output as static HTML at build time.

There is no client/server split, no runtime data fetching, and no state management — all data is resolved at build time.

## Folder Structure

```
wekaflow-portfolio/
├── astro.config.mjs        # Astro config; sets `site` to the Vercel URL
├── package.json             # Dependencies + npm scripts
├── tsconfig.json            # Extends Astro's strict TS config
├── src/
│   ├── content.config.ts    # Defines the `blog` collection + Zod schema
│   ├── components/
│   │   └── Schema.astro     # JSON-LD structured data component
│   ├── layouts/
│   │   └── MainLayout.astro # Shared <head>/<nav> wrapper for every page
│   ├── content/
│   │   └── blog/
│   │       └── llm-retrieval-blind-spot.md   # The one existing post
│   └── pages/
│       ├── index.astro              # Homepage / case study listing
│       └── blog/
│           └── [...slug].astro      # Dynamic per-post route
└── .astro/                  # Astro's generated type/content cache (auto-generated, not hand-authored)
```

There is **no `public/` directory**, no `.gitignore`, and no `LICENSE` file in the repository as cloned.

## Installation

```bash
git clone https://github.com/mellowmellow23/wekaflow-portfolio.git
cd wekaflow-portfolio
npm install
```

Requires Node.js compatible with Astro 6 (Node 18.20.8+, 20.3.0+, or 22.0.0+ per Astro's current requirements — verify against your installed version).

## Environment Variables

**None are used anywhere in this codebase.** There is no `.env`, `.env.example`, or any reference to `process.env` / `import.meta.env` in the source. No API keys, secrets, or configuration are required to run or build the project locally.

## Running Locally

```bash
npm run dev       # starts the Astro dev server (default: http://localhost:4321)
```

Other available scripts (from `package.json`):

```bash
npm run build     # Type-checks and builds the static site to ./dist
npm run preview   # Serves the built ./dist output locally
npm run astro     # Direct access to the Astro CLI (e.g. `npm run astro -- add`)
```

## Deployment

`astro.config.mjs` sets `site: 'https://wekaflow.vercel.app'`, and a commit message in the repo's history (`fix: update git config email to authorize vercel build`) indicates the project is deployed via **Vercel**. However:

- There is **no `vercel.json`** or Vercel-specific configuration file in the repository.
- There is **no Astro adapter installed** (e.g. `@astrojs/vercel`) — the project currently relies on Astro's default static output, which Vercel can serve directly as a static build, but no explicit adapter/integration is configured in `astro.config.mjs`.
- There is no CI/CD workflow file; deployment is presumably triggered by Vercel's Git integration outside of this repo (not something visible in the codebase itself).

To deploy elsewhere (Netlify, GitHub Pages, any static host), run `npm run build` and serve the resulting `dist/` directory — no server runtime is required since the project is fully static.

## Screenshots

> _Screenshots not yet included in the repository. Add images here once available._

| Homepage | Blog Post |
|---|---|
| `![Homepage placeholder](docs/screenshots/homepage.png)` | `![Blog post placeholder](docs/screenshots/blog-post.png)` |

## Roadmap

The following are gaps identified during this audit, not confirmed future plans by the maintainer — listed here as reasonable next steps given the current state of the code:

- [ ] Add a `.gitignore` and remove the accidentally committed `node_modules/` directory from version control.
- [ ] Add a `LICENSE` file.
- [ ] Add the missing `/assets/logo.png` referenced by `Schema.astro`, or remove that reference until it exists.
- [ ] Fix the placeholder LinkedIn URL (`your-profile-url`) hardcoded in `Schema.astro`.
- [ ] Add a `public/` directory for static assets (favicon, logo, social preview image) — none currently exist.
- [ ] Add a testing setup (even minimal smoke tests for build output).
- [ ] Add CI (GitHub Actions or similar) to run builds/lint on PRs.
- [ ] Consider adding an explicit Vercel adapter/config or a `vercel.json` if any non-static behavior is planned.
- [ ] Add more blog/case-study content — only one post currently exists.
- [ ] Extract the large amount of inline CSS in `MainLayout.astro` and `index.astro` into a proper stylesheet as the site grows.

## Challenges & Lessons Learned

These are inferred from the commit history and code, not stated explicitly by the maintainer, so treat them as an outside reviewer's read rather than the author's own account:

- **Astro v6 Content Collections API churn:** Comments in `content.config.ts` (`// v6 Requirement: Explicitly define the loader and the path`) and `[...slug].astro` (`// v6 Requirement: Use the new render function`) indicate the author had to adapt from older Content Collections patterns (implicit loaders, `entry.render()`) to Astro v6's explicit `loader: glob(...)` and standalone `render(entry)` function — a common upgrade pain point for anyone following older Astro tutorials.
- **Iterative deployment fixes:** The commit history shows a fix specifically for authorizing the Vercel build via git config email, suggesting an early deployment/CI hiccup tied to commit author identity rather than application code.
- **Structured-data-first design:** The project deliberately treats JSON-LD schema injection as a first-class, reusable component (`Schema.astro`) rather than an afterthought — reflecting the site's core thesis that machine-readable metadata matters as much as human-readable content.

---

*This README reflects the state of the repository at the time of this audit and should be updated as the project evolves.*
