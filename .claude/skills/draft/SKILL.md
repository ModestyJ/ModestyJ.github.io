---
name: draft
description: Build an image-first blog publication candidate in the Cairn Obsidian vault from the Draft Basket (or stones/memos the user names). Use when the user explicitly asks to draft or write up a post from their notes.
---

# Draft a publication candidate

Author an image-first **publication candidate** inside the user's Obsidian vault ("Cairn"). The user reviews it in Obsidian, then the `publish` skill turns it into a Jekyll post.

## Resolve paths (portable — no hardcoded absolute path)
First that exists wins (`test -d`):
- **VAULT** (Cairn): `$CAIRN_VAULT_DIR` → sibling `../obsidian` → `/Users/jckim/Library/CloudStorage/OneDrive-Personal/obsidian`
- **BLOG**: `$CAIRN_BLOG_DIR` → sibling `../ModestyJ.github.io` → current repo root if it is the Jekyll site
If none resolve, ask the user.

## Choose the sources
- If the user named a topic or specific stones/memos, use those.
- **Otherwise read the Draft Basket** at `<VAULT>/Publish/_basket.md`: collect the `- [[...]]` links under its `## Items` section — those are the sources. **After** the candidate is written, empty the basket (clear the list under `## Items`).
- If both are empty, ask the user what to draft.
- Sources come from `Projects/*/Stones/*.md` (🪨 Stones) and interest folders `1. Language`, `2. Wiki`, `3. Papers`, `4. Research`, `5. Tech`. Read each in full. Never invent facts — trace everything to real notes (or clearly-marked general knowledge).

## Create the candidate
1. **Slug & category**: slug = kebab-case of the title. Category ∈ `arch | ml | chip | lang | env` — infer, confirm if unsure.
2. Write `<VAULT>/Publish/<slug>/<slug>.md`:
   ```
   ---
   type: publication
   status: draft
   title: "<Title>"
   category: <key>
   tags: [ ... ]
   slug: <slug>
   sources: ["[[note A]]", "[[note B]]"]
   created: <today YYYY-MM-DD>
   published_url:
   ---
   ```
3. **Write image-first**: diagrams over prose — a short hook, then lead each concept with a diagram and tight text beneath. Prefer one diagram per major idea.
4. **Diagrams as Excalidraw** in `<VAULT>/Publish/<slug>/diagrams/`, one `.excalidraw.md` per concept, embedded via `![[diagrams/<name>.excalidraw]]` (use the skeleton in `reference/excalidraw-skeleton.md`). Then tell the user to **open each drawing once in Obsidian** so its SVG auto-exports.
5. Report the candidate path; remind the user to review, mark **ready** (Publish Queue button or `status: ready`), then run the `publish` skill.

Write files with the Write tool at the resolved vault paths. Do not run git — publishing is the `publish` skill's job.
