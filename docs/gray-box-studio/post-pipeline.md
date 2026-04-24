# Gray Box Studio — Post-Production Pipeline

## Overview

```
Capture on gray stage
    ↓
Camera Solve (track camera from markers or robot arm data)
    ↓
Segmentation (separate performer from gray background)
    ↓
Environment Generation (AI creates the background)
    ↓
Relighting (match performer lighting to environment)
    ↓
Compositing (layer performer over environment)
    ↓
Refinement (human artist review and fix)
    ↓
Final output
```

## Step 1: Camera Solve

**What:** extract camera position, rotation, and lens parameters for every frame.

**Tools:**
- **Robot arm data** (primary) — if shot on the robot arm, camera position is already known from encoders. export as a camera path file.
- **COLMAP** (backup/manual shots) — open source structure-from-motion. reads the tracking markers on the gray walls and calculates camera position. `colmap automatic_reconstructor --image_path frames/ --workspace_path colmap_out/`
- **PFTrack / SynthEyes** (commercial alternatives if more precision needed)

**Output:** camera path file (position + rotation + focal length per frame) in a format your compositor can read (Alembic, FBX, or raw CSV).

## Step 2: Segmentation

**What:** separate the performer from the gray background, producing a clean alpha matte per frame.

**Tools:**
- **SAM2** (Segment Anything Model 2, Meta) — state of the art. click on the performer in frame 1, it tracks them through the entire video. handles hair, transparent fabrics, motion blur.
- **YOLO 11 + SAM2** — YOLO detects the person bounding box, SAM2 refines to pixel-perfect matte. fully automatic.
- **roto-anything** (if SAM2 struggles) — manual roto with AI assist via After Effects or Nuke.

**Output:** per-frame PNG sequence with alpha channel (RGBA), or EXR with embedded matte.

**Our advantage:** gray background is much easier to segment than complex real environments. SAM2 will produce cleaner mattes than it would on location footage.

## Step 3: Environment Generation

**What:** create the background environment that replaces the gray walls.

**Multiple approaches — pick based on the shot:**

### Option A: Gaussian Splat Environments (from our scanner)
- Scan a real location with the studio's gaussian scanner
- Load the scan into the Three.js VP lab or Nerfstudio
- Render the scan from the tracked camera path (matching the camera solve from step 1)
- Output: rendered background plate that perfectly matches camera movement
- **Best for:** real locations you have access to scan

### Option B: AI Image Generation
- Use production design reference art as img2img input
- Generate environments with Flux, SDXL, or Seedance (frame by frame or video)
- Use the camera solve data to ensure perspective and parallax are correct
- **Best for:** fictional or inaccessible locations

### Option C: Unreal Engine Rendered Environments
- Same content you'd display on the LED volume, rendered offline instead of real-time
- Higher quality than real-time (full ray tracing, unlimited resolution)
- Camera path from step 1 drives the virtual camera in Unreal
- **Best for:** environments you've already built for the LED volume

### Option D: Hybrid
- Gaussian splat base (real scanned location) + AI enhancement (add time of day, weather, set dressing)
- This is probably the most powerful approach: start with reality, enhance with AI

**Output:** rendered background plate sequence matching the camera path

## Step 4: Relighting

**What:** adjust the performer's lighting to match the generated environment. the flat stage lighting needs to be transformed to look like the performer is actually IN the environment.

**Tools:**
- **IC-Light** (Tencent, open source) — AI relighting from a reference image. give it the performer frame + the environment, it adjusts lighting on the performer to match.
- **SwitchLight** (open source) — similar, specifically designed for compositing relighting.
- **DaVinci Resolve 20+** — has AI-powered relighting tools built in.
- **Nuke + ML tools** — production-standard compositing with AI assist.

**Output:** relit performer frames that look like they're actually standing in the environment.

## Step 5: Compositing

**What:** layer the relit performer over the generated environment.

**Tools:**
- **DaVinci Resolve Fusion** (free) — node-based compositing, handles the full pipeline
- **Nuke** (industry standard, expensive)
- **After Effects** (accessible, good for simpler comps)
- **Natron** (open source Nuke alternative)

**Layers:**
1. Background environment plate (from step 3)
2. Performer with alpha matte (from step 2, relit from step 4)
3. Shadow/contact layer (AI-generated shadows where performer meets the ground)
4. Atmospheric effects (haze, particles, light leaks matching the environment)
5. Lens effects (matching the RED's lens characteristics — bokeh, distortion, aberration)

**Output:** composited frames

## Step 6: Refinement

**What:** human artist reviews every shot and fixes issues.

**Common fixes:**
- Matte edge cleanup (hair, transparent objects)
- Color matching between performer and environment
- Shadow direction and intensity
- Reflection consistency
- Continuity between cuts
- Fine detail in environment (remove AI artifacts, add texture)

**This is the step that separates good from great.** The AI does 80% of the work. The artist does the 20% that makes it look real.

## Hardware Requirements

All processing runs on the studio's 32-40 GB VRAM GPU machine (via Parsec):

| Step | GPU VRAM | Time per frame |
|------|----------|---------------|
| COLMAP camera solve | 4-8 GB | batch (minutes for full sequence) |
| SAM2 segmentation | 4-8 GB | ~0.5-1 sec/frame |
| AI environment generation | 8-24 GB | 2-30 sec/frame depending on method |
| IC-Light relighting | 8-12 GB | 1-5 sec/frame |
| Compositing | CPU-bound | real-time in Resolve |

For a 2-minute scene at 24fps (2,880 frames), full pipeline: ~4-12 hours depending on environment generation method.

## What Liman's Team Does Differently

- Proprietary AI system (not off-the-shelf tools)
- 55 dedicated AI artists (we'd start with 1-2 people + AI tools)
- 30-week post schedule (acceptable for a $70M feature film)
- Custom training data for their AI (we'd use general-purpose models)

The open-source pipeline above won't match Acme AI's proprietary system on day one. But it's free, it's improvable, and it gets better every month as the tools advance.
