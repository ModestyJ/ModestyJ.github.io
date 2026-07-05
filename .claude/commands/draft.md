---
description: Draft an image-first publication candidate in the Cairn vault from selected Stones/memos
---

Draft a **publication candidate** for the github.io blog, authored inside the user's Obsidian vault ("Cairn"). The user reviews it in Obsidian, then `/publish` converts it to a Jekyll post.

## Resolve paths (portable — no hardcoded absolute path)
Find each repo by checking these in order and using the first that exists (`test -d`):
- **VAULT** (Obsidian/Cairn): `$CAIRN_VAULT_DIR` → sibling `../obsidian` → `/Users/jckim/Library/CloudStorage/OneDrive-Personal/obsidian`
- **BLOG**: `$CAIRN_BLOG_DIR` → sibling `../ModestyJ.github.io` → the current repo root if it looks like the Jekyll site
If none resolve, ask the user for the path. (This lets the same command run on any machine: sibling clones use `../`, this Mac uses the OneDrive/`~/work` paths.)

## Layout inside VAULT
- Candidates: `Publish/<slug>/<slug>.md` (per-candidate folder)
- Diagrams: `Publish/<slug>/diagrams/*.excalidraw.md` (Excalidraw; auto-export SVG is ON → a sibling `.svg` appears after the drawing is opened/saved in Obsidian)
- Sources to pull from: `Projects/*/Stones/*.md` (🪨 Stones) and interest folders `1. Language`, `2. Wiki`, `3. Papers`, `4. Research`, `5. Tech`.

## Input
`$ARGUMENTS` is the topic and/or specific Stones/memos. If vague, search the vault for candidates, list the strongest 3–8, and ask which to include before drafting. Never invent facts — everything traces to real notes (or clearly-marked general knowledge).

## Steps
1. **Gather**: read the chosen Stones/memos in full; collect their vault paths for `sources` (as `"[[note name]]"`).
2. **Slug & category**: slug = kebab-case of the title. Category ∈ `arch | ml | chip | lang | env` — infer, confirm if unsure.
3. **Create** `Publish/<slug>/<slug>.md` with frontmatter:
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
4. **Write image-first.** Diagrams over prose: a short hook, then lead each concept with a diagram and tight text under it. Prefer one diagram per major idea.
5. **Diagrams as Excalidraw** in `Publish/<slug>/diagrams/`, one `.excalidraw.md` per concept, embedded via `![[diagrams/<name>.excalidraw]]`. Use the minimal skeleton below (labeled boxes + arrows as a starting point). Then tell the user to **open each drawing once in Obsidian** so the SVG auto-exports.
6. Report the candidate path; remind the user to review, mark **ready** (Publish Queue button or `status: ready`), then run `/publish`.

## Minimal Excalidraw file skeleton (`.excalidraw.md`)
```
---
excalidraw-plugin: parsed
tags: [excalidraw]
---
==⚠  Switch to EXCALIDRAW VIEW in the MORE OPTIONS menu of this document. ⚠==

# Excalidraw Data
## Text Elements
%%
## Drawing
​```json
{"type":"excalidraw","version":2,"source":"draft","elements":[
  {"type":"rectangle","x":100,"y":100,"width":180,"height":70,"id":"r1","strokeColor":"#1e1e1e","backgroundColor":"transparent","fillStyle":"solid","strokeWidth":2,"roughness":1,"seed":1,"version":1,"versionNonce":1,"isDeleted":false,"groupIds":[],"boundElements":[],"updated":1,"link":null,"locked":false},
  {"type":"text","x":120,"y":125,"width":140,"height":25,"id":"t1","text":"Concept A","fontSize":20,"fontFamily":1,"textAlign":"left","verticalAlign":"top","strokeColor":"#1e1e1e","backgroundColor":"transparent","fillStyle":"solid","strokeWidth":2,"roughness":1,"seed":2,"version":1,"versionNonce":2,"isDeleted":false,"groupIds":[],"boundElements":[],"updated":1,"link":null,"locked":false,"containerId":null,"originalText":"Concept A","lineHeight":1.25,"baseline":18}
],"appState":{"gridSize":null,"viewBackgroundColor":"#ffffff"},"files":{}}
​```
%%
```
Extend `elements` with more rectangles, `arrow`/`line`, and `text`. Keep it legible; the user refines it in Obsidian.

Write files with the Write tool at the resolved vault paths. Do not run git here — publishing is `/publish`'s job.
