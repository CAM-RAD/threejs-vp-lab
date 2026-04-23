# DESIGN-LAB — GS Fox Demo

## Current State
**Active file:** `gs-fox-v4.html` — the latest, best version
**Status:** Working, interactive, animated
**Run:** `cd DESIGN-LAB && npx -y http-server -p 8080 -c-1` then open `localhost:8080/gs-fox-v4.html`

## What It Is
A single-file Three.js demo rendering a procedural red fox using ~100k+ gaussian splat-style particles with PBD (Position-Based Dynamics) soft-body deformation and 8 interactive animations.

## Features
- **~100k particles** rendered as screen-space smoothstep splats
- **65+ body parts** with fox-accurate proportions (slender body, pointed snout, tall ears)
- **Directional lighting** — key light + fill + specular + ambient occlusion + warm rim
- **PBD deformation** — tail, ears, and ruff jiggle on camera orbit (body is rigid)
- **Root-dragging** — particles near body surface barely deform, tips swing freely
- **Depth-sorted** every 5 frames for correct alpha compositing
- **8 animations** — Jump, Shake, Wag Tail, Look Around, Spin, Nod, Bounce, Wiggle
- **Environment** — mossy ground (10k particles), grass tufts, mushrooms, fallen leaves, pebbles, flowers, ground shadow, bokeh background, floating dust

## File Inventory
| File | Description | Status |
|------|-------------|--------|
| `gs-fox-v4.html` | Latest fox — animations, smoothstep rendering, slim proportions | **Active** |
| `gs-fox-hd.html` | Earlier HD attempt — gaussian falloff, 85k splats | Superseded |
| `gs-fox-demo.html` | First fox version — 26k splats, basic | Superseded |
| `gs-poodle-demo.html` | Original poodle version — 18k splats | Superseded |
| `gs-real-viewer.html` | Attempted real .splat file viewer (gsplat.js) | Broken — library issues |

## Key Technical Decisions
- **Smoothstep falloff** instead of gaussian — gives opaque center with sharp edge (looks solid, not cloudy)
- **Shell sampling** — particles generated in thin 12% shell near surface, not inside volume (prevents visible internal primitives)
- **Rigid body + deformable zones** — per arXiv:2604.14782v1, body is locked rigid, only tail/ears/ruff deform
- **Root-dragging** — deformation scaled by distance from zone pivot point
- **Single continuous leg ellipsoids** — no separate paw/knee/transition blobs

## Known Issues / Next Steps
- Body still shows some ellipsoid seams at certain angles
- Legs could use more blend shapes at body connection
- Could add more animations (lie down, pounce, stretch)
- Could add sound effects on button click
- Could try loading a real .splat file for photorealistic quality (gsplat.js had issues)
- Performance ~47fps at 100k splats — could optimize with WebGPU

## Inspiration
- arXiv:2604.14782v1 — "One-shot Compositional 3D Head Avatars with Deformable Hair"
- antimatter15/splat — real GS viewer reference
- Viral GS deformation demos on X/Twitter
