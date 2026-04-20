# Scalable Video Research

> **Original concept and invention by Joel Fricke (CAM-RAD / CAMANO Enterprises)**
> All ideas in this document originated with Joel Fricke on April 20, 2026.
> This document serves as a dated record of authorship and prior art.

## The Question

Is there a way to generate video content (via AI or otherwise) that scales resolution like an SVG, where the underlying representation is resolution-independent and you render at whatever quality you need at playback time?

---

## Idea 1: Implicit Neural Representations (INR)

**Concept:** Store a video clip as a tiny neural network (1-5 MB) instead of as pixels. The network is a continuous function: given (timestamp, x, y) it outputs the RGB color. Query at any resolution to render.

**Key research:**
- NeRV (Neural Representations for Videos, 2021, Meta/UCB)
- HNeRV (improved architecture, hierarchical)
- SIREN (sinusoidal activations, smooth interpolation)
- FFNeRV (flow-guided, state of art neural video codec)

**What we could build:**
1. CLI encoder: takes an mp4 (e.g., 720p Seedance output), trains a small INR, outputs a .safetensors file
2. CLI player: takes the .safetensors file, renders at any resolution (4K, 8K, etc.)
3. Web player: ONNX.js or WebGPU in-browser playback

**Advantages:**
- Resolution is a render-time choice
- Temporal interpolation for free (smooth slow-mo, arbitrary framerate)
- Extreme compression (2-5 MB vs 30 MB for H.264)
- No hallucination; the network learned THIS clip's structure

**Challenges:**
- Training time (minutes per clip on a GPU)
- Playback requires GPU inference (not CPU-friendly yet)
- Subtle quality artifacts vs raw source
- No standard format or ecosystem

**Status:** research code is open source, needs packaging into usable tools. Integration work, not novel research.

---

## Idea 2: Layered Decomposition + Tiled Super-Resolution

**Concept:** Decompose AI-generated video into independently scalable layers (background plate, character mattes, motion vectors, depth map), upscale each layer with the optimal method for its content type, recomposite at any resolution.

**Complementary approach:** Tiled super-resolution (slice frames into overlapping tiles, enhance each with an image model, stitch back) for fast production-phase upscaling before the final layered master.

**Tools (all existing):**
- SAM2 for character segmentation
- Topaz / ESRGAN for image/video upscaling
- RAFT for optical flow estimation
- DaVinci Resolve for compositing

**Best for:** Final mastering of longer content where layer-level editability matters.

---

## Idea 3: AI-Generated SVG Keyframes + Mathematical Tweening (PRIMARY CONCEPT)

> **Invented by Joel Fricke, April 20, 2026.**
> This is the primary original concept in this research project.

**The Core Idea:**

Use AI SVG generators (Quiver, Recraft, etc.) to generate KEYFRAME images as vector graphics (SVGs). Then use mathematical path interpolation to generate all in-between frames. The result is a video where every single frame is resolution-independent vectors, not pixels. Infinitely scalable. No upscaling needed, ever.

**Why this is novel:**
- SVG generators exist (Quiver, Recraft)
- SVG animation tools exist (Rive, Lottie, GSAP)
- SVG morphing libraries exist (flubber.js, paper.js)
- Nobody has connected them into: "AI generates keyframes as vectors, math fills in the frames, the result is infinitely scalable video"
- This is a product, not a research project

**Architecture (Approach D: Keyframe + Normalize + Tween):**

1. **AI generates keyframes as SVG** (every 1-2 seconds of final video). Using Quiver or equivalent SVG generator. Each keyframe is a full vector illustration.

2. **Path normalization** aligns SVG topologies between consecutive keyframes. Forces both SVGs into the same path structure (same number of paths, same number of anchor points per path). Libraries: paper.js `path.resample()`, snap.svg, computational geometry "shape correspondence" algorithms.

3. **Mathematical tweening** generates intermediate frames by interpolating path coordinates between normalized keyframes. Libraries: flubber.js (by Mike Bostock / D3 creator, specifically built for SVG shape morphing), GSAP MorphSVG, anime.js.

4. **Playback** in any browser. Simple div swapping SVG innerHTML at 24fps, animated SVG with SMIL, or export to Lottie for mobile. No custom player, no GPU, runs on a phone.

**Key Properties:**
- Every frame is an SVG. Resolution-independent from birth. No pixels anywhere.
- File size: ~50-200 KB per keyframe SVG, ~12-48 MB for 10 seconds at 24fps (compresses further with gzip)
- Frame interpolation is pure math (path coordinate lerp), not AI hallucination
- Want 60fps from 24fps? Interpolate more points between keyframes. Perfect tweening for free.
- Any frame can be opened in Illustrator/Figma and edited by hand
- Standard format. No new player needed. Browsers already render SVGs.

**Challenges to Solve:**
- **Temporal consistency of AI-generated keyframes.** Does frame 2's dog look like frame 1's dog? Current SVG generators don't have character/style locking across multiple outputs. This is the primary technical risk.
- **Path normalization between arbitrary SVGs.** Getting two independently-generated SVGs into the same topology is a known computational geometry problem with existing solutions, but it may need tuning for complex illustrations.
- **Visual style.** Current AI SVG output is illustrative/stylized, not photorealistic. This approach produces animated illustration, not live-action-style video. That's a feature for some use cases and a limitation for others.

**Build Plan:**
- Phase 1 (prototype, 2-3 days): take 2 keyframe SVGs, normalize paths with flubber.js, output a 24-frame tweened animation as a playable HTML file
- Phase 2: wrap Quiver API to auto-generate keyframes from text prompts
- Phase 3: full pipeline. prompt in, scalable animated video out. web player. export to Lottie/MP4.

**Potential Names:** VectorVideo, ScaleMotion, InfiniFrame, SVGflix, VecFilm

---

## Attribution and Prior Art

All concepts in this document were originated by **Joel Fricke** (GitHub: CAM-RAD, CAMANO Enterprises Inc., Vancouver, BC, Canada) on **April 20, 2026**, during a conversation documented in a Claude Code Telegram session running on his Mac Mini workstation.

The specific innovation of **connecting AI SVG generation with mathematical path tweening to produce infinitely scalable video** (Idea 3) is Joel Fricke's original concept. While the individual components (SVG generators, path morphing libraries, vector animation tools) exist independently, the synthesis of these components into an end-to-end scalable video pipeline was conceived and articulated by Joel Fricke on this date.

This document, committed to the private GitHub repository CAM-RAD/threejs-vp-lab, serves as a timestamped record of authorship and prior art.

---

*Created: 2026-04-20*
*Author: Joel Fricke (CAM-RAD / CAMANO Enterprises)*
*Documented by: Claude Code (Anthropic) acting as technical collaborator*
