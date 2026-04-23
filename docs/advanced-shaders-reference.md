# Advanced Shader Techniques in Three.js

> Weapon-grade reference for custom shaders, post-processing, raymarching, volumetrics, particles, water, procedural textures, PBR extensions, and instancing in Three.js.

Last updated: 2026-04-22

---

## Table of Contents

1. [Custom ShaderMaterial / RawShaderMaterial](#1-custom-shadermaterial--rawshadermaterial)
2. [Post-Processing Effects](#2-post-processing-effects)
3. [Screen-Space Effects](#3-screen-space-effects)
4. [Raymarching in Three.js](#4-raymarching-in-threejs)
5. [Volumetric Effects](#5-volumetric-effects)
6. [Particle Systems](#6-particle-systems)
7. [Water / Ocean](#7-water--ocean)
8. [Procedural Textures](#8-procedural-textures)
9. [PBR Extensions](#9-pbr-extensions)
10. [Instancing + Merged Geometries](#10-instancing--merged-geometries)

---

## 1. Custom ShaderMaterial / RawShaderMaterial

### Overview

`ShaderMaterial` lets you write custom GLSL vertex and fragment shaders while Three.js automatically injects built-in uniforms (`projectionMatrix`, `modelViewMatrix`, `normalMatrix`, `cameraPosition`) and attributes (`position`, `normal`, `uv`). `RawShaderMaterial` strips all of that out -- you declare everything yourself.

Use `ShaderMaterial` when you want Three.js lighting/fog infrastructure available. Use `RawShaderMaterial` when you want total control or are targeting GLSL3 (`#version 300 es`).

### Constructor Options

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `uniforms` | Object | `{}` | Key-value uniform objects `{ name: { value: ... } }` |
| `vertexShader` | string | — | GLSL vertex shader source |
| `fragmentShader` | string | — | GLSL fragment shader source |
| `defines` | Object | `{}` | Custom `#define` directives injected before shader source |
| `fog` | boolean | `false` | Inject fog uniforms (requires `UniformsLib['fog']`) |
| `lights` | boolean | `false` | Inject lighting uniforms |
| `clipping` | boolean | `false` | Enable clipping plane uniforms |
| `glslVersion` | `null\|GLSL3` | `null` | Set to `THREE.GLSL3` for `#version 300 es` |
| `transparent` | boolean | `false` | Enable alpha blending |
| `depthWrite` | boolean | `true` | Write to depth buffer |
| `side` | int | `FrontSide` | `FrontSide`, `BackSide`, `DoubleSide` |

### Basic Setup

```javascript
import * as THREE from 'three';

const material = new THREE.ShaderMaterial({
  uniforms: {
    uTime:       { value: 0.0 },
    uResolution: { value: new THREE.Vector2(window.innerWidth, window.innerHeight) },
    uColor:      { value: new THREE.Color(0xff4400) },
    uTexture:    { value: someTexture },
  },
  vertexShader: vertexSource,
  fragmentShader: fragmentSource,
});

// Update in animation loop
function animate(time) {
  material.uniforms.uTime.value = time * 0.001;
  renderer.render(scene, camera);
  requestAnimationFrame(animate);
}
```

### Built-in Uniforms (ShaderMaterial only)

These are automatically available in your GLSL -- do NOT declare them:

```glsl
// Vertex shader built-ins
uniform mat4 modelMatrix;         // object.matrixWorld
uniform mat4 modelViewMatrix;     // camera.matrixWorldInverse * object.matrixWorld
uniform mat4 projectionMatrix;    // camera.projectionMatrix
uniform mat4 viewMatrix;          // camera.matrixWorldInverse
uniform mat3 normalMatrix;        // inverse transpose of modelViewMatrix
uniform vec3 cameraPosition;      // camera world position

// Built-in attributes
attribute vec3 position;
attribute vec3 normal;
attribute vec2 uv;
```

### Basic Vertex Shader

```glsl
varying vec2 vUv;
varying vec3 vNormal;
varying vec3 vWorldPosition;

void main() {
  vUv = uv;
  vNormal = normalize(normalMatrix * normal);

  vec4 worldPos = modelMatrix * vec4(position, 1.0);
  vWorldPosition = worldPos.xyz;

  gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
}
```

### Basic Fragment Shader

```glsl
uniform float uTime;
uniform vec3 uColor;

varying vec2 vUv;
varying vec3 vNormal;

void main() {
  // Simple directional light
  vec3 lightDir = normalize(vec3(1.0, 1.0, 1.0));
  float diffuse = max(dot(vNormal, lightDir), 0.0);

  vec3 color = uColor * (0.3 + 0.7 * diffuse);
  gl_FragColor = vec4(color, 1.0);
}
```

### Custom Attributes

```javascript
const geometry = new THREE.BufferGeometry();

// Standard attributes
const positions = new Float32Array([...]);
const normals = new Float32Array([...]);
geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
geometry.setAttribute('normal', new THREE.BufferAttribute(normals, 3));

// Custom per-vertex attribute
const displacement = new Float32Array(vertexCount);
for (let i = 0; i < vertexCount; i++) {
  displacement[i] = Math.random();
}
geometry.setAttribute('aDisplacement', new THREE.BufferAttribute(displacement, 1));
```

In the vertex shader:

```glsl
attribute float aDisplacement;
uniform float uTime;

void main() {
  vec3 pos = position;
  pos += normal * aDisplacement * sin(uTime + position.x * 5.0) * 0.3;
  gl_Position = projectionMatrix * modelViewMatrix * vec4(pos, 1.0);
}
```

To update attributes at runtime:

```javascript
geometry.attributes.aDisplacement.array[i] = newValue;
geometry.attributes.aDisplacement.needsUpdate = true;
```

### RawShaderMaterial (GLSL3)

```javascript
const material = new THREE.RawShaderMaterial({
  glslVersion: THREE.GLSL3,
  uniforms: {
    uTime: { value: 0 },
  },
  vertexShader: `#version 300 es
    precision highp float;

    in vec3 position;
    in vec2 uv;
    uniform mat4 projectionMatrix;
    uniform mat4 modelViewMatrix;
    out vec2 vUv;

    void main() {
      vUv = uv;
      gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
    }
  `,
  fragmentShader: `#version 300 es
    precision highp float;

    in vec2 vUv;
    out vec4 fragColor;
    uniform float uTime;

    void main() {
      fragColor = vec4(vUv, sin(uTime) * 0.5 + 0.5, 1.0);
    }
  `,
});
```

### Extending Built-in Materials with onBeforeCompile

Inject custom code into `MeshStandardMaterial` or `MeshPhysicalMaterial` without rewriting everything:

```javascript
const material = new THREE.MeshStandardMaterial({
  color: 0x888888,
  roughness: 0.4,
  metalness: 0.1,
});

// Custom uniforms -- store reference for animation loop
const customUniforms = {
  uTime: { value: 0 },
  uNoiseScale: { value: 3.0 },
};

material.onBeforeCompile = (shader) => {
  // Inject custom uniforms
  shader.uniforms.uTime = customUniforms.uTime;
  shader.uniforms.uNoiseScale = customUniforms.uNoiseScale;

  // Inject into vertex shader: add displacement
  shader.vertexShader = shader.vertexShader.replace(
    '#include <begin_vertex>',
    `
    #include <begin_vertex>
    float noise = sin(position.x * uNoiseScale + uTime) * 0.1;
    transformed += normal * noise;
    `
  );

  // Inject uniforms declaration at top
  shader.vertexShader = 'uniform float uTime;\nuniform float uNoiseScale;\n' + shader.vertexShader;
};
```

### THREE-CustomShaderMaterial (CSM) Library

A cleaner API for extending built-in materials:

```bash
npm install three-custom-shader-material
```

```javascript
import CustomShaderMaterial from 'three-custom-shader-material/vanilla';

const material = new CustomShaderMaterial({
  baseMaterial: THREE.MeshPhysicalMaterial,
  vertexShader: `
    uniform float uTime;
    void main() {
      float wave = sin(position.x * 5.0 + uTime) * 0.2;
      csm_Position = position + normal * wave;
    }
  `,
  fragmentShader: `
    uniform float uTime;
    void main() {
      float pulse = sin(uTime * 2.0) * 0.5 + 0.5;
      csm_DiffuseColor = vec4(pulse, 0.2, 0.8, 1.0);
    }
  `,
  uniforms: { uTime: { value: 0 } },
  // All MeshPhysicalMaterial props work
  clearcoat: 1.0,
  roughness: 0.3,
  metalness: 0.6,
});
```

**CSM output variables (vertex shader):**

| Variable | Type | Purpose |
|----------|------|---------|
| `csm_Position` | `vec3` | Vertex position (auto-projected) |
| `csm_PositionRaw` | `vec4` | Direct `gl_Position` override |
| `csm_Normal` | `vec3` | Custom normals |

**CSM output variables (fragment shader):**

| Variable | Type | Purpose |
|----------|------|---------|
| `csm_DiffuseColor` | `vec4` | Diffuse color (preserves lighting) |
| `csm_FragColor` | `vec4` | Final color (overrides all shading) |
| `csm_Roughness` | `float` | Per-pixel roughness |
| `csm_Metalness` | `float` | Per-pixel metalness |
| `csm_Emissive` | `vec3` | Emissive color |
| `csm_AO` | `float` | Ambient occlusion |
| `csm_Clearcoat` | `float` | Clearcoat intensity |
| `csm_Transmission` | `float` | Glass-like transparency |
| `csm_Iridescence` | `float` | Iridescence intensity |
| `csm_FragNormal` | `float` | Fragment normal override |
| `csm_UnlitFac` | `float` | Blend between lit/unlit (0=lit, 1=unlit) |

**patchMap for chunk replacement:**

```javascript
const material = new CustomShaderMaterial({
  baseMaterial: THREE.MeshPhysicalMaterial,
  patchMap: {
    "*": {
      "vec4 diffuseColor = vec4( diffuse, opacity );":
      "vec4 diffuseColor = vec4( myCustomColor, opacity );"
    }
  }
});
```

### Performance Tips

- Minimize uniform count. Pack related data into `vec4` where possible.
- Use `defines` for compile-time constants instead of uniforms for values that never change.
- Use `#pragma unroll_loop_start` / `#pragma unroll_loop_end` for known-length loops.
- Avoid `discard` in fragment shaders when possible -- it breaks early-Z optimisation.
- Use `lowp`/`mediump` precision where acceptable (color channels, UVs).
- If extending built-in materials, `onBeforeCompile` has zero overhead vs writing from scratch. CSM adds minimal overhead through shader string manipulation at compile time.

### Key References

- [Three.js ShaderMaterial Docs](https://threejs.org/docs/pages/ShaderMaterial.html)
- [THREE-CustomShaderMaterial (GitHub)](https://github.com/FarazzShaikh/THREE-CustomShaderMaterial)
- [Three.js Custom Materials (cjgammon)](https://blog.cjgammon.com/threejs-custom-shader-material/)
- [GLSL Shaders for Beginners (Wael Yasmina)](https://waelyasmina.net/articles/glsl-and-shaders-tutorial-for-beginners-webgl-threejs/)

---

## 2. Post-Processing Effects

### Overview

Post-processing applies screen-space effects after the main render. Three.js has two ecosystems:
1. **Built-in**: `EffectComposer` from `three/addons/postprocessing/` -- uses ShaderPass with render-to-texture chains
2. **pmndrs/postprocessing**: Higher-performance library that merges compatible effects into a single pass

### Built-in EffectComposer Setup

```javascript
import { EffectComposer } from 'three/addons/postprocessing/EffectComposer.js';
import { RenderPass } from 'three/addons/postprocessing/RenderPass.js';
import { ShaderPass } from 'three/addons/postprocessing/ShaderPass.js';
import { UnrealBloomPass } from 'three/addons/postprocessing/UnrealBloomPass.js';
import { OutputPass } from 'three/addons/postprocessing/OutputPass.js';
import { FXAAShader } from 'three/addons/shaders/FXAAShader.js';

// Create composer
const composer = new EffectComposer(renderer);

// Pass 1: Render the scene
const renderPass = new RenderPass(scene, camera);
composer.addPass(renderPass);

// Pass 2: Bloom
const bloomPass = new UnrealBloomPass(
  new THREE.Vector2(window.innerWidth, window.innerHeight),
  1.5,   // strength
  0.4,   // radius
  0.85   // threshold
);
composer.addPass(bloomPass);

// Pass 3: FXAA antialiasing
const fxaaPass = new ShaderPass(FXAAShader);
const pixelRatio = renderer.getPixelRatio();
fxaaPass.material.uniforms['resolution'].value.x = 1 / (window.innerWidth * pixelRatio);
fxaaPass.material.uniforms['resolution'].value.y = 1 / (window.innerHeight * pixelRatio);
composer.addPass(fxaaPass);

// Pass 4: Output (handles sRGB + tone mapping -- MUST be last)
const outputPass = new OutputPass();
composer.addPass(outputPass);

// Render loop -- use composer instead of renderer
function animate() {
  requestAnimationFrame(animate);
  composer.render();
}
```

### Writing a Custom ShaderPass

```javascript
const myCustomShader = {
  uniforms: {
    tDiffuse: { value: null },  // Previous pass output (auto-bound)
    uTime: { value: 0 },
    uIntensity: { value: 0.5 },
  },
  vertexShader: `
    varying vec2 vUv;
    void main() {
      vUv = uv;
      gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
    }
  `,
  fragmentShader: `
    uniform sampler2D tDiffuse;
    uniform float uTime;
    uniform float uIntensity;
    varying vec2 vUv;

    void main() {
      vec4 color = texture2D(tDiffuse, vUv);

      // Chromatic aberration
      float offset = uIntensity * 0.01;
      float r = texture2D(tDiffuse, vUv + vec2(offset, 0.0)).r;
      float g = color.g;
      float b = texture2D(tDiffuse, vUv - vec2(offset, 0.0)).b;

      gl_FragColor = vec4(r, g, b, color.a);
    }
  `,
};

const customPass = new ShaderPass(myCustomShader);
composer.addPass(customPass);

// Update in animation loop
customPass.uniforms.uTime.value = elapsedTime;
```

### Writing a Custom Pass Class

For more control (multi-render-target, custom geometry, etc.):

```javascript
import { Pass, FullScreenQuad } from 'three/addons/postprocessing/Pass.js';

class CustomRenderPass extends Pass {
  constructor(scene, camera) {
    super();
    this.scene = scene;
    this.camera = camera;
    this.material = new THREE.ShaderMaterial({
      uniforms: {
        tDiffuse: { value: null },
        uStrength: { value: 1.0 },
      },
      vertexShader: `...`,
      fragmentShader: `...`,
    });
    this.fsQuad = new FullScreenQuad(this.material);
  }

  render(renderer, writeBuffer, readBuffer) {
    this.material.uniforms.tDiffuse.value = readBuffer.texture;

    if (this.renderToScreen) {
      renderer.setRenderTarget(null);
    } else {
      renderer.setRenderTarget(writeBuffer);
    }
    this.fsQuad.render(renderer);
  }

  dispose() {
    this.material.dispose();
    this.fsQuad.dispose();
  }
}
```

### pmndrs/postprocessing Library

Higher performance -- merges multiple effects into fewer passes:

```bash
npm install postprocessing
```

```javascript
import { EffectComposer, RenderPass, EffectPass, BloomEffect, VignetteEffect } from 'postprocessing';

const composer = new EffectComposer(renderer);
composer.addPass(new RenderPass(scene, camera));

// Multiple effects merged into ONE pass
composer.addPass(new EffectPass(camera,
  new BloomEffect({ intensity: 1.5, luminanceThreshold: 0.8 }),
  new VignetteEffect({ darkness: 0.5, offset: 0.3 })
));
```

### WebGPU Post-Processing (TSL)

For the new WebGPU renderer, post-processing uses node-based composition:

```javascript
import * as THREE from 'three/webgpu';
import { bloom, pass } from 'three/tsl';

const postProcessing = new THREE.PostProcessing(renderer);
const scenePass = pass(scene, camera);
const bloomPass = bloom(scenePass, {
  threshold: 0.8,
  intensity: 1.5,
});
postProcessing.outputNode = bloomPass;

// Render loop
function animate() {
  postProcessing.render();
}
```

### Performance Considerations

- Each pass in the built-in `EffectComposer` renders a full-screen quad with a texture read/write. 5+ passes can hurt on mobile.
- The `pmndrs/postprocessing` library merges effects into fewer shader programs, reducing texture reads. Prefer it for production.
- Use `HalfFloatType` for the EffectComposer framebuffer: `new EffectComposer(renderer, { frameBufferType: THREE.HalfFloatType })`.
- Render expensive effects at half resolution and upscale.
- The `OutputPass` must be the final pass when using the built-in chain (it applies tone mapping and sRGB encoding).

### Key References

- [Three.js EffectComposer Docs](https://threejs.org/docs/pages/EffectComposer.html)
- [pmndrs/postprocessing (GitHub)](https://github.com/pmndrs/postprocessing)
- [Post-Processing in Three.js (Sangil Lee, 2025)](https://sangillee.com/2025-01-15-post-processing/)
- [Sketchy Pencil Effect (Codrops)](https://tympanus.net/codrops/2022/11/29/sketchy-pencil-effect-with-three-js-post-processing/)

---

## 3. Screen-Space Effects

### Screen-Space Reflections (SSR)

SSR uses ray marching in screen space to find reflections from the depth/normal buffer. Only reflects what is visible on screen -- objects behind the camera or occluded produce artifacts.

**Setup with `screen-space-reflections` (pmndrs/postprocessing compatible):**

```bash
npm install postprocessing screen-space-reflections
```

```javascript
import { EffectComposer, RenderPass, EffectPass } from 'postprocessing';
import { SSREffect } from 'screen-space-reflections';

const composer = new EffectComposer(renderer, {
  frameBufferType: THREE.HalfFloatType,
});
composer.addPass(new RenderPass(scene, camera));

const ssrEffect = new SSREffect(scene, camera, {
  intensity: 1,
  distance: 10,           // Max ray travel distance
  thickness: 10,          // Depth tolerance (pre-refinement)
  ior: 1.45,              // Fresnel index of refraction
  maxRoughness: 1,        // Only reflect surfaces below this roughness
  maxDepthDifference: 10, // Post-refinement depth tolerance
  blend: 0.9,             // Temporal accumulation factor (0=no history, 1=full history)
  blur: 0.5,              // Ratio of blurred to raw reflection
  blurKernel: 1,          // Box blur kernel size
  blurSharpness: 10,      // Edge-aware blur sharpness
  jitter: 0,              // Random ray deviation (reduces banding)
  steps: 20,              // Ray march iterations
  refineSteps: 5,         // Binary search refinement steps
  resolutionScale: 1,     // Render at fraction of screen resolution
  missedRays: true,       // Show env map for unresolved rays
});

composer.addPass(new EffectPass(camera, ssrEffect));
```

**For animated materials:**

```javascript
mesh.material.userData.needsUpdatedReflections = true;
```

**Limitations:**
- Only reflects visible screen content; use envMap as fallback for off-screen reflections.
- Expensive at full resolution -- use `resolutionScale: 0.5` for 4x speedup.
- The `screen-space-reflections` repo is archived; active development continues at [realism-effects](https://github.com/0beqz/realism-effects).

### Screen-Space Ambient Occlusion (SSAO)

SSAO darkens crevices and contact points by sampling depth around each pixel. Three.js ships `SSAOPass` and `SAOPass` built-in. The modern community standard is **N8AOPass**.

**N8AOPass (recommended):**

```bash
npm install n8ao
```

```javascript
import { N8AOPass } from 'n8ao';

const composer = new EffectComposer(renderer);
composer.addPass(new RenderPass(scene, camera));

const aoPass = new N8AOPass(scene, camera, window.innerWidth, window.innerHeight);
aoPass.configuration.aoRadius = 5.0;          // World-space AO radius
aoPass.configuration.distanceFalloff = 1.0;   // Falloff speed
aoPass.configuration.intensity = 3.0;         // Darkness multiplier
aoPass.configuration.color = new THREE.Color(0x000000);  // AO tint
aoPass.configuration.halfRes = true;          // 2-4x perf boost
composer.addPass(aoPass);
```

**Built-in SSAOPass:**

```javascript
import { SSAOPass } from 'three/addons/postprocessing/SSAOPass.js';

const ssaoPass = new SSAOPass(scene, camera, window.innerWidth, window.innerHeight);
ssaoPass.kernelRadius = 16;
ssaoPass.minDistance = 0.005;
ssaoPass.maxDistance = 0.1;
composer.addPass(ssaoPass);
```

### God Rays (Volumetric Light Scattering)

Two approaches: classical screen-space radial blur (GPU Gems 3), or shadow-map ray marching.

**three-good-godrays (shadow map approach, higher quality):**

```bash
npm install three-good-godrays
```

```javascript
import { EffectComposer, RenderPass } from 'postprocessing';
import { GodraysPass } from 'three-good-godrays';

// Shadow mapping required
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;

// Light source
const light = new THREE.DirectionalLight(0xffffff, 2);
light.castShadow = true;
light.shadow.mapSize.set(1024, 1024);
light.shadow.camera.near = 0.1;
light.shadow.camera.far = 100;
light.position.set(5, 10, 5);
scene.add(light);

// Ensure meshes cast/receive shadows
scene.traverse((obj) => {
  if (obj.isMesh) {
    obj.castShadow = true;
    obj.receiveShadow = true;
  }
});

// Composer
const composer = new EffectComposer(renderer, {
  frameBufferType: THREE.HalfFloatType,
});
composer.addPass(new RenderPass(scene, camera));

const godraysPass = new GodraysPass(light, camera, {
  density: 1 / 128,       // Ray march density
  maxDensity: 0.5,         // Cap maximum density
  edgeStrength: 2,         // Edge detection intensity
  edgeRadius: 2,           // Edge detection radius
  distanceAttenuation: 2,  // Distance-based falloff
  color: new THREE.Color(0xffffff),
  raymarchSteps: 60,       // Quality vs performance
  blur: true,              // Post-process blur
  gammaCorrection: true,   // Set false if other passes follow
});
godraysPass.renderToScreen = true;
composer.addPass(godraysPass);
```

**Classical radial blur approach (GPU Gems 3):**

The algorithm:
1. Render the scene with light source as white, all occluders as black, to an offscreen texture.
2. Apply a radial blur centered on the light's screen-space position.
3. Additively blend the result over the normal render.

Fragment shader for radial blur:

```glsl
uniform sampler2D tOcclusion;
uniform vec2 uLightPos;    // Light position in screen space [0,1]
uniform float uDecay;      // 0.96
uniform float uDensity;    // 1.0
uniform float uWeight;     // 0.5
uniform float uExposure;   // 0.12
const int NUM_SAMPLES = 64;

varying vec2 vUv;

void main() {
  vec2 texCoord = vUv;
  vec2 deltaTexCoord = (texCoord - uLightPos) * uDensity / float(NUM_SAMPLES);

  float illuminationDecay = 1.0;
  vec4 color = vec4(0.0);

  for (int i = 0; i < NUM_SAMPLES; i++) {
    texCoord -= deltaTexCoord;
    vec4 sample = texture2D(tOcclusion, texCoord);
    sample *= illuminationDecay * uWeight;
    color += sample;
    illuminationDecay *= uDecay;
  }

  gl_FragColor = color * uExposure;
}
```

### Screen-Space Shadows

Contact shadows that darken the ground near objects. Three.js r155+ includes experimental contact shadows. A common approach:

```javascript
// Render depth from a light's perspective
const shadowCamera = new THREE.OrthographicCamera(-10, 10, 10, -10, 0.1, 50);
const shadowRT = new THREE.WebGLRenderTarget(1024, 1024, {
  type: THREE.FloatType,
  minFilter: THREE.NearestFilter,
  magFilter: THREE.NearestFilter,
});

// In the screen-space shadow pass shader, project each pixel into light space
// and compare depth to determine shadow
```

### Performance Considerations

- SSR is the most expensive screen-space effect. Always use `resolutionScale: 0.5` or lower.
- N8AOPass `halfRes` mode is 2-4x faster with minimal quality loss.
- God rays: reduce `raymarchSteps` for real-time. 30-60 steps is typical for production.
- All screen-space effects scale with screen resolution -- lower pixel ratio on mobile.

### Key References

- [screen-space-reflections (GitHub)](https://github.com/0beqz/screen-space-reflections)
- [realism-effects (successor)](https://github.com/0beqz/realism-effects)
- [N8AOPass](https://www.npmjs.com/package/n8ao)
- [three-good-godrays (GitHub)](https://github.com/Ameobea/three-good-godrays)
- [Three.js SSR WebGPU Example](https://threejs.org/examples/webgpu_postprocessing_ssr.html)
- [3D Game Shaders for Beginners: SSR](https://lettier.github.io/3d-game-shaders-for-beginners/screen-space-reflection.html)

---

## 4. Raymarching in Three.js

### Overview

Raymarching renders geometry by stepping rays from the camera through each pixel and testing against Signed Distance Functions (SDFs). No mesh geometry needed -- shapes are defined mathematically. The scene lives entirely in the fragment shader on a fullscreen quad.

### Fullscreen Quad Setup

```javascript
// Minimal geometry covering the viewport
const geometry = new THREE.PlaneGeometry(2, 2);
const material = new THREE.ShaderMaterial({
  uniforms: {
    uTime: { value: 0 },
    uResolution: { value: new THREE.Vector2(window.innerWidth, window.innerHeight) },
    uCameraPos: { value: new THREE.Vector3(0, 0, -3) },
    uCameraTarget: { value: new THREE.Vector3(0, 0, 0) },
  },
  vertexShader: `
    varying vec2 vUv;
    void main() {
      vUv = uv;
      gl_Position = vec4(position.xy, 0.0, 1.0);
    }
  `,
  fragmentShader: raymarchFragmentSource,
  depthWrite: false,
  depthTest: false,
});

const quad = new THREE.Mesh(geometry, material);
scene.add(quad);

// Use OrthographicCamera for fullscreen quad
const camera = new THREE.OrthographicCamera(-1, 1, 1, -1, 0, 1);
```

### SDF Primitives

```glsl
// Sphere: distance from point p to surface of sphere at origin with radius r
float sdSphere(vec3 p, float r) {
  return length(p) - r;
}

// Box: half-extents b
float sdBox(vec3 p, vec3 b) {
  vec3 q = abs(p) - b;
  return length(max(q, 0.0)) + min(max(q.x, max(q.y, q.z)), 0.0);
}

// Rounded Box
float sdRoundBox(vec3 p, vec3 b, float r) {
  vec3 q = abs(p) - b + r;
  return length(max(q, 0.0)) + min(max(q.x, max(q.y, q.z)), 0.0) - r;
}

// Torus: t.x = major radius, t.y = minor radius
float sdTorus(vec3 p, vec2 t) {
  vec2 q = vec2(length(p.xz) - t.x, p.y);
  return length(q) - t.y;
}

// Cylinder: h = half-height, r = radius
float sdCylinder(vec3 p, float h, float r) {
  vec2 d = abs(vec2(length(p.xz), p.y)) - vec2(r, h);
  return min(max(d.x, d.y), 0.0) + length(max(d, 0.0));
}

// Infinite Plane at y=0
float sdPlane(vec3 p) {
  return p.y;
}

// Capsule: from a to b with radius r
float sdCapsule(vec3 p, vec3 a, vec3 b, float r) {
  vec3 pa = p - a, ba = b - a;
  float h = clamp(dot(pa, ba) / dot(ba, ba), 0.0, 1.0);
  return length(pa - ba * h) - r;
}
```

### SDF Operations

```glsl
// Union: combine shapes
float opUnion(float d1, float d2) {
  return min(d1, d2);
}

// Subtraction: carve d2 out of d1
float opSubtraction(float d1, float d2) {
  return max(d1, -d2);
}

// Intersection: keep only overlap
float opIntersection(float d1, float d2) {
  return max(d1, d2);
}

// Smooth union (metaball / gloopy blend)
float opSmoothUnion(float d1, float d2, float k) {
  float h = clamp(0.5 + 0.5 * (d2 - d1) / k, 0.0, 1.0);
  return mix(d2, d1, h) - k * h * (1.0 - h);
}

// Smooth subtraction
float opSmoothSubtraction(float d1, float d2, float k) {
  float h = clamp(0.5 - 0.5 * (d2 + d1) / k, 0.0, 1.0);
  return mix(d2, -d1, h) + k * h * (1.0 - h);
}

// Repetition (infinite tiling)
float opRepeat(vec3 p, vec3 spacing, float sdFunc) {
  vec3 q = mod(p + 0.5 * spacing, spacing) - 0.5 * spacing;
  // Use q instead of p in your SDF
}
```

### Core Raymarching Loop

```glsl
#define MAX_STEPS 128
#define MAX_DIST 100.0
#define SURFACE_DIST 0.001

float scene(vec3 p) {
  float sphere = sdSphere(p - vec3(0, 1, 0), 1.0);
  float plane = sdPlane(p);
  return opSmoothUnion(sphere, plane, 0.5);
}

float raymarch(vec3 ro, vec3 rd) {
  float t = 0.0;
  for (int i = 0; i < MAX_STEPS; i++) {
    vec3 p = ro + rd * t;
    float d = scene(p);
    if (d < SURFACE_DIST) break;
    if (t > MAX_DIST) break;
    t += d;
  }
  return t;
}
```

### Normal Calculation (Central Differences)

```glsl
vec3 calcNormal(vec3 p) {
  const float e = 0.0001;
  return normalize(vec3(
    scene(p + vec3(e, 0, 0)) - scene(p - vec3(e, 0, 0)),
    scene(p + vec3(0, e, 0)) - scene(p - vec3(0, e, 0)),
    scene(p + vec3(0, 0, e)) - scene(p - vec3(0, 0, e))
  ));
}
```

### Lighting

```glsl
vec3 render(vec3 ro, vec3 rd) {
  float t = raymarch(ro, rd);
  if (t > MAX_DIST) return vec3(0.0); // Background

  vec3 p = ro + rd * t;
  vec3 n = calcNormal(p);
  vec3 lightDir = normalize(vec3(1.0, 2.0, -1.0));

  // Diffuse
  float diff = max(dot(n, lightDir), 0.0);

  // Specular (Blinn-Phong)
  vec3 halfVec = normalize(lightDir - rd);
  float spec = pow(max(dot(n, halfVec), 0.0), 32.0);

  // Ambient occlusion (cheap SDF-based)
  float ao = 0.0;
  for (int i = 1; i <= 5; i++) {
    float dist = 0.05 * float(i);
    ao += (dist - scene(p + n * dist)) / pow(2.0, float(i));
  }
  ao = 1.0 - ao;

  // Soft shadows
  float shadow = 1.0;
  float ph = 1e20;
  float st = 0.01;
  for (int i = 0; i < 64; i++) {
    float h = scene(p + lightDir * st);
    float y = h * h / (2.0 * ph);
    float d = sqrt(h * h - y * y);
    shadow = min(shadow, 10.0 * d / max(0.0, st - y));
    ph = h;
    st += h;
    if (shadow < 0.001 || st > 20.0) break;
  }
  shadow = clamp(shadow, 0.0, 1.0);

  vec3 color = vec3(0.9);
  color *= 0.2 + 0.8 * diff * shadow;
  color += spec * shadow * 0.3;
  color *= ao;

  return color;
}
```

### TSL Raymarching (WebGPU)

Using Three.js Shading Language for raymarching:

```javascript
import { Fn, vec3, float, uv, normalize, Loop, If, Break } from 'three/tsl';

const sdSphere = Fn(([p, r]) => {
  return p.length().sub(r);
});

const smin = Fn(([a, b, k]) => {
  const h = max(k.sub(abs(a.sub(b))), 0).div(k);
  return min(a, b).sub(h.mul(h).mul(k).mul(0.25));
});

const scene = Fn(([p]) => {
  const s1 = sdSphere(p.sub(vec3(-1, 0, 0)), 0.8);
  const s2 = sdSphere(p.sub(vec3(1, 0, 0)), 0.8);
  return smin(s1, s2, float(0.5));
});

const calcNormal = Fn(([p]) => {
  const eps = float(0.0001);
  const h = vec2(eps, 0);
  return normalize(vec3(
    scene(p.add(h.xyy)).sub(scene(p.sub(h.xyy))),
    scene(p.add(h.yxy)).sub(scene(p.sub(h.yxy))),
    scene(p.add(h.yyx)).sub(scene(p.sub(h.yyx)))
  ));
});

const raymarch = Fn(() => {
  const rayOrigin = vec3(0, 0, -3);
  const rayDirection = vec3(uv().sub(0.5).mul(2.0), 1).normalize();
  const t = float(0).toVar();
  const ray = rayOrigin.add(rayDirection.mul(t)).toVar();

  Loop({ start: 0, end: 80 }, () => {
    const d = scene(ray);
    t.addAssign(d);
    ray.assign(rayOrigin.add(rayDirection.mul(t)));
    If(d.lessThan(0.001), () => Break());
    If(t.greaterThan(100), () => Break());
  });

  // Return color based on hit/miss
  const normal = calcNormal(ray);
  const lightDir = normalize(vec3(1, 2, -1));
  const diff = max(dot(normal, lightDir), 0.0);
  return vec4(vec3(diff), 1.0);
});
```

### Combining Raymarching with Mesh Scenes

To blend raymarched SDFs into a scene with regular meshes:
1. Render the mesh scene normally to get a depth buffer.
2. In the raymarching pass, sample the depth buffer and stop marching when the ray exceeds mesh depth.
3. Composite using depth comparison.

```glsl
uniform sampler2D tDepth;
uniform float cameraNear;
uniform float cameraFar;

float linearizeDepth(float d) {
  return cameraNear * cameraFar / (cameraFar + d * (cameraNear - cameraFar));
}

// In raymarch loop:
float meshDepth = linearizeDepth(texture2D(tDepth, vUv).r);
if (t > meshDepth) break; // Stop at mesh surface
```

### Performance Considerations

- Raymarching cost scales with `MAX_STEPS * pixel_count`. Start with 64-128 steps.
- Use adaptive step sizes: multiply distance by 0.8-0.9 for safety margin near thin features.
- For complex scenes, use a bounding volume check before the main loop.
- Soft shadows are expensive (inner loop). Use 16-32 steps for shadows.
- Render at half resolution and upscale for real-time on moderate GPUs.

### Key References

- [Liquid Raymarching with TSL (Codrops, 2024)](https://tympanus.net/codrops/2024/07/15/how-to-create-a-liquid-raymarching-scene-using-three-js-shading-language/)
- [Painting with Math (Maxime Heckel)](https://blog.maximeheckel.com/posts/painting-with-math-a-gentle-study-of-raymarching/)
- [Inigo Quilez SDF Functions](https://iquilezles.org/articles/distfunctions/)
- [raymarching-for-THREE (GitHub)](https://github.com/nicoptere/raymarching-for-THREE)
- [Iridescent Crystal with SDF (Varun Vachhar)](https://varun.ca/ray-march-sdf/)

---

## 5. Volumetric Effects

### Overview

Volumetric rendering samples the density and lighting of a 3D volume along each ray. Unlike surface raymarching (which stops at the first hit), volumetric marching accumulates density and color through the entire volume using front-to-back compositing.

### Volumetric Fog / Clouds -- Core Algorithm

```glsl
#define MAX_STEPS 100
#define MARCH_SIZE 0.08

uniform float uTime;
uniform vec3 uSunDirection;

// 3D noise from a 2D texture
float noise(vec3 x) {
  vec3 p = floor(x);
  vec3 f = fract(x);
  vec2 u = f.xy * f.xy * (3.0 - 2.0 * f.xy);

  vec2 uv = (p.xy + vec2(37.0, 239.0) * p.z) + u;
  vec2 tex = textureLod(uNoiseTex, (uv + 0.5) / 256.0, 0.0).yx;

  return mix(tex.x, tex.y, f.z) * 2.0 - 1.0;
}

// Fractal Brownian Motion for detail
float fbm(vec3 p) {
  vec3 q = p + uTime * 0.5 * vec3(1.0, -0.2, -1.0);

  float f = 0.0;
  float scale = 0.5;
  float factor = 2.02;

  for (int i = 0; i < 6; i++) {
    f += scale * noise(q);
    q *= factor;
    factor += 0.21;
    scale *= 0.5;
  }
  return f;
}

// Volume density: SDF inverted + noise
float volumeDensity(vec3 p) {
  float shape = -(length(p) - 2.0); // Sphere volume
  float detail = fbm(p * 1.5);
  return shape + detail;
}

// Front-to-back compositing raymarch
vec4 raymarchVolume(vec3 ro, vec3 rd) {
  float depth = 0.0;
  vec4 result = vec4(0.0);

  for (int i = 0; i < MAX_STEPS; i++) {
    if (result.a > 0.99) break; // Early exit when opaque

    vec3 p = ro + rd * depth;
    float density = volumeDensity(p);

    if (density > 0.0) {
      // Directional derivative lighting (cheap: 2 samples vs 4 for normals)
      float diffuse = clamp(
        (volumeDensity(p) - volumeDensity(p + 0.3 * uSunDirection)) / 0.3,
        0.0, 1.0
      );

      // Lighting: ambient + directional
      vec3 lighting = vec3(0.60, 0.60, 0.75) * 1.1
                    + 0.8 * vec3(1.0, 0.6, 0.3) * diffuse;

      vec4 color = vec4(mix(vec3(1.0), vec3(0.0), density), density);
      color.rgb *= lighting;
      color.rgb *= color.a;  // Pre-multiply alpha

      // Front-to-back compositing
      result += color * (1.0 - result.a);
    }

    depth += MARCH_SIZE;
  }
  return result;
}
```

### Volumetric Lighting (Light Shafts)

Volumetric lighting samples the shadow map along the view ray to determine which points are illuminated:

```glsl
uniform sampler2D uShadowMap;
uniform mat4 uShadowMatrix;
uniform vec3 uLightPos;

float volumetricLighting(vec3 ro, vec3 rd, float maxDist) {
  float accumulation = 0.0;
  float stepSize = maxDist / float(VOLUME_STEPS);

  for (int i = 0; i < VOLUME_STEPS; i++) {
    float t = stepSize * (float(i) + 0.5);
    vec3 p = ro + rd * t;

    // Project point into shadow map space
    vec4 shadowCoord = uShadowMatrix * vec4(p, 1.0);
    shadowCoord.xyz /= shadowCoord.w;
    shadowCoord.xyz = shadowCoord.xyz * 0.5 + 0.5;

    // Sample shadow map
    float shadowDepth = texture2D(uShadowMap, shadowCoord.xy).r;
    float inLight = step(shadowCoord.z, shadowDepth + 0.005);

    // Distance attenuation
    float dist = length(p - uLightPos);
    float attenuation = 1.0 / (1.0 + dist * dist * 0.1);

    accumulation += inLight * attenuation * stepSize;
  }

  return accumulation;
}
```

### Blue Noise Dithering (Anti-Banding)

Without dithering, low sample counts produce visible banding. Blue noise breaks up the pattern:

```glsl
uniform sampler2D uBlueNoise;

vec4 raymarchVolume(vec3 ro, vec3 rd) {
  // Sample blue noise for this pixel
  vec2 noiseUV = gl_FragCoord.xy / 256.0;
  float blueNoise = texture2D(uBlueNoise, noiseUV).r;

  // Offset initial ray position
  float depth = MARCH_SIZE * blueNoise;

  // ... rest of loop
}
```

### Performance Optimizations

1. **Half-resolution rendering**: Render volumetrics at 50% resolution, blur, then composite.
2. **Temporal reprojection**: Reuse last frame's result with motion vectors to reduce per-frame samples.
3. **Blue noise dithering**: 32-64 steps with blue noise looks as good as 128+ without.
4. **Exponential step sizes**: Start with small steps near the camera, increase for distant samples.
5. **Early termination**: Break when accumulated alpha > 0.99.
6. **Bounding volume**: Only sample inside a known bounding box/sphere.

### Cloud Rendering Specifics

For physically plausible clouds:
- Use **Worley noise** (cellular) + **Perlin noise** combined for cloud shapes
- Layer multiple octaves of 3D noise at different frequencies
- Apply **Beer's Law** for light absorption: `transmission = exp(-density * stepSize * absorptionCoeff)`
- Use **Henyey-Greenstein phase function** for directional scattering:

```glsl
float henyeyGreenstein(float cosTheta, float g) {
  float g2 = g * g;
  return (1.0 - g2) / (4.0 * PI * pow(1.0 + g2 - 2.0 * g * cosTheta, 1.5));
}
```

### Key References

- [Real-time Cloudscapes with Volumetric Raymarching (Maxime Heckel)](https://blog.maximeheckel.com/posts/real-time-cloudscapes-with-volumetric-raymarching/)
- [Volumetric Clouds - Game Ready (Three.js Forum)](https://discourse.threejs.org/t/volumetric-clouds-game-ready/86598)
- [Volumetric Lighting in WebGPU (Three.js Forum)](https://discourse.threejs.org/t/volumetric-lighting-in-webgpu/87959)
- [Volumetric Light Example (GitHub)](https://github.com/netpraxis/volumetric_light_example)
- [Sebastian Hillaire - Atmosphere Rendering (Epic Games)](https://sebh.github.io/publications/egsr2020.pdf)

---

## 6. Particle Systems

### GPU Particles with GPGPU (WebGL)

GPGPU uses render-to-texture to run physics on the GPU. Each pixel in a data texture represents one particle's state (position, velocity). `GPUComputationRenderer` manages the ping-pong framebuffer approach.

**Setup:**

```javascript
import { GPUComputationRenderer } from 'three/addons/misc/GPUComputationRenderer.js';

const PARTICLE_COUNT = 256; // Texture size (256x256 = 65,536 particles)
const gpgpu = new GPUComputationRenderer(PARTICLE_COUNT, PARTICLE_COUNT, renderer);

// Create data textures
const posTexture = gpgpu.createTexture();
const velTexture = gpgpu.createTexture();

// Initialize positions (RGBA per pixel, XYZ + W)
const posData = posTexture.image.data;
for (let i = 0; i < posData.length; i += 4) {
  posData[i + 0] = (Math.random() - 0.5) * 10; // x
  posData[i + 1] = Math.random() * 5;            // y
  posData[i + 2] = (Math.random() - 0.5) * 10; // z
  posData[i + 3] = 1.0;                          // w (unused or lifetime)
}

// Register simulation shaders
const posVar = gpgpu.addVariable('uCurrentPosition', simPositionShader, posTexture);
const velVar = gpgpu.addVariable('uCurrentVelocity', simVelocityShader, velTexture);

// Dependencies: position reads velocity, velocity reads position
gpgpu.setVariableDependencies(posVar, [posVar, velVar]);
gpgpu.setVariableDependencies(velVar, [posVar, velVar]);

// Custom uniforms for simulation
posVar.material.uniforms.uDeltaTime = { value: 0 };
velVar.material.uniforms.uMouse = { value: new THREE.Vector3() };
velVar.material.uniforms.uOriginalPosition = { value: posTexture.clone() };

gpgpu.init();
```

**Position simulation shader (GLSL):**

```glsl
void main() {
  vec2 uv = gl_FragCoord.xy / resolution.xy;
  vec3 position = texture2D(uCurrentPosition, uv).xyz;
  vec3 velocity = texture2D(uCurrentVelocity, uv).xyz;

  position += velocity;

  gl_FragColor = vec4(position, 1.0);
}
```

**Velocity simulation shader (GLSL):**

```glsl
uniform sampler2D uOriginalPosition;
uniform vec3 uMouse;

void main() {
  vec2 uv = gl_FragCoord.xy / resolution.xy;
  vec3 position = texture2D(uCurrentPosition, uv).xyz;
  vec3 original = texture2D(uOriginalPosition, uv).xyz;
  vec3 velocity = texture2D(uCurrentVelocity, uv).xyz;

  // Damping
  velocity *= 0.95;

  // Spring back to original position
  vec3 toOrigin = original - position;
  float dist = length(toOrigin);
  if (dist > 0.001) {
    velocity += normalize(toOrigin) * 0.0005;
  }

  // Mouse repulsion
  float mouseDist = distance(position, uMouse);
  if (mouseDist < 1.0) {
    vec3 pushDir = normalize(position - uMouse);
    velocity += pushDir * (1.0 - mouseDist) * 0.01;
  }

  // Gravity
  velocity.y -= 0.0001;

  gl_FragColor = vec4(velocity, 1.0);
}
```

**Particle rendering geometry:**

```javascript
const particleCount = PARTICLE_COUNT * PARTICLE_COUNT;
const positions = new Float32Array(particleCount * 3);
const uvs = new Float32Array(particleCount * 2);

// Map each particle to a UV in the data texture
for (let i = 0; i < particleCount; i++) {
  const x = (i % PARTICLE_COUNT) / PARTICLE_COUNT;
  const y = Math.floor(i / PARTICLE_COUNT) / PARTICLE_COUNT;
  uvs[i * 2 + 0] = x;
  uvs[i * 2 + 1] = y;
}

const geometry = new THREE.BufferGeometry();
geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
geometry.setAttribute('uv', new THREE.BufferAttribute(uvs, 2));
```

**Particle render shaders:**

```glsl
// Vertex shader
uniform sampler2D uPositionTexture;
uniform float uParticleSize;
varying vec2 vUv;

void main() {
  vUv = uv;
  vec3 pos = texture2D(uPositionTexture, uv).xyz;
  vec4 mvPos = modelViewMatrix * vec4(pos, 1.0);
  gl_PointSize = uParticleSize / -mvPos.z;
  gl_Position = projectionMatrix * mvPos;
}
```

```glsl
// Fragment shader
uniform sampler2D uVelocityTexture;
varying vec2 vUv;

void main() {
  // Circular particle shape
  float dist = length(gl_PointCoord - 0.5);
  if (dist > 0.5) discard;

  // Color based on velocity
  vec3 velocity = texture2D(uVelocityTexture, vUv).xyz;
  float speed = clamp(length(velocity) * 50.0, 0.1, 1.0);

  gl_FragColor = vec4(vec3(0.8, 0.6, 0.2), speed);
}
```

**Animation loop:**

```javascript
function animate() {
  gpgpu.compute();

  // Pass computed textures to render material
  particleMaterial.uniforms.uPositionTexture.value =
    gpgpu.getCurrentRenderTarget(posVar).texture;
  particleMaterial.uniforms.uVelocityTexture.value =
    gpgpu.getCurrentRenderTarget(velVar).texture;

  renderer.render(scene, camera);
  requestAnimationFrame(animate);
}
```

### Compute Shader Particles (WebGPU + TSL)

Modern approach using WebGPU compute shaders. No texture ping-pong needed -- direct storage buffer read/write.

```javascript
import * as THREE from 'three/webgpu';
import {
  Fn, instanceIndex, instancedArray, storage,
  vec3, vec4, float, time, If
} from 'three/tsl';

const PARTICLE_COUNT = 1000000;

// Persistent GPU storage buffers
const positionBuffer = instancedArray(PARTICLE_COUNT, 'vec3');
const velocityBuffer = instancedArray(PARTICLE_COUNT, 'vec3');

// Initialize positions on CPU, then upload
const initPositions = new Float32Array(PARTICLE_COUNT * 3);
for (let i = 0; i < PARTICLE_COUNT; i++) {
  initPositions[i * 3 + 0] = (Math.random() - 0.5) * 20;
  initPositions[i * 3 + 1] = Math.random() * 10;
  initPositions[i * 3 + 2] = (Math.random() - 0.5) * 20;
}
// Upload to buffer...

// Compute shader: update particles
const updateParticles = Fn(() => {
  const i = instanceIndex;
  const pos = positionBuffer.element(i);
  const vel = velocityBuffer.element(i);

  const dt = float(0.016);
  const gravity = vec3(0, -9.8, 0);

  // Apply gravity
  vel.addAssign(gravity.mul(dt));

  // Update position
  pos.addAssign(vel.mul(dt));

  // Ground bounce
  If(pos.y.lessThan(0.0), () => {
    pos.y.assign(0.0);
    vel.y.assign(vel.y.negate().mul(0.6)); // Bounce with energy loss
  });
})().compute(PARTICLE_COUNT);

// Render particles
const particleMaterial = new THREE.SpriteNodeMaterial();
particleMaterial.positionNode = positionBuffer.toAttribute();
particleMaterial.scaleNode = float(0.05);
particleMaterial.colorNode = vec4(1, 0.8, 0.3, 0.8);

const particles = new THREE.InstancedMesh(
  new THREE.PlaneGeometry(1, 1),
  particleMaterial,
  PARTICLE_COUNT
);
scene.add(particles);

// Animation loop
async function animate() {
  await renderer.computeAsync(updateParticles);
  renderer.render(scene, camera);
  requestAnimationFrame(animate);
}
```

### Performance Comparison

| Approach | Max Particles (60fps) | Setup Complexity |
|----------|----------------------|-----------------|
| CPU `THREE.Points` | ~10,000 | Low |
| GPGPU (WebGL) | ~500,000 | Medium |
| Compute Shaders (WebGPU) | ~1,000,000+ | Medium |

### Key References

- [GPGPU Particle Effect (Codrops, Dec 2024)](https://tympanus.net/codrops/2024/12/19/crafting-a-dreamy-particle-effect-with-three-js-and-gpgpu/)
- [Galaxy Simulation with WebGPU Compute (Three.js Roadmap)](https://threejsroadmap.com/blog/galaxy-simulation-webgpu-compute-shaders)
- [Field Guide to TSL and WebGPU (Maxime Heckel)](https://blog.maximeheckel.com/posts/field-guide-to-tsl-and-webgpu/)
- [GPGPU Particles with TSL (Wawa Sensei)](https://wawasensei.dev/courses/react-three-fiber/lessons/tsl-gpgpu)
- [WebGPU Migration Checklist (Utsubo, 2026)](https://www.utsubo.com/blog/webgpu-threejs-migration-guide)

---

## 7. Water / Ocean

### Approach Comparison

| Method | Quality | Performance | Complexity |
|--------|---------|-------------|------------|
| Simple sine waves | Low | Excellent | Low |
| Gerstner (trochoidal) waves | Good | Good | Medium |
| FFT ocean simulation | Excellent | Medium | High |
| Three.js Water Pro | Production | Good | Low (API) |

### Gerstner Waves

Gerstner waves displace vertices horizontally (creating crests) in addition to vertically. The formula:

```
x' = x + Q * A * Dx * cos(k * dot(D, xz) - w * t)
y' = A * sin(k * dot(D, xz) - w * t)
z' = z + Q * A * Dz * cos(k * dot(D, xz) - w * t)
```

Where:
- `Q` = steepness (0-1, >1 causes loops)
- `A` = amplitude
- `D` = wave direction (normalized 2D vector)
- `k` = 2*PI / wavelength
- `w` = angular frequency = sqrt(g * k)

**GLSL Implementation:**

```glsl
struct GerstnerWave {
  vec2 direction;
  float amplitude;
  float steepness;
  float frequency;
  float speed;
};

#define NUM_WAVES 4

uniform GerstnerWave uWaves[NUM_WAVES];
uniform float uTime;

vec3 gerstnerWave(vec3 pos, GerstnerWave wave) {
  float phase = wave.frequency * dot(wave.direction, pos.xz) - wave.speed * uTime;
  float c = cos(phase);
  float s = sin(phase);
  float q = wave.steepness / (wave.frequency * wave.amplitude * float(NUM_WAVES));

  return vec3(
    q * wave.amplitude * wave.direction.x * c,
    wave.amplitude * s,
    q * wave.amplitude * wave.direction.y * c
  );
}

// In vertex shader:
void main() {
  vec3 pos = position;
  for (int i = 0; i < NUM_WAVES; i++) {
    pos += gerstnerWave(position, uWaves[i]);
  }

  // Recalculate normal
  vec3 tangent = vec3(1.0, 0.0, 0.0);
  vec3 binormal = vec3(0.0, 0.0, 1.0);
  for (int i = 0; i < NUM_WAVES; i++) {
    float phase = uWaves[i].frequency * dot(uWaves[i].direction, position.xz) - uWaves[i].speed * uTime;
    float wa = uWaves[i].frequency * uWaves[i].amplitude;
    float s = sin(phase);
    float c = cos(phase);
    float q = uWaves[i].steepness / (wa * float(NUM_WAVES));

    tangent += vec3(
      -q * uWaves[i].direction.x * uWaves[i].direction.x * wa * s,
       q * uWaves[i].direction.x * wa * c,
      -q * uWaves[i].direction.x * uWaves[i].direction.y * wa * s
    );
    binormal += vec3(
      -q * uWaves[i].direction.x * uWaves[i].direction.y * wa * s,
       q * uWaves[i].direction.y * wa * c,
      -q * uWaves[i].direction.y * uWaves[i].direction.y * wa * s
    );
  }
  vec3 normal = normalize(cross(binormal, tangent));

  vNormal = normalMatrix * normal;
  gl_Position = projectionMatrix * modelViewMatrix * vec4(pos, 1.0);
}
```

**Three.js wave configuration:**

```javascript
const waveParams = [
  { direction: [1.0, 0.0],  amplitude: 0.3, steepness: 0.5, frequency: 1.0, speed: 1.5 },
  { direction: [0.7, 0.7],  amplitude: 0.15, steepness: 0.3, frequency: 2.0, speed: 1.2 },
  { direction: [-0.4, 0.9], amplitude: 0.1,  steepness: 0.4, frequency: 3.0, speed: 0.8 },
  { direction: [0.2, -0.8], amplitude: 0.05, steepness: 0.2, frequency: 5.0, speed: 2.0 },
];

const uniforms = {
  uTime: { value: 0 },
  'uWaves[0].direction': { value: new THREE.Vector2(...waveParams[0].direction) },
  'uWaves[0].amplitude': { value: waveParams[0].amplitude },
  'uWaves[0].steepness': { value: waveParams[0].steepness },
  'uWaves[0].frequency': { value: waveParams[0].frequency },
  'uWaves[0].speed':     { value: waveParams[0].speed },
  // ... repeat for each wave
};

// Geometry needs enough subdivisions
const waterGeometry = new THREE.PlaneGeometry(100, 100, 256, 256);
waterGeometry.rotateX(-Math.PI / 2);
```

### FFT Ocean Simulation

FFT-based ocean generates displacement and normal maps from an ocean wave spectrum (Phillips or JONSWAP). The pipeline:

1. **Generate spectrum** (H0): Initial wave amplitudes from wind speed, direction, and a statistical distribution.
2. **Evolve spectrum** (H(t)): Animate using dispersion relation.
3. **Inverse FFT**: Convert frequency-domain data to spatial displacement map.
4. **Apply to mesh**: Use displacement map in vertex shader, normal map in fragment shader.

```javascript
// Using jbouny/fft-ocean approach
// Spectrum generation happens on GPU in a compute/render pass
// The output is a displacement texture + normal texture

const oceanMaterial = new THREE.ShaderMaterial({
  uniforms: {
    uDisplacementMap: { value: displacementRT.texture },
    uNormalMap: { value: normalRT.texture },
    uSunDirection: { value: new THREE.Vector3(1, 1, 0).normalize() },
    uOceanColor: { value: new THREE.Color(0x006994) },
    uSkyColor: { value: new THREE.Color(0x87CEEB) },
  },
  vertexShader: `
    uniform sampler2D uDisplacementMap;
    varying vec2 vUv;
    varying vec3 vWorldPos;

    void main() {
      vUv = uv;
      vec3 displaced = position + texture2D(uDisplacementMap, uv).xyz;
      vWorldPos = (modelMatrix * vec4(displaced, 1.0)).xyz;
      gl_Position = projectionMatrix * modelViewMatrix * vec4(displaced, 1.0);
    }
  `,
  fragmentShader: `
    uniform sampler2D uNormalMap;
    uniform vec3 uSunDirection;
    uniform vec3 uOceanColor;
    uniform vec3 uSkyColor;
    varying vec2 vUv;
    varying vec3 vWorldPos;

    void main() {
      vec3 normal = normalize(texture2D(uNormalMap, vUv).xyz * 2.0 - 1.0);
      vec3 viewDir = normalize(cameraPosition - vWorldPos);

      // Fresnel
      float fresnel = pow(1.0 - max(dot(viewDir, normal), 0.0), 5.0);

      // Specular (sun reflection)
      vec3 halfVec = normalize(uSunDirection + viewDir);
      float spec = pow(max(dot(normal, halfVec), 0.0), 256.0);

      // Subsurface scattering approximation
      float sss = pow(max(dot(viewDir, -uSunDirection), 0.0), 4.0) * 0.3;

      vec3 color = mix(uOceanColor, uSkyColor, fresnel);
      color += vec3(1.0, 0.9, 0.7) * spec;
      color += vec3(0.0, 0.5, 0.3) * sss;

      gl_FragColor = vec4(color, 0.95);
    }
  `,
});
```

### Caustics

Caustics are the light patterns on surfaces underwater created by refraction through the wave surface.

```glsl
// Caustic pattern generator (fragment shader on underwater surfaces)
uniform float uTime;
uniform float uCausticScale;
uniform float uCausticSpeed;

// Two-layer animated noise for caustic effect
float caustic(vec2 uv) {
  vec2 p = mod(uv * uCausticScale, 1.0) - 0.5;

  float t = uTime * uCausticSpeed;
  float a = length(p + vec2(sin(t * 0.3 + p.y * 3.0) * 0.3,
                            cos(t * 0.4 + p.x * 3.0) * 0.3));
  float b = length(p + vec2(cos(t * 0.2 + p.x * 2.0) * 0.4,
                            sin(t * 0.5 + p.y * 2.0) * 0.3));
  return pow(min(a, b), 3.0) * 5.0;
}

// Apply to underwater surface
vec3 surfaceColor = baseColor.rgb;
float c = caustic(worldPos.xz * 0.5);
surfaceColor += vec3(0.2, 0.4, 0.5) * c;
```

### Foam

Foam appears at wave crests (high Jacobian determinant in FFT) and shore interactions:

```glsl
// Foam from wave steepness (vertex shader output)
varying float vFoam;

void main() {
  // ... wave displacement ...

  // Jacobian determinant approximation: negative = folding surface = foam
  float jacobian = ...; // From FFT or estimated from displacement gradient
  vFoam = smoothstep(-0.1, 0.0, -jacobian);
}

// Fragment shader
uniform sampler2D uFoamTexture;

void main() {
  // ... lighting ...

  // Blend foam
  vec3 foamColor = texture2D(uFoamTexture, vUv * 10.0).rgb;
  color = mix(color, foamColor, vFoam * 0.7);
}
```

### Three.js Built-in Water

```javascript
import { Water } from 'three/addons/objects/Water.js';

const waterGeometry = new THREE.PlaneGeometry(10000, 10000);
const water = new Water(waterGeometry, {
  textureWidth: 512,
  textureHeight: 512,
  waterNormals: new THREE.TextureLoader().load('waternormals.jpg', (tex) => {
    tex.wrapS = tex.wrapT = THREE.RepeatWrapping;
  }),
  sunDirection: new THREE.Vector3(),
  sunColor: 0xffffff,
  waterColor: 0x001e0f,
  distortionScale: 3.7,
});
water.rotation.x = -Math.PI / 2;
scene.add(water);

// Animate
water.material.uniforms['time'].value += 1.0 / 60.0;
```

### Key References

- [Three.js Ocean Shader Example](https://threejs.org/examples/webgl_shaders_ocean.html)
- [FFT Ocean (jbouny)](https://github.com/jbouny/fft-ocean)
- [Gerstner Waves (CaffeineViking)](https://github.com/CaffeineViking/osgw)
- [Three.js Water Pro (docs)](https://docs.threejswaterpro.com/)
- [Water Simulation with RGB Caustics](https://water-simulation.vercel.app/)
- [GPU Gems: Effective Water Simulation](https://developer.nvidia.com/gpugems/gpugems/part-i-natural-effects/chapter-1-effective-water-simulation-physical-models)

---

## 8. Procedural Textures

### Noise Functions

#### Perlin Noise (3D)

```glsl
vec3 hash(vec3 p) {
  p = vec3(dot(p, vec3(127.1, 311.7, 74.7)),
           dot(p, vec3(269.5, 183.3, 246.1)),
           dot(p, vec3(113.5, 271.9, 124.6)));
  return -1.0 + 2.0 * fract(sin(p) * 43758.5453123);
}

float noise(vec3 p) {
  vec3 i = floor(p);
  vec3 f = fract(p);
  vec3 u = f * f * (3.0 - 2.0 * f); // Smoothstep interpolation

  return mix(
    mix(mix(dot(hash(i + vec3(0,0,0)), f - vec3(0,0,0)),
            dot(hash(i + vec3(1,0,0)), f - vec3(1,0,0)), u.x),
        mix(dot(hash(i + vec3(0,1,0)), f - vec3(0,1,0)),
            dot(hash(i + vec3(1,1,0)), f - vec3(1,1,0)), u.x), u.y),
    mix(mix(dot(hash(i + vec3(0,0,1)), f - vec3(0,0,1)),
            dot(hash(i + vec3(1,0,1)), f - vec3(1,0,1)), u.x),
        mix(dot(hash(i + vec3(0,1,1)), f - vec3(0,1,1)),
            dot(hash(i + vec3(1,1,1)), f - vec3(1,1,1)), u.x), u.y),
    u.z
  );
}
```

#### Worley (Cellular) Noise

```glsl
float worley(vec3 coords) {
  vec2 gridBase = floor(coords.xy);
  vec2 gridOffset = fract(coords.xy);
  float closest = 1.0;

  for (float y = -2.0; y <= 2.0; y += 1.0) {
    for (float x = -2.0; x <= 2.0; x += 1.0) {
      vec2 neighborCell = vec2(x, y);
      vec2 cellWorld = gridBase + neighborCell;
      vec2 cellOffset = vec2(
        noise(vec3(cellWorld, coords.z) + vec3(243.432, 324.235, 0.0)),
        noise(vec3(cellWorld, coords.z))
      );
      float dist = length(neighborCell + cellOffset - gridOffset);
      closest = min(closest, dist);
    }
  }
  return closest;
}
```

### Fractal Brownian Motion (fBm)

Layer noise at different frequencies for natural complexity:

```glsl
float fbm(vec3 p, int octaves, float persistence, float lacunarity) {
  float amplitude = 0.5;
  float frequency = 1.0;
  float total = 0.0;
  float normalization = 0.0;

  for (int i = 0; i < octaves; i++) {
    float noiseValue = noise(p * frequency);
    total += noiseValue * amplitude;
    normalization += amplitude;
    amplitude *= persistence;  // Typically 0.5
    frequency *= lacunarity;   // Typically 2.0
  }

  return total / normalization;
}
```

### Turbulence (Absolute-value fBm)

Takes absolute value of noise for sharp ridges:

```glsl
float turbulence(vec3 p, int octaves, float persistence, float lacunarity) {
  float amplitude = 0.5;
  float frequency = 1.0;
  float total = 0.0;
  float normalization = 0.0;

  for (int i = 0; i < octaves; i++) {
    float noiseValue = abs(noise(p * frequency)); // abs() creates ridges
    total += noiseValue * amplitude;
    normalization += amplitude;
    amplitude *= persistence;
    frequency *= lacunarity;
  }

  return total / normalization;
}
```

### Marble Texture

```glsl
vec3 marbleTexture(vec3 p) {
  float n = fbm(p * 2.0, 6, 0.5, 2.0);
  // Sine creates veins; noise breaks them into organic patterns
  float veins = sin(p.x * 8.0 + n * 10.0) * 0.5 + 0.5;

  vec3 darkColor = vec3(0.1, 0.1, 0.12);
  vec3 lightColor = vec3(0.95, 0.93, 0.90);
  vec3 veinColor = vec3(0.3, 0.28, 0.25);

  vec3 base = mix(darkColor, lightColor, veins);
  // Add subtle vein coloring
  float veinIntensity = pow(1.0 - veins, 3.0);
  return mix(base, veinColor, veinIntensity * 0.5);
}
```

### Wood Texture

```glsl
vec3 woodTexture(vec3 p) {
  // Ring pattern from distance to center axis
  float dist = length(p.xz);
  float rings = fract(dist * 10.0 + fbm(p * 3.0, 4, 0.5, 2.0) * 2.0);

  // Stepped rings for visible grain
  float stepped = floor(rings * 10.0) / 10.0;
  float remainder = fract(rings * 10.0);
  stepped = (stepped - remainder) * 0.5 + 0.5;

  vec3 darkWood = vec3(0.35, 0.2, 0.08);
  vec3 lightWood = vec3(0.7, 0.5, 0.25);

  return mix(darkWood, lightWood, stepped);
}
```

### Terrain Generation

```glsl
float terrain(vec2 p) {
  float height = 0.0;

  // Large landforms
  height += fbm(vec3(p * 0.01, 0.0), 4, 0.5, 2.0) * 50.0;

  // Medium features (hills)
  height += fbm(vec3(p * 0.05, 1.0), 4, 0.5, 2.0) * 10.0;

  // Fine detail (rocks)
  height += fbm(vec3(p * 0.2, 2.0), 4, 0.5, 2.0) * 2.0;

  // Ridges (turbulence)
  height += turbulence(vec3(p * 0.03, 3.0), 5, 0.5, 2.0) * 20.0;

  return height;
}
```

### Domain Warping

Feed noise output back as input coordinates for alien/organic patterns:

```glsl
float domainWarp(vec3 p) {
  vec3 q = vec3(
    fbm(p, 4, 0.5, 2.0),
    fbm(p + vec3(5.2, 1.3, 2.8), 4, 0.5, 2.0),
    fbm(p + vec3(9.7, 3.4, 6.1), 4, 0.5, 2.0)
  );

  vec3 r = vec3(
    fbm(p + 4.0 * q + vec3(1.7, 9.2, 3.4), 4, 0.5, 2.0),
    fbm(p + 4.0 * q + vec3(8.3, 2.8, 7.1), 4, 0.5, 2.0),
    fbm(p + 4.0 * q + vec3(4.5, 6.7, 1.2), 4, 0.5, 2.0)
  );

  return fbm(p + 4.0 * r, 4, 0.5, 2.0);
}
```

### TSL Noise Functions (WebGPU)

Three.js TSL provides built-in noise nodes:

```javascript
import {
  mx_noise_float, mx_noise_vec3,     // Perlin noise
  mx_worley_noise_float,              // Worley noise
  mx_fractal_noise_float,             // fBm
  hash,                               // Fast hash
} from 'three/tsl';

// Usage in a material
const material = new THREE.MeshStandardNodeMaterial();
material.colorNode = Fn(() => {
  const p = positionLocal.mul(5.0).add(time.mul(0.3));
  const n = mx_fractal_noise_float(p, 6, 2.0, 0.5, 1.0);
  return mix(vec3(0.1, 0.1, 0.8), vec3(1.0, 0.9, 0.7), n.add(0.5));
})();
```

### Three.js Setup for Procedural Materials

```javascript
const material = new THREE.ShaderMaterial({
  uniforms: {
    uTime: { value: 0 },
    uScale: { value: 5.0 },
    uOctaves: { value: 6 },
  },
  vertexShader: `
    varying vec2 vUv;
    varying vec3 vPosition;
    void main() {
      vUv = uv;
      vPosition = position;
      gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
    }
  `,
  fragmentShader: `
    uniform float uTime;
    uniform float uScale;

    varying vec2 vUv;
    varying vec3 vPosition;

    // ... noise functions above ...

    void main() {
      vec3 p = vPosition * uScale + vec3(0, 0, uTime * 0.5);
      vec3 color = marbleTexture(p); // or woodTexture, etc.
      gl_FragColor = vec4(color, 1.0);
    }
  `,
});
```

### Performance Tips

- Noise functions with `sin()` hashing are cheaper than texture-based noise on modern GPUs.
- For 3D noise, 6 octaves of fBm is usually the visual sweet spot. More octaves are rarely distinguishable.
- Pre-compute noise into a 3D texture for heavy use (256x256x256 R8 = 16MB).
- Domain warping is expensive (3 nested fBm calls). Use 2-3 octaves per layer.
- Worley noise with a 5x5 neighbor search is expensive. A 3x3 search works for most cases.

### Key References

- [10 Noise Functions for TSL (Three.js Roadmap)](https://threejsroadmap.com/blog/10-noise-functions-for-threejs-tsl-shaders)
- [Procedural Materials with Shaders (Balazs Farago)](https://www.balazsfarago.dev/blog/procedural-materials)
- [The Book of Shaders: Noise](https://thebookofshaders.com/11/)
- [Noise Generation in Shaders (glsl.site)](https://glsl.site/post/noise-generation-in-shaders/)
- [THREE.Terrain (GitHub)](https://github.com/IceCreamYou/THREE.Terrain)

---

## 9. PBR Extensions

### MeshPhysicalMaterial Properties

`MeshPhysicalMaterial` extends `MeshStandardMaterial` with advanced physically-based features. Most are disabled by default (0.0) and add GPU cost when enabled.

#### Clearcoat (Car Paint, Lacquer, Wet Surfaces)

```javascript
const carPaint = new THREE.MeshPhysicalMaterial({
  color: 0xcc0000,
  metalness: 0.9,
  roughness: 0.4,
  clearcoat: 1.0,              // 0-1, coating intensity
  clearcoatRoughness: 0.05,    // 0-1, coating roughness
  // Optional texture maps
  clearcoatMap: clearcoatTexture,         // Per-pixel clearcoat intensity
  clearcoatRoughnessMap: roughTexture,    // Per-pixel coating roughness
  clearcoatNormalMap: normalTexture,      // Independent normal for coating layer
  clearcoatNormalScale: new THREE.Vector2(0.3, 0.3),
});
```

#### Iridescence (Soap Bubbles, Oil Films, Insect Wings)

```javascript
const oilSlick = new THREE.MeshPhysicalMaterial({
  color: 0x222222,
  metalness: 0.8,
  roughness: 0.15,
  iridescence: 1.0,                      // 0-1, effect intensity
  iridescenceIOR: 1.3,                   // 1.0-2.333, color shift strength
  iridescenceThicknessRange: [100, 400], // [min, max] thin-film thickness in nm
  iridescenceMap: iridescenceTexture,    // Per-pixel intensity
  iridescenceThicknessMap: thicknessTexture, // Per-pixel thickness (green channel)
});
```

The green channel of `iridescenceThicknessMap` maps: 0.0 = minimum thickness, 1.0 = maximum thickness.

#### Transmission (Glass, Transparent Plastics)

```javascript
const glass = new THREE.MeshPhysicalMaterial({
  color: 0xffffff,
  metalness: 0,
  roughness: 0.05,           // 0 = clear glass, higher = frosted
  transmission: 1.0,         // 0-1, optical transparency
  thickness: 0.5,            // Volume depth in mesh coordinate space
  ior: 1.5,                  // Index of refraction (glass=1.5, water=1.33, diamond=2.42)
  attenuationColor: new THREE.Color(0x88ccff),  // Tint from absorption
  attenuationDistance: 0.5,   // Distance for attenuation (world units)
  dispersion: 0.3,           // Chromatic aberration strength (0-1)
  // IMPORTANT: set opacity to 1 when using transmission
  opacity: 1.0,
  transparent: true,
  side: THREE.DoubleSide,
});
```

#### Sheen (Fabric, Velvet)

```javascript
const velvet = new THREE.MeshPhysicalMaterial({
  color: 0x330033,
  metalness: 0,
  roughness: 0.8,
  sheen: 0.8,                            // 0-1, sheen intensity
  sheenColor: new THREE.Color(0xff88ff), // Tint color
  sheenRoughness: 0.3,                   // 0-1 (default 1.0)
  sheenColorMap: sheenTexture,           // Per-pixel sheen color
  sheenRoughnessMap: sheenRoughTexture,  // Per-pixel sheen roughness
});
```

#### Specular Control (Non-Metallic)

```javascript
const material = new THREE.MeshPhysicalMaterial({
  color: 0xffffff,
  metalness: 0,
  roughness: 0.5,
  specularIntensity: 0.8,               // 0-1, scales F0 reflectance
  specularColor: new THREE.Color(1, 1, 1), // Tint at normal incidence
  specularIntensityMap: specTexture,     // Per-pixel intensity (alpha channel)
  specularColorMap: specColorTexture,    // Per-pixel tint (RGB)
});
```

### Subsurface Scattering (Custom Implementation)

Three.js does not have built-in SSS. Here is a fast approximation using `onBeforeCompile`:

```javascript
const sssUniforms = {
  uThicknessMap: { value: thicknessTexture },
  uThicknessPower: { value: 20.0 },    // Light falloff exponent
  uThicknessScale: { value: 4.0 },     // Intensity multiplier
  uThicknessDistortion: { value: 0.185 }, // Normal distortion
  uThicknessAmbient: { value: 0.0 },   // Ambient SSS contribution
  uSubsurfaceColor: { value: new THREE.Color(0xff4400) }, // SSS tint
};

const material = new THREE.MeshStandardMaterial({ color: 0xffddcc, roughness: 0.5 });

material.onBeforeCompile = (shader) => {
  // Add uniforms
  Object.assign(shader.uniforms, sssUniforms);

  // Inject thickness sampling and translucency calculation
  shader.fragmentShader = shader.fragmentShader.replace(
    '#include <lights_fragment_begin>',
    `
    #include <lights_fragment_begin>

    // SSS: Fast subsurface scattering approximation
    {
      float thickness = texture2D(uThicknessMap, vUv).r;
      vec3 scatterNormal = normalize(normal + geometry.viewDir * uThicknessDistortion);

      // For each directional/point light, calculate light transmitted through surface
      #if NUM_DIR_LIGHTS > 0
      for (int i = 0; i < NUM_DIR_LIGHTS; i++) {
        vec3 lightDir = directionalLights[i].direction;
        float lightTransmit = pow(
          max(dot(geometry.viewDir, -scatterNormal), 0.0),
          uThicknessPower
        ) * uThicknessScale + uThicknessAmbient;

        reflectedLight.directDiffuse += lightTransmit *
          thickness * directionalLights[i].color * uSubsurfaceColor;
      }
      #endif
    }
    `
  );

  // Add uniform declarations
  shader.fragmentShader = `
    uniform sampler2D uThicknessMap;
    uniform float uThicknessPower;
    uniform float uThicknessScale;
    uniform float uThicknessDistortion;
    uniform float uThicknessAmbient;
    uniform vec3 uSubsurfaceColor;
  ` + shader.fragmentShader;
};
```

### Custom BRDF via onBeforeCompile

Replace the default Cook-Torrance BRDF with a custom one by targeting shader chunks:

```javascript
material.onBeforeCompile = (shader) => {
  // Replace the BRDF calculation chunk
  shader.fragmentShader = shader.fragmentShader.replace(
    '#include <lights_physical_fragment>',
    `
    // Custom BRDF: wrap diffuse + custom specular
    vec3 customDiffuse = diffuseColor.rgb * (0.5 + 0.5 * dot(normal, geometry.viewDir));

    // Ashikhmin-Shirley anisotropic specular (simplified)
    vec3 halfVector = normalize(geometry.viewDir + directLight.direction);
    float NdotH = max(dot(normal, halfVector), 0.0);
    float spec = pow(NdotH, 64.0 * (1.0 - roughnessFactor));

    reflectedLight.directDiffuse += customDiffuse * directLight.color;
    reflectedLight.directSpecular += spec * directLight.color;
    `
  );
};
```

### Anisotropy (Brushed Metal, Hair)

Available in three.js r160+:

```javascript
const brushedMetal = new THREE.MeshPhysicalMaterial({
  color: 0xcccccc,
  metalness: 1.0,
  roughness: 0.3,
  anisotropy: 1.0,                    // 0-1, anisotropy strength
  anisotropyRotation: Math.PI / 4,    // Radians, direction of anisotropy
  anisotropyMap: anisotropyTexture,   // Per-pixel anisotropy (RG channels = direction)
});
```

### Performance Notes

| Feature | GPU Cost | Notes |
|---------|----------|-------|
| `clearcoat` | Low | Adds one extra specular lobe |
| `iridescence` | Low | Thin-film interference lookup |
| `sheen` | Low | Simple additional lobe |
| `transmission` | High | Requires separate render pass for refraction |
| `dispersion` | Very High | Multiple refraction samples |
| Custom SSS | Medium | Extra texture samples + lighting pass |
| `anisotropy` | Low | Modified GGX distribution |

Always specify an `envMap` for MeshPhysicalMaterial -- reflections and transmission rely heavily on it.

### Key References

- [MeshPhysicalMaterial Docs](https://threejs.org/docs/pages/MeshPhysicalMaterial.html)
- [MeshPhysicalMaterial Tutorial (sbcode)](https://sbcode.net/threejs/meshphysicalmaterial/)
- [Fast SSS in Three.js (Matt DesLauriers gist)](https://gist.github.com/mattdesl/2ee82157a86962347dedb6572142df7c)
- [OpenPBR Surface Specification](https://dl.acm.org/doi/10.1145/3744199.3744632)
- [Enterprise PBR Shading Model (Dassault)](https://dassaultsystemes-technology.github.io/EnterprisePBRShadingModel/user_guide.md.html)

---

## 10. Instancing + Merged Geometries

### InstancedMesh

Renders many copies of the same geometry + material in a single draw call. Each instance can have its own transform and color.

**Basic setup:**

```javascript
const geometry = new THREE.BoxGeometry(1, 1, 1);
const material = new THREE.MeshStandardMaterial({ color: 0xffffff });
const count = 10000;

const instancedMesh = new THREE.InstancedMesh(geometry, material, count);

const dummy = new THREE.Object3D();
const color = new THREE.Color();

for (let i = 0; i < count; i++) {
  // Set transform
  dummy.position.set(
    (Math.random() - 0.5) * 100,
    (Math.random() - 0.5) * 100,
    (Math.random() - 0.5) * 100
  );
  dummy.rotation.set(
    Math.random() * Math.PI * 2,
    Math.random() * Math.PI * 2,
    0
  );
  dummy.scale.setScalar(0.5 + Math.random() * 1.5);
  dummy.updateMatrix();
  instancedMesh.setMatrixAt(i, dummy.matrix);

  // Set per-instance color
  color.setHSL(Math.random(), 0.7, 0.5);
  instancedMesh.setColorAt(i, color);
}

instancedMesh.instanceMatrix.needsUpdate = true;
instancedMesh.instanceColor.needsUpdate = true;

scene.add(instancedMesh);
```

**Updating at runtime:**

```javascript
function animate() {
  const dummy = new THREE.Object3D();

  for (let i = 0; i < updateCount; i++) {
    instancedMesh.getMatrixAt(i, dummy.matrix);
    dummy.matrix.decompose(dummy.position, dummy.quaternion, dummy.scale);

    // Modify
    dummy.position.y += Math.sin(time + i * 0.1) * 0.01;
    dummy.updateMatrix();

    instancedMesh.setMatrixAt(i, dummy.matrix);
  }
  instancedMesh.instanceMatrix.needsUpdate = true;
}
```

**Custom per-instance attributes (shader access):**

```javascript
// Add custom per-instance data
const phases = new Float32Array(count);
for (let i = 0; i < count; i++) phases[i] = Math.random() * Math.PI * 2;

const phaseAttr = new THREE.InstancedBufferAttribute(phases, 1);
instancedMesh.geometry.setAttribute('aPhase', phaseAttr);
```

In vertex shader:

```glsl
attribute float aPhase;
varying float vPhase;

void main() {
  vPhase = aPhase;
  // ... standard instanced vertex transform (automatic with InstancedMesh)
}
```

### InstancedBufferGeometry

Lower-level than InstancedMesh. You build the geometry + instance attributes manually. More flexible but more work.

```javascript
const baseGeometry = new THREE.BoxGeometry(1, 1, 1);
const instancedGeometry = new THREE.InstancedBufferGeometry();

// Copy base geometry attributes
instancedGeometry.index = baseGeometry.index;
instancedGeometry.setAttribute('position', baseGeometry.getAttribute('position'));
instancedGeometry.setAttribute('normal', baseGeometry.getAttribute('normal'));
instancedGeometry.setAttribute('uv', baseGeometry.getAttribute('uv'));

// Create per-instance attributes
const instanceCount = 50000;
const offsets = new Float32Array(instanceCount * 3);
const colors = new Float32Array(instanceCount * 3);
const scales = new Float32Array(instanceCount);

for (let i = 0; i < instanceCount; i++) {
  offsets[i * 3 + 0] = (Math.random() - 0.5) * 200;
  offsets[i * 3 + 1] = (Math.random() - 0.5) * 200;
  offsets[i * 3 + 2] = (Math.random() - 0.5) * 200;

  colors[i * 3 + 0] = Math.random();
  colors[i * 3 + 1] = Math.random();
  colors[i * 3 + 2] = Math.random();

  scales[i] = 0.5 + Math.random();
}

instancedGeometry.setAttribute('aOffset', new THREE.InstancedBufferAttribute(offsets, 3));
instancedGeometry.setAttribute('aColor', new THREE.InstancedBufferAttribute(colors, 3));
instancedGeometry.setAttribute('aScale', new THREE.InstancedBufferAttribute(scales, 1));

// Custom shader material
const material = new THREE.ShaderMaterial({
  vertexShader: `
    attribute vec3 aOffset;
    attribute vec3 aColor;
    attribute float aScale;
    varying vec3 vColor;

    void main() {
      vColor = aColor;
      vec3 pos = position * aScale + aOffset;
      gl_Position = projectionMatrix * modelViewMatrix * vec4(pos, 1.0);
    }
  `,
  fragmentShader: `
    varying vec3 vColor;
    void main() {
      gl_FragColor = vec4(vColor, 1.0);
    }
  `,
});

const mesh = new THREE.Mesh(instancedGeometry, material);
scene.add(mesh);
```

### BatchedMesh (r155+)

Combines multiple different geometries into a single draw call using MultiDraw. Unlike InstancedMesh (same geometry), BatchedMesh handles mixed geometry types.

**Complete API:**

```javascript
const material = new THREE.MeshStandardMaterial({ color: 0x888888 });

// Constructor: (maxInstances, maxVertices, maxIndices, material)
const batchedMesh = new THREE.BatchedMesh(1000, 50000, 100000, material);

// Add different geometries
const boxGeoId = batchedMesh.addGeometry(
  new THREE.BoxGeometry(1, 1, 1),
  100,   // reservedVertexCount (optional, for future geometry swaps)
  200    // reservedIndexCount (optional)
);
const sphereGeoId = batchedMesh.addGeometry(new THREE.SphereGeometry(0.5, 16, 16));
const coneGeoId = batchedMesh.addGeometry(new THREE.ConeGeometry(0.3, 1, 8));

// Create instances of each geometry
const dummy = new THREE.Matrix4();
const instances = [];

for (let i = 0; i < 300; i++) {
  const geoId = [boxGeoId, sphereGeoId, coneGeoId][i % 3];
  const instanceId = batchedMesh.addInstance(geoId);

  dummy.makeTranslation(
    (Math.random() - 0.5) * 50,
    (Math.random() - 0.5) * 50,
    (Math.random() - 0.5) * 50
  );
  batchedMesh.setMatrixAt(instanceId, dummy);

  // Per-instance color (Vector4 for alpha support)
  const color = new THREE.Color().setHSL(Math.random(), 0.7, 0.5);
  batchedMesh.setColorAt(instanceId, color);

  instances.push(instanceId);
}

// Visibility toggle
batchedMesh.setVisibleAt(instances[5], false);

// Swap instance geometry at runtime
batchedMesh.setGeometryIdAt(instances[0], sphereGeoId);

// Per-object frustum culling (enabled by default)
batchedMesh.perObjectFrustumCulled = true;

// Sorting (enabled by default for transparency)
batchedMesh.sortObjects = true;

scene.add(batchedMesh);
```

**Geometry management:**

```javascript
// Replace geometry (must fit within reserved space)
const newSphere = new THREE.SphereGeometry(0.8, 32, 32);
batchedMesh.setGeometryAt(sphereGeoId, newSphere);

// Delete
batchedMesh.deleteInstance(instanceId);
batchedMesh.deleteGeometry(geoId); // Also deletes all instances using this geometry

// Defragment after many deletes
batchedMesh.optimize();

// Resize buffers
batchedMesh.setInstanceCount(2000);
batchedMesh.setGeometrySize(100000, 200000);

// Query
const geoId = batchedMesh.getGeometryIdAt(instanceId);
const visible = batchedMesh.getVisibleAt(instanceId);
const count = batchedMesh.instanceCount; // Current count (readonly)
```

**Custom sorting:**

```javascript
batchedMesh.setCustomSort((list, camera) => {
  // list is an array of { z: number } objects
  // Sort by z (camera depth) for correct transparency
  list.sort((a, b) => b.z - a.z);
});
```

### When to Use Which

| Scenario | Best Choice | Why |
|----------|-------------|-----|
| Same geometry, 1000+ instances | **InstancedMesh** | Lowest overhead, `drawElementsInstanced` |
| Same geometry, 100k+ instances | **InstancedMesh** | MultiDraw is slower at high counts |
| 2-10 different geometries | **BatchedMesh** | Combines into single MultiDraw call |
| Need per-instance visibility | **BatchedMesh** | Has `setVisibleAt()` API |
| Custom per-instance shader data | **InstancedBufferGeometry** | Full attribute control |
| Dynamic geometry swapping | **BatchedMesh** | `setGeometryIdAt()` at runtime |
| Static scene, many materials | **Merge geometries** | `BufferGeometryUtils.mergeGeometries()` |

### Merged Geometries (Static Optimization)

For static scenes where instances share a material but never move independently:

```javascript
import { mergeGeometries } from 'three/addons/utils/BufferGeometryUtils.js';

const geometries = [];
const dummy = new THREE.Object3D();

for (let i = 0; i < 500; i++) {
  const geo = new THREE.BoxGeometry(1, 1 + Math.random(), 1);

  // Apply transform directly to geometry
  dummy.position.set(i * 2, 0, 0);
  dummy.updateMatrix();
  geo.applyMatrix4(dummy.matrix);

  geometries.push(geo);
}

const merged = mergeGeometries(geometries);
const mesh = new THREE.Mesh(merged, material);
scene.add(mesh);
// One draw call, zero per-frame overhead, but no per-instance control
```

### InstancedMesh2 (Community Extension)

Extends `InstancedMesh` with culling, LOD, sorting, raycasting via BVH:

```bash
npm install @three.ez/instanced-mesh
```

Features:
- Frustum culling per instance (uses BVH)
- LOD system per instance
- Efficient raycasting
- Built-in sorting for transparency
- Visibility toggling

### Performance Benchmarks (Approximate)

| Method | 10k objects | 100k objects | 1M objects |
|--------|------------|-------------|-----------|
| Individual Meshes | 10-15 fps | Unusable | Unusable |
| InstancedMesh | 60 fps | 55-60 fps | 30-45 fps |
| BatchedMesh (mixed geo) | 55-60 fps | 40-50 fps | 20-30 fps |
| Merged Geometry | 60 fps | 60 fps | 50-60 fps |

Note: BatchedMesh can underperform at very high counts (100k+) due to MultiDraw overhead vs instanced draw. For homogeneous geometry at scale, InstancedMesh wins.

### Key References

- [InstancedMesh Docs](https://threejs.org/docs/pages/InstancedMesh.html)
- [BatchedMesh Docs](https://threejs.org/docs/pages/BatchedMesh.html)
- [BatchedMesh for High-Performance Rendering (Wael Yasmina)](https://waelyasmina.net/articles/batchedmesh-for-high-performance-rendering-in-three-js/)
- [BatchedMesh + WebGPURenderer (Codrops, Oct 2024)](https://tympanus.net/codrops/2024/10/30/interactive-3d-with-three-js-batchedmesh-and-webgpurenderer/)
- [Instanced Rendering (Wael Yasmina)](https://waelyasmina.net/articles/instanced-rendering-in-three-js/)
- [batched-mesh-extensions (GitHub)](https://github.com/agargaro/batched-mesh-extensions)
- [InstancedMesh2 (@three.ez)](https://www.npmjs.com/package/@three.ez/instanced-mesh)

---

## Appendix: WebGPU / TSL Quick Reference

Three.js r171+ ships a production-ready WebGPU renderer via `import * as THREE from 'three/webgpu'`.

### TSL (Three.js Shading Language)

TSL replaces GLSL for WebGPU. It compiles to WGSL (WebGPU) or GLSL (WebGL fallback) from a single JavaScript-based authoring syntax.

**GLSL to TSL cheat sheet:**

| GLSL | TSL |
|------|-----|
| `float x = 1.0;` | `const x = float(1.0)` |
| `float x = 0.0; x = 1.0;` | `const x = float(0).toVar(); x.assign(1.0)` |
| `vec3(1, 0, 0)` | `vec3(1, 0, 0)` |
| `sin(x)` | `sin(x)` |
| `dot(a, b)` | `dot(a, b)` |
| `mix(a, b, t)` | `mix(a, b, t)` |
| `position` | `positionLocal` |
| `normal` | `normalLocal` |
| `uv` | `uv()` |
| `gl_FragColor = ...` | `return vec4(...)` |
| `uniform float uTime;` | `const uTime = uniform(float(0))` |
| `attribute float aPhase;` | Via `instancedArray` or buffer attributes |
| `varying float vX;` | `const vX = varying(float())` |
| `for (int i...)` | `Loop({ start, end }, () => { ... })` |
| `if (x > 0.0)` | `If(x.greaterThan(0.0), () => { ... })` |

**Key TSL imports:**

```javascript
import {
  // Types
  float, vec2, vec3, vec4, mat3, mat4, int, uint,
  // Math
  sin, cos, pow, sqrt, abs, min, max, clamp, mix, step, smoothstep,
  dot, cross, normalize, length, distance, reflect, refract,
  // Geometry
  positionLocal, positionWorld, normalLocal, normalWorld, uv,
  // Camera
  cameraPosition, viewDirection,
  // Time
  time,
  // Control flow
  Fn, Loop, If, Break,
  // Uniforms/buffers
  uniform, instancedArray, instanceIndex,
  // Noise
  mx_noise_float, mx_fractal_noise_float,
  // Post-processing
  pass, bloom,
} from 'three/tsl';
```

**Compute shader template:**

```javascript
const computeShader = Fn(() => {
  const i = instanceIndex;
  const pos = positionBuffer.element(i);
  const vel = velocityBuffer.element(i);

  vel.addAssign(vec3(0, -9.8, 0).mul(0.016));
  pos.addAssign(vel.mul(0.016));
})().compute(PARTICLE_COUNT);

// Execute
await renderer.computeAsync(computeShader);
```

**Feature detection:**

```javascript
async function initRenderer(canvas) {
  if (navigator.gpu) {
    const adapter = await navigator.gpu.requestAdapter();
    if (adapter) {
      const renderer = new THREE.WebGPURenderer({ canvas, antialias: true });
      await renderer.init();
      return renderer;
    }
  }
  // Fallback
  return new THREE.WebGLRenderer({ canvas, antialias: true });
}
```

### Key References

- [WebGPU Migration Checklist (Utsubo, 2026)](https://www.utsubo.com/blog/webgpu-threejs-migration-guide)
- [TSL Tutorials (GitHub)](https://github.com/cmhhelgeson/Threejs_TSL_Tutorials)
- [WebGPU + TSL (Threlte Docs)](https://threlte.xyz/docs/learn/advanced/webgpu/)
