# Scalable Vector Video — Master Plan

> **Inventor:** Joel Fricke (CAM-RAD / CAMANO Enterprises)
> **Conceived:** April 20, 2026
> **Status:** Brainstorming complete. Ready for build session on workstation/laptop.

## Goal

Build a working prototype that takes a text prompt, generates SVG keyframes via AI, tweens intermediate frames mathematically, and outputs an infinitely scalable animation playable in a browser.

## Core Concept

AI generates keyframes as SVG vector graphics. Mathematical path interpolation fills in intermediate frames. Every single frame is resolution-independent. No pixels anywhere. Scalable like an SVG, animated like a video.

See `scalable-video-research.md` for full concept writeup, prior art analysis, and attribution.

---

## Phase 0: Validate the Hard Part First (1-2 days)

**Goal:** Prove the riskiest assumption before building anything real. Can flubber.js (or similar) produce clean, artifact-free tweens between two AI-generated SVGs?

**Deliverable:** A single HTML file. Paste in two SVGs (manually generated from Quiver), hit a button, see a smooth 24-frame morph animation between them.

**Tasks:**
- [ ] Generate 2 test SVGs from Quiver (same subject, slightly different pose)
- [ ] Build a minimal HTML page with flubber.js that morphs SVG A to SVG B over 24 frames
- [ ] Evaluate: is the morph smooth? Do shapes hold? Does it look like animation or like melting?
- [ ] If flubber fails, test alternatives: GSAP MorphSVG, paper.js resample, anime.js
- [ ] Document which library wins and why

**Success criteria:** The morph looks like intentional animation, not a glitchy transition. Shapes maintain structural identity throughout the tween.

**If this fails:** The concept needs a different interpolation approach (possibly neural latent space interpolation via DeepSVG, or constraining the SVG generator to produce animation-compatible output). Reassess before proceeding.

---

## Phase 1: Single-Clip Prototype (3-5 days)

**Goal:** Build the minimum viable pipeline end to end.

**Deliverable:** A CLI tool. Input: a text prompt + duration. Output: an HTML file containing a playable, scalable SVG animation.

**Tasks:**
- [ ] Pick the SVG generator API (Quiver, Recraft, or whichever has best API access and style consistency)
- [ ] Write a script that takes a prompt and generates N keyframe SVGs (e.g., 1 per second for a 5-second clip = 5 keyframes)
- [ ] Build the path normalization step (force all keyframes into the same SVG topology)
- [ ] Build the tweening step (interpolate 23 frames between each pair of keyframes for 24fps)
- [ ] Build the player (HTML page that cycles through SVG frames at 24fps)
- [ ] Add resolution controls (render at 720p / 1080p / 4K viewport, same SVG data)
- [ ] Test with 3 different subjects: simple (geometric shape), medium (animal), complex (person)

**Success criteria:** A 5-second clip generated from a text prompt, playable in a browser, visually scalable from 720p to 4K with no quality loss.

---

## Phase 2: Quality + Consistency (1-2 weeks)

**Goal:** Make it actually look good, not just technically working.

**Tasks:**
- [ ] Solve character/style consistency between keyframes (prompt engineering, style references, seed control)
- [ ] Experiment with keyframe density (1 per second vs 1 per 2 seconds vs 1 per 0.5 seconds, find sweet spot)
- [ ] Add easing curves to the tweening (linear interpolation looks robotic, cubic-bezier looks natural)
- [ ] Handle color interpolation properly (SVG fill/stroke colors need to tween smoothly)
- [ ] Handle path count mismatches (what if keyframe A has 47 paths and keyframe B has 52?)
- [ ] Add audio track support (sync to a provided audio file)
- [ ] Build a gallery of 5-10 example clips to demonstrate different styles and subjects

**Success criteria:** Output looks like intentional stylized animation, not a tech demo. Someone watching it would think "that's a cool animated video" not "that's an SVG morph experiment."

---

## Phase 3: Web Player + Export (1 week)

**Goal:** Make it usable by someone who isn't a developer.

**Tasks:**
- [ ] Build a web UI: text input for prompt, generate button, preview player, resolution slider
- [ ] Add export to MP4 (render SVG frames to canvas at target resolution, encode with ffmpeg.wasm or server-side ffmpeg)
- [ ] Add export to Lottie (convert SVG keyframes + tween data to Lottie JSON for mobile playback)
- [ ] Add export to GIF (for quick sharing)
- [ ] Host on Cloudflare Pages (existing stack)

**Success criteria:** A non-technical person can type a prompt, preview the animation, choose a resolution, and download an MP4.

---

## Phase 4: Refine + Decide (ongoing)

- [ ] Test with real creative briefs from VP work
- [ ] Compare output quality to Seedance 720p → Topaz 4K upscale pipeline (the current alternative)
- [ ] Gather feedback from a few trusted people
- [ ] Decide: open source launch? product? tool for internal use only?
- [ ] If open sourcing: choose a name, write README, clean up code, launch publicly

---

## Tech Stack

All lightweight, runs on existing machines.

| Component | Tool | Why |
|-----------|------|-----|
| Language | JavaScript / Node.js | Runs everywhere, browser-native for the player |
| Path morphing | flubber.js (or winner from Phase 0) | Built by D3 creator, MIT, designed for SVG shape morphing |
| SVG generation | Quiver / Recraft API | Best current AI SVG generators |
| Player | Vanilla HTML/CSS/JS | No framework needed, maximum portability |
| MP4 export | ffmpeg.wasm | Client-side, no server dependency |
| Hosting | Cloudflare Pages | Already in Joel's stack |
| Source control | CAM-RAD GitHub (private) | Until ready to go public |

## What Runs Where

| Activity | Machine |
|----------|---------|
| Brainstorming + planning | Mac mini (this session) |
| Building Phase 0-3 | Workstation or laptop (new session) |
| Hosting | Cloudflare Pages |
| SVG generation | Cloud API (Quiver/Recraft) |

---

## Prior Art Summary

No existing patent covers this exact concept. Closest academic work is LottieGPT (Tsinghua, April 2026) and OmniLottie (Fudan, March 2026), but both use autoregressive Lottie token generation, not the generate-then-normalize-then-tween pipeline. The field is moving fast. Full prior art analysis is in `scalable-video-research.md`.

## Key Risks

1. **Temporal consistency of AI-generated keyframes.** The SVG generator may produce structurally different SVGs for each keyframe, making normalization difficult. Mitigate with Phase 0 validation.
2. **Path normalization quality.** Getting two independently-generated SVGs into the same topology is a known problem with known solutions, but may need tuning for complex illustrations.
3. **Visual style ceiling.** Current AI SVG output is illustrative/stylized, not photorealistic. This produces animated illustration, not live-action video. That's a feature for some use cases and a limitation for others.
4. **Competing research pace.** Multiple papers per month in this space. Building fast matters.

---

## How to Start a Build Session

Open a new Claude Code session on the workstation or laptop and say:

> "Let's work on scalable vector video. The master plan is at CAM-RAD/threejs-vp-lab/docs/scalable-video-master-plan.md. Start with Phase 0."

Everything needed is in this doc and `scalable-video-research.md` in the same repo.

---

*Author: Joel Fricke (CAM-RAD / CAMANO Enterprises)*
*Plan drafted: April 20, 2026*
