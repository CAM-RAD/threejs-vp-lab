# Scalable Video Research

## The Question

Is there a way to generate video content (via AI or otherwise) that scales resolution like an SVG, where the underlying representation is resolution-independent and you render at whatever quality you need at playback time?

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

**Status:** research code is open source, needs packaging into usable tools. integration work, not novel research.

**Next step:** deep research pass on which INR architecture gives best quality-to-size for AI-generated content.

---

## Idea 2: [TBD — exploring more elegant / simpler approaches]

Looking for solutions that:
- Work on regular people's hardware (no GPU requirement for playback)
- Use existing tools and infrastructure
- Could run on the cloud as a service
- Are realistic to build and ship, not research projects

---

*Created: 2026-04-20*
*Context: conversation between Joel and Claude about vector-like scalability for AI-generated video*
