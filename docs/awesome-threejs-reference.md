# Awesome Three.js — Full Reference

Source: [github.com/AxiomeCG/awesome-threejs](https://github.com/AxiomeCG/awesome-threejs) (888 stars, CC0 license)
Last synced: 2026-04-23

---

## Core

- [Three.js official site](https://threejs.org/)
- [Three.js examples](https://threejs.org/examples/#webgl_animation_keyframes)
- [Three.js documentation](https://threejs.org/docs/index.html#manual/en/introduction/Creating-a-scene)

---

## Books

### 3D Theory
- [3D Math Primer for Graphics and Game Development](https://gamemath.com/book/intro.html) — Must-read for 3D math comfort
- [Physically Based Rendering](https://pbr-book.org/) — The PBR bible (theory to implementation)

### Creative Coding
- [The Nature of Code](https://natureofcode.com/) by Daniel Shiffman — Natural simulations with Processing

### Three.js
- [Discover Three.js](https://discoverthreejs.com/)
- [Learn Three.js - Third Edition](https://www.packtpub.com/product/learn-three-js-third-edition/9781788833288)

---

## Courses

- [Three.js Journey](https://threejs-journey.com/) by Bruno Simon — Best beginner course, builds 3D experiences from scratch
- [The Easiest Way to Learn GLSL](https://simondev.teachable.com/p/glsl-shaders-from-scratch) by SimonDev
- [The Book of Shaders](https://thebookofshaders.com/) — Free, best GLSL reference

---

## Articles

### Documentation
- [Three.js Fundamentals](https://threejs.org/manual/#en/fundamentals)
- [Shaderific for OpenGL](https://shaderific.com/index.html) — GLSL docs
- [GLSL documentation](https://docs.gl/sl4/clamp)

### 3D Theory
- [Explaining Homogeneous Coordinates & Projective Geometry](https://www.tomdalling.com/blog/modern-opengl/explaining-homogenous-coordinates-and-projective-geometry/)

### Tutorials
- [Surface Sampling in Three.js](https://tympanus.net/codrops/2021/08/31/surface-sampling-in-three-js/) — MeshSurfaceSampler usage
- [Fake 3D Image Effect with WebGL](https://tympanus.net/codrops/2019/02/20/how-to-create-a-fake-3d-image-effect-with-webgl/) — Depth from 2D images
- [Tutorial on Matrices](http://www.opengl-tutorial.org/beginners-tutorials/tutorial-3-matrices/) — Visual shader matrix tutorial

### Water
- [Real-time rendering of water caustics](https://medium.com/@martinRenou/real-time-rendering-of-water-caustics-59cda1d74aa) — Screen-space caustics approach
- [Gentle intro to fluid simulation](https://shahriyarshahrabi.medium.com/gentle-introduction-to-fluid-simulation-for-programmers-and-technical-artists-7c0045c40bac) — Realistic water creation

### Generative Art
- [Generative Artistry tutorials](https://generativeartistry.com/tutorials/) — Evolutionary generative art lessons

### Collision Detection
- [Bounding volume collision detection](https://developer.mozilla.org/en-US/docs/Games/Techniques/3D_collision_detection/Bounding_volume_collision_detection_with_THREE.js)
- [Physics-based collision with Ammo.js](https://medium.com/@bluemagnificent/collision-detection-in-javascript-3d-physics-using-ammo-js-and-three-js-31a5569291ef)

---

## Inspiration

- [same.energy](https://same.energy/) — Visual search engine by keyword or picture
- [ShaderToy](https://www.shadertoy.com/) — Shader sharing/discovery platform
- [Pinterest](https://www.pinterest.fr/)

---

## Resources

### Asset Libraries
| Source | Type | License |
|--------|------|---------|
| [Poly Haven](https://polyhaven.com/) | Textures, Models, HDRI | CC0 |
| [ambientCG](https://ambientcg.com/) | PBR Textures | CC0 |
| [3D Textures](https://3dtextures.me/) | PBR (all maps) | Free |
| [Poliigon](https://www.poliigon.com/) | Textures, Models, HDRI | Paid |
| [Arroway Textures](https://www.arroway-textures.ch/) | Digital textures | Paid |
| [Matcap repository](https://github.com/nidorx/matcaps) | Matcaps | Free |
| [Three.js Resources](https://threejsresources.com/) | Curated everything | Free |

### GLSL Shader References
- [Signal shaping functions](https://iquilezles.org/articles/functions/) by Inigo Quilez — Essential GLSL patterns
- [Shaping functions](http://www.flong.com/archive/texts/code/) by Golan Levin
- [Cheat sheet on curves](https://www.flickr.com/photos/kynd/9546075099/) by kynd
- [GLSL Noises](https://gist.github.com/patriciogonzalezvivo/670c22f3966e662d2f83) — Perlin, simplex, etc.
- [Realistic water shader](https://github.com/jbouny/ocean) — With full GLSL explanations
- [PixelSpirit Deck](https://pixelspiritdeck.com/) — Tarot-ordered GLSL exercises (simple to complex)
  - [GitHub source](https://github.com/patriciogonzalezvivo/PixelSpiritDeck)

---

## Tools

### Debug / Optimization
- [GLTF Report](https://gltf.report) — Web app to diagnose + optimize GLTF, supports BASIS/KTX2
- [gltf-transform](https://gltf-transform.dev/) — CLI: weld, prune, draco compress, KTX2 convert

### Scene Creation
- [Polygonjs](https://polygonjs.com) — Node-based WebGL design tool (procedural geometry, particles, materials)
- [Three-Blender](https://github.com/ppmpreetham/three-blender) — Blender to Three.js converter
- [Three.js Editor](https://threejs.org/editor/) — Built-in scene editor

### 3D Modeling
- [Blender](https://www.blender.org/) — Free, powerful
- [Houdini](https://www.sidefx.com/products/houdini/) — Procedural (Apprentice = free)
- [Spline](https://spline.design/) — Collaborative 3D modeling, exports to three.js

### Materials
- [Adobe Substance3D](https://www.adobe.com/fr/products/substance3d/3d-augmented-reality.html)

### Shader Tools (Online)
| Tool | What it does |
|------|-------------|
| [GraphToy](https://graphtoy.com/) | Test shaping signals (by Inigo Quilez) |
| [ShaderShop](http://tobyschachman.com/Shadershop/editor/) | Visual drag-and-drop pattern editor |
| [NodeToy](https://app.nodetoy.co/) | Full web shader editor, React integration via react-nodetoy |
| [Shader Park](https://shaderpark.com/) | JS-based procedural shaders with built-in raymarcher |

### Shader Tools (Installed)
- [glslViewer](https://github.com/patriciogonzalezvivo/glslViewer) — Console-based GLSL sandbox

### Other
- [HDRI-to-CubeMap](https://matheowis.github.io/HDRI-to-CubeMap/) — Convert HDRI for skyboxes

---

## Libraries

### Shaders
- [lygia](https://github.com/patriciogonzalezvivo/lygia) — Granular multi-language shader library (GLSL/HLSL)

### Animation
- [GSAP](https://greensock.com/gsap/) — Industry standard (ScrollTrigger, Flip, etc.)

### Framework Integrations

**React:**
- [react-three-fiber (R3F)](https://github.com/pmndrs/react-three-fiber) — Declarative Three.js
- [drei](https://github.com/pmndrs/drei) — R3F helpers (controls, text, environments, etc.)
- [react-postprocessing](https://github.com/pmndrs/react-postprocessing) — Bloom, glitch, DOF
- [react-spring](https://www.react-spring.dev/) — Physics-based animations (has @react-spring/three)
- [framer-motion-3d](https://www.framer.com/motion/three-introduction/) — Framer Motion for R3F

**Vue:**
- [TresJs](https://github.com/tresjs/tres) — Declarative Three.js with Vue components
- [Cientos](https://github.com/Tresjs/cientos) — TresJs helpers
- [tres-post-processing](https://github.com/Tresjs/post-processing)

**Svelte:**
- [Threlte](https://github.com/threlte/threlte) — Three.js component library for Svelte
- [threlte-postprocessing](https://github.com/1bye/threlte-postprocessing)

**Angular:**
- [angular-three](https://github.com/nartc/angular-three) — Declarative Three.js for Angular

### Physics
| Engine | Notes |
|--------|-------|
| [Rapier](https://github.com/dimforge/rapier) | Modern, Rust-based, best performance |
| [cannon-es](https://github.com/pmndrs/cannon-es) | JS-native, good for simple scenes |
| [Ammo.js](https://github.com/kripken/ammo.js/) | Bullet port, most features, heaviest |
| [Oimo.js](https://lo-th.github.io/Oimo.js/#basic) | Lightweight, basic |

### Spatial Querying & Raycasting
- [three-mesh-bvh](https://github.com/gkjohnson/three-mesh-bvh) — BVH acceleration for fast raycasting/collision

### Constructive Solid Geometry
- [three-bvh-csg](https://github.com/gkjohnson/three-bvh-csg) — Boolean ops (cut, merge, subtract meshes)

### Pathfinding
- [three-pathfinding](https://github.com/donmccurdy/three-pathfinding) — Navmesh-based
- [Pathfinding.js](https://github.com/qiao/PathFinding.js) — Grid-based, many algorithms
- [Kompute](https://github.com/oguzeroglu/Kompute) — Steering behaviors

### Characters
- [ossos](https://github.com/sketchpunklabs/ossos) — Full skinning + animation library for web
- [mannequin.js](https://boytchev.github.io/mannequin.js/) — Procedural character generation with armature

---

## Demonstrations

### Water
- [fft-ocean](https://github.com/jbouny/fft-ocean) — FFT-based ocean rendering
- [skunami.js](https://github.com/skeelogy/skunami.js/) — Realistic water interaction
- [Shallow water demo](https://vuoriov4.github.io/webgl-water-demo/) — Ripple effect

### Collision Detection
- [AABB collision](https://github.com/mozdevs/gamedev-js-3d-aabb) — Axis-aligned bounding boxes
- [Raycast collision](http://stemkoski.github.io/Three.js/Collision-Detection.html)

---

## Community

- [Three.js Discord](https://discord.com/invite/56GBJwAnUS) — Official
- [Three.js Forum](https://discourse.threejs.org/) — Official
- [Stack Overflow](https://stackoverflow.com/questions/tagged/three.js)
- [Reddit r/threejs](https://www.reddit.com/r/threejs/)
- [Twitter @threejs](https://twitter.com/threejs)

---

## Related Awesome Lists

- [awesome-glsl](https://github.com/vanrez-nez/awesome-glsl)
- [awesome-webgl](https://github.com/sjfricke/awesome-webgl)
- [awesome-webgpu](https://github.com/mikbry/awesome-webgpu)
- [awesome-creative-coding](https://github.com/terkelg/awesome-creative-coding)
- [awesome-computer-vision](https://github.com/jbhuang0604/awesome-computer-vision)
- [graphics-resources](https://github.com/mattdesl/graphics-resources)

---

## VP Lab Takeaways

1. **Shaders > polygons** for visual impact, especially mobile
2. **GSAP** for complex camera animations (timeline, ScrollTrigger)
3. **R3F + drei** if building interactive React tools
4. **Rapier** for physics (modern, fastest)
5. **three-mesh-bvh** for any scene with complex raycasting
6. **Poly Haven + ambientCG** for free CC0 assets
7. **gltf-transform** CLI before loading any imported model
8. **ShaderToy** for discovering cool effects to port
9. **lygia** shader library for reusable GLSL building blocks
10. **ossos** if we ever need character animation in browser
