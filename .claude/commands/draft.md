---
description: Draft an image-first publication candidate in the Cairn vault from selected Stones/memos
---

You are drafting a **publication candidate** for the user's github.io blog, authored inside their Obsidian vault ("Cairn"). The user reviews it in Obsidian, then `/publish` converts it to a Jekyll post.

## Paths
- Vault root: `/Users/jckim/Library/CloudStorage/OneDrive-Personal/obsidian`
- Candidates live at: `Publish/<slug>/<slug>.md` (per-candidate folder)
- Diagrams: `Publish/<slug>/diagrams/*.excalidraw.md` (Excalidraw; auto-export SVG is ON, so a sibling `.svg` appears after the user opens/saves each drawing in Obsidian)
- Knowledge sources to pull from: `Projects/*/Stones/*.md` (🪨 Stones) and interest folders `1. Language`, `2. Wiki`, `3. Papers`, `4. Research`, `5. Tech`.

## Input
`$ARGUMENTS` is the topic and/or the specific Stones/memos to summarize. If it's vague or no sources are named, first search the vault for candidate Stones/memos on the topic, list the strongest 3–8, and ask the user which to include before drafting. Do not invent facts — everything must trace to real notes (or clearly-marked general knowledge).

## Steps
1. **Gather**: read the chosen Stones/memos in full. Note their key claims, and any existing images/links worth carrying over. Collect their paths for the `sources` frontmatter (as `"[[note name]]"`).
2. **Pick slug & category**: slug = kebab-case of the title. Category is one of `arch | ml | chip | lang | env` (Architecture / Machine Learning / Chip / Language / Environment) — infer from the sources, confirm if unsure.
3. **Create the candidate** at `Publish/<slug>/<slug>.md` with this frontmatter:
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
4. **Write it image-first.** This blog favors diagrams over prose. Structure: a short hook, then **lead each concept with a diagram**, with tight explanatory text underneath. Prefer one diagram per major idea. Keep prose lean — the images carry the load.
5. **Make the diagrams as Excalidraw files** in `Publish/<slug>/diagrams/`, one `.excalidraw.md` per concept, and embed them in the candidate with `![[diagrams/<name>.excalidraw]]`. Use the minimal Obsidian-Excalidraw format below; lay out labeled boxes + arrows for the concept (best-effort starting point the user will refine). After creating them, tell the user to **open each drawing once in Obsidian** so the SVG auto-exports (publish needs the `.svg`).
6. Report the candidate path and remind the user to review it, mark it **ready** (Publish Queue button or `status: ready`), then run `/publish`.

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
Extend `elements` with more rectangles, `arrow`/`line`, and `text` to express the concept. Keep it simple and legible; the user refines it in Obsidian.

Write files with the Write tool at the absolute vault paths. Do not run git here — publishing is `/publish`'s job.
