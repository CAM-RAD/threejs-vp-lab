# Cutting-Edge Three.js Reference (r160-r175+)

> Comprehensive technical reference covering WebGPU, TSL, compute shaders, BatchedMesh,
> advanced materials, WebXR, gaussian splatting, texture compression, and glTF extensions.
> Last updated: April 2026.

---

## Table of Contents

1. [WebGPU in Three.js](#1-webgpu-in-threejs)
2. [TSL (Three Shading Language)](#2-tsl-three-shading-language)
3. [Compute Shaders](#3-compute-shaders)
4. [BatchedMesh](#4-batchedmesh)
5. [Advanced Material Features](#5-advanced-material-features)
6. [Release Changelog Highlights (r160-r175)](#6-release-changelog-highlights-r160-r175)
7. [WebXR (VR/AR)](#7-webxr-vrar)
8. [Gaussian Splatting](#8-gaussian-splatting)
9. [Texture Compression (KTX2 / Basis Universal)](#9-texture-compression-ktx2--basis-universal)
10. [glTF Extensions](#10-gltf-extensions)

---

## 1. WebGPU in Three.js

### Overview

`WebGPURenderer` is Three.js's modern rendering backend targeting the WebGPU API, with automatic fallback to WebGL 2 when WebGPU is unavailable. Production-ready since **r171** (September 2025).

### Key Differences from WebGLRenderer

| Aspect | WebGLRenderer | WebGPURenderer |
|--------|---------------|----------------|
| **API** | WebGL 2 | WebGPU (fallback to WebGL 2) |
| **Shaders** | GLSL strings | TSL (node-based, compiles to GLSL or WGSL) |
| **Compute** | Not supported | Full compute shader support |
| **Init** | Synchronous | Asynchronous (`await renderer.init()`) |
| **Import** | `import * as THREE from 'three'` | `import * as THREE from 'three/webgpu'` |
| **Node Materials** | Not supported (removed in r164) | Full support |
| **Performance** | Baseline | 20-40% faster at 10,000+ draw calls |

### Setup

```javascript
import * as THREE from 'three/webgpu';

const renderer = new THREE.WebGPURenderer({
  antialias: true,
  alpha: true
});
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(window.devicePixelRatio);
document.body.appendChild(renderer.domElement);

// CRITICAL: WebGPU init is async
await renderer.init();

// Then render as usual
renderer.setAnimationLoop(() => {
  renderer.render(scene, camera);
});
```

### Constructor Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `alpha` | boolean | `true` | Transparent default framebuffer |
| `depth` | boolean | `true` | Include depth buffer |
| `stencil` | boolean | `false` | Include stencil buffer |
| `antialias` | boolean | `false` | MSAA anti-aliasing |
| `samples` | number | `0` | MSAA sample count (4 when antialias=true) |
| `forceWebGL` | boolean | `false` | Force WebGL 2 backend |
| `logarithmicDepthBuffer` | boolean | `false` | Logarithmic depth |
| `reversedDepthBuffer` | boolean | `false` | Reversed depth |
| `multiview` | boolean | `false` | Multiview for WebXR |
| `outputBufferType` | number | `HalfFloatType` | Output buffer precision |

### Migration Path

**Import changes (r171+):**
```javascript
// OLD
import * as THREE from 'three';
import { ShaderMaterial } from 'three';

// NEW — for WebGPU + node materials
import * as THREE from 'three/webgpu';
import { color, texture, uniform } from 'three/tsl';
```

**Migration time estimates:**
- Simple projects with standard materials: 1-2 hours
- Projects with custom GLSL shaders: 1-2 days
- Large apps with complex post-processing: 1-2 weeks

**Feature detection:**
```javascript
if (navigator.gpu) {
  renderer = new THREE.WebGPURenderer();
} else {
  renderer = new THREE.WebGLRenderer(); // fallback
}
```

### Browser Support (as of early 2026)

| Browser | WebGPU Status |
|---------|---------------|
| Chrome 113+ | Shipped |
| Edge 113+ | Shipped |
| Firefox 130+ | Shipped |
| Safari 18+ | Shipped (late 2025) |

### Performance Notes

- WebGPU shows measurable gains only with **10,000+ draw calls** (20-40% improvement via command buffer batching)
- Under 1,000 draw calls: negligible difference; WebGL may even be slightly faster
- Data that stays on the GPU avoids CPU-GPU transfer bottleneck
- The same scene graph works across both renderers without modification

---

## 2. TSL (Three Shading Language)

### Overview

TSL is Three.js's node-based shader abstraction written in JavaScript. Instead of writing raw GLSL/WGSL strings, you compose shader logic from JavaScript expressions that compile to both GLSL (WebGL 2) and WGSL (WebGPU).

**Key advantages:**
- Write once, run on both WebGL and WebGPU
- No string manipulation or shader concatenation
- Full tree-shaking support
- Composable nodes shared across materials and post-processing
- Automatic expression deduplication and optimization

### Imports

```javascript
// TSL functions
import {
  float, int, vec2, vec3, vec4, color, mat3, mat4,
  uniform, attribute, texture, uv, time, deltaTime,
  positionLocal, positionWorld, normalLocal, normalWorld,
  sin, cos, mix, step, smoothstep, clamp, fract,
  Fn, If, Loop, Break, Continue, Return, Discard
} from 'three/tsl';

// Node materials
import {
  MeshStandardNodeMaterial,
  MeshPhysicalNodeMaterial,
  NodeMaterial,
  SpriteNodeMaterial
} from 'three/webgpu';
```

### Data Types

```javascript
float(1.0)              // scalar float
int(5)                  // integer
uint(3)                 // unsigned integer
bool(true)              // boolean
vec2(1.0, 2.0)          // 2D vector
vec3(1.0, 2.0, 3.0)     // 3D vector
vec4(1, 0, 0, 1)        // 4D vector / RGBA color
color(0xff0000)          // color (accepts hex, CSS names)
color('crimson')         // CSS color name
mat2(...)               // 2x2 matrix
mat3(...)               // 3x3 matrix
mat4(...)               // 4x4 matrix
ivec2(), ivec3(), ivec4() // integer vectors
uvec2(), uvec3(), uvec4() // unsigned integer vectors
bvec2(), bvec3(), bvec4() // boolean vectors
```

### Type Conversions

```javascript
someNode.toFloat()
someNode.toVec3()
someNode.toColor()
someNode.toInt()
```

### Swizzling

```javascript
const v = vec3(1.0, 2.0, 3.0);
v.xy   // vec2(1.0, 2.0)
v.zyx  // vec3(3.0, 2.0, 1.0)
v.r    // 1.0 (rgba aliases)
v.stpq // stpq aliases also work
```

### Uniforms

```javascript
const myColor = uniform(new THREE.Color(0x0066FF));
const scale = uniform(float(1.0));

material.colorNode = myColor;

// Update in animation loop:
scale.value = Math.sin(Date.now() * 0.001);

// Auto-update callbacks:
scale.onFrameUpdate(() => performance.now() * 0.001);
scale.onRenderUpdate((frame) => frame.time);
scale.onObjectUpdate((object, frame) => object.position.y);
```

### Attributes & Geometry Data

```javascript
attribute('customAttr', 'float')  // custom attributes
uv()                              // UV coordinates (alias for uv(0))
uv(1)                             // second UV set
vertexColor()                     // vertex colors
positionGeometry                  // raw geometry position
positionLocal                     // transformed local position
positionWorld                     // world-space position
positionView                      // view-space position
normalGeometry                    // raw geometry normal
normalLocal                       // local-space normal
normalView                        // view-space normal
normalWorld                       // world-space normal
tangentLocal, tangentView, tangentWorld
bitangentLocal, bitangentView, bitangentWorld
instanceIndex                     // for instanced rendering
vertexIndex                       // vertex index
```

### Operators

```javascript
// Arithmetic (chainable)
a.add(b)          // a + b
a.sub(b)          // a - b
a.mul(b)          // a * b
a.div(b)          // a / b
a.mod(b)          // a % b

// Comparison
a.equal(b)
a.notEqual(b)
a.lessThan(b)
a.greaterThan(b)
a.lessThanEqual(b)
a.greaterThanEqual(b)

// Logical
a.and(b)
a.or(b)
a.not()

// Assignment (for mutable variables)
myVar.assign(newValue)
myVar.addAssign(delta)
myVar.subAssign(delta)
myVar.mulAssign(factor)
```

### Functions (Fn)

The `Fn()` wrapper creates reusable shader functions:

```javascript
// Array parameter style
const wobble = Fn(([amplitude, frequency]) => {
  return sin(time.mul(frequency)).mul(amplitude);
});

// Object parameter style
const blend = Fn(({ colorA, colorB, factor }) => {
  return mix(colorA, colorB, factor);
});

// Usage
material.colorNode = blend({
  colorA: color(0xff0000),
  colorB: color(0x0000ff),
  factor: wobble([float(0.5), float(2.0)])
});
```

Functions receive a second parameter with build context:
```javascript
const myFn = Fn(([param], { material, geometry, object, camera, scene, renderer }) => {
  // Access build-time context
});
```

### Variables

```javascript
const uvScaled = uv().mul(10).toVar('uvScaled');  // mutable variable
const constant = uv().mul(10).toConst();           // inline constant
```

### Conditionals

```javascript
// If/ElseIf/Else
If(condition, () => {
  col.assign(color(1, 0, 0));
}).ElseIf(condition2, () => {
  col.assign(color(0, 1, 0));
}).Else(() => {
  col.assign(color(0, 0, 1));
});

// Switch/Case
Switch(value)
  .Case(0, () => { col.assign(color(1, 0, 0)); })
  .Case(1, () => { col.assign(color(0, 1, 0)); })
  .Default(() => { col.assign(color(1, 1, 1)); });

// Ternary
const result = select(value.greaterThan(1), 1.0, value);
```

### Loops

```javascript
Loop(10, ({ i }) => {
  // executes 10 times, i is loop counter
});

// With explicit range
Loop({ start: int(0), end: int(10), type: 'int' }, ({ i }) => {});

// Nested loops
Loop(10, 5, ({ i, j }) => {});

// While loop (r175+)
Loop(() => {
  // body
});

Break();      // exit loop
Continue();   // next iteration
```

### Math Functions

```javascript
// Trigonometric
sin(x), cos(x), tan(x), asin(x), acos(x), atan(y, x)
sinh(x), cosh(x), tanh(x)
degrees(x), radians(x)

// Power/Root
pow(x, y), sqrt(x), cbrt(x), inverseSqrt(x)
exp(x), exp2(x), log(x), log2(x)

// Utility
abs(x), sign(x), floor(x), ceil(x), round(x), trunc(x), fract(x)
clamp(value, low, high), saturate(x)    // clamp to 0-1
mix(a, b, t), step(edge, x), smoothstep(edge0, edge1, x)
min(...values), max(...values)

// Vector
dot(a, b), cross(a, b), distance(a, b), length(a)
normalize(a), reflect(I, N), refract(I, N, eta)
faceforward(N, I, Nref)

// Derivatives
dFdx(x), dFdy(x), fwidth(x)

// Constants
PI, TWO_PI, HALF_PI, EPSILON, INFINITY
```

### Textures

```javascript
texture(myTexture)                          // sample at default UV
texture(myTexture, uv())                    // explicit UV
texture(myTexture, uv(), float(2.0))        // explicit LOD level
textureLoad(myTexture, ivec2(10, 20))       // texel fetch (no filtering)
textureSize(myTexture, int(0))              // dimensions
textureBicubic(myTexture, float(1.0))       // bicubic filtering
cubeTexture(envMap, direction)              // cubemap sampling
texture3D(volTexture, uvw, level)           // 3D texture
triplanarTexture(texX, texY, texZ, scale, position, normal)
```

### Varyings & Vertex Stage

```javascript
// Compute in vertex shader, interpolate in fragment
const viewNormal = toVarying(modelNormalMatrix.mul(normalLocal));

// Force computation to vertex stage
const vertexData = toVertexStage(someExpensiveCalc);
```

> **Note (r174+):** `varying()` was renamed to `toVarying()`, `vertexStage()` to `toVertexStage()`.

### Camera & Model Matrices

```javascript
// Camera
cameraNear, cameraFar, cameraPosition
cameraProjectionMatrix, cameraViewMatrix
cameraNormalMatrix, cameraWorldMatrix

// Model
modelWorldMatrix, modelViewMatrix, modelNormalMatrix
modelPosition, modelDirection, modelScale
modelViewPosition

// Screen/Viewport
screenUV              // normalized [0,1] frame buffer coords
screenCoordinate      // physical pixels
screenSize            // frame buffer dimensions
viewportUV            // normalized viewport coords
viewportCoordinate    // viewport pixels
viewportSize          // viewport dimensions
```

### Oscillators & Timing

```javascript
time              // elapsed seconds (float)
deltaTime         // frame delta
oscSine(time)     // sine wave 0-1
oscSquare(time)   // square wave 0-1
oscTriangle(time) // triangle wave 0-1
oscSawtooth(time) // sawtooth wave 0-1
```

### Color Operations

```javascript
luminance(color)           // perceived brightness
saturation(color, adj)     // adjust saturation
vibrance(color, adj)       // selective saturation
hue(color, rotation)       // rotate hue (radians)
posterize(color, steps)    // reduce color levels
grayscale(color)           // desaturate
sepia(color)               // sepia tone

// Blend modes
blendBurn(a, b), blendDodge(a, b)
blendOverlay(a, b), blendScreen(a, b), blendColor(a, b)
```

### UV Utilities

```javascript
matcapUV                              // matcap texture coords
rotateUV(uv, rotation, center)       // rotate UVs
spherizeUV(uv, strength, center)     // spherical distortion
spritesheetUV(count, uv, frame)      // spritesheet animation
equirectUV(direction)                // equirectangular mapping
```

### Remapping

```javascript
remap(value, inLow, inHigh, outLow, outHigh)
remapClamp(value, inLow, inHigh, outLow, outHigh)
```

### Node Material Properties

Override specific parts of the lighting pipeline:

```javascript
const material = new MeshStandardNodeMaterial();

// Core overrides
material.colorNode = texture(colorMap).mul(color(0xff0000));
material.roughnessNode = float(0.5);
material.metalnessNode = float(1.0);
material.normalNode = normalMap(texture(normalTex));
material.emissiveNode = color(0x003300);
material.opacityNode = float(0.8);

// Vertex deformation
material.positionNode = Fn(() => {
  const pos = positionLocal;
  const displacement = sin(time.mul(3.0).add(pos.y.mul(5.0))).mul(0.1);
  return pos.add(normalLocal.mul(displacement));
})();

// Full fragment override (replaces entire lighting model)
material.fragmentNode = color(0xff0000);

// Full vertex override
material.vertexNode = /* ... */;

// Shadow control
material.castShadowNode = /* ... */;
material.receivedShadowNode = /* ... */;
```

### Post-Processing with TSL

```javascript
import { pass, bloom, fxaa, gaussianBlur, dof } from 'three/webgpu';

const postProcessing = new THREE.PostProcessing(renderer);
const scenePass = pass(scene, camera);

// Chain effects
postProcessing.outputNode = bloom(fxaa(scenePass), 1.0, 0.5, 0.5);
```

**Available effects:**
- `bloom(node, strength, radius, threshold)` — Glow
- `fxaa(node)` / `smaa(node)` / `traa(...)` — Anti-aliasing
- `gaussianBlur(node, direction, sigma)` — Blur
- `dof(node, viewZ, focus, focalLength, bokeh)` — Depth of field
- `chromaticAberration(node, strength)` — Color fringing
- `ssao(depth, normal, camera)` — Ambient occlusion
- `ssr(color, depth, normal, metalness, roughness, camera)` — Screen-space reflections
- `motionBlur(node, velocity, samples)` — Motion blur
- `film(node, intensity)` — Film grain
- `dotScreen(node, angle, scale)` — Halftone
- `anamorphic(node, threshold, scale, samples)` — Anamorphic flare
- `lut3D(node, lut, size, intensity)` — Color grading
- `afterImage(node, damp)` — Persistence
- `godrays(depth, camera, light)` — Volumetric rays
- `lensflare(bloom, options)` — Lens flare
- `vignette()` — Vignette
- `denoise(node, depth, normal, camera)` — Denoising

### Multiple Render Targets (MRT)

```javascript
const scenePass = pass(scene, camera);
scenePass.setMRT(mrt({
  output: output,
  normal: directionToColor(normalView),
  velocity: velocity
}));

const colorTexture = scenePass.getTextureNode('output');
const normalTexture = scenePass.getTextureNode('normal');
const depthTexture = scenePass.getTextureNode('depth');
```

### Fog

```javascript
import { fog, rangeFogFactor, densityFogFactor } from 'three/tsl';

// Linear fog
scene.fogNode = fog(color(0x000000), rangeFogFactor(10, 100));

// Exponential fog
scene.fogNode = fog(color(0x333333), densityFogFactor(0.02));
```

### Structs (r173+)

```javascript
const BoundingBox = struct({ min: 'vec3', max: 'vec3' });
const bb = BoundingBox(vec3(0), vec3(1));
const minVal = bb.get('min');
```

### Arrays

```javascript
const colors = array([vec3(1, 0, 0), vec3(0, 1, 0), vec3(0, 0, 1)]);
const green = colors.element(1);

const uniformColors = uniformArray([
  new THREE.Color(1, 0, 0),
  new THREE.Color(0, 1, 0)
], 'color');
```

### Debugging

```javascript
// Inspect compiled shader code
renderer.debug.getShaderAsync(scene, camera, mesh).then((info) => {
  console.log(info.vertexShader);
  console.log(info.fragmentShader);
});

// Name variables for readable shader output
const myVar = someNode.toVar('myReadableName');
```

---

## 3. Compute Shaders

### Overview

Compute shaders run on the GPU but do not render anything. They process data in parallel, reading from and writing to GPU buffers. In Three.js, compute shaders are written with TSL and dispatched via `renderer.computeAsync()`.

**Requirements:** WebGPURenderer (no compute on WebGL backend).

### Storage Buffers

```javascript
import { instancedArray, attributeArray, storage } from 'three/tsl';

// instancedArray — creates a persistent GPU storage buffer
// Used for per-instance data (positions, velocities, etc.)
const positionBuffer = instancedArray(PARTICLE_COUNT, 'vec3');
const velocityBuffer = instancedArray(PARTICLE_COUNT, 'vec3');
const colorBuffer = instancedArray(PARTICLE_COUNT, 'vec4');

// attributeArray — creates a per-vertex attribute buffer
const vertexData = attributeArray(VERTEX_COUNT, 'vec3');
```

### Defining a Compute Shader

```javascript
import { Fn, instanceIndex, float, vec3, storage } from 'three/tsl';

const PARTICLE_COUNT = 100000;

const positionBuffer = instancedArray(PARTICLE_COUNT, 'vec3');
const velocityBuffer = instancedArray(PARTICLE_COUNT, 'vec3');

// Initialize positions
const initCompute = Fn(() => {
  const i = instanceIndex;
  const angle = float(i).div(PARTICLE_COUNT).mul(Math.PI * 2);
  const radius = float(i).div(PARTICLE_COUNT).mul(5.0);

  positionBuffer.element(i).assign(vec3(
    cos(angle).mul(radius),
    sin(float(i).mul(0.01)).mul(2.0),
    sin(angle).mul(radius)
  ));

  velocityBuffer.element(i).assign(vec3(0, 0, 0));
})().compute(PARTICLE_COUNT);

// Update positions each frame
const updateCompute = Fn(() => {
  const i = instanceIndex;
  const pos = positionBuffer.element(i);
  const vel = velocityBuffer.element(i);

  // Simple gravity toward origin
  const toCenter = pos.negate().normalize().mul(0.01);
  vel.addAssign(toCenter);

  // Damping
  vel.mulAssign(0.99);

  // Update position
  pos.addAssign(vel.mul(deltaTime));
})().compute(PARTICLE_COUNT);
```

### Dispatching Compute

```javascript
// One-time init
await renderer.computeAsync(initCompute);

// In animation loop
renderer.setAnimationLoop(async () => {
  await renderer.computeAsync(updateCompute);
  renderer.render(scene, camera);
});
```

### Rendering Compute Results

Convert storage buffers to vertex attributes using `.toAttribute()`:

```javascript
import { SpriteNodeMaterial } from 'three/webgpu';

const particleMaterial = new SpriteNodeMaterial();
particleMaterial.positionNode = positionBuffer.toAttribute();
particleMaterial.colorNode = colorBuffer.toAttribute();
particleMaterial.scaleNode = float(0.05);

const particles = new THREE.Mesh(
  new THREE.PlaneGeometry(1, 1),
  particleMaterial
);
particles.count = PARTICLE_COUNT;
scene.add(particles);
```

### Workgroup Sizes

```javascript
// Default workgroup size
const compute = myFn().compute(totalInvocations);

// Explicit workgroup size
const compute = myFn().compute(totalInvocations, [256]);
// Use multiples of 16-32 for best performance
```

### Atomic Operations

For thread-safe shared writes:

```javascript
import { atomicAdd, atomicStore, atomicLoad, atomicMax, atomicMin } from 'three/tsl';

atomicAdd(buffer.element(index), value);
atomicStore(buffer.element(index), value);
const val = atomicLoad(buffer.element(index));
```

### Barriers

```javascript
import { workgroupBarrier, storageBarrier, textureBarrier } from 'three/tsl';

workgroupBarrier();  // sync threads within workgroup
storageBarrier();    // sync storage buffer access
textureBarrier();    // sync texture access
```

### Compute Builtins

```javascript
globalId            // uvec3 - global invocation ID
localId             // uvec3 - local invocation within workgroup
workgroupId         // uvec3 - workgroup index
numWorkgroups       // uvec3 - total workgroups dispatched
instanceIndex       // uint  - 1D invocation index
subgroupSize        // uint  - subgroup size
```

### Performance Notes

- Keep data on the GPU — avoid CPU-GPU transfers per frame
- Use `instancedArray` for persistent buffers, `attributeArray` for per-vertex
- Compute output fed directly to render pipeline via `.toAttribute()` avoids readback
- Operations that take milliseconds on CPU complete in microseconds when parallelized
- Replaces the older FBO/texture-based GPGPU approach entirely

---

## 4. BatchedMesh

### Overview

`BatchedMesh` renders large numbers of objects sharing the same material but with **different geometries** and transforms in a single draw call. Available since r156, it extends `InstancedMesh` by supporting heterogeneous geometry.

### When to Use

| Scenario | Use |
|----------|-----|
| Same geometry, same material, many instances | `InstancedMesh` |
| Different geometries, same material, many instances | `BatchedMesh` |
| Different materials | Separate meshes or multiple BatchedMesh |

### Setup

```javascript
const material = new THREE.MeshStandardMaterial({ color: 0x00ff00 });

// Pre-allocate capacity
const batchedMesh = new THREE.BatchedMesh(
  10000,   // maxInstanceCount
  50000,   // maxVertexCount (total across all geometries)
  100000,  // maxIndexCount
  material
);

scene.add(batchedMesh);
```

### Adding Geometries and Instances

```javascript
// Create geometries
const boxGeo = new THREE.BoxGeometry(1, 1, 1);
const sphereGeo = new THREE.SphereGeometry(0.5, 16, 16);
const coneGeo = new THREE.ConeGeometry(0.5, 1, 8);

// Register geometries — returns geometry IDs
const boxId = batchedMesh.addGeometry(boxGeo);
const sphereId = batchedMesh.addGeometry(sphereGeo);
const coneId = batchedMesh.addGeometry(coneGeo);

// Create instances — returns instance IDs
const inst1 = batchedMesh.addInstance(boxId);
const inst2 = batchedMesh.addInstance(sphereId);
const inst3 = batchedMesh.addInstance(coneId);

// Position via matrices
const matrix = new THREE.Matrix4();
matrix.makeTranslation(2, 0, 0);
batchedMesh.setMatrixAt(inst1, matrix);

matrix.makeTranslation(-2, 0, 0);
batchedMesh.setMatrixAt(inst2, matrix);

matrix.compose(
  new THREE.Vector3(0, 2, 0),
  new THREE.Quaternion().setFromEuler(new THREE.Euler(0, Math.PI / 3, 0)),
  new THREE.Vector3(1.5, 1, 1)
);
batchedMesh.setMatrixAt(inst3, matrix);
```

### Instance Colors

```javascript
batchedMesh.setColorAt(inst1, new THREE.Color(0xff0000));
batchedMesh.setColorAt(inst2, new THREE.Color(0x00ff00));

// With alpha (Vector4)
batchedMesh.setColorAt(inst3, new THREE.Vector4(0, 0, 1, 0.5));
```

### Dynamic Operations

```javascript
// Change which geometry an instance uses
batchedMesh.setGeometryIdAt(inst1, sphereId);

// Show/hide individual instances
batchedMesh.setVisibleAt(inst2, false);

// Remove instances and geometries
batchedMesh.deleteInstance(inst3);
batchedMesh.deleteGeometry(coneId); // also deletes referencing instances

// Repack after deletions to reclaim space
batchedMesh.optimize();
```

### Frustum Culling & Sorting

```javascript
batchedMesh.perObjectFrustumCulled = true;  // default: true
batchedMesh.sortObjects = true;              // default: true
// Sorts front-to-back for opaque, back-to-front for transparent

// Custom sort function
batchedMesh.setCustomSort((list, camera) => {
  // list[i] has .z for depth
  list.sort((a, b) => a.z - b.z);
});
```

### Loading GLB Models into BatchedMesh

```javascript
const loader = new GLTFLoader();
loader.load('tree.glb', (glb) => {
  const treeMesh = glb.scene.getObjectByName('Tree');
  const geo = treeMesh.geometry;

  const bm = new THREE.BatchedMesh(
    5000,
    geo.attributes.position.count,
    geo.index.count,
    treeMesh.material
  );

  const geoId = bm.addGeometry(geo);

  const matrix = new THREE.Matrix4();
  for (let i = 0; i < 5000; i++) {
    const id = bm.addInstance(geoId);
    matrix.makeTranslation(
      (Math.random() - 0.5) * 100,
      0,
      (Math.random() - 0.5) * 100
    );
    bm.setMatrixAt(id, matrix);
  }

  scene.add(bm);
});
```

### Performance Notes

- Reduces draw calls from N to 1 for objects sharing a material
- 50,000 instances (mixed box + sphere) run at interactive frame rates
- Set accurate capacity values — don't over-allocate vertex/index counts
- Combine with array textures for diverse appearances with minimal draw calls
- Added in r156; `addInstance()` required since r166

---

## 5. Advanced Material Features

### MeshPhysicalMaterial

The most feature-rich standard material in Three.js, supporting PBR with extensions for glass, fabric, car paint, and more.

#### Transmission (Glass / Translucency)

```javascript
const glass = new THREE.MeshPhysicalMaterial({
  transmission: 1.0,        // 0-1, degree of optical transparency
  thickness: 0.5,           // volume thickness in world units (0 = thin-walled)
  ior: 1.5,                 // index of refraction (1.0 - 2.333)
  roughness: 0.0,           // smoother = clearer glass
  attenuationColor: new THREE.Color(0x88ccff),  // color absorption
  attenuationDistance: 2.0,  // absorption distance
  // IMPORTANT: set opacity to 1 when using transmission
  opacity: 1.0,
});
```

**Texture maps:**
- `transmissionMap` — red channel controls per-pixel transmission
- `thicknessMap` — green channel controls per-pixel thickness

#### Iridescence (Soap Bubbles / Oil Films)

```javascript
const iridescent = new THREE.MeshPhysicalMaterial({
  iridescence: 1.0,                    // 0-1, intensity
  iridescenceIOR: 1.3,                // 1.0 - 2.333, shift strength
  iridescenceThicknessRange: [100, 400], // nm, min and max layer thickness
  // iridescenceMap — red channel for per-pixel control
  // iridescenceThicknessMap — green channel maps to thickness range
});
```

#### Anisotropy (Brushed Metal)

```javascript
const brushedMetal = new THREE.MeshPhysicalMaterial({
  anisotropy: 0.8,            // 0-1, strength
  anisotropyRotation: 0,      // radians, direction in tangent space
  metalness: 1.0,
  roughness: 0.3,
  // anisotropyMap — RG channels define direction, B channel multiplies strength
});
```

#### Sheen (Fabric / Cloth)

```javascript
const fabric = new THREE.MeshPhysicalMaterial({
  sheen: 1.0,                               // 0-1, intensity
  sheenColor: new THREE.Color(0x663399),    // tint color
  sheenRoughness: 0.8,                       // 0-1, roughness
  // sheenColorMap — RGB multiplied against sheenColor
  // sheenRoughnessMap — alpha channel multiplied against sheenRoughness
});
```

#### Clearcoat (Car Paint / Carbon Fiber)

```javascript
const carPaint = new THREE.MeshPhysicalMaterial({
  clearcoat: 1.0,              // 0-1, clear layer intensity
  clearcoatRoughness: 0.1,     // 0-1
  // clearcoatMap — red channel for per-pixel intensity
  // clearcoatNormalMap — independent normals for clear layer
  // clearcoatNormalScale — Vector2, how much normal map affects
  // clearcoatRoughnessMap — green channel for roughness
});
```

#### Dispersion (Chromatic Aberration Through Glass)

```javascript
const prism = new THREE.MeshPhysicalMaterial({
  transmission: 1.0,
  dispersion: 0.5,   // strength of angular color separation
  ior: 1.8,
  thickness: 1.0,
});
```

#### Specular

```javascript
const material = new THREE.MeshPhysicalMaterial({
  specularIntensity: 1.0,                       // 0-1
  specularColor: new THREE.Color(0xffffff),     // tint at normal incidence
  // specularIntensityMap — alpha channel
  // specularColorMap — RGB channels
});
```

### MeshPhysicalNodeMaterial

The node-based version allows dynamic/procedural control via TSL:

```javascript
import { MeshPhysicalNodeMaterial } from 'three/webgpu';
import { float, vec3, color, texture, sin, time } from 'three/tsl';

const material = new MeshPhysicalNodeMaterial();

// Override PBR properties with nodes
material.colorNode = texture(colorMap).mul(color(0xaaccff));
material.roughnessNode = float(0.3);
material.metalnessNode = float(1.0);
material.normalNode = normalMap(texture(normalTex));

// Animated transmission
material.transmissionNode = sin(time).mul(0.5).add(0.5);
material.thicknessNode = float(0.5);
material.iorNode = float(1.5);

// Iridescence
material.iridescenceNode = float(1.0);
material.iridescenceIORNode = float(1.3);
material.iridescenceThicknessNode = float(200);

// Clearcoat
material.clearcoatNode = float(1.0);
material.clearcoatRoughnessNode = float(0.1);

// Sheen
material.sheenNode = vec3(0.4, 0.2, 0.6);
material.sheenRoughnessNode = float(0.8);

// Anisotropy
material.anisotropyNode = float(0.8);

// Specular
material.specularColorNode = vec3(1, 1, 1);
material.specularIntensityNode = float(1.0);

// Attenuation
material.attenuationColorNode = vec3(0.5, 0.8, 1.0);
material.attenuationDistanceNode = float(2.0);

// Dispersion
material.dispersionNode = float(0.3);
```

**Feature flags** — enable features explicitly when using node properties:
```javascript
material.useTransmission = true;
material.useClearcoat = true;
material.useSheen = true;
material.useAnisotropy = true;
material.useIridescence = true;
material.useDispersion = true;
```

### NodeMaterial (Fully Custom)

For materials outside the PBR model:

```javascript
import { NodeMaterial } from 'three/webgpu';

const material = new NodeMaterial();

// Full fragment override (replaces lighting model entirely)
material.fragmentNode = vec4(
  sin(positionWorld.x.mul(10).add(time)).mul(0.5).add(0.5),
  cos(positionWorld.y.mul(10).add(time)).mul(0.5).add(0.5),
  sin(positionWorld.z.mul(10).add(time.mul(2))).mul(0.5).add(0.5),
  1.0
);

// Or just override specific slots while keeping lighting:
material.colorNode = color(0xff0000);
material.emissiveNode = color(0x003300);
```

---

## 6. Release Changelog Highlights (r160-r175)

### r160 (January 2024)

**Breaking:**
- `build/three.js` and `build/three.min.js` removed — use ES Modules
- Equirectangular env maps auto-convert to cubemaps (more memory)

**Features:**
- AgX tone mapping support
- `WEBGL_clip_cull_distance` extension support
- WebGPU initial MaterialX support and PostProcessing framework
- UBO improvements (3x3→3x4 matrices, boolean/array support)

### r161 (February 2024)

**Breaking:**
- `WebGLMultipleRenderTargets` removed — use `count` property on RenderTarget
- Hand-tracking no longer requested by default in WebXR
- Renderers use `naturalWidth/naturalHeight` for HTMLImageElement

### r162 (March 2024)

**Breaking:**
- WebGL 1 support fully dropped — WebGL 2 minimum

### r163 (April 2024)

**Breaking:**
- Stencil context attribute defaults to `false`
- `TextGeometry` parameter `height` renamed to `depth`

**Features:**
- `Scene.environmentIntensity` for global env map control

### r164 (May 2024)

**Breaking:**
- Legacy `WebGLNodeBuilder` removed — node materials only work with WebGPURenderer
- `copyTextureToTexture()` signature changed

### r165 (June 2024)

**Breaking:**
- BatchedMesh requires `addInstance()` after `addGeometry()` for rendering (r166)

### r166-r167 (July-August 2024)

**Breaking:**
- WebGPURenderer and TSL imports restructured
- `HDRJPGLoader` removed → use `UltraHDRLoader`

### r168 (September 2024)

**Breaking:**
- TSL chaining removed: `outputPass.fxaa()` → `fxaa(outputPass)`
- `viewportTopLeft` → `viewportUV`
- `uniforms()` → `uniformArray()`
- DragControls: `activate()/deactivate()` → `connect()/disconnect()`

### r169 (October 2024)

**Breaking:**
- TransformControls derives from Controls: `scene.add(controls.getHelper())`
- Several async methods: `EXRExporter.parse()`, `KTX2Exporter.parse()`
- `PackedPhongMaterial`, `SDFGeometryGenerator`, `TiltLoader` removed

### r170 (November 2024)

**Breaking:**
- `Material.type` is now a static property (cannot be changed)
- `WebGLRenderer.copyTextureToTexture3D()` removed → use `copyTextureToTexture()`
- `CinematicCamera` removed
- MMD modules deprecated

### r171 (November 2024)

**Major:**
- **Codesplit WebGL/WebGPU entry points** — `three/webgpu` and `three/tsl` imports
- **Introduced `three.tsl.js`** build file
- `attributeArray()` and `instancedArray()` for compute shaders
- ClippingGroup support for WebGPU
- Hardware clipping support
- `storageObject()` deprecated → use `storage().setPBO(true)`

**Breaking:**
- TSL blend functions renamed: `burn()` → `blendBurn()`, etc.

### r172 (December 2024)

**Major:**
- `RenderTarget3D` and `RenderTargetArray` classes
- `customCacheKey()` for node caching
- `PostProcessingUtils` renamed to `RendererUtils`

**Breaking:**
- `TextureNode.uv()` → `TextureNode.sample()`
- `rangeFog()` → `fog(color, rangeFogFactor(near, far))`
- `densityFog()` → `fog(color, densityFogFactor(density))`
- `materialAOMap` → `materialAO`

### r173 (January 2025)

**Major:**
- **XR Manager for WebGPURenderer** with layers and MSAA
- `struct()` for TSL complex types
- `array()` for TSL arrays
- `mat2` matrix type support
- `VideoFrameTexture` for WebCodecs API

**Breaking:**
- `varying()` → `toVarying()`, `vertexStage()` → `toVertexStage()`
- `.monitor` → `.observer` in NodeBuilder
- `Controls.connect()` requires DOM element
- `ParametricGeometries` → `ParametricFunctions`

### r174 (February 2025)

**Breaking:**
- `Timer` no longer auto-uses Page Visibility API — call `timer.connect(document)`
- `RenderTarget.clone()` performs full structural clone (textures no longer shared)

### r175 (March 2025)

**Major:**
- `Material.allowOverride` property
- NodeMaterial `compute()` integrated into materials
- TSL: `samplerComparison`, `debug()`, while loops in `Loop()`
- TSL: `max()`/`min()` accept arbitrary argument count
- WebXR Layers support enhanced
- Earcut library copied into core

**Breaking:**
- `Controls.connect()` requires `element` parameter
- `AnimationClip.parseAnimation()` deprecated
- `MeshGouraudMaterial` deprecated → use `MeshLambertMaterial`
- `InstancedPointsNodeMaterial` removed → use `PointsNodeMaterial`
- `modInt()` deprecated
- Luminance and LuminanceAlpha formats removed

### Migration Best Practice

> When updating old projects, it's recommended to **update the library in increments of 10 versions** to manage API changes systematically.

---

## 7. WebXR (VR/AR)

### Overview

Three.js provides built-in WebXR support through `WebXRManager`, accessible via `renderer.xr`. It handles VR/AR session initialization, stereoscopic rendering, controller input, and hand tracking.

### VR Setup

```javascript
import * as THREE from 'three/webgpu';
import { VRButton } from 'three/addons/webxr/VRButton.js';

const renderer = new THREE.WebGPURenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.xr.enabled = true;
document.body.appendChild(renderer.domElement);

// Add VR enter button
document.body.appendChild(VRButton.createButton(renderer));

// Use setAnimationLoop (not requestAnimationFrame) for XR
renderer.setAnimationLoop(() => {
  renderer.render(scene, camera);
});
```

### AR Setup

```javascript
import { ARButton } from 'three/addons/webxr/ARButton.js';

renderer.xr.enabled = true;

// AR with optional features
document.body.appendChild(ARButton.createButton(renderer, {
  requiredFeatures: ['hit-test'],
  optionalFeatures: ['dom-overlay'],
  domOverlay: { root: document.getElementById('overlay') }
}));
```

### Session Options

```javascript
// VR with hand tracking
VRButton.createButton(renderer, {
  requiredFeatures: ['local-floor'],
  optionalFeatures: [
    'bounded-floor',
    'hand-tracking',
    'layers'
  ]
});

// AR with hit testing and plane detection
ARButton.createButton(renderer, {
  requiredFeatures: ['hit-test', 'local'],
  optionalFeatures: [
    'plane-detection',
    'anchors',
    'dom-overlay'
  ]
});
```

### Controllers

```javascript
import { XRControllerModelFactory } from 'three/addons/webxr/XRControllerModelFactory.js';

const controllerModelFactory = new XRControllerModelFactory();

// Controller 0 (left)
const controller0 = renderer.xr.getController(0);
controller0.addEventListener('selectstart', onSelectStart);
controller0.addEventListener('selectend', onSelectEnd);
controller0.addEventListener('connected', (event) => {
  // event.data contains XRInputSource
});
scene.add(controller0);

// Controller grip (for model visualization)
const grip0 = renderer.xr.getControllerGrip(0);
grip0.add(controllerModelFactory.createControllerModel(grip0));
scene.add(grip0);

// Controller 1 (right)
const controller1 = renderer.xr.getController(1);
scene.add(controller1);
const grip1 = renderer.xr.getControllerGrip(1);
grip1.add(controllerModelFactory.createControllerModel(grip1));
scene.add(grip1);

// Visual ray for aiming
const lineGeometry = new THREE.BufferGeometry().setFromPoints([
  new THREE.Vector3(0, 0, 0),
  new THREE.Vector3(0, 0, -1)
]);
const line = new THREE.Line(lineGeometry);
controller0.add(line.clone());
controller1.add(line.clone());
```

### Hand Tracking

```javascript
import { XRHandModelFactory } from 'three/addons/webxr/XRHandModelFactory.js';

const handModelFactory = new XRHandModelFactory();

// Left hand
const hand0 = renderer.xr.getHand(0);
hand0.add(handModelFactory.createHandModel(hand0, 'mesh'));
// Available models: 'spheres', 'boxes', 'mesh'
scene.add(hand0);

// Right hand
const hand1 = renderer.xr.getHand(1);
hand1.add(handModelFactory.createHandModel(hand1, 'mesh'));
scene.add(hand1);

// Hand joint events
hand0.addEventListener('pinchstart', (event) => {
  // Pinch gesture detected
});
hand0.addEventListener('pinchend', (event) => {
  // Pinch released
});
```

### Hit Testing (AR)

```javascript
let hitTestSource = null;
let hitTestSourceRequested = false;

renderer.setAnimationLoop((timestamp, frame) => {
  if (frame) {
    const referenceSpace = renderer.xr.getReferenceSpace();
    const session = renderer.xr.getSession();

    if (!hitTestSourceRequested) {
      session.requestReferenceSpace('viewer').then((viewerSpace) => {
        session.requestHitTestSource({ space: viewerSpace }).then((source) => {
          hitTestSource = source;
        });
      });
      hitTestSourceRequested = true;
    }

    if (hitTestSource) {
      const hitTestResults = frame.getHitTestResults(hitTestSource);
      if (hitTestResults.length > 0) {
        const hit = hitTestResults[0];
        const pose = hit.getPose(referenceSpace);
        // Use pose.transform.position and pose.transform.orientation
        reticle.visible = true;
        reticle.matrix.fromArray(pose.transform.matrix);
      } else {
        reticle.visible = false;
      }
    }
  }

  renderer.render(scene, camera);
});
```

### WebXR Features (r173-r175)

- **XR Manager for WebGPURenderer** — full XR support with WebGPU backend
- **WebXR Layers support** — composited rendering layers for text clarity
- **Multiview rendering** — `multiview: true` option for performance
- **MSAA in XR** — improved antialiasing for XR render targets

### Platform Support (2026)

| Platform | Features |
|----------|----------|
| Meta Quest 3 | 90Hz, passthrough AR, hand+controller simultaneous, plane detection |
| Apple Vision Pro | Immersive-vr, hand tracking, eye tracking |
| Chrome/Edge Desktop | WebXR with emulator |
| Safari 18+ | Core WebXR Device API |

### Important Notes

- WebXR requires HTTPS for immersive sessions
- Use `renderer.setAnimationLoop()` instead of `requestAnimationFrame()` for XR
- Only one hand model type shows at a time (based on active input mode)
- r161+: hand-tracking not requested by default — add to session features manually

---

## 8. Gaussian Splatting

### Overview

3D Gaussian Splatting is a real-time radiance field rendering technique that represents scenes as collections of 3D Gaussians instead of meshes or point clouds. Several Three.js libraries enable loading and rendering gaussian splats in the browser.

### Libraries

#### GaussianSplats3D (by Mark Kellogg)

The original and most established Three.js implementation. **Note: no longer in active development** — author recommends Spark.

**Supported formats:** `.ply`, `.splat`, `.ksplat`

```bash
npm install @mkkellogg/gaussian-splats-3d
```

```javascript
import * as GaussianSplats3D from '@mkkellogg/gaussian-splats-3d';

// Standalone viewer
const viewer = new GaussianSplats3D.Viewer({
  cameraUp: [0, -1, -0.6],
  initialCameraPosition: [-1, -4, 6],
  initialCameraLookAt: [0, 4, 0]
});

viewer.addSplatScene('scene.ply', {
  splatAlphaRemovalThreshold: 5,
  showLoadingUI: true,
  position: [0, 1, 0],
  rotation: [0, 0, 0, 1],
  scale: [1.5, 1.5, 1.5]
}).then(() => {
  viewer.start();
});
```

**Integrating with existing Three.js scene:**

```javascript
const threeScene = new THREE.Scene();
// ... add your meshes, lights, etc.

const viewer = new GaussianSplats3D.Viewer({
  threeScene: threeScene,
  selfDrivenMode: false  // you control the render loop
});

viewer.addSplatScene('scene.ksplat').then(() => {
  // In your animation loop:
  viewer.update();
  viewer.render();
});
```

**DropInViewer (simplest integration):**

```javascript
const splatViewer = new GaussianSplats3D.DropInViewer({
  gpuAcceleratedSort: true
});
splatViewer.addSplatScenes([
  { path: 'scene1.ply', splatAlphaRemovalThreshold: 20 },
  { path: 'scene2.splat', position: [-3, -2, -3.2], scale: [1.5, 1.5, 1.5] }
]);
threeScene.add(splatViewer);
```

**Performance characteristics:**
- Custom octree for pre-render splat culling
- WASM + SIMD for CPU sorting
- Optional GPU-accelerated distance pre-calculation
- Spherical harmonics: 0th, 1st, 2nd degree

**Splat capacity limits:**

| SH Degree | Max Splats |
|-----------|------------|
| 0 | ~16,000,000 |
| 1 | ~11,000,000 |
| 2 | ~8,000,000 |

#### Spark (by World Labs) — Recommended

Actively maintained, advanced renderer with more formats and features.

**Supported formats:** `.ply`, `.splat`, `.ksplat`, `.sogs`, `.spz`

```bash
npm install @worldlabs/spark
```

**Key features:**
- Fast rendering on all devices
- Integrates with other meshes and splats in the scene
- Programmable dynamic splat effects
- Level-of-Detail (LOD) optimization
- Multiple splat format support

**Components:** SparkRenderer, SplatMesh, PackedSplats, ExtSplats

#### Other Libraries

- **gle-gs3d** — TypeScript port with `PlyLoader` and `SplatLoader`
- **ThreeSplat** (Gsplat-Spaces) — Alternative Three.js-based implementation

### Performance Notes

- CPU-based sorting creates visible artifacts during rapid camera movement
- `.ksplat` format is compressed and loads faster than raw `.ply`
- Progressive loading streams splats but loses cache-optimized ordering
- Mobile performance is sub-optimal with all current libraries
- For large scenes, use integer-based sorting and increased `splatSortDistanceMapPrecision`
- Currently only works correctly with objects that write to the depth buffer (opaque objects)

### File Format Comparison

| Format | Size | Load Speed | Optimization |
|--------|------|------------|--------------|
| `.ply` | Large | Slow | Original INRIA format |
| `.splat` | Medium | Medium | Standard web format |
| `.ksplat` | Small | Fast | Compressed for web |
| `.spz` | Small | Fast | Google compressed format |
| `.sogs` | Small | Fast | Self-Organizing Gaussians |

---

## 9. Texture Compression (KTX2 / Basis Universal)

### The Problem

Standard image formats (PNG, JPEG, WebP) require full CPU decompression before GPU upload. A single 4096x4096 texture with mipmaps consumes **90 MB of GPU memory** regardless of file size. This causes:
- Framerate stalls during upload
- Excessive memory consumption
- Slow initial loading

### GPU Texture Compression

Specialized formats (ETC, ASTC, BC, PVRTC) remain compressed on the GPU, reducing memory by **4-8x**. But different platforms require different formats — Basis Universal solves this.

### Basis Universal

A "universal" codec system that **transcodes** (not decompresses) at runtime into the device-appropriate GPU format. Delivered via the KTX2 container format.

#### Two Codecs

| Codec | Quality | File Size | Best For |
|-------|---------|-----------|----------|
| **ETC1S** | Low-medium (JPEG-like) | Small | Color textures, diffuse maps |
| **UASTC** | High (BC7-equivalent) | Larger (1-2x JPEG with Zstd) | Normal maps, data textures, anything quality-critical |

#### Transcoding Targets

Basis Universal transcodes to the optimal format per device:

| Device | ETC1S Target | UASTC Target |
|--------|-------------|--------------|
| Desktop (NVIDIA/AMD) | BC1/BC3 | BC7 |
| Desktop (Intel) | BC1/BC3 | BC7 |
| Android | ETC1/ETC2 | ASTC |
| iOS | PVRTC | ASTC |
| WebGPU | BC/ETC/ASTC | BC7/ASTC |

### KTX2Loader Setup

```javascript
import { KTX2Loader } from 'three/addons/loaders/KTX2Loader.js';

const ktx2Loader = new KTX2Loader();

// 1. Set path to WASM transcoder files
ktx2Loader.setTranscoderPath('examples/jsm/libs/basis/');

// 2. Detect hardware support (REQUIRED before loading)
ktx2Loader.detectSupport(renderer);

// 3. Load textures
const texture = await ktx2Loader.loadAsync('diffuse.ktx2');
material.map = texture;

// Callback style
ktx2Loader.load('normal.ktx2', (texture) => {
  material.normalMap = texture;
}, undefined, (error) => {
  console.error('KTX2 load error:', error);
});

// 4. Dispose when done loading all textures
ktx2Loader.dispose();
```

### With GLTFLoader

```javascript
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import { KTX2Loader } from 'three/addons/loaders/KTX2Loader.js';

const ktx2Loader = new KTX2Loader();
ktx2Loader.setTranscoderPath('examples/jsm/libs/basis/');
ktx2Loader.detectSupport(renderer);

const gltfLoader = new GLTFLoader();
gltfLoader.setKTX2Loader(ktx2Loader);

const gltf = await gltfLoader.loadAsync('model.glb');
scene.add(gltf.scene);
```

### Creating KTX2 Files

**Using `toktx` (from KTX-Software):**

```bash
# ETC1S (small, lossy)
toktx --encode basis-lz --clevel 2 --qlevel 128 output.ktx2 input.png

# UASTC (high quality, larger)
toktx --encode uastc --uastc_quality 2 --zcmp 18 output.ktx2 input.png

# UASTC for normal maps
toktx --encode uastc --uastc_quality 3 --zcmp 18 --assign_oetf linear \
  --assign_primaries none output_normal.ktx2 normal.png
```

**Using `gltf-transform`:**

```bash
# Compress all textures in a glTF to KTX2
npx gltf-transform uastc input.glb output.glb --level 2 --zstd 18
npx gltf-transform etc1s input.glb output.glb --quality 128
```

### When to Use What

| Scenario | Format | Rationale |
|----------|--------|-----------|
| First paint / hero images | WebP/AVIF | Smallest download |
| Interactive 3D scenes | KTX2 (ETC1S) | Best runtime performance |
| Normal/data maps | KTX2 (UASTC) | Quality preservation |
| HDR environment maps | `.hdr` or `.exr` | No universal compressed GPU HDR format |
| Maximum compatibility | JPEG/PNG | Simplest pipeline |

### Performance Impact

- **Memory:** 4-8x reduction vs uncompressed
- **Upload:** Near-instant (no CPU decompression)
- **Quality:** ETC1S comparable to JPEG; UASTC comparable to BC7
- **Tip:** Use `renderer.initTexture(texture)` to spread upload costs across frames

---

## 10. glTF Extensions

### Overview

GLTFLoader supports numerous Khronos (KHR) and vendor (EXT/MSFT) extensions for materials, compression, lighting, and instancing.

### Built-in Extensions

These work automatically when detected in glTF files:

#### KHR_draco_mesh_compression

Compresses mesh vertex data by 60-95%.

```javascript
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import { DRACOLoader } from 'three/addons/loaders/DRACOLoader.js';

const dracoLoader = new DRACOLoader();
dracoLoader.setDecoderPath('examples/jsm/libs/draco/');

const gltfLoader = new GLTFLoader();
gltfLoader.setDRACOLoader(dracoLoader);

const gltf = await gltfLoader.loadAsync('compressed-model.glb');
```

**Performance:**
- 60-90% vertex data reduction typical
- 20MB GLB → 3-5MB with Draco
- Decode cost: small CPU overhead on load
- Best for: geometry-heavy models

#### KHR_meshopt_compression

Alternative to Draco with different trade-offs.

```javascript
import { MeshoptDecoder } from 'three/addons/libs/meshopt_decoder.module.js';

const gltfLoader = new GLTFLoader();
gltfLoader.setMeshoptDecoder(MeshoptDecoder);
```

**Meshopt vs Draco:**

| Aspect | Draco | Meshopt |
|--------|-------|---------|
| Compression ratio | Higher | Slightly lower |
| Decode speed | Slower | Faster |
| Animation support | No | Yes (keyframe compression) |
| Streaming | No | Yes (progressive decode) |

#### KHR_lights_punctual

Adds point, spot, and directional lights to glTF scenes.

```javascript
// Lights are automatically added to the scene
const gltf = await gltfLoader.loadAsync('scene-with-lights.glb');
scene.add(gltf.scene);
// Lights are now in the scene graph
```

**Supported light types:**
- `point` → `THREE.PointLight`
- `spot` → `THREE.SpotLight`
- `directional` → `THREE.DirectionalLight`

#### KHR_texture_basisu

Enables Basis Universal compressed textures (KTX2) in glTF files. Requires KTX2Loader:

```javascript
const ktx2Loader = new KTX2Loader();
ktx2Loader.setTranscoderPath('examples/jsm/libs/basis/');
ktx2Loader.detectSupport(renderer);

gltfLoader.setKTX2Loader(ktx2Loader);
```

#### KHR_materials_* (PBR Extensions)

All mapped to `MeshPhysicalMaterial` properties:

| Extension | Material Property |
|-----------|------------------|
| `KHR_materials_transmission` | `transmission`, `transmissionMap` |
| `KHR_materials_volume` | `thickness`, `thicknessMap`, `attenuationColor`, `attenuationDistance` |
| `KHR_materials_ior` | `ior` |
| `KHR_materials_specular` | `specularIntensity`, `specularColor` |
| `KHR_materials_clearcoat` | `clearcoat`, `clearcoatRoughness`, `clearcoatNormalMap` |
| `KHR_materials_iridescence` | `iridescence`, `iridescenceIOR`, `iridescenceThicknessRange` |
| `KHR_materials_anisotropy` | `anisotropy`, `anisotropyRotation` |
| `KHR_materials_sheen` | `sheen`, `sheenColor`, `sheenRoughness` |
| `KHR_materials_emissive_strength` | `emissiveIntensity` (beyond 1.0) |
| `KHR_materials_unlit` | `MeshBasicMaterial` |
| `KHR_materials_dispersion` | `dispersion` |

#### EXT_mesh_gpu_instancing

Enables GPU instancing within glTF, so the same mesh can be efficiently rendered many times with different transforms.

```javascript
// Automatically handled by GLTFLoader
// Results in InstancedMesh in the scene graph
const gltf = await gltfLoader.loadAsync('instanced-scene.glb');
```

#### KHR_texture_transform

UV offset, rotation, and scale transforms on textures.

```javascript
// Handled automatically — maps to texture.offset, texture.rotation, texture.repeat
```

#### KHR_mesh_quantization

Reduces vertex attribute precision (float32 → int16/int8) for smaller file sizes with minimal quality loss.

#### EXT_texture_webp / EXT_texture_avif

Allows WebP or AVIF textures inside glTF files for smaller texture payloads.

### Third-Party Plugin Extensions

These require manual registration:

#### KHR_materials_variants

Allows switching between predefined material variations (e.g., color options for a product configurator).

```javascript
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';

const gltf = await gltfLoader.loadAsync('shoe.glb');
scene.add(gltf.scene);

// Access variants
const variants = gltf.userData.variants;  // ['Midnight', 'Beach', 'Street']

// Switch variant
function selectVariant(scene, variantName) {
  scene.traverse((child) => {
    if (child.isMesh && child.userData.variantMaterials) {
      const variantMaterial = child.userData.variantMaterials[variantName];
      if (variantMaterial) {
        child.material = variantMaterial;
      }
    }
  });
}

selectVariant(gltf.scene, 'Beach');
```

### Full Extension List (Built-in)

```
KHR_draco_mesh_compression     KHR_materials_ior
KHR_lights_punctual            KHR_materials_specular
KHR_materials_anisotropy       KHR_materials_transmission
KHR_materials_clearcoat        KHR_materials_iridescence
KHR_materials_dispersion       KHR_materials_unlit
KHR_materials_emissive_strength KHR_materials_volume
KHR_mesh_quantization          KHR_meshopt_compression
KHR_texture_basisu             KHR_texture_transform
EXT_materials_bump             EXT_mesh_gpu_instancing
EXT_meshopt_compression        EXT_texture_avif
EXT_texture_webp
```

### Optimization Pipeline

For production glTF files, use `gltf-transform`:

```bash
# Full optimization: Draco + KTX2 + dedup + quantize
npx gltf-transform optimize input.glb output.glb

# Individual steps
npx gltf-transform dedup input.glb temp1.glb          # remove duplicate data
npx gltf-transform quantize temp1.glb temp2.glb        # reduce precision
npx gltf-transform draco temp2.glb temp3.glb            # compress geometry
npx gltf-transform uastc temp3.glb output.glb           # compress textures

# Or meshopt instead of draco
npx gltf-transform meshopt input.glb output.glb
```

---

## Quick Reference: Import Cheatsheet

```javascript
// Core (WebGPU + fallback)
import * as THREE from 'three/webgpu';

// TSL functions
import {
  float, int, vec2, vec3, vec4, color, mat4,
  uniform, uniformArray, attribute, texture, uv,
  positionLocal, positionWorld, normalLocal, normalWorld,
  time, deltaTime, instanceIndex,
  sin, cos, mix, clamp, smoothstep, fract, step,
  Fn, If, Loop, Break, Discard, Return,
  instancedArray, attributeArray,
  fog, rangeFogFactor, densityFogFactor,
  pass, bloom, fxaa, gaussianBlur, dof,
  toVarying, toVertexStage
} from 'three/tsl';

// Node materials
import {
  NodeMaterial,
  MeshStandardNodeMaterial,
  MeshPhysicalNodeMaterial,
  SpriteNodeMaterial
} from 'three/webgpu';

// Loaders
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import { DRACOLoader } from 'three/addons/loaders/DRACOLoader.js';
import { KTX2Loader } from 'three/addons/loaders/KTX2Loader.js';

// WebXR
import { VRButton } from 'three/addons/webxr/VRButton.js';
import { ARButton } from 'three/addons/webxr/ARButton.js';
import { XRControllerModelFactory } from 'three/addons/webxr/XRControllerModelFactory.js';
import { XRHandModelFactory } from 'three/addons/webxr/XRHandModelFactory.js';
```

---

## Sources

- [Three.js Official Docs — WebGPURenderer](https://threejs.org/docs/pages/WebGPURenderer.html)
- [Three.js Official Docs — TSL](https://threejs.org/docs/pages/TSL.html)
- [Three.js Official Docs — BatchedMesh](https://threejs.org/docs/pages/BatchedMesh.html)
- [Three.js Official Docs — MeshPhysicalMaterial](https://threejs.org/docs/pages/MeshPhysicalMaterial.html)
- [Three.js Official Docs — MeshPhysicalNodeMaterial](https://threejs.org/docs/pages/MeshPhysicalNodeMaterial.html)
- [Three.js Official Docs — KTX2Loader](https://threejs.org/docs/pages/KTX2Loader.html)
- [Three.js Official Docs — GLTFLoader](https://threejs.org/docs/pages/GLTFLoader.html)
- [Three.js Official Docs — VRButton](https://threejs.org/docs/pages/VRButton.html)
- [Three.js Official Docs — ARButton](https://threejs.org/docs/pages/ARButton.html)
- [Three.js Wiki — TSL](https://github.com/mrdoob/three.js/wiki/Three.js-Shading-Language)
- [Three.js Wiki — Migration Guide](https://github.com/mrdoob/three.js/wiki/Migration-Guide)
- [Three.js GitHub Releases](https://github.com/mrdoob/three.js/releases) — r160, r171, r172, r173, r175
- [Three.js Roadmap — TSL](https://threejsroadmap.com/blog/tsl-a-better-way-to-write-shaders-in-threejs)
- [Three.js Roadmap — Galaxy Compute Shaders](https://threejsroadmap.com/blog/galaxy-simulation-webgpu-compute-shaders)
- [Three.js Roadmap — Draw Calls](https://threejsroadmap.com/blog/draw-calls-the-silent-killer)
- [SBCode — TSL Tutorials](https://sbcode.net/tsl/what-do-we-have/)
- [SBCode — WebGPU Renderer](https://sbcode.net/threejs/webgpu-renderer/)
- [Maxime Heckel — Field Guide to TSL and WebGPU](https://blog.maximeheckel.com/posts/field-guide-to-tsl-and-webgpu/)
- [Wael Yasmina — BatchedMesh Guide](https://waelyasmina.net/articles/batchedmesh-for-high-performance-rendering-in-three-js/)
- [Don McCurdy — Web Texture Formats](https://www.donmccurdy.com/2024/02/11/web-texture-formats/)
- [IGC — Three.js WebGPU Architecture](https://www.intelligentgraphicandcode.com/development/threejs-interfaces/webgpu)
- [Codrops — BatchedMesh and WebGPURenderer](https://tympanus.net/codrops/2024/10/30/interactive-3d-with-three-js-batchedmesh-and-webgpurenderer/)
- [GaussianSplats3D](https://github.com/mkkellogg/GaussianSplats3D)
- [Spark Gaussian Splatting](https://sparkjs.dev/)
- [Basis Universal](https://github.com/BinomialLLC/basis_universal)
- [KhronosGroup — glTF Extensions](https://github.com/KhronosGroup/glTF/tree/main/extensions)
- [VR Me Up — WebXR Hands](https://www.vrmeup.com/devlog/devlog_12_webxr_hands_and_gestures.html)
- [Utsubo — Three.js 2026 Changes](https://www.utsubo.com/blog/threejs-2026-what-changed)
- [Utsubo — WebGPU Migration Guide](https://www.utsubo.com/blog/webgpu-threejs-migration-guide)
- [NiksCourses — TSL Tutorial](https://niklever.com/tutorials/getting-to-grips-with-threejs-shading-language-tsl/)
- [Pragmattic — React Three Fiber + WebGPU + TSL](https://blog.pragmattic.dev/react-three-fiber-webgpu-typescript)
- [boytchev/tsl-textures](https://github.com/boytchev/tsl-textures)
