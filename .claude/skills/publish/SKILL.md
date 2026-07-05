---
name: publish
description: Convert a ready publication candidate from the Cairn vault into a Jekyll post on the ModestyJ.github.io blog. Use only when the user explicitly asks to publish a Cairn candidate.
---

# Publish a candidate to github.io

Convert a reviewed **publication candidate** from the user's Obsidian vault ("Cairn") into a Jekyll post in the `ModestyJ.github.io` repo. Follow that repo's `CLAUDE.md` conventions exactly.

## Resolve paths (portable — no hardcoded absolute path)
First that exists wins (`test -d`):
- **VAULT**: `$CAIRN_VAULT_DIR` → sibling `../obsidian` → `/Users/jckim/Library/CloudStorage/OneDrive-Personal/obsidian`
- **BLOG** (cwd is usually this): `$CAIRN_BLOG_DIR` → sibling `../ModestyJ.github.io` → `/Users/jckim/work/ModestyJ.github.io` (this machine)
If none resolve, ask the user.

**Never rely on the shell's current directory.** It may be pointing at the vault. Every git and file operation must target an explicitly-resolved absolute path: run git as `git -C "<BLOG>" …` or `git -C "<VAULT>" …`, and read/write files by their full resolved path. A bare `git commit` is a bug.

## Layout
- Candidates: `<VAULT>/98. Publish/<slug>/<slug>.md`; diagrams + auto-exported SVGs in `<VAULT>/98. Publish/<slug>/diagrams/`
- Posts: `<BLOG>/_posts/`; images: `<BLOG>/assets/images/<slug>/`

## Select the candidate
If the user named a slug/title, use it. Otherwise scan `<VAULT>/98. Publish/*/` for `status: ready` candidates and list them; if more than one, ask which. Only publish `ready` unless the user overrides.

## Category mapping (candidate `category` key → blog taxonomy)
| key | permalink base | `categories:` value | title prefix |
|-----|----------------|---------------------|--------------|
| arch | `/categories/arch/` | `ARCHITECTURE` | `[Architecture]` |
| ml   | `/categories/ml/`   | `Machine Learning` | `[ML]` |
| chip | `/categories/chip/` | `Chip` | `[Chip]` |
| lang | `/categories/lang/` | `Language` | `[Language]` |
| env  | `/categories/env/`  | `Environment` | `[Environment]` |

## Steps
1. **Read** the candidate `.md` + frontmatter (title, category, tags, slug, created).
2. **Copy images** into `<BLOG>/assets/images/<slug>/`. For each embed:
   - `![[diagrams/<name>.excalidraw]]` → find the exported SVG sibling (`<name>.excalidraw.svg` or `<name>.svg`) and copy it as `<name>.svg`. **If the SVG is missing, STOP** and tell the user to open that drawing in Obsidian once (auto-export) — never publish with missing diagrams.
   - Other embeds (`![[img.png]]` / `![](path)`) → copy too.
3. **Convert body**: drop the `> [!info]` helper callout; rewrite every embed to Jekyll form, e.g. `![<name>](/assets/images/<slug>/<name>.svg)`; convert or drop unresolved `[[wikilinks]]`. Keep the image-first structure.
4. **Write the post** `<BLOG>/_posts/<created>-<slug>.md`, matching a recent file's frontmatter shape (e.g. `2022-09-26-arch-eyeriss.md`):
   ```
   ---
   title: "<prefix> <Title>"
   excerpt: "<one-line>"
   categories:
     - <categories value>
   tags:
     - [<tag>, <tag>]
   permalink: /categories/<key>/<slug>
   date: <created>
   last_modified_at: <today>
   ---
   ```
5. **Verify**: `bundle exec jekyll build` if feasible, or confirm post path, permalink, and that every referenced image exists.
6. **Update the candidate** (in VAULT): set `status: published` and `published_url: https://ModestyJ.github.io/categories/<key>/<slug>`.
7. **Commit for portability** — always use `git -C` so the shell's cwd is irrelevant:
   - BLOG: `git -C "<BLOG>" add _posts/<created>-<slug>.md assets/images/<slug>` then `git -C "<BLOG>" commit -m "…"`. Push only if the user asked (`git -C "<BLOG>" push`); they publish from the default branch.
   - VAULT: `git -C "<VAULT>" add "98. Publish/<slug>"` (candidate status change **and** the exported `.svg` files, so a fresh clone can publish without re-rendering) then `git -C "<VAULT>" commit -m "…"`. Push only if asked.
   - Verify each commit landed in the intended repo (`git -C "<repo>" log --oneline -1`) before reporting.

Report: post path, permalink, image count, and both commits. Do not delete anything from the vault.
