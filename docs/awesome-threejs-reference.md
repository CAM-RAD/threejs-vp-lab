# Awesome Three.js — Curated Reference

Source: github.com/AxiomeCG/awesome-threejs

---

## Key Resources for VP Lab Work

### Learning
- **Three.js Journey** — Best beginner course
- **The Book of Shaders** — Free GLSL reference (thebookofshaders.com)
- **Three.js Fundamentals** — Official docs walkthrough
- **The Easiest Way to Learn GLSL** — Shader intro

### Shaders (the cool visual stuff)
- **ShaderToy** — Shader sharing/discovery (shadertoy.com)
- **lygia** — Granular multi-language shader library
- **NodeToy** — Web-based visual shader editor with React integration
- **Shader Park** — JS-based procedural shader library
- **GraphToy** — Signal shaping visualization
- **ShaderShop** — Visual shader pattern editor
- **GLSL Noises** — Perlin and other noise implementations
- **Realistic water shader** — With full explanations

### Post-Processing
- **react-postprocessing** — Bloom, glitch, DOF, etc. (React)
- **UnrealBloomPass** — Built into three.js examples (we're using this)

### Physics
- **cannon-es** — Physics engine (objects falling, colliding)
- **Rapier** — Modern physics library
- **Ammo.js** — Bullet physics port
- **Oimo.js** — Lightweight physics

### Animation
- **GSAP** — Industry standard animation library (works with three.js)
- **react-spring** — Physics-based animations

### Assets
- **Poly Haven** — Free CC0 HDRIs, textures, models
- **Poliigon** — Textures, models, HDRI (paid)
- **ambientCG** — Free CC0 PBR textures
- **3D Textures** — Free PBR with multiple maps
- **Matcap repository** — Pre-baked material captures

### Frameworks (if we go React)
- **react-three-fiber (R3F)** — Declarative three.js in React
- **drei** — Helper utilities for R3F (controls, text, environments, etc.)
- **Cientos/TresJs** — Vue equivalent

### Tools
- **GLTF Report** — Diagnose and optimize .gltf/.glb files
- **gltf-transform** — CLI for GLTF optimization + KTX2 compression
- **Spline** — Collaborative 3D modeling (exports to three.js)
- **Three-Blender** — Blender to three.js converter
- **HDRI-to-CubeMap** — Convert HDRIs for skyboxes

### Scene Creation
- **Polygonjs** — Node-based WebGL design tool
- **Three.js Editor** — Official built-in editor

### Advanced
- **three-mesh-bvh** — Fast raycasting/collision acceleration
- **three-bvh-csg** — Boolean operations (cut holes, merge shapes)

---

## Water Resources (for VP ocean scenes)
- Real-time rendering of water caustics (article)
- Gentle introduction to fluid simulation for programmers
- Realistic water shader with GLSL explanations
- Three.js Water Pro (threejswaterpro.com) — Commercial, FFT-based

## Key Takeaways for Our Workflow
1. **Shaders > polygons** for visual impact on mobile
2. **GSAP** for complex animations (camera paths, reveals)
3. **R3F** if we want to build interactive tools (React-based)
4. **Poly Haven** for free textures and HDRIs
5. **ShaderToy** for discovering cool effects to port
6. **gltf-transform** for optimizing imported 3D models
