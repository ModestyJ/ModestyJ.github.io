---
description: Convert a ready publication candidate from the Cairn vault into a Jekyll post on github.io
---

You are publishing a reviewed **publication candidate** from the user's Obsidian vault ("Cairn") into this Jekyll blog (`ModestyJ.github.io`). Follow this repo's conventions in `CLAUDE.md` exactly.

## Paths
- Vault root: `/Users/jckim/Library/CloudStorage/OneDrive-Personal/obsidian`
- Candidates: `Publish/<slug>/<slug>.md`, diagrams in `Publish/<slug>/diagrams/` (Excalidraw `.excalidraw.md` + auto-exported `.svg` siblings)
- Blog repo (cwd): `/Users/jckim/work/ModestyJ.github.io`

## Select the candidate
`$ARGUMENTS` may name a slug/title. Otherwise, scan `Publish/*/` for candidates with `status: ready` and list them; if more than one, ask which to publish. Only publish `ready` candidates unless the user explicitly overrides.

## Category mapping (candidate `category` key → blog taxonomy)
| key | permalink base | `categories:` value | title prefix |
|-----|----------------|---------------------|--------------|
| arch | `/categories/arch/` | `ARCHITECTURE` | `[Architecture]` |
| ml   | `/categories/ml/`   | `Machine Learning` | `[ML]` |
| chip | `/categories/chip/` | `Chip` | `[Chip]` |
| lang | `/categories/lang/` | `Language` | `[Language]` |
| env  | `/categories/env/`  | `Environment` | `[Environment]` |

## Steps
1. **Read** the candidate `.md` and its frontmatter (title, category, tags, slug, created).
2. **Copy images**: create `assets/images/<slug>/` in the blog repo. For every embed in the candidate:
   - `![[diagrams/<name>.excalidraw]]` → find the exported SVG sibling (`diagrams/<name>.excalidraw.svg` or `diagrams/<name>.svg`) and copy it to `assets/images/<slug>/<name>.svg`. If no SVG exists yet, STOP and tell the user to open that drawing in Obsidian once (auto-export) — do not publish half the images.
   - Any other embedded image (`![[img.png]]` / `![](path)`) → copy into `assets/images/<slug>/` too.
3. **Convert the body**: strip the candidate's `> [!info]` helper callout; rewrite every image embed to Jekyll form pointing at the copied file, e.g. `![<name>](/assets/images/<slug>/<name>.svg)`. Convert any `[[wikilinks]]` that don't resolve on the blog into plain text or drop them. Keep the image-first structure.
4. **Write the post** at `_posts/<created>-<slug>.md` with frontmatter following existing posts:
   ```
   ---
   title: "<title prefix> <Title>"
   excerpt: "<one-line summary>"
   categories:
     - <categories value>
   tags:
     - [<tag>, <tag>, ...]
   permalink: /categories/<key>/<slug>
   date: <created>
   last_modified_at: <today>
   ---
   ```
   (Match the exact shape of a recent file in `_posts/`, e.g. `2022-09-26-arch-eyeriss.md`.)
5. **Verify** the build if feasible (`bundle exec jekyll build`), or at minimum confirm the post path, permalink, and that every image referenced exists under `assets/images/<slug>/`.
6. **Update the candidate**: set `status: published` and `published_url: https://ModestyJ.github.io/categories/<key>/<slug>` in the vault candidate's frontmatter, so the Publish Queue shows it as published.
7. **Commit** the blog repo (post + images). Branch first if on the default branch per repo policy. Ask before `git push` unless the user already said to push.

Report: the created post path, the permalink, how many images were carried over, and the commit. Do not delete anything from the vault.
