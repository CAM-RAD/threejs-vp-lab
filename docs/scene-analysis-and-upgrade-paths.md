# Scene Analysis and Upgrade Paths

**Date:** 2026-04-22
**Scope:** All scenes in `threejs-vp-lab/` (desktop + mobile) and `DESIGN-LAB/gs-fox-v4.html`

---

## 1. Per-Scene Deep Analysis

### 1.1 Greek Epic (`scenes/desktop/greek-epic.html`)

**Three.js Features Used:**
- `MeshLambertMaterial` exclusively (no PBR)
- `Water` addon (Three.js example water with normal map texture)
- `DirectionalLight` (single, shadow-casting, 1024x1024 shadow map)
- `AmbientLight` + `HemisphereLight` (3-point ambient strategy)
- `InstancedMesh` for olive trees and cypress trees (60 + 80 instances)
- `FogExp2`-style linear `Fog` (near=400, far=1200)
- `ACESFilmicToneMapping` at 0.75 exposure
- `CatmullRomCurve3` for flythrough camera path (14 waypoints, closed loop)

**Architecture Pattern:**
- Builder functions (`col()`, `temple()`, `shop()`, `house()`, `statue()`, `boat()`) return `THREE.Group` objects
- Flat material dictionary `M = {}` at the top
- All geometry is procedural primitives (Box, Cylinder, Cone, Sphere, Ring, Circle, Plane)
- No post-processing pipeline
- Single animation loop with `requestAnimationFrame`
- UI via native HTML range inputs controlling `tod`, `waves`, `spd`, flythrough toggle

**What's Impressive:**
- Dense, believable city layout with zoned districts (acropolis, agora, harbor, residential, olive groves)
- Well-thought-out flythrough path that tells a story (ocean approach -> dock -> gate -> agora -> acropolis -> residential -> coastal)
- Time-of-day system dynamically adjusts sun position, intensity, exposure, sky/fog color from a single slider
- Instanced meshes for vegetation keep draw calls manageable (~20-30 unique meshes + 2 instanced)

**What Could Be Improved:**
- Lambert materials everywhere means no roughness/metalness variation -- marble, wood, terracotta all have identical surface response
- Single 1024x1024 shadow map for the entire 700-unit scene means shadow resolution is roughly 1 pixel per 0.7 scene units -- visually blurry
- No shadow on instanced meshes' receiving side (trees don't cast shadows onto ground properly because Lambert + no shadow bias tuning)
- Water uses CDN normal map texture (network dependency) but no fallback
- The flythrough speed formula `0.00004 + (spdEl.value/100)*0.00025` makes the speed slider feel non-linear -- exponential mapping would feel better
- Sky is a flat `Color`, not a gradient -- sunrise/sunset doesn't read as convincing

### 1.2 Medieval Town (`scenes/desktop/medieval-town.html`)

**Three.js Features Used:**
- `MeshStandardMaterial` (PBR) for all materials -- roughness/metalness properly varied
- `PCFSoftShadowMap` with 2048x2048 shadow map (biggest in the lab)
- `DirectionalLight` as key + separate `DirectionalLight` as fill + `AmbientLight` + `HemisphereLight` (4-light cinematic rig)
- `PointLight` per torch (13+ torches) with flicker animation
- `ExtrudeGeometry` for roofs (triangular cross-section extruded along depth)
- `TorusGeometry` for bridge arches and cart wheels
- `ShapeGeometry` for river
- `CanvasTexture` for dynamic sky gradient (2x256 canvas, repainted per frame)
- `GLTFLoader` for Objaverse 3D assets (windmill, barrels, lanterns, swords, shields, horses)
- `FogExp2`-style linear `Fog` with dynamic near/far from slider

**Architecture Pattern:**
- Same builder-function pattern (`buildHouse()`, `buildStall()`, `addBarrel()`, `addCrate()`, `addTorch()`, `addSign()`, `addTree()`)
- Material dictionary `M = {}` but now with `emissive` and `metalness` properties
- `placeModel()` helper auto-scales GLTFs to a target height and zero-floors them via bounding box computation
- Sky system is a standalone `updateSky(tod)` function that paints a canvas gradient based on 4 time zones (night/dusk/golden/day)
- `lerpColor()` utility for hex color interpolation

**Custom Techniques:**
- Canvas-based sky gradient is more sophisticated than any other scene -- 4-band gradient with smooth interpolation between night, dusk, golden hour, and day
- Fog color is sampled from the sky canvas at y=200 (horizon) -- `skyCtx.getImageData(0, 200, 1, 1).data` -- this is clever because it auto-matches fog to sky without manual color picking
- Torch flicker uses `Math.sin(time * 8 + Math.random() * 2) * 0.15` -- the `Math.random()` here is called every frame, which gives a noisy flicker but also means it's not deterministic (each frame gets a fresh random, not seeded per torch)
- Torch intensity scales with `nightFactor = 1 - dayFactor` so torches auto-dim at daytime

**What's Impressive:**
- Best lighting setup in the lab: warm key + cool fill + ambient + hemisphere creates genuine cinematic depth
- Half-timber buildings with cross-bracing, window grids, door frames -- more architectural detail than any other scene
- The sky system is production-quality for a no-build-step project
- GLTF asset loading with auto-scaling is reusable infrastructure

**What Could Be Improved:**
- `PointLight` per torch (13+) with no shadow casting is fine for performance, but the torches don't cast any light pools on the ground because the buildings don't have proper receiving surfaces angled toward them
- `ExtrudeGeometry` for roofs is noted as dangerous on mobile in the progress reference -- desktop-only is fine, but it limits scene portability
- No environment map or reflections -- the wet stone, glass windows, and river all use `roughness` but have nothing to reflect
- Flythrough is a simple circular orbit (`Math.sin(flyAngle) * radius`) -- doesn't showcase the town's depth like the Greek Epic's spline path
- The `buildHouse()` function generates windows on the Z+ face only -- side-facing and back-facing walls are blank
- Shadow bias is set to -0.001 but shadow camera bounds are 120x120 -- with 2048 shadow map that's 0.058 units per pixel, which is decent but could be tighter with CSM
- Torch `Math.random()` per frame means flicker is truly random (not just noisy) -- this causes occasional jumps rather than smooth oscillation. Better: per-torch phase offset with multiple sine waves

### 1.3 NYC Chase v2 (`scenes/desktop/nyc-chase-v2.html`)

**Three.js Features Used:**
- `MeshStandardMaterial` with full PBR parameter range (wet asphalt at roughness 0.25, metalness 0.05; glass tower at roughness 0.08, metalness 0.6, transparent)
- `emissive` + `emissiveIntensity` on neon materials (up to 2.5 intensity)
- `PCFSoftShadowMap` at 2048x2048
- `FogExp2` (exponential falloff at 0.005 density)
- `SpotLight` for streetlights (with `penumbra`, `decay`, and explicit `target`)
- `PointLight` for police strobes and neon sign glow
- `BufferGeometry` + `Points` for particle rain (12,000 particles)
- `GLTFLoader` with full preloader system (progress bar, skip button, 10s timeout fallback)
- `powerPreference: 'high-performance'` explicitly requested

**Architecture Pattern:**
- Material dictionary `M = {}` is the most extensive in the lab (28+ materials)
- `addBuilding()` is the most complex builder function: generates storefront, window grid (random lit/cool/off), cornice, ledges, fire escapes, water towers, AC units
- `modelCache` preloads all 24 GLTF models into memory before placing any
- `place()` function clones from cache (`.clone()`), auto-scales, floors, and enables shadow casting
- Preloader is a real UI component with progress bar, loading text, skip button, and timeout

**Custom Techniques:**
- Wet road reflections are faked with a separate plane of `puddle` material (roughness 0.02, metalness 0.6, transparent 0.4) layered 0.07 units above the road -- this is cheap but effective
- Police light alternation uses a `Math.sin(t * 14)` threshold to swap red/blue intensity -- creates the 7Hz strobe effect
- Random window lighting assignment (`Math.random() > 0.55 ? M.winLit : ...`) runs once during building generation, not per frame -- windows stay consistently lit/dark
- Steam vents use scale animation (`Math.sin(t * 3 + s.position.x) * 0.4`) on a cylinder to fake billowing
- Rain particles have per-particle velocity arrays for varied fall speed
- Building generation is semi-procedural: z-position loop with random offsets for width, depth, height, material

**What's Impressive:**
- The preloader is production-grade (model count tracking, skip fallback, timeout)
- Wet surface treatment (low roughness + puddle overlay) reads convincingly as rain-soaked
- The density of detail: fire escapes, water towers, lane markings, crosswalks, parked cars, traffic cones, dumpsters, benches
- Chase cam that travels along the road with slight sinusoidal sway creates a handheld-camera feel
- 24 unique GLTF models placed via cache-and-clone pattern

**What Could Be Improved:**
- No post-processing at all -- this scene begs for bloom (neon signs), chromatic aberration, and film grain for a noir/action feel
- SpotLights per streetlight (one every 25 units, both sides = ~60+ spotlights) is extremely expensive -- most GPUs will bottleneck on the light count. Should use baked light cookies or spot-only-near-camera approach
- Rain particles are `Points` with default `PointsMaterial` -- no streak/line rendering, so rain doesn't look like rain at speed. Should be `LineSegments` or stretched point sprites
- No screen-space reflections -- the wet road is a flat overlay that doesn't actually reflect the neon signs or buildings above it
- Police lights and headlights have no volumetric cone effect -- adding fog-interacting light shafts would sell the atmosphere
- Cars are placed statically -- no animation of oncoming traffic or the chase itself moving
- The `reflectPlane` (puddle overlay) covers the entire 800-unit road length at roughness 0.02 -- with no environment map, `metalness: 0.6` reflects nothing. It's just a dark transparent plane

### 1.4 Alpine Highway (`scenes/desktop/alpine-highway.html`)

**Three.js Features Used:**
- `PlaneGeometry(2000, 2000, 256, 256)` with vertex displacement for terrain
- Vertex colors for height-based biome coloring (snow/rock/meadow/valley)
- `MeshStandardMaterial` with `vertexColors: true`
- `ExtrudeGeometry` with `extrudePath` (CatmullRomCurve3) for the road and center line
- `InstancedMesh` for trees (800 instances, cone + cylinder)
- `FogExp2` for distance fade
- `PCFSoftShadowMap` at 2048x2048

**Custom Techniques:**
- Pseudo-noise terrain using layered `Math.sin/cos` at 4 octaves (frequencies 0.008 to 0.08) -- not true Simplex/Perlin but visually plausible
- Valley carving function subtracts a parabolic channel centered on the road path
- Height-based vertex coloring with distinct snow line, tree line, and valley floor thresholds
- Road follows terrain height with +0.5 offset so it sits on the surface
- Tree placement filters: only between y=15-75, farther than 20 units from road center

**What's Impressive:**
- The terrain generation is genuinely good for a sin-based approach -- 4 octaves with offset phases avoids obvious tiling
- Road following terrain via CatmullRomCurve3 + ExtrudeGeometry is elegant
- Vertex coloring avoids texture dependencies entirely

**What Could Be Improved:**
- Terrain is a single 256x256 plane -- at 2000 units, that's ~7.8 units per vertex. Close-up, the terrain looks blocky. A clipmap or LOD approach would fix this
- No normal map on terrain means the rocky slopes look smooth
- Shadow camera frustum is 1000x1000 units -- at 2048x2048 shadow map, that's 0.49 units/pixel. Shadows are blurry at any reasonable camera distance
- Trees are all identical cones -- no variation in shape, no billboarding, no wind animation
- No guardrails, no vehicles, no tunnel through mountains -- the road feels empty
- Sky is a flat color matching the fog -- no gradient, no clouds

### 1.5 Neon Alley v2 (`scenes/mobile/neon-alley-v2.html`)

**Three.js Features Used:**
- `EffectComposer` + `RenderPass` + `UnrealBloomPass` -- **only scene in the lab using post-processing**
- `MeshPhongMaterial` (Lambert-level performance with specular)
- `logarithmicDepthBuffer: true` (prevents z-fighting in tight alley geometry)
- `pixelRatio: 1` (mobile optimization)
- `antialias: false` (mobile optimization)
- `GLTFLoader` for DamagedHelmet and Fox models
- `AnimationMixer` for GLTF animation playback

**Custom Techniques:**
- Fake neon reflections on ground: thin emissive boxes positioned just above ground plane to simulate reflected neon glow
- Neon flicker with threshold-based blink: `Math.sin(t * 8) > 0.92 ? 3 : 0` creates intermittent buzzing effect
- Bloom strength is slider-controlled in real-time

**What's Impressive:**
- The only scene that uses post-processing -- bloom makes the neon signs glow properly
- Ground reflections are a clever fake that costs almost nothing
- Logarithmic depth buffer prevents z-fighting in narrow alley geometry where camera gets close

**What Could Be Improved:**
- Labeled as mobile but uses bloom (post-processing) which is expensive on mobile GPUs -- contradicts the mobile optimization rules in the progress reference
- Using `MeshPhongMaterial` mixed with bloom creates an inconsistent look -- PhongMaterial's specular model doesn't interact with bloom the way PBR emissive does
- GLTF model loading uses `await new Promise()` with no timeout or fallback -- if either model fails to load, the scene still works but the error handling is just `console.log`
- Rain is only 500 particles (vs NYC's 12,000) -- reads as light drizzle rather than rain
- No flythrough or automated camera movement

### 1.6 GS Fox v4 (`DESIGN-LAB/gs-fox-v4.html`)

**Three.js Features Used:**
- Custom `ShaderMaterial` with vertex + fragment shaders (VS/FS)
- `BufferGeometry` with custom attributes (`aS`, `aC`, `aF`, `aV`, `aAO`, `aN`)
- `Points` rendering for gaussian splat emulation
- Manual depth sorting via index buffer manipulation
- `AdditiveBlending` for bokeh background and dust particles
- `NormalBlending` for the fox body
- Separate shader pairs for fox, bokeh, and dust (3 distinct ShaderMaterials)

**Custom Shaders:**
- **Vertex shader**: Full per-vertex lighting with key light (warm, upper-right), fill light (cool, left-low), ambient, specular (Blinn-Phong half-vector, power 40), rim light (Fresnel-based warm silhouette glow), and ambient occlusion
- **Fragment shader**: Custom point sprite with smooth opaque center and soft edge -- NOT Gaussian falloff. Uses `smoothstep(.5, edge, r)` where `edge` varies between fur (0.30) and core (0.38)
- **Bokeh background shader**: Soft circles with additive ring glow effect
- **Dust shader**: Additive exponential falloff particles with drift animation

**Custom Techniques:**
- Procedural fox body: 70+ body parts defined as ellipsoids with parameters `[name, cx,cy,cz, rx,ry,rz, count, r,g,b, furWeight, furRatio, furMaxLen]`
- Surface shell sampling: core particles use 88-100% depth of ellipsoid (thin shell, not solid volume); fur particles use `surfTight()` which places particles just outside the surface
- Color jitter (`jit()`) adds per-particle variation with occasional dark/light mutations for realistic fur pattern
- PBD (Position-Based Dynamics) spring-damper deformation: `vel += (-springK*dx - damping*vel + inertia_force) * dt`
- Root-dragging for deformable zones: softness scales with distance from pivot point (tail root, ear base, neck)
- Animated rest position system: animations modify `aRst[]` array, then PBD uses `aRst[]` as target -- deformation is always relative to the animated pose
- 8 animation types (jump, shake, bigWag, lookAround, spin, nod, bounce, wiggle) all modify per-particle positions procedurally
- Depth sorting every 5 frames (not every frame) for performance
- 130k+ splats with ~5.5x quality multiplier

**What's Impressive:**
- This is genuinely advanced rendering. The per-vertex lighting model in the shader is more sophisticated than anything in the VP lab scenes
- The PBD deformation system creates physically plausible secondary motion on fur and tail
- The animation system that modifies rest positions and lets PBD add secondary motion on top is a solid architecture pattern
- Root-dragging (softness proportional to distance from pivot) prevents the "whole tail moves as one piece" artifact
- The surface shell sampling technique (88-100% depth) prevents interior particles from showing through alpha
- 130k particles at 60fps with depth sorting is well-optimized

**What Could Be Improved:**
- Depth sorting runs on CPU (`sortIdx.sort()`) -- at 130k particles, this is O(n log n) every 5 frames. A GPU-based bitonic sort or a radix sort would be significantly faster
- No shadow casting or receiving -- the fox has a baked ground shadow circle but doesn't respond to directional light
- The splat shading is baked into the vertex shader (fixed light directions) -- can't change lighting at runtime
- Color is per-particle with no texture lookup -- adding a UV-mapped color/detail texture would add realism
- No SSR, bloom, or tone mapping -- the fox exists in a raw linear color space
- The scene background is CSS-only (radial gradient) with a vignette overlay div -- no 3D skybox or environment map

---

## 2. Cross-Cutting Patterns and Conventions

### Architecture Pattern (Consistent Across All Scenes)
1. Single HTML file, no build step, no bundler
2. Three.js via CDN importmap (v0.172.0 for VP lab, v0.167.0 for DESIGN-LAB)
3. Material dictionary `M = {}` or `const M = {}` at top of script
4. Builder functions that return `THREE.Group` (e.g., `temple()`, `buildHouse()`, `addBuilding()`)
5. `OrbitControls` always present, optionally disabled during flythrough
6. HTML range sliders for runtime parameter control
7. `requestAnimationFrame` loop with `THREE.Clock` for delta time
8. `window.addEventListener('resize', ...)` for responsive viewport
9. Cache-busting meta tag during development

### Material Strategy
| Scene | Material Class | Reason |
|-------|---------------|--------|
| Greek Epic | `MeshLambertMaterial` | Performance (many objects, mobile target) |
| Medieval Town | `MeshStandardMaterial` | Visual quality (PBR roughness/metalness) |
| NYC Chase | `MeshStandardMaterial` | Wet surfaces need PBR |
| Alpine Highway | `MeshStandardMaterial` | Vertex colors need Standard |
| Neon Alley | `MeshPhongMaterial` | Mobile target, needs specular |
| GS Fox | `ShaderMaterial` | Custom splat rendering |

### Lighting Convention
- 3-4 light sources per scene: 1 directional (key), 1 ambient, 1 hemisphere, optional fill/point
- Shadow maps are 1024-2048, never 4096+
- `ACESFilmicToneMapping` is universal
- Time-of-day systems adjust light intensity and exposure in the animation loop

### Performance Conventions
- `pixelRatio` capped at 1.5 (desktop) or 1 (mobile)
- `InstancedMesh` used for vegetation (olive trees, cypress, pine trees)
- `powerPreference: 'high-performance'` on desktop
- No shadow on point lights (torches, neon glow)
- Mobile scenes: no shadows, no sky shader, Lambert only, antialias off

---

## 3. Gaps -- Techniques Not Yet Used

### Post-Processing (Critical Gap)
Only `neon-alley-v2` uses any post-processing (bloom). None of the desktop scenes use:
- **Bloom** -- NYC Chase and Medieval Town both need it desperately (neon signs, torch fire, emissive windows)
- **SSAO** (Screen-Space Ambient Occlusion) -- would add massive depth to Greek Epic's tightly packed buildings and Medieval Town's half-timber structures
- **Color grading / LUT** -- every scene uses raw ACES mapping; a scene-specific color grade would unify the look
- **Depth of Field** -- the flythrough cameras would benefit hugely from DoF to guide the viewer's eye
- **Film grain** -- VP scenes should look filmic
- **Chromatic aberration** -- subtle use on NYC Chase would sell the action-cam feel

### Environment and Reflections (Critical Gap)
No scene uses:
- **HDR environment maps** (HDRI) -- the `pathtracer.html` demo loads one but no VP scene does
- **PMREMGenerator** for roughness-dependent reflections -- Standard materials have roughness but reflect nothing
- **Screen-space reflections (SSR)** -- wet surfaces in NYC and Medieval Town's river are reflectionless
- **Reflection probes / cube camera** -- even a baked cubemap would help

The progress reference notes "NO Sky shader (PMREMGenerator kills mobile GPUs)" for mobile, but desktop scenes should be using it.

### Shadow Quality (Significant Gap)
No scene uses:
- **Cascaded Shadow Maps (CSM)** -- critical for large outdoor scenes where a single shadow map is spread too thin
- **Shadow bias tuning** -- only Medieval Town and NYC set `shadow.bias`
- **VSM or PCSS** for softer shadow falloff
- Contact hardening shadows would dramatically improve close-up shadow quality

### Geometry (Moderate Gap)
- **No normal/bump maps** anywhere -- every surface is geometrically flat
- **No displacement maps** -- terrain is vertex-displaced but nothing uses texture-based displacement
- **No decals** -- cracks, stains, posters would add life
- **No LOD (Level of Detail)** -- buildings at 800 units away render with the same geometry as buildings at 10 units
- **No terrain LOD** (clipmap, quadtree) -- Alpine Highway's 256x256 grid is fixed resolution

### Animation (Moderate Gap)
- No skeletal animation in VP scenes (only GLTF Fox in Neon Alley has it)
- No morph targets
- No particle systems beyond rain (no fire, smoke, dust, sparks)
- No cloth/flag simulation
- No vehicle animation in NYC Chase

### Audio (Complete Gap)
- No scene has any audio -- ambience, music, or spatial sound
- `THREE.AudioListener` + `THREE.PositionalAudio` could add water sounds (Greek Epic), market crowd (Medieval), traffic/sirens (NYC)

---

## 4. Specific Upgrade Opportunities

### Greek Epic: Acropolis at Golden Hour

| Upgrade | Effort | Impact |
|---------|--------|--------|
| Upgrade to `MeshStandardMaterial` for marble (roughness 0.3) and stone (roughness 0.85) | Low | High -- marble will read as marble |
| Add HDR environment map (Mediterranean sky) | Low | High -- reflective water, marble highlights |
| Add CSM shadows (3 cascades) | Medium | High -- fixes the blurry-shadow problem across the whole city |
| Add `UnrealBloomPass` for sunset glow and water sparkle | Low | Medium |
| Replace flat sky color with sky gradient canvas (steal from Medieval Town) | Low | Medium |
| Add simple flag cloth on boats (sine-displaced PlaneGeometry) | Low | Medium |
| Animate boats with gentle rocking (sin wave on rotation.x/z) | Trivial | Medium |
| Add vertex displacement to Water plane for foam near cliffs | Medium | Medium |

### Medieval Town: VP Volume Background

| Upgrade | Effort | Impact |
|---------|--------|--------|
| Add `UnrealBloomPass` (strength ~0.4) -- torches, fire, emissive windows will glow | Low | **Critical** -- the torch system is built but bloom is missing to sell it |
| Add SSAO pass -- half-timber building corners and cobblestone square will gain depth | Medium | High |
| Fix torch flicker to use per-torch phase offsets: `0.85 + Math.sin(time * 8 + torchIndex * 1.7) * 0.15` | Trivial | Medium |
| Add environment cube map (even a low-res baked one) for glass windows and river | Low | High |
| Replace circular flythrough with CatmullRomCurve3 path through town (steal pattern from Greek Epic) | Low | High |
| Add DoF on flythrough (focus on town center, blur edges) | Medium | High |
| Add volumetric fog (screen-space raymarched or particle-based) around river and torches | High | **Jaw-dropping** |
| Add ground contact shadows (small dark circles under objects) or PCSS | Medium | Medium |
| Generate windows on all 4 faces of `buildHouse()`, not just Z+ | Low | Medium |

### NYC Chase: Rainy Night Pursuit

| Upgrade | Effort | Impact |
|---------|--------|--------|
| Add `UnrealBloomPass` + `FilmPass` (grain + scanlines) | Low | **Critical** -- this scene is designed for noir but has no post-processing |
| Add `SSRPass` or planar reflections on wet road | Medium | **Critical** -- wet road must reflect neon to sell the rain |
| Replace rain `Points` with `LineSegments` (stretched in velocity direction) | Medium | High |
| Add volumetric light cones on headlights and police lights (raymarched or sprite-based) | Medium | High -- sells the rain-through-headlight beam look |
| Cap active spotlights to nearest N from camera (frustum culling) | Medium | High -- fixes the 60+ spotlight performance cliff |
| Animate a few cars moving (simple z-translation in animation loop) | Low | Medium |
| Add screen-shake effect tied to police lights or speed | Trivial | Medium |
| Add lens flare (`Lensflare` addon) on headlights and streetlights | Low | Medium |
| Add chromatic aberration pass (subtle, 0.5-1px offset) | Low | Medium |

### Alpine Highway: Mountain Drive

| Upgrade | Effort | Impact |
|---------|--------|--------|
| Replace sin-based terrain with proper Simplex noise (use a noise library or compute shader) | Medium | High |
| Add normal map generation from heightfield for rocky detail | Medium | High |
| Add CSM shadows (3 cascades) | Medium | High |
| Add cloud layer (translucent planes or volumetric particles at mountain-top height) | Medium | High |
| Add guardrails along road (instanced box geometry following road curve) | Low | Medium |
| Add a vehicle model following the road curve | Low | Medium |
| Add atmospheric scattering sky shader (Preetham or Hosek-Wilkie via Three.js `Sky` addon) | Low | High -- the flat sky is the weakest element |
| Add wind animation on trees (vertex shader displacement) | Medium | Medium |
| Add LOD for terrain (higher resolution near camera) | High | Medium |

### GS Fox v4: Splat Rendering

| Upgrade | Effort | Impact |
|---------|--------|--------|
| Move depth sort to GPU (bitonic sort via compute shader or transform feedback) | High | Medium -- current CPU sort at Q=5.5 is the bottleneck above 200k splats |
| Add bloom pass -- rim light and specular highlights would glow | Low | High |
| Make lighting dynamic (pass light direction as uniform, not hardcoded) | Low | Medium |
| Add shadow receiving (project a directional shadow onto a ground plane in the shader) | Medium | Medium |
| Add environment probe reflection on eyes and nose (high-gloss surfaces) | Medium | Medium |
| Add tone mapping (`ACESFilmicToneMapping`) -- currently rendering in linear space | Trivial | Medium |

---

## 5. "Next Level" Roadmap

### Tier 1: Quick Wins (1-2 hours each, massive visual improvement)

**1. Universal Post-Processing Stack**
Create a shared post-processing setup that every desktop scene imports:
```javascript
// post.js -- shared across scenes
import { EffectComposer } from 'three/addons/postprocessing/EffectComposer.js';
import { RenderPass } from 'three/addons/postprocessing/RenderPass.js';
import { UnrealBloomPass } from 'three/addons/postprocessing/UnrealBloomPass.js';
import { SMAAPass } from 'three/addons/postprocessing/SMAAPass.js';

export function createComposer(renderer, scene, camera, opts = {}) {
  const composer = new EffectComposer(renderer);
  composer.addPass(new RenderPass(scene, camera));
  if (opts.bloom !== false) {
    composer.addPass(new UnrealBloomPass(
      new THREE.Vector2(window.innerWidth, window.innerHeight),
      opts.bloomStrength || 0.4,
      opts.bloomRadius || 0.3,
      opts.bloomThreshold || 0.8
    ));
  }
  composer.addPass(new SMAAPass(window.innerWidth, window.innerHeight));
  return composer;
}
```
Since these are single-file scenes (no bundler), this would be inlined per scene. But the pattern should be standardized.

**2. Environment Maps for Desktop Scenes**
Three.js ships with several free HDRIs. Loading one transforms every `MeshStandardMaterial` surface:
```javascript
import { RGBELoader } from 'three/addons/loaders/RGBELoader.js';
new RGBELoader().load('path/to/env.hdr', (hdr) => {
  hdr.mapping = THREE.EquirectangularReflectionMapping;
  scene.environment = hdr; // all PBR materials now reflect this
});
```
For Greek Epic: a warm Mediterranean sky HDRI.
For Medieval Town: a moody overcast HDRI.
For NYC Chase: a dark city night HDRI (even a 256x128 one adds life to wet surfaces).

**3. Greek Epic Material Upgrade**
Swap `MeshLambertMaterial` to `MeshStandardMaterial` with proper roughness values:
```javascript
marble: new THREE.MeshStandardMaterial({ color: 0xeee8d5, roughness: 0.25, metalness: 0.0 }),
stone:  new THREE.MeshStandardMaterial({ color: 0xb8a88a, roughness: 0.85, metalness: 0.0 }),
wood:   new THREE.MeshStandardMaterial({ color: 0x5c3a1e, roughness: 0.7, metalness: 0.0 }),
bronze: new THREE.MeshStandardMaterial({ color: 0x8B7355, roughness: 0.4, metalness: 0.8 }),
water:  // keep the Water addon, it already handles PBR reflection
```
This single change + an environment map = the scene looks 10x better.

### Tier 2: Medium Effort (3-6 hours each)

**4. Cascaded Shadow Maps**
Three.js has a CSM implementation in the examples:
```javascript
import { CSM } from 'three/addons/csm/CSM.js';
const csm = new CSM({
  maxFar: camera.far,
  cascades: 3,
  shadowMapSize: 2048,
  lightDirection: new THREE.Vector3(-0.5, -1, -0.3).normalize(),
  camera: camera,
  parent: scene,
});
```
Priority scenes: Greek Epic (huge scene, worst shadow quality) and Alpine Highway.

**5. Screen-Space Reflections for NYC Chase**
The `SSRPass` in Three.js examples would transform the wet road:
```javascript
import { SSRPass } from 'three/addons/postprocessing/SSRPass.js';
const ssr = new SSRPass({ renderer, scene, camera, width: innerWidth, height: innerHeight });
ssr.maxDistance = 10;
ssr.opacity = 0.5;
composer.addPass(ssr);
```
The wet asphalt (roughness 0.25, metalness 0.05) will start reflecting neon signs. The effect is transformative for a rain scene.

**6. Volumetric Fog / Light Shafts**
Two approaches:
- **Particle-based**: Place transparent billboard sprites in a cone from each light source. Cheap, good for torches.
- **Screen-space raymarched**: More expensive but physically accurate. Three.js doesn't ship this natively; requires a custom shader pass.

For Medieval Town's torches, the particle approach is the right call:
```javascript
const smokeMat = new THREE.SpriteMaterial({
  color: 0xff8833,
  transparent: true,
  opacity: 0.08,
  blending: THREE.AdditiveBlending,
});
// Place 10-20 sprites in a cone above each torch
```

**7. Rain as LineSegments (NYC Chase)**
Replace `Points` with velocity-stretched lines:
```javascript
const lineGeo = new THREE.BufferGeometry();
// For each raindrop, create two vertices: current position and position + velocity * stretch
// Update both positions each frame
const rain = new THREE.LineSegments(lineGeo, new THREE.LineBasicMaterial({
  color: 0x8888bb, transparent: true, opacity: 0.3,
}));
```

### Tier 3: High Effort, High Reward (1-2 days each)

**8. Real-Time Sky System (Preetham/Hosek)**
Three.js ships a `Sky` addon that generates physically-based sky colors from sun position:
```javascript
import { Sky } from 'three/addons/objects/Sky.js';
const sky = new Sky();
sky.scale.setScalar(10000);
const skyUniforms = sky.material.uniforms;
skyUniforms['turbidity'].value = 10;
skyUniforms['rayleigh'].value = 2;
skyUniforms['mieCoefficient'].value = 0.005;
skyUniforms['mieDirectionalG'].value = 0.8;
skyUniforms['sunPosition'].value.copy(sunLight.position);
scene.add(sky);
```
This replaces flat sky colors AND generates a proper environment map via `PMREMGenerator` that all PBR materials can reflect. Works for Greek Epic, Alpine Highway.

Note: The progress reference warns PMREMGenerator kills mobile GPUs. Desktop-only.

**9. Proper Terrain with Noise + Triplanar Texturing (Alpine Highway)**
Replace sin-based heightfield with a proper noise function (FBM over Simplex), then apply triplanar texturing for seamless rocky detail without UV coordinates:
```glsl
// Fragment shader for triplanar
vec3 blending = abs(vNormal);
blending = normalize(max(blending, 0.00001));
blending /= (blending.x + blending.y + blending.z);
vec4 xaxis = texture2D(rockTex, vWorldPos.yz * scale);
vec4 yaxis = texture2D(grassTex, vWorldPos.xz * scale);
vec4 zaxis = texture2D(rockTex, vWorldPos.xy * scale);
vec4 color = xaxis * blending.x + yaxis * blending.y + zaxis * blending.z;
```

**10. GPU Particle System for Atmospheric Effects**
Build a reusable GPU particle system using transform feedback or a compute-shader approach:
- Fire (torches, campfires)
- Smoke (chimneys, steam vents)
- Dust motes (indoor scenes)
- Sparks (blacksmith in Medieval Town)
- Leaves falling (forest scenes)
- Snow

This would be a shared utility across all scenes.

**11. Deferred or Clustered Lighting for NYC Chase**
60+ spotlights is unsustainable with forward rendering. Options:
- **Clustered forward**: Partition the frustum into tiles, each tile only evaluates nearby lights. Three.js doesn't ship this, but the `WebGPURenderer` in development supports it natively.
- **Short term**: Distance-cull lights based on camera position. Only enable the nearest 8-10 spotlights.

### Tier 4: Experimental / Frontier

**12. WebGPU Migration**
Three.js `WebGPURenderer` is maturing. Benefits:
- Compute shaders for particle systems, terrain generation, depth sorting
- Clustered lighting built in
- Bindless textures for better material batching

**13. Gaussian Splatting with Real Captures**
The GS Fox v4 demonstrates procedural splat rendering. The next step: load real 3D Gaussian Splat captures (`.ply` or `.splat` files from Luma AI, Polycam, or nerfstudio) into the VP scenes. Imagine a photogrammetry-captured Greek column placed in the Greek Epic scene.

**14. Path Tracing for Hero Shots**
`pathtracer.html` already demonstrates the `three-gpu-pathtracer` library. Use it to render hero still frames of each VP scene at 1000+ samples for portfolio/presentation use -- then fall back to rasterization for real-time.

---

## 6. Priority Ranking (What to Do First)

| Priority | Upgrade | Scenes Affected | ROI |
|----------|---------|-----------------|-----|
| 1 | Add bloom post-processing | Medieval Town, NYC Chase | Highest -- unlocks all existing emissive work |
| 2 | Add HDRI environment maps | All desktop | Highest -- single change transforms every PBR surface |
| 3 | Upgrade Greek Epic to StandardMaterial | Greek Epic | High -- marble looks like marble |
| 4 | Add SSR or planar reflections to NYC | NYC Chase | High -- wet road becomes convincing |
| 5 | Add CSM shadows | Greek Epic, Alpine Highway | High -- fixes blurry shadows |
| 6 | Fix Medieval Town flythrough path | Medieval Town | Medium -- low effort, much better showcase |
| 7 | Add Sky addon to outdoor scenes | Greek Epic, Alpine Highway | Medium -- proper sky gradient and env map in one |
| 8 | Rain as LineSegments | NYC Chase | Medium -- rain reads as rain |
| 9 | Add SSAO | Medieval Town, NYC Chase | Medium -- adds depth to geometry-heavy scenes |
| 10 | Volumetric torch lights | Medieval Town | Medium -- the atmosphere multiplier |

---

## 7. Summary of Current Skill Level

The VP lab demonstrates solid intermediate-to-advanced Three.js skills:

**Strong areas:**
- Procedural geometry construction with builder functions
- Instanced mesh usage for vegetation
- Material variation and selection
- Camera path animation (CatmullRomCurve3)
- GLTF asset pipeline with auto-scaling
- Dynamic sky/time-of-day systems
- Custom GLSL shaders (GS Fox demonstrates vertex lighting, alpha compositing, PBD physics)
- Performance awareness (pixel ratio capping, mobile degradation path, shadow map sizing)

**Growth areas:**
- Post-processing pipeline (bloom, SSAO, DoF, SSR, film grain)
- Environment maps and PBR reflections
- Shadow quality (CSM, VSM, contact-hardening)
- Texture-based detail (normal maps, roughness maps, decals)
- Particle/VFX systems beyond basic rain
- Audio integration
- GPU-side computation (compute shaders, transform feedback)

The gap between the current state and "jaw-dropping" is primarily in the post-processing and environment/reflection pipeline. The scene construction, layout, and procedural geometry work is already strong. Adding bloom + HDRI to the existing scenes would likely double their visual impact with less than an hour of work per scene.
