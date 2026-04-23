# World-Class Three.js Inspiration

> Mood board of what's possible. Every entry has a URL, what makes it jaw-dropping, techniques used, and how we could apply it in VP lab scenes.
>
> Last updated: 2026-04-22

---

## Table of Contents

1. [Award-Winning Sites (Awwwards / FWA / SOTD)](#1-award-winning-sites)
2. [The Gold Standard: Bruno Simon](#2-the-gold-standard-bruno-simon)
3. [World-Class Portfolios](#3-world-class-portfolios)
4. [Elite Studios & Agencies](#4-elite-studios--agencies)
5. [Codrops Demos & Tutorials (2024-2026)](#5-codrops-demos--tutorials)
6. [Three.js + AI / Gaussian Splatting](#6-threejs--ai--gaussian-splatting)
7. [pmndrs / React Three Fiber Ecosystem](#7-pmndrs--react-three-fiber-ecosystem)
8. [Shader Techniques & Ports](#8-shader-techniques--ports)
9. [WebGPU & TSL (Three Shading Language)](#9-webgpu--tsl)
10. [Specific Effect Techniques](#10-specific-effect-techniques)
11. [Viral Demos & Experiments](#11-viral-demos--experiments)
12. [Key Takeaways for VP Lab](#12-key-takeaways-for-vp-lab)

---

## 1. Award-Winning Sites

### Awwwards Site of the Year 2025

| Site | URL | What Makes It Special |
|------|-----|----------------------|
| **Lando Norris (OFF+BRAND)** | [landonorris.com](https://landonorris.com) | McLaren F1 driver's official site. Webflow + WebGL + Rive motion design. Immersive motorsport experience. |
| **Messenger** | — | Not a messaging app — a tiny WebGL planet with physics, lighting, and animation. GPU-powered rendering for deliveries experience. |
| **Bruno Simon v2** | [bruno-simon.com](https://bruno-simon.com) | Awwwards Site of the Month (Jan 2026). Browser-based 3D environment with vehicle controls and spatialized audio. |

### Recent Awwwards Three.js Winners (2025-2026)

- **MANGO IN SPACE 6.0** — Site of the Day (Apr 20, 2026)
- **ilithya.rocks** — Developer portfolio
- **OceanX 2025** — Ocean exploration
- **D2C Life Science** — Site of the Day + Developer Award
- **Voku.Studio** — Site of the Day
- **Vibrant Wellness** — Site of the Day + Developer Award

> Browse the full collection: [awwwards.com/websites/three-js](https://www.awwwards.com/websites/three-js/)

---

## 2. The Gold Standard: Bruno Simon

**URL:** [bruno-simon.com](https://bruno-simon.com)
**Case Study:** [Medium deep-dive](https://medium.com/@bruno_simon/bruno-simon-portfolio-case-study-960402cc259b)
**Course:** [threejs-journey.com](https://threejs-journey.com/)

### Technical Breakdown

| Technique | Detail |
|-----------|--------|
| **Rendering** | Three.js with **Matcap shading** — zero dynamic lights, zero real shadows. "It's just illusions." Normal-relative-to-camera picks color from a provided texture. |
| **Physics** | **Cannon.js** with primitive collision shapes decoupled from visual meshes. Primitives handle physics; Three.js meshes update position each frame. Objects set to sleep after inactivity to prevent FPS drops. |
| **Shadows** | Static shadows **baked in Blender** (Cycles renderer), saved as PNG-8 via TinyPNG. Dynamic shadows are simple planes beneath objects with opacity based on height from ground. |
| **Light bounce** | Faked by calculating vertex distance to ground + dot product math. Closer + downward-facing surfaces get stronger orange coloration. |
| **Modeling** | All assets modeled in **Blender (Eevee)**, exported as **glTF with Draco compression** — files 50-70% lighter. |
| **Postprocessing** | Vertical/horizontal blur on screen edges + large light flare on left for warmth. |
| **Antenna** | No physics — rotation inversely matches car acceleration, spring-force proportional to displacement pulls it back to center. |
| **Audio** | **Howler.js** with speed control. Collision sounds randomized from multiple similar samples with volume/speed variation. Simultaneous sound limits prevent distortion. |
| **Optimization** | VAO (Vertex Array Object) optimization by Samuel Honigstein drastically reduced WebGL calls. Mobile: removed blur, clamped pixel ratio, disabled matrix auto-update. Debug panel via `#debug` URL hash. |
| **Total size** | 2.8 MB for entire site. |

### VP Lab Application
- Matcap shading for stylized scenes (zero light setup, maximum performance)
- Baked shadows from Blender for static environments
- Cannon.js primitives for interactive physics without visual complexity penalty
- Draco-compressed glTF as standard export pipeline
- Debug panel pattern via URL hash

---

## 3. World-Class Portfolios

### Samuel Honigstein (Samsy) — Cyberpunk WebGPU World
**URL:** [samsy.ninja](https://samsy.ninja/)

- **120+ FPS** cyberpunk 3D world powered by **WebGPU**
- Neon-lit cityscape with first-person controls, holographic interfaces, Japanese design elements
- Stack: **Vue.js + Three.js + TSL + GSAP**
- Custom post-processing pipeline: bloom and reflections without compute shaders
- TSL auto-compiles shaders to WebGPU when supported, falls back to WebGL

> **VP Lab:** WebGPU + TSL as the future rendering path. Cyberpunk aesthetic as a scene template.

### Sebastien Lempens — Scroll-Driven Paris Tour
**URL:** [sebastien-lempens.com](https://sebastien-lempens.com/)

- Scroll-driven tour through 3D Paris
- Dynamic camera shifts: first-person, following on scooter, skydiving
- Cinematic storytelling through camera transitions

> **VP Lab:** Scroll-driven camera choreography for narrative scenes.

### Jordan Breton — Floating Island
**URL:** [jordan-breton.com](https://jordan-breton.com/)

- FWA Site of the Day (Oct 2, 2025)
- Floating island with grass, waterfall, fire, wind, trees, butterflies
- Particle systems for every natural element
- Fixed-point camera navigation

> **VP Lab:** Particle-driven natural environments. Fire, water, wind as particle systems.

### Bilal El Moussaoui — Music Box Story
**URL:** [bilal.show](https://bilal.show/)

- Scroll-driven 3D story following a character through a music-box environment
- Proves developers can be exceptional visual storytellers

> **VP Lab:** Character-following scroll narrative as a scene type.

### JReyes MC — Minecraft Portfolio
**URL:** [jreyes-mc-portfolio.com](https://jreyes-mc-portfolio.com/)

- Minecraft-inspired 3D environment (Awwwards Honorable Mention)
- Voxel aesthetic with scroll-based navigation

### Thibault Introvigne — Spaceman Explorer
**URL:** [thibault-introvigne.com](https://www.thibault-introvigne.com/)

- Control a spaceman in a colorful world with 10 collectibles
- Blade Runner / Cyberpunk / The Witness inspiration
- Built with **React Three Fiber**

### Aimee Weis — Papercraft World
**URL:** [aimees-papercraft-world.com](https://aimees-papercraft-world.com)

- Scroll-driven portfolio with hand-drawn papercraft environment
- 2D-to-3D asset baking technique
- Built with **React Three Fiber**

> **VP Lab:** 2D illustration baked onto 3D geometry for stylized looks.

---

## 4. Elite Studios & Agencies

### Tier 1: The Masters

| Studio | URL | Specialty | Notable Work |
|--------|-----|-----------|-------------|
| **Lusion** | [lusion.co](https://lusion.co/) | Buttery motion, inventive interactions | My Little Storybook (WebGL + illustrated animation), Porsche Dream Machine (generative CG film), Oryzo AI |
| **Active Theory** | [activetheory.net](https://activetheory.net/) | Elegant design + engineering, multi-user metaverse spaces | Neon WebGL installation, real-time rendered worlds |
| **Resn** | [resn.co.nz](https://resn.co.nz/) | Large-scale interactive builds, surrealist playgrounds | Multi-user "metaverse" spaces, experiential storytelling |
| **Unseen Studio** | [unseen.co](https://unseen.co/) | Refined visuals + solid engineering | BlueYard (fluid-simulated orb, reactive cursor, custom shaders) |

### Tier 2: Specialized Excellence

| Studio | URL | Specialty |
|--------|-----|-----------|
| **Utsubo** | [utsubo.com](https://utsubo.com/) | Engineering-led WebGPU/WebGL, performance budgets baked in |
| **Merci-Michel** | [merci-michel.com](https://merci-michel.com/) | Web games, playful microsites that get shared |
| **Immersive Garden** | [immersive-g.com](https://immersive-g.com/) | Calm premium design, precise motion, luxury/editorial |
| **Dogstudio** | [dogstudio.co](https://dogstudio.co/) | Bold creative execution, distinctive visual signatures |
| **Garden Eight** | [garden-eight.com](https://garden-eight.com/) | Minimal Japanese aesthetics, illustration-driven storytelling |
| **Noomo Agency** | [noomoagency.com](https://noomoagency.com/) | Web3 + Webflow + custom Three.js, quick-to-ship |

### Key Insight from Lusion
> "Every project gets its own system, its own logic, and its own flavour" — no templates, bespoke technical solutions per project. Self-initiated R&D projects (My Little Storybook) are used to test ideas and grow the team.

**Codrops interview:** [Lusion: Where Digital Craft Meets Ambitious Experimentation](https://tympanus.net/codrops/2026/04/13/lusion-where-digital-craft-meets-ambitious-experimentation/)

---

## 5. Codrops Demos & Tutorials

The absolute best source for learning production Three.js techniques. 51 tutorials published in 2025 alone.

### 2026 Highlights

| Demo | Techniques | VP Lab Application |
|------|-----------|-------------------|
| [**False Earth: WebGPU-Driven World**](https://tympanus.net/codrops/2026/04/21/false-earth-from-webgl-limits-to-a-webgpu-driven-world/) | Infinite procedural landscape, compute shaders, indirect drawing, interactive grass blades | Procedural terrain generation, grass rendering |
| [**160,000 Cubes Dithering Visualization**](https://tympanus.net/codrops/2026/04/01/animating-160000-cubes-in-three-js-to-visualize-dithering/) | Large-scale instanced animation, custom dithering shaders | Massive instanced geometry scenes |
| [**Dual-Scene Fluid X-Ray Reveal**](https://tympanus.net/codrops/2026/03/23/building-a-dual-scene-fluid-x-ray-reveal-effect-in-three-js/) | Two-scene blending, fluid simulation, dynamic reveal masks | Scene transition effects |
| [**Seamless 3D Transitions (Webflow + GSAP + Barba.js)**](https://tympanus.net/codrops/2026/03/18/building-seamless-3d-transitions-with-webflow-gsap-and-three-js/) | Persistent Three.js scene across page navigations, GSAP-driven transitions | Never-reload scene architecture |
| [**Scroll-Reactive 3D Gallery**](https://tympanus.net/codrops/2026/03/09/building-a-scroll-reactive-3d-gallery-with-three-js-velocity-and-mood-based-backgrounds/) | Depth-layered images, velocity-responsive motion, mood-based backgrounds | Scroll-driven galleries with depth |
| [**Corentin Bernadou Portfolio Case Study**](https://tympanus.net/codrops/2026/03/05/inside-corentin-bernadous-portfolio-swiss-inspired-layouts-webgl-geometry-and-thoughtful-motion/) | Swiss-inspired layouts, WebGL geometry, editorial motion | Design-forward WebGL integration |
| [**Three.js Conference Website**](https://tympanus.net/codrops/2026/02/28/when-community-becomes-ui-building-the-website-for-the-first-three-js-conference/) | SSGI (Screen-Space Global Illumination), community-driven UI | Raytraced color bleeding for realism |
| [**3D Product Grid (R3F)**](https://tympanus.net/codrops/2026/02/24/from-flat-to-spatial-creating-a-3d-product-grid-with-react-three-fiber/) | Curved 3D grid, GLSL shaders, performance optimization | Product showcases |
| [**Composite Rendering Transitions**](https://tympanus.net/codrops/2026/02/23/composite-rendering-the-brilliance-behind-inspiring-webgl-transitions/) | Render targets, composite rendering, advanced scene transitions | Multi-scene compositing |
| [**Horizontal Parallax Gallery (DOM to WebGL)**](https://tympanus.net/codrops/2026/02/19/creating-a-smooth-horizontal-parallax-gallery-from-dom-to-webgl/) | DOM-to-WebGL upgrade, shader-driven parallax | Hybrid DOM/WebGL layouts |
| [**Scroll-Driven 3D Image Tube (R3F)**](https://tympanus.net/codrops/2026/02/17/reactive-depth-building-a-scroll-driven-3d-image-tube-with-react-three-fiber/) | Infinite looping tube, shader-driven scrolling, inertia | Cylindrical image galleries |
| [**Endless Procedural Snake**](https://tympanus.net/codrops/2026/02/10/building-an-endless-procedural-snake-with-three-js-and-webgl/) | GPU-enhanced procedural curves, steering rules, Bezier paths | Procedural path/ribbon generation |
| [**WebGPU Gommage: MSDF Text Dissolution**](https://tympanus.net/codrops/2026/01/28/webgpu-gommage-effect-dissolving-msdf-text-into-dust-and-petals-with-three-js-tsl/) | MSDF text, noise-driven TSL shaders, particle dissolution | Text-to-particle effects |
| [**Pixel-to-Voxel Video Drop (Rapier)**](https://tympanus.net/codrops/2026/01/05/how-to-create-a-pixel-to-voxel-video-drop-effect-with-three-js-and-rapier/) | Video voxelization, Rapier physics simulation | Video-to-3D transitions |

### 2025 Highlights

| Demo | Techniques | VP Lab Application |
|------|-----------|-------------------|
| [**Interactive Video Projection Mapping**](https://tympanus.net/codrops/2025/08/28/interactive-video-projection-mapping-with-three-js/) | Cube grid, UV-mapped video textures, masking | Virtual projection mapping |
| [**Interactive Text Destruction (WebGPU + TSL)**](https://tympanus.net/codrops/2025/07/22/interactive-text-destruction-with-three-js-webgpu-and-tsl/) | Letters explode into dynamic shapes, TSL shaders | Destructible text effects |
| [**3D Audio Visualizer (GSAP + Web Audio)**](https://tympanus.net/codrops/2025/06/18/coding-a-3d-audio-visualizer-with-three-js-gsap-web-audio-api/) | Sci-fi "anomaly detector" reacting to music in real-time | Audio-reactive scenes |
| [**3D World in Browser (Blender + Three.js)**](https://tympanus.net/codrops/2025/04/08/3d-world-in-the-browser-with-blender-and-three-js/) | Immersive 3D museum, Blender-to-web pipeline | Full environment construction |
| [**Living Particle System (UntilLabs)**](https://tympanus.net/codrops/2025/12/10/simulating-life-in-the-browser-creating-a-living-particle-system-for-the-untillabs-website/) | Physics-driven particles, data-driven motion, shaders | Organic particle ecosystems |
| [**3D Hand Controller (MediaPipe + Three.js)**](https://tympanus.net/codrops/2024/10/24/creating-a-3d-hand-controller-using-a-webcam-with-mediapipe-and-three-js/) | Webcam hand tracking, depth-based 3D control | Gesture-controlled scenes |
| [**Dreamy GPGPU Particle Effect**](https://tympanus.net/codrops/2024/12/19/crafting-a-dreamy-particle-effect-with-three-js-and-gpgpu/) | GPU-computed particles, ethereal motion | Atmospheric particle effects |
| [**Audio-Reactive Dynamic Particles**](https://tympanus.net/codrops/2023/12/19/creating-audio-reactive-visuals-with-dynamic-particles-in-three-js/) | Web Audio API frequency data driving particle motion | Music visualization |

> Full archive: [tympanus.net/codrops/tag/three-js](https://tympanus.net/codrops/tag/three-js/)
> Demo hub (1000+ demos): [tympanus.net/codrops/hub/tag/three-js](https://tympanus.net/codrops/hub/tag/three-js/)

---

## 6. Three.js + AI / Gaussian Splatting

### Gaussian Splatting Viewers

| Library | URL | Key Feature |
|---------|-----|------------|
| **Spark** | [sparkjs.dev](https://sparkjs.dev/) / [GitHub](https://github.com/sparkjsdev/spark) | Advanced 3DGS renderer for Three.js. Fuses splats with mesh objects. 98%+ WebGL2 device support. Multiple format support (ply, sogs, spz, splat, ksplat). |
| **GaussianSplats3D** | [Demo](https://projects.markkellogg.org/threejs/demo_gaussian_splats_3d.php) / [GitHub](https://github.com/mkkellogg/GaussianSplats3D) | Works with .ply, .splat, .ksplat files. The original Three.js gaussian splat renderer. |
| **Luma AI Library** | [three.js forum](https://discourse.threejs.org/t/luma-ais-three-js-and-r3f-gaussian-splatting-library/58960) | R3F-compatible gaussian splatting from Luma AI captures. |
| **SplatLabJs** | [Live demo](https://cs-util-com.github.io/SplatLabJs/) / [GitHub](https://github.com/cs-util-com/SplatLabJs) | Viewer + editor for blending, combining, measuring multiple splats. Transform controls. |

### AI + 3D Style Transfer (Research)

- **CLIPGaussian** — Universal multimodal style transfer on 3D Gaussian Splatting (NeurIPS 2025)
- **Styl3R** — Instant 3D stylized reconstruction for arbitrary scenes/styles (NeurIPS 2025)
- **ABC-GS** — 3D Gaussian Splatting framework for high-quality 3D style transfer

### VP Lab Application
- Capture real environments with phone/drone photos, reconstruct as Gaussian splats, render in Three.js
- Mix photorealistic splat environments with stylized Three.js meshes
- Style transfer on 3DGS scenes for artistic virtual production looks
- Spark library is production-ready for integrating splats into Three.js scenes

---

## 7. pmndrs / React Three Fiber Ecosystem

### Core Libraries

| Package | Purpose |
|---------|---------|
| **react-three-fiber** | React renderer for Three.js — declarative 3D scene graphs |
| **drei** | 100+ ready-made abstractions: Environment, Stage, useGLTF, Text3D, Float, MeshDistortMaterial, etc. |
| **postprocessing** | High-performance post-processing: Bloom, SSAO, God Rays, Glitch, ChromaticAberration |
| **react-spring/three** | Physics-based animation for 3D |
| **rapier** | Rust-based physics via WASM — modern Cannon.js replacement |
| **zustand** | Lightweight state management (3D scene state) |

### Drei Highlights for VP Lab

| Helper | What It Does |
|--------|-------------|
| `<Environment preset="sunset" />` | One-line HDRI environment maps from presets |
| `<Stage>` | Studio lighting + ground shadows + auto-zoom to fit |
| `useGLTF()` | Load GLTF/GLB models with auto-Draco decompression |
| `<Float>` | Gentle floating animation for objects |
| `<MeshDistortMaterial>` | Organic distortion on meshes |
| `<MeshReflectorMaterial>` | Reflective floors/surfaces |
| `<Text3D>` | 3D extruded text from fonts |
| `<Sparkles>` | Particle sparkle effects |
| `<Stars>` | Starfield backgrounds |
| `<Sky>` | Procedural sky with sun position |
| `<Cloud>` | Volumetric cloud meshes |

### Realism Effects (0beqz)

**URL:** [GitHub](https://github.com/0beqz/realism-effects)

- **SSGIEffect** — Screen-Space Global Illumination (raytraced color bleeding)
- **TRAAEffect** — Temporal Resolve Anti-Aliasing
- **MotionBlurEffect** — Per-object motion blur
- SSGI replaces RenderPass entirely (does its own rendering), saving a full pass

> **VP Lab:** R3F + drei + realism-effects = fastest path to cinematic quality in the browser.

---

## 8. Shader Techniques & Ports

### Shadertoy to Three.js Conversion

**Key guide:** [Three.js Fundamentals — Shadertoy](https://threejs.org/manual/en/shadertoy.html)
**Practical guide:** [Felix Rieseberg's blog](https://felixrieseberg.com/using-webgl-shadertoy-shaders-in-three-js/)

**Conversion recipe:**
1. Map Shadertoy uniforms: `iTime`, `iTimeDelta`, `iResolution`, `iMouse`
2. Default vertex shader with `varying vec2 vUv`
3. Wrap `mainImage()` call in a `main()` function
4. Most conversions require redeclaring Shadertoy-specific built-in uniforms

### Maxime Heckel's Shader Blog (Must-Read)

**URL:** [blog.maximeheckel.com](https://blog.maximeheckel.com)

| Article | What You Learn |
|---------|---------------|
| [Study of Shaders with R3F](https://blog.maximeheckel.com/posts/the-study-of-shaders-with-react-three-fiber/) | Shader fundamentals in React Three Fiber context |
| [Magical World of Particles](https://blog.maximeheckel.com/posts/the-magical-world-of-particles-with-react-three-fiber-and-shaders/) | 8 unique particle scenes, attributes, buffer geometries, FBO technique |
| [Dithering & Retro Shading](https://blog.maximeheckel.com/posts/the-art-of-dithering-and-retro-shading-web/) | Dithering as a shader-based style technique |
| [Moebius Post-Processing](https://blog.maximeheckel.com/posts/moebius-style-post-processing/) | Stylized post-processing reproducing Jean Giraud's art style |
| [Refraction & Dispersion](https://blog.maximeheckel.com/posts/refraction-dispersion-and-other-shader-light-effects/) | Glass, diamond, prismatic light effects |
| [Caustics in WebGL](https://blog.maximeheckel.com/posts/caustics-in-webgl/) | Light caustic patterns with shaders |
| [TSL & WebGPU Field Guide](https://blog.maximeheckel.com/posts/field-guide-to-tsl-and-webgpu/) | Modern shader development with Three.js Shading Language |

> **VP Lab:** Maxime's blog is the single best resource for learning production shader techniques with R3F. Every article has interactive playgrounds.

### Holographic / Cyberpunk Shaders

| Resource | URL | Details |
|----------|-----|---------|
| **Holographic Material (R3F)** | [GitHub](https://github.com/ektogamat/threejs-holographic-material) | fresnelAmount, scanlineSize, hologramBrightness, signalSpeed, blinking |
| **Holographic Material (Vanilla)** | [GitHub](https://github.com/ektogamat/threejs-vanilla-holographic-material) | Same effects without React |
| **Hologram Shader Lesson** | [Three.js Journey](https://threejs-journey.com/lessons/hologram-shader) | Fresnel effect + stripe pattern (modulo on vPosition.y) + vertex glitch via sine waves |

---

## 9. WebGPU & TSL

### Three.js Shading Language (TSL)

TSL is Three.js's higher-level, JavaScript-like shading language. Write shaders in JS, they auto-compile to:
- **WGSL** (WebGPU) when supported
- **GLSL** (WebGL) as fallback

**Official docs:** [threejs.org/docs/pages/TSL.html](https://threejs.org/docs/pages/TSL.html)
**Wiki:** [GitHub wiki](https://github.com/mrdoob/three.js/wiki/Three.js-Shading-Language)

### Key TSL Capabilities
- Multiple Render Targets (MRT) — capture color, normals, depth, velocity in a single pass
- Compute shaders for particle physics (millions of particles in <1ms)
- Cross-browser shader portability (write once, run on WebGPU or WebGL)

### Notable WebGPU + TSL Demos

| Demo | URL | Techniques |
|------|-----|-----------|
| **Samsy.ninja cyberpunk world** | [samsy.ninja](https://samsy.ninja/) | TSL shaders, custom bloom/reflections, 120+ FPS |
| **Interactive 3D Particle Scene** | [three.js forum](https://discourse.threejs.org/t/interactive-3d-scene-built-with-three-js-webgpu-and-tsl/90492) | 4,000 particles orbiting glowing cube, custom TSL physics |
| **Patina: Real-Time Weathering** | [webgpu.com](https://www.webgpu.com/showcase/patina-preview-real-time-weathering-threejs-tsl/) | Procedural PBR playground — metal aging, oxidation in cavities, highlight dulling via TSL node graph |
| **Shader.se: 80s Corporate Tape** | [webgpu.com](https://www.webgpu.com/showcase/shader-se-webgpu-tsl-studio-site/) | 1987 training reel aesthetic, bold geometric scenes in real-time WebGPU |
| **False Earth** | [Codrops](https://tympanus.net/codrops/2026/04/21/false-earth-from-webgl-limits-to-a-webgpu-driven-world/) | Infinite procedural landscape, compute shaders, interactive grass |

### WebGPU Compute: Million-Particle Galaxy

- CPU particle updates hit bottleneck at ~50,000 particles
- WebGPU compute shaders push to **millions** — update in <1ms
- Three.js TSL `StorageBufferAttribute` stores 1M+ positions on GPU
- **Reference:** [Interactive Galaxy with WebGPU Compute Shaders](https://threejsroadmap.com/blog/galaxy-simulation-webgpu-compute-shaders)

### Migration Checklist

**Reference:** [Migrate Three.js to WebGPU (2026)](https://www.utsubo.com/blog/webgpu-threejs-migration-guide)

> **VP Lab:** TSL is the future. Write shaders in JS, get WebGPU performance with WebGL fallback. Prioritize TSL over raw GLSL for new effects.

---

## 10. Specific Effect Techniques

### Water & Ocean

| Resource | URL | Quality |
|----------|-----|---------|
| **Three.js Water Pro** | [threejsroadmap.com](https://threejsroadmap.com/assets/threejs-water-pro) | Physically-based WebGPU ocean: FFT wave sim, dynamic foam, caustics, atmospheric rendering. Hurricane to tropical shallows. |
| **Realistic Water Simulation** | [water-simulation.vercel.app](https://water-simulation.vercel.app/) | RGB caustics, underwater distortion, screen droplets. R3F + GLSL. |
| **Official Ocean Shader** | [threejs.org/examples](https://threejs.org/examples/webgl_shaders_ocean.html) | Built-in ocean example |
| **GPGPU Water** | threejs.org/examples | GPU-computed water wave simulation |
| **jbouny/ocean** | [GitHub](https://github.com/jbouny/ocean) | Realistic plane water shader |

### Volumetric Lighting / God Rays

| Resource | URL | Technique |
|----------|-----|-----------|
| **Codrops Tutorial** | [Volumetric Light Rays](https://tympanus.net/codrops/2022/06/27/volumetric-light-rays-with-three-js/) | Fragment shader god rays |
| **Fake Volumetric** | three.js forum | Translucent ConeGeometry with additive blending |
| **Path Tracing Light Shafts** | [THREE.js-PathTracing-Renderer](https://erichlof.github.io/THREE.js-PathTracing-Renderer/) | True volumetric via path tracing |
| **WebGPU Volumetric** | [three.js forum](https://discourse.threejs.org/t/volumetric-lighting-in-webgpu/87959) | Modern WebGPU approach |

### Path Tracing / Ray Tracing

| Project | URL | Achievement |
|---------|-----|------------|
| **THREE.js-PathTracing-Renderer** | [Live Demo](https://erichlof.github.io/THREE.js-PathTracing-Renderer/Geometry_Showcase.html) / [GitHub](https://github.com/erichlof/THREE.js-PathTracing-Renderer) | ~1M polygons path-traced in real-time in browser. Stanford Dragon (100K tris) fully path traced even on mobile. Custom denoiser for clean diffuse. |
| **three-gpu-pathtracer** | [GitHub](https://github.com/gkjohnson/three-gpu-pathtracer) | Modular shader-based path tracing extension. BVH-accelerated. |
| **WebGPU Raytracer** | [threejsresources.com](https://threejsresources.com/tool/webgpu-raytracer) | WebGPU-native ray tracing engine |

### Postprocessing Pipeline

| Effect | Library | Notes |
|--------|---------|-------|
| **Bloom** | pmndrs/postprocessing | UnrealBloomPass — resolution, strength, radius, threshold |
| **SSGI** | 0beqz/realism-effects | Replaces RenderPass, raytraced color bleeding |
| **TRAA** | 0beqz/realism-effects | Temporal anti-aliasing |
| **Motion Blur** | 0beqz/realism-effects | Per-object motion blur |
| **Selective Bloom** | pmndrs/postprocessing | Bloom only specific objects by layer |
| **Glitch** | three.js built-in | GlitchPass for datamoshing effects |

### Scroll-Driven Animation

**Best approach:** Three.js + GSAP ScrollTrigger

Key patterns:
- Camera translate following scroll position
- Parallax: camera moves horizontally/vertically with mouse for depth perception
- Mood-based: each section defines a color palette driving background gradients
- Velocity as signal: scroll speed modulates scene intensity
- Progressive revelation for storytelling pacing

**References:**
- [Builder.io — Apple-style 3D scroll animations](https://www.builder.io/blog/webgl-scroll-animation)
- [Codrops — Crafting Scroll Based Animations](https://tympanus.net/codrops/2022/01/05/crafting-scroll-based-animations-in-three-js/)
- [Three.js Journey lesson](https://threejs-journey.com/lessons/scroll-based-animation)

### Audio-Reactive Visuals

**Core technique:** Web Audio API `AnalyserNode` → `getFrequencyData()` → feed into shader uniforms or mesh deformation each frame.

- Modulate sphere size based on beat signature
- Deform surface with procedural noise using audio data as input
- Layer sound-reactive shader animations with event-based GSAP tweens

**References:**
- [Codrops — 3D Audio Visualizer (2025)](https://tympanus.net/codrops/2025/06/18/coding-a-3d-audio-visualizer-with-three-js-gsap-web-audio-api/)
- [Codrops — Audio-Reactive Particles](https://tympanus.net/codrops/2023/12/19/creating-audio-reactive-visuals-with-dynamic-particles-in-three-js/)

### Procedural Generation

| Technique | Resource |
|-----------|---------|
| **Terrain** | [THREE.Terrain](https://github.com/IceCreamYou/THREE.Terrain) — Diamond-Square, Perlin, Simplex, Worley noise, Brownian motion, auto-texturing by elevation/slope |
| **Cities** | Grid subdivision + random building dimensions + instanced meshes |
| **Infinite worlds** | Chunk-based loading, LOD, frustum culling |

---

## 11. Viral Demos & Experiments

### Iconic Three.js Experiences

| Demo | URL | Why It's Famous |
|------|-----|----------------|
| **Bruno Simon's Portfolio** | [bruno-simon.com](https://bruno-simon.com) | The 3D driving game that proved portfolios don't need to be boring |
| **NASA Eyes on the Solar System** | [eyes.nasa.gov](https://eyes.nasa.gov/apps/solar-system/#/home) | Real-time explorable solar system with actual orbital data |
| **Paper Planes** | [paperplanes.world](https://paperplanes.world) | Throw/catch paper planes globally — shared multiplayer |
| **The Boat (SBS)** | [sbs.com.au/theboat](https://www.sbs.com.au/theboat/) | Interactive graphic novel, Vietnamese refugee story |
| **Inside Music (Google)** | [experiments.withgoogle.com](https://experiments.withgoogle.com/inside-music) | Explore song layers in 3D space, isolate instruments |
| **Chartogne-Taillet** | [chartogne-taillet.com](https://chartogne-taillet.com/en) | Immersive vineyard virtual tour |
| **Wayfinder (NFB Canada)** | [nfb.ca](https://www.nfb.ca/interactive/wayfinder) | Contemplative art game, procedural generative landscapes |
| **GPGPU Birds** | [threejs.org](https://threejs.org/examples/webgl_gpgpu_birds.html) | GPU-accelerated flocking simulation with mouse interaction |

### Three.js Conference 2026

- **When:** September 10-11, 2026, Paris
- **Website:** Features SSGI (Screen-Space Global Illumination) — raytraced color bleeding
- **Case study:** [Codrops](https://tympanus.net/codrops/2026/02/28/when-community-becomes-ui-building-the-website-for-the-first-three-js-conference/)

---

## 12. Key Takeaways for VP Lab

### Rendering Strategy

1. **Matcap shading** for maximum performance on stylized scenes (zero lights needed)
2. **PBR (MeshPhysicalMaterial)** for photorealistic scenes — metalness, roughness, clearcoat, sheen
3. **SSGI via realism-effects** for production-quality global illumination
4. **Baked shadows from Blender** for static environments (PNG-8, TinyPNG compressed)
5. **TSL over raw GLSL** for new shader work — future-proof, WebGPU-ready

### Performance Patterns

1. **Instanced Mesh** for repeated geometry (100K+ objects at 60fps)
2. **WebGPU compute shaders** for million-particle simulations
3. **Draco compression** on all glTF exports (50-70% file size reduction)
4. **VAO optimization** for reduced WebGL draw calls
5. **Mobile: clamp pixel ratio, disable matrix auto-update, reduce postprocessing**

### Scene Types to Build

| Scene Type | Inspiration | Techniques |
|-----------|-------------|-----------|
| **Cyberpunk cityscape** | Samsy.ninja | WebGPU + TSL, neon bloom, holographic UIs, first-person controls |
| **Natural environment** | Jordan Breton, False Earth | Particle-driven fire/water/wind, procedural terrain, grass blades via compute |
| **Scroll narrative** | Bilal.show, Sebastien Lempens | GSAP ScrollTrigger, camera choreography, progressive revelation |
| **Product showcase** | Gucci Virtual, 3D Product Grid | PBR materials, environment maps, Stage helper, MeshReflectorMaterial |
| **Audio-reactive** | Codrops visualizers | Web Audio API + shader uniforms, beat-driven deformation |
| **Gaussian splat hybrid** | Spark + Three.js | Real captured environments mixed with stylized 3D objects |
| **Stylized/artistic** | Moebius post-processing, dithering | Custom post-processing for non-photorealistic rendering |
| **Interactive game world** | Bruno Simon, Thibault Introvigne | Physics (Rapier/Cannon), character controls, collectibles |
| **Portal/transition** | Codrops composite rendering | Render targets, scene blending, displacement morphing |

### Essential Libraries

```
three.js          — Core renderer
@react-three/fiber — React integration (if using React)
@react-three/drei  — 100+ helpers (Environment, Stage, useGLTF, etc.)
postprocessing     — Bloom, SSAO, God Rays, Glitch
realism-effects    — SSGI, TRAA, Motion Blur
gsap + ScrollTrigger — Scroll-driven animation
rapier3d           — Modern physics (Rust/WASM)
spark              — Gaussian splatting integration
howler.js          — Spatialized audio
lil-gui / tweakpane — Debug panels
```

### Learning Path

1. **Start:** [threejs-journey.com](https://threejs-journey.com/) (Bruno Simon's course)
2. **Shaders:** [blog.maximeheckel.com](https://blog.maximeheckel.com) (interactive shader tutorials)
3. **Production techniques:** [tympanus.net/codrops/tag/three-js](https://tympanus.net/codrops/tag/three-js/) (51+ tutorials/year)
4. **WebGPU/TSL:** [TSL Field Guide](https://blog.maximeheckel.com/posts/field-guide-to-tsl-and-webgpu/)
5. **Performance:** [100 Three.js Tips (2026)](https://www.utsubo.com/blog/threejs-best-practices-100-tips)
6. **Awards gallery:** [awwwards.com/websites/three-js](https://www.awwwards.com/websites/three-js/)

---

## Source Index

### Award Sites & Galleries
- [Awwwards Three.js Collection](https://www.awwwards.com/websites/three-js/)
- [Awwwards WebGL Collection](https://www.awwwards.com/websites/webgl/)
- [Codrops Creative Hub (1000+ demos)](https://tympanus.net/codrops/hub/tag/three-js/)
- [Official Three.js Examples](https://threejs.org/examples/)
- [threejsdemos.com](https://threejsdemos.com/)

### Deep-Dive Articles
- [Bruno Simon Case Study (Medium)](https://medium.com/@bruno_simon/bruno-simon-portfolio-case-study-960402cc259b)
- [Best Three.js Portfolio Examples 2026 (CreativeDevJobs)](https://www.creativedevjobs.com/blog/best-threejs-portfolio-examples-2025)
- [10 Best Three.js Agencies (Utsubo)](https://www.utsubo.com/blog/top-threejs-agencies)
- [Lusion Interview (Codrops)](https://tympanus.net/codrops/2026/04/13/lusion-where-digital-craft-meets-ambitious-experimentation/)
- [10 Impressive Three.js Examples](https://threejsresources.com/blog/10-impressive-threejs-examples-and-what-we-can-learn-from-them)

### Technical References
- [TSL Documentation](https://threejs.org/docs/pages/TSL.html)
- [WebGPU Migration Guide](https://www.utsubo.com/blog/webgpu-threejs-migration-guide)
- [100 Three.js Performance Tips](https://www.utsubo.com/blog/threejs-best-practices-100-tips)
- [Maxime Heckel's Shader Blog](https://blog.maximeheckel.com)
- [Shadertoy to Three.js Guide](https://threejs.org/manual/en/shadertoy.html)
- [realism-effects (SSGI/TRAA)](https://github.com/0beqz/realism-effects)
- [Spark Gaussian Splatting](https://sparkjs.dev/)

### Studio Websites
- [Lusion](https://lusion.co/) | [Active Theory](https://activetheory.net/) | [Resn](https://resn.co.nz/) | [Unseen Studio](https://unseen.co/) | [Merci-Michel](https://merci-michel.com/) | [Immersive Garden](https://immersive-g.com/) | [Dogstudio](https://dogstudio.co/) | [Garden Eight](https://garden-eight.com/)
