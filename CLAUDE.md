# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This is **ModestyJ's personal Jekyll blog** (`ModestyJ.github.io`, deployed via GitHub Pages), built by directly vendoring the source of the **Minimal Mistakes** Jekyll theme rather than installing it as a gem or remote theme. That means the theme's own source (`_layouts/`, `_includes/`, `_sass/`, `assets/js/`) lives at the repo root right alongside the actual blog content — `_config.yml` at root has `theme:`/`remote_theme:` commented out, confirming the direct-copy install method.

Because of this, the repo also still carries the upstream theme project's own dev/demo scaffolding: `docs/` (the theme's documentation/demo site, with its own `_config.yml`, `_posts`, `_pages`) and `test/` (a minimal Jekyll site used only to preview theme changes). **These are not part of the blog** — do not edit content there when the task is about the actual site. The real blog config is the root `_config.yml` (title "Modesty Studylog"), and the real blog content is the root `_posts/` and `_pages/`.

## Commands

- Install dependencies: `bundle install`
- Serve the site locally (root blog, not the theme test site): `bundle exec jekyll serve` (also available as `./run.sh`), then browse `http://localhost:4000`
- Preview theme changes in isolation using the vendored theme's own demo site: `bundle exec rake preview`, then browse `http://localhost:4000/test/` (this uses `test/` as the Jekyll source, per the `Rakefile`, and only matters when editing theme internals like `_layouts`/`_includes`/`_sass`, not when writing posts)
- Rebuild the bundled/minified theme JS after editing `assets/js/_main.js` or `assets/js/plugins/*`: `npm run build:js` (runs `uglifyjs` over the vendor+plugin files into `assets/js/main.min.js`, then stamps a banner via `banner.js`); `npm run watch:js` watches for changes
- There is no application test suite to run for content changes; `test/` is theme-preview scaffolding, not a test runner

## Content architecture

- **Posts** live in root `_posts/`, one file per post (`YYYY-MM-DD-slug.md`). Slugs and titles follow a `<topic>-<subject>` convention (e.g. `2022-09-26-arch-eyeriss.md`, title `"[Architecture] EYERISS"`).
- **Categories** are the primary taxonomy. Current set: `ARCHITECTURE` (arch), `Machine Learning` (ml), `Chip` (chip), `Language` (lang), `Environment` (env). Each has:
  - an entry in `_data/navigation.yml` under `categories:` (sidebar nav, `url: /categories/<key>`)
  - a page in `_pages/categories/category-<key>.md` with `layout: category`, `permalink: /categories/<key>/`, and a `taxonomy:` field that must match the `categories:` value used in post front matter
  - Post front matter should set `permalink: /categories/<key>/<slug>` and `categories: [<TAXONOMY_VALUE>]` to match its category page.
- Post front matter conventions to follow (see any existing post for a template): `title`, `excerpt`, `categories`, `tags` (an array), `permalink`, `date`, `last_modified_at`.
- `_pages/about.md` is the About page (`permalink: /about/`); `_data/navigation.yml` also defines the top masthead nav (`main:`).
- Post images go under `assets/images/<post-topic>/` (one subfolder per post/topic, matching the post slug's subject).
- Site-wide settings (skin, comments provider, search, analytics, author bio/links, footer links, defaults for `_posts`) are all in root `_config.yml`. Note `_config.yml` is not reloaded by `jekyll serve` on change — restart the server after editing it.
