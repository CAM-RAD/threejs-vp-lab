# Three.js VP Lab — Progress Reference

## Project Purpose
Browser-based 3D scene prototyping for virtual production. Build previz environments that run on any device via URL.

## Tech Stack
- Three.js (via CDN, no build step)
- Single HTML files per scene
- GitHub Pages for hosting
- Repo: CAM-RAD/threejs-vp-lab (public for GitHub Pages)

## Project Structure
```
index.html          — Scene directory / landing page
scenes/
  desktop/          — Full-detail scenes (desktop GPU required)
  mobile/           — Optimized scenes (phones/tablets)
docs/               — Learnings, tips, references
progress-reference.md
```

## Scenes Built

| Scene | Desktop | Mobile | Description |
|-------|---------|--------|-------------|
| Alpine Highway | scenes/desktop/alpine-highway.html | — | Procedural mountains, winding road, pine trees, fog |
| Greek City | scenes/desktop/greek-city.html | — | 3 temples, agora, shops, stoa, fountain, dock, water |
| Greek Epic | scenes/desktop/greek-epic.html | scenes/mobile/greek-epic-mobile.html | Full city: acropolis, amphitheatre, aqueduct, walls, boats, olive groves |
| Medieval Town | scenes/desktop/medieval-town.html | — | VP volume bg: half-timber buildings, church, castle wall, market, well, torches, dynamic sky |

## Mobile Optimization Rules (learned the hard way)
- NO Sky shader (PMREMGenerator kills mobile GPUs)
- NO shadows (shadowMap.enabled = false)
- Pixel ratio = 1 (never use devicePixelRatio on mobile)
- antialias: false
- powerPreference: 'low-power'
- MeshLambertMaterial only (not Standard or Physical)
- Water texture: 128x128 max
- Cylinder/cone segments: 3-5 (not 8-12)
- Max instanced objects: ~40-50
- Fog near plane: 250 (hide distant complexity)
- No ExtrudeGeometry (can crash some mobile GPUs)
- Always add cache-busting meta tag during dev

## Deployment
- GitHub Pages: cam-rad.github.io/threejs-vp-lab/
- Repo is PUBLIC (required for free GitHub Pages)
- Cache-busting: add ?v=N to URL, or use no-cache meta tag
