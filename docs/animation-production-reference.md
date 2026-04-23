# Three.js Animation & Production Reference

> Practical code patterns for camera animation, skeletal animation, morph targets, procedural simulation, scene optimization, lighting, shadows, and asset loading.

---

## Table of Contents

1. [Camera Animation](#1-camera-animation)
2. [Skeletal Animation](#2-skeletal-animation)
3. [Morph Targets](#3-morph-targets)
4. [Procedural Animation](#4-procedural-animation)
5. [Scene Optimization](#5-scene-optimization)
6. [Lighting](#6-lighting)
7. [Shadows](#7-shadows)
8. [Loading](#8-loading)

---

## 1. Camera Animation

### 1.1 GSAP + Three.js Camera

GSAP animates any JavaScript object properties, which makes it perfect for Three.js camera transforms. The key: animate `camera.position`, `camera.rotation`, or a proxy object, then call `renderer.render()` on each tick.

**Basic camera move:**

```js
import gsap from 'gsap';

// Animate camera position
gsap.to(camera.position, {
  x: 5, y: 3, z: 8,
  duration: 2,
  ease: 'power2.inOut',
  onUpdate: () => {
    camera.lookAt(target);
    renderer.render(scene, camera);
  }
});
```

**Timeline with multiple keyframes:**

```js
const tl = gsap.timeline({ defaults: { ease: 'power2.inOut' } });

tl.to(camera.position, { x: 0, y: 5, z: 10, duration: 2 })
  .to(camera.position, { x: -3, y: 2, z: 6, duration: 1.5 }, '+=0.5')
  .to(camera.position, { x: 0, y: 1, z: 4, duration: 2 }, '+=0.3');

// In render loop
function animate() {
  requestAnimationFrame(animate);
  renderer.render(scene, camera);
}
```

**GSAP + ScrollTrigger (scroll-driven camera):**

```js
import gsap from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';
gsap.registerPlugin(ScrollTrigger);

// Proxy object pattern — avoids OrbitControls conflicts
const params = { camX: 0, camY: 5, camZ: 10, lookX: 0, lookY: 0, lookZ: 0 };

gsap.timeline({
  scrollTrigger: {
    trigger: '#canvas-wrapper',
    start: 'top top',
    end: '+=3000',
    scrub: 1,
    pin: true
  }
})
.to(params, { camX: 5, camY: 2, camZ: 3, duration: 1 })
.to(params, { lookX: 2, lookY: 1, lookZ: 0, duration: 1 }, '<');

// In render loop
function animate() {
  requestAnimationFrame(animate);
  camera.position.set(params.camX, params.camY, params.camZ);
  camera.lookAt(params.lookX, params.lookY, params.lookZ);
  renderer.render(scene, camera);
}
```

> **Warning:** OrbitControls overwrites camera transforms every frame. If you use GSAP to animate the camera, either disable OrbitControls during the tween or animate `controls.target` instead.

### 1.2 CameraControls Library

`camera-controls` by yomotsu is a drop-in replacement for OrbitControls that supports smooth transitions, dolly, truck, crane, and programmatic animation.

```bash
npm install camera-controls
```

**Setup:**

```js
import * as THREE from 'three';
import CameraControls from 'camera-controls';

CameraControls.install({ THREE });

const clock = new THREE.Clock();
const camera = new THREE.PerspectiveCamera(60, w / h, 0.01, 1000);
const controls = new CameraControls(camera, renderer.domElement);

function animate() {
  requestAnimationFrame(animate);
  const delta = clock.getDelta();
  controls.update(delta);          // drives smooth interpolation
  renderer.render(scene, camera);
}
```

**Cinematic moves:**

```js
// Dolly — move camera closer/farther (physical move, not zoom)
controls.dolly(-5, true);           // true = enable smooth transition
controls.dollyTo(10, true);         // absolute distance

// Truck — pan left/right, up/down
controls.truck(2, 0, true);         // move 2 units right

// Orbit — rotate around target
controls.rotate(
  45 * THREE.MathUtils.DEG2RAD,     // azimuth (horizontal)
  -15 * THREE.MathUtils.DEG2RAD,    // polar (vertical)
  true
);

// Set exact camera position + look-at in one call
controls.setLookAt(
  5, 5, 5,     // camera position
  0, 0, 0,     // look-at target
  true          // smooth transition
);

// Lerp between two camera poses (great for cinematic cuts)
controls.lerpLookAt(
  0, 10, 0,    // position A
  0, 0, 0,     // target A
  5, 3, 8,     // position B
  2, 1, 0,     // target B
  0.5,         // interpolation factor (0–1)
  true
);

// Fit a bounding box in view
const box = new THREE.Box3().setFromObject(myModel);
controls.fitToBox(box, true, { paddingTop: 1, paddingBottom: 1 });
```

**Transition speed:**

```js
controls.smoothTime = 0.5;         // seconds for smooth damping (default 0.25)
controls.draggingSmoothTime = 0.1; // while user is dragging
```

### 1.3 CatmullRomCurve3 Camera Paths

Define a spline path through keyframes, then move the camera along it. The camera can look ahead on the path for natural direction changes.

**Create the path:**

```js
const points = [
  new THREE.Vector3(0, 5, 20),
  new THREE.Vector3(10, 3, 10),
  new THREE.Vector3(15, 4, -5),
  new THREE.Vector3(5, 6, -15),
  new THREE.Vector3(0, 5, -20),
];

const curve = new THREE.CatmullRomCurve3(points, false, 'centripetal', 0.5);

// Optional: visualize the path
const lineGeo = new THREE.BufferGeometry().setFromPoints(curve.getPoints(200));
const line = new THREE.Line(lineGeo, new THREE.LineBasicMaterial({ color: 0xff0000 }));
scene.add(line);
```

**Animate along the path:**

```js
let progress = 0;
const speed = 0.0005;

function animate() {
  requestAnimationFrame(animate);

  progress += speed;
  if (progress > 1) progress = 0;

  // Get current position on curve
  const pos = curve.getPointAt(progress);
  camera.position.copy(pos);

  // Look slightly ahead on the curve for smooth turning
  const lookAhead = Math.min(progress + 0.02, 1);
  const lookAtPos = curve.getPointAt(lookAhead);
  camera.lookAt(lookAtPos);

  renderer.render(scene, camera);
}
```

**Scroll-driven path (0–1 mapped to scroll):**

```js
window.addEventListener('scroll', () => {
  const scrollY = window.scrollY;
  const maxScroll = document.body.scrollHeight - window.innerHeight;
  const t = Math.min(scrollY / maxScroll, 1);

  const pos = curve.getPointAt(t);
  camera.position.copy(pos);

  const lookT = Math.min(t + 0.02, 1);
  camera.lookAt(curve.getPointAt(lookT));
});
```

**GSAP-driven path (timeline scrub):**

```js
const pathState = { t: 0 };

gsap.to(pathState, {
  t: 1,
  duration: 10,
  ease: 'none',
  onUpdate: () => {
    const pos = curve.getPointAt(pathState.t);
    camera.position.copy(pos);
    const lookT = Math.min(pathState.t + 0.02, 1);
    camera.lookAt(curve.getPointAt(lookT));
  }
});
```

**Dual-path pattern (camera + target on separate splines):**

```js
const cameraCurve = new THREE.CatmullRomCurve3(cameraPoints);
const targetCurve = new THREE.CatmullRomCurve3(targetPoints);

function animate(t) {
  camera.position.copy(cameraCurve.getPointAt(t));
  camera.lookAt(targetCurve.getPointAt(t));
}
```

### 1.4 three-story-controls (NYT)

The New York Times' `three-story-controls` library provides a `CameraRig` wrapper for cinematic camera actions (pan, tilt, dolly, pedestal, truck, zoom, roll) plus scroll-based story integration.

```bash
npm install three-story-controls
```

```js
import { CameraRig, ScrollControls } from 'three-story-controls';

const rig = new CameraRig(camera, scene);
const controls = new ScrollControls(rig, {
  scrollElement: document.querySelector('.scroll-container'),
  cameraKeyframes: [
    { position: [0, 5, 10], target: [0, 0, 0] },
    { position: [5, 2, 5],  target: [2, 1, 0] },
    { position: [0, 1, 3],  target: [0, 0, 0] },
  ]
});

function animate() {
  requestAnimationFrame(animate);
  controls.update();
  renderer.render(scene, camera);
}
```

---

## 2. Skeletal Animation

### 2.1 AnimationMixer Fundamentals

The three.js animation system is built on `AnimationClip` (the data), `AnimationMixer` (the player), and `AnimationAction` (a clip bound to the mixer).

**Load and play a glTF animation:**

```js
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';

const loader = new GLTFLoader();
let mixer;

loader.load('character.glb', (gltf) => {
  const model = gltf.scene;
  scene.add(model);

  // One mixer per animated object
  mixer = new THREE.AnimationMixer(model);

  // Clips are in gltf.animations[]
  const idleClip = gltf.animations[0];
  const walkClip = gltf.animations[1];
  const runClip  = gltf.animations[2];

  // Create actions (cached internally — clipAction is idempotent)
  const idleAction = mixer.clipAction(idleClip);
  const walkAction = mixer.clipAction(walkClip);
  const runAction  = mixer.clipAction(runClip);

  idleAction.play();
});

// Update in render loop
const clock = new THREE.Clock();
function animate() {
  requestAnimationFrame(animate);
  const delta = clock.getDelta();
  if (mixer) mixer.update(delta);
  renderer.render(scene, camera);
}
```

### 2.2 Action Controls

```js
const action = mixer.clipAction(clip);

// Playback
action.play();
action.stop();
action.reset();             // rewind to start
action.halt(0.5);           // decelerate to stop over 0.5s

// Speed
action.timeScale = 0.5;             // half speed
action.setEffectiveTimeScale(2.0);  // double speed (respects warping)

// Loop modes
action.loop = THREE.LoopRepeat;     // default — infinite loop
action.loop = THREE.LoopOnce;       // play once
action.loop = THREE.LoopPingPong;   // forward then reverse
action.clampWhenFinished = true;    // hold last frame (with LoopOnce)
action.repetitions = 3;             // loop 3 times then stop

// Weight — influence on the final pose (0.0 = none, 1.0 = full)
action.setEffectiveWeight(0.5);

// Direction
action.timeScale = -1;              // play in reverse
```

### 2.3 Crossfading Between Clips

Crossfading blends one action out while blending another in.

```js
let currentAction = idleAction;

function crossFade(fromAction, toAction, duration = 0.5) {
  toAction.reset();
  toAction.setEffectiveTimeScale(1);
  toAction.setEffectiveWeight(1);
  toAction.play();

  fromAction.crossFadeTo(toAction, duration, true);
  currentAction = toAction;
}

// Usage
crossFade(idleAction, walkAction, 0.4);

// Wait for walk cycle to end, then fade to run
setTimeout(() => crossFade(walkAction, runAction, 0.3), 2000);
```

**Synchronized crossfade (waits for current loop to finish):**

```js
function syncCrossFade(fromAction, toAction, duration) {
  mixer.addEventListener('loop', onLoopFinished);

  function onLoopFinished(e) {
    if (e.action === fromAction) {
      mixer.removeEventListener('loop', onLoopFinished);
      crossFade(fromAction, toAction, duration);
    }
  }
}
```

### 2.4 Additive Blending

Additive blending layers an animation on top of a base pose. Use it for expressions, breathing, head turns, or hit reactions over locomotion.

```js
// Convert a clip to additive
const headTurnClip = THREE.AnimationUtils.makeClipAdditive(
  gltf.animations.find(c => c.name === 'headTurn')
);

// For a pose clip, extract a single frame first
const poseClip = THREE.AnimationUtils.subclip(rawClip, 'pose', 2, 3, 30);
THREE.AnimationUtils.makeClipAdditive(poseClip);

// Create action with additive blend mode
const headTurnAction = mixer.clipAction(headTurnClip);
headTurnAction.blendMode = THREE.AdditiveAnimationBlendMode;
headTurnAction.setEffectiveWeight(0.7);   // 70% influence
headTurnAction.play();

// Base action plays normally
idleAction.play();

// Result: idle + 70% head turn layered on top
```

**Controlling additive weight dynamically:**

```js
function setAdditiveWeight(action, weight) {
  action.setEffectiveWeight(weight);
  // Weight 0 = no influence, 1 = full additive contribution
}

// Example: breathe faster when running
const breatheAction = mixer.clipAction(breatheClip);
breatheAction.blendMode = THREE.AdditiveAnimationBlendMode;
breatheAction.play();

// Ramp up breathing with speed
function updateBreathing(speed) {
  const weight = THREE.MathUtils.clamp(speed / 10, 0, 1);
  breatheAction.setEffectiveWeight(weight);
  breatheAction.timeScale = 1 + speed * 0.3;
}
```

### 2.5 Multiple Mixers for Multiple Characters

```js
const mixers = [];

function loadCharacter(url, position) {
  loader.load(url, (gltf) => {
    const model = gltf.scene;
    model.position.copy(position);
    scene.add(model);

    const mixer = new THREE.AnimationMixer(model);
    const action = mixer.clipAction(gltf.animations[0]);
    action.play();
    mixers.push(mixer);
  });
}

// Update all mixers
function animate() {
  requestAnimationFrame(animate);
  const delta = clock.getDelta();
  mixers.forEach(m => m.update(delta));
  renderer.render(scene, camera);
}
```

### 2.6 Inverse Kinematics (IK)

Three.js includes `CCDIKSolver` for CCD-based IK. For FABRIK, use `IK-threejs`.

**Built-in CCDIKSolver:**

```js
import { CCDIKSolver } from 'three/addons/animation/CCDIKSolver.js';

const iks = [
  {
    target: targetBoneIndex,
    effector: effectorBoneIndex,
    links: [
      { index: bone3Index },
      { index: bone2Index, limitation: new THREE.Vector3(1, 0, 0) },
      { index: bone1Index },
    ],
    iteration: 10,
    minAngle: 0,
    maxAngle: Math.PI,
  }
];

const solver = new CCDIKSolver(skinnedMesh, iks);

// In render loop
function animate() {
  solver.update();
  renderer.render(scene, camera);
}
```

**IK-threejs (FABRIK + CCD with constraints):**

```bash
npm install ik-threejs   # or import from source
```

```js
import { FABRIKSolver } from 'ik-threejs';

const skeleton = model.skeleton;
const solver = new FABRIKSolver(skeleton);

solver.setConfiguration({
  iterations: 5,
  thresholdTargetSq: 0.0001,
  constraintsEnabler: true,
});

// Create an IK chain from effector to root
const target = new THREE.Object3D();
scene.add(target);

solver.createChain(
  [5, 4, 3, 2, 1, 0],  // bone indices: effector → root
  [
    null,  // effector joint (ignored)
    { type: FABRIKSolver.JOINTTYPES.OMNI, twist: [0, 0.0001] },
    { type: FABRIKSolver.JOINTTYPES.HINGE,
      axis: [1, 0, 0], min: 0, max: Math.PI * 0.5 },
    { type: FABRIKSolver.JOINTTYPES.BALLSOCKET,
      twist: [-Math.PI * 0.25, Math.PI * 0.25],
      polar: [0, Math.PI * 0.5],
      azimuth: [-Math.PI * 0.6, Math.PI * 0.4] },
    null, null
  ],
  target,
  'rightArm'
);

// In render loop
target.position.set(mouseX, mouseY, 0);
solver.update();
```

Joint types: `OMNI` (unconstrained), `HINGE` (single axis), `BALLSOCKET` (polar/azimuth limits).

---

## 3. Morph Targets

### 3.1 Morph Targets in Three.js

Morph targets (blend shapes) deform a mesh by interpolating between its base geometry and one or more target shapes. Each target has an influence value from 0 (base) to 1 (fully morphed).

**Manual morph target setup:**

```js
const geometry = new THREE.BoxGeometry(2, 2, 2, 32, 32, 32);

// Create a morph target by cloning and modifying positions
const positions = geometry.attributes.position.array.slice();
for (let i = 0; i < positions.length; i += 3) {
  positions[i]     *= 1.5;   // scale X
  positions[i + 1] *= 0.5;   // squash Y
}

geometry.morphAttributes.position = [
  new THREE.Float32BufferAttribute(positions, 3)
];

const mesh = new THREE.Mesh(
  geometry,
  new THREE.MeshStandardMaterial({ morphTargets: true })
);

// Set morph influence (0 = cube, 1 = squashed)
mesh.morphTargetInfluences[0] = 0.5;
```

### 3.2 glTF Morph Targets

glTF models from Blender export morph targets as named blend shapes. Three.js maps them to `morphTargetDictionary` and `morphTargetInfluences`.

```js
loader.load('face.glb', (gltf) => {
  const model = gltf.scene;
  scene.add(model);

  // Find the mesh with morph targets
  let faceMesh;
  model.traverse((child) => {
    if (child.isMesh && child.morphTargetDictionary) {
      faceMesh = child;
    }
  });

  // List available morph targets
  console.log(faceMesh.morphTargetDictionary);
  // e.g. { mouthOpen: 0, eyeBlinkLeft: 1, eyeBlinkRight: 2, smile: 3 }

  // Drive morph targets by name
  const dict = faceMesh.morphTargetDictionary;
  faceMesh.morphTargetInfluences[dict['smile']] = 0.8;
  faceMesh.morphTargetInfluences[dict['eyeBlinkLeft']] = 1.0;
});
```

### 3.3 Animating Morph Targets

**With GSAP:**

```js
// Animate a smile from 0 to 1 over 0.5 seconds
gsap.to(faceMesh.morphTargetInfluences, {
  [dict['smile']]: 1.0,
  duration: 0.5,
  ease: 'power2.out'
});

// Blink sequence
const blinkTl = gsap.timeline({ repeat: -1, repeatDelay: 3 });
blinkTl.to(faceMesh.morphTargetInfluences, {
  [dict['eyeBlinkLeft']]: 1,
  [dict['eyeBlinkRight']]: 1,
  duration: 0.1,
}).to(faceMesh.morphTargetInfluences, {
  [dict['eyeBlinkLeft']]: 0,
  [dict['eyeBlinkRight']]: 0,
  duration: 0.1,
});
```

**With AnimationMixer (clip-based):**

```js
// glTF files can embed morph target animations
// They show up in gltf.animations just like skeletal clips
loader.load('face_animated.glb', (gltf) => {
  const model = gltf.scene;
  scene.add(model);

  const mixer = new THREE.AnimationMixer(model);

  // Morph target animation clips
  gltf.animations.forEach((clip) => {
    console.log(clip.name, clip.tracks.length);
    // Tracks named like: "faceMesh.morphTargetInfluences[smile]"
  });

  const talkAction = mixer.clipAction(gltf.animations[0]);
  talkAction.play();
});
```

**With KeyframeTrack (procedural clip):**

```js
// Create a morph target animation clip from scratch
const smileTrack = new THREE.NumberKeyframeTrack(
  'faceMesh.morphTargetInfluences[smile]',
  [0, 0.5, 1.0, 1.5],      // times
  [0, 1.0, 1.0, 0]          // values
);

const blinkTrack = new THREE.NumberKeyframeTrack(
  'faceMesh.morphTargetInfluences[eyeBlinkLeft]',
  [0, 0.8, 0.9, 1.0],
  [0, 0, 1, 0]
);

const clip = new THREE.AnimationClip('expression', 1.5, [smileTrack, blinkTrack]);
const action = mixer.clipAction(clip);
action.play();
```

### 3.4 Face Animation with Blend Shapes

Common ARKit-style blend shapes for facial animation (52 shapes). If your glTF model follows Apple ARKit naming:

```js
// Map face tracking data to morph targets
function applyFaceData(faceMesh, blendShapes) {
  const dict = faceMesh.morphTargetDictionary;
  for (const [name, value] of Object.entries(blendShapes)) {
    if (dict[name] !== undefined) {
      faceMesh.morphTargetInfluences[dict[name]] = value;
    }
  }
}

// Example: Apply from a face tracking library
applyFaceData(faceMesh, {
  jawOpen: 0.3,
  mouthSmileLeft: 0.7,
  mouthSmileRight: 0.7,
  eyeBlinkLeft: 0.0,
  eyeBlinkRight: 0.0,
  browInnerUp: 0.2,
});
```

---

## 4. Procedural Animation

### 4.1 Spring Physics

A damped spring creates natural secondary motion — use it for follow cameras, bouncy UI, character accessories, and anything that needs to settle smoothly.

**Simple spring class:**

```js
class Spring {
  constructor(stiffness = 180, damping = 12, mass = 1) {
    this.stiffness = stiffness;
    this.damping = damping;
    this.mass = mass;
    this.value = 0;
    this.velocity = 0;
    this.target = 0;
  }

  update(dt) {
    const force = -this.stiffness * (this.value - this.target);
    const dampingForce = -this.damping * this.velocity;
    const acceleration = (force + dampingForce) / this.mass;
    this.velocity += acceleration * dt;
    this.value += this.velocity * dt;
    return this.value;
  }
}

// Usage: bouncy follow camera
const springY = new Spring(100, 8, 1);

function animate() {
  requestAnimationFrame(animate);
  const dt = clock.getDelta();

  springY.target = character.position.y + 5;
  camera.position.y = springY.update(dt);

  renderer.render(scene, camera);
}
```

**Spring3D for Vector3 properties:**

```js
class Spring3D {
  constructor(stiffness = 180, damping = 12, mass = 1) {
    this.stiffness = stiffness;
    this.damping = damping;
    this.mass = mass;
    this.value = new THREE.Vector3();
    this.velocity = new THREE.Vector3();
    this.target = new THREE.Vector3();
    this._force = new THREE.Vector3();
  }

  update(dt) {
    this._force
      .copy(this.value)
      .sub(this.target)
      .multiplyScalar(-this.stiffness)
      .addScaledVector(this.velocity, -this.damping);

    this.velocity.addScaledVector(this._force, dt / this.mass);
    this.value.addScaledVector(this.velocity, dt);
    return this.value;
  }
}

// Usage: antenna/hair wobble on a character
const antennaSpring = new Spring3D(200, 6, 0.5);
antennaSpring.target.set(0, 2, 0);   // rest position

function animate() {
  // Jolt the antenna when character jumps
  if (justJumped) antennaSpring.velocity.set(0, 5, 0);

  const tip = antennaSpring.update(clock.getDelta());
  antennaBone.position.copy(tip);
}
```

### 4.2 Verlet Integration

Verlet stores current and previous positions instead of velocities. This makes constraint solving (ropes, cloth, ragdolls) trivial.

**Core Verlet particle:**

```js
class VerletParticle {
  constructor(x, y, z, pinned = false) {
    this.pos = new THREE.Vector3(x, y, z);
    this.prev = new THREE.Vector3(x, y, z);
    this.pinned = pinned;
  }

  update(gravity, damping = 0.99) {
    if (this.pinned) return;

    const vel = new THREE.Vector3().subVectors(this.pos, this.prev);
    vel.multiplyScalar(damping);

    this.prev.copy(this.pos);
    this.pos.add(vel);
    this.pos.y -= gravity;    // simple gravity
  }
}
```

**Distance constraint (stick):**

```js
class Constraint {
  constructor(p1, p2, restLength = null) {
    this.p1 = p1;
    this.p2 = p2;
    this.restLength = restLength ??
      p1.pos.distanceTo(p2.pos);
  }

  solve() {
    const diff = new THREE.Vector3().subVectors(this.p2.pos, this.p1.pos);
    const dist = diff.length();
    const correction = (dist - this.restLength) / dist * 0.5;
    const offset = diff.multiplyScalar(correction);

    if (!this.p1.pinned) this.p1.pos.add(offset);
    if (!this.p2.pinned) this.p2.pos.sub(offset);
  }
}
```

### 4.3 Cloth Simulation

A cloth grid is a 2D array of Verlet particles connected by distance constraints.

```js
class Cloth {
  constructor(width, height, segments, restDist) {
    this.particles = [];
    this.constraints = [];
    const w = segments;
    const h = segments;

    // Create particles
    for (let y = 0; y <= h; y++) {
      for (let x = 0; x <= w; x++) {
        const px = (x / w - 0.5) * width;
        const py = 3;   // hang from top
        const pz = (y / h - 0.5) * height;
        const pinned = (y === 0);  // pin top row
        this.particles.push(new VerletParticle(px, py, pz, pinned));
      }
    }

    // Structural constraints (horizontal + vertical)
    for (let y = 0; y <= h; y++) {
      for (let x = 0; x <= w; x++) {
        const i = y * (w + 1) + x;
        if (x < w) this.constraints.push(
          new Constraint(this.particles[i], this.particles[i + 1], restDist)
        );
        if (y < h) this.constraints.push(
          new Constraint(this.particles[i], this.particles[i + w + 1], restDist)
        );
      }
    }

    // Shear constraints (diagonal) for stability
    for (let y = 0; y < h; y++) {
      for (let x = 0; x < w; x++) {
        const i = y * (w + 1) + x;
        this.constraints.push(
          new Constraint(this.particles[i], this.particles[i + w + 2], restDist * 1.414)
        );
        this.constraints.push(
          new Constraint(this.particles[i + 1], this.particles[i + w + 1], restDist * 1.414)
        );
      }
    }
  }

  update(gravity = 0.01, iterations = 5) {
    // Update particles
    this.particles.forEach(p => p.update(gravity));

    // Solve constraints multiple times for stiffness
    for (let i = 0; i < iterations; i++) {
      this.constraints.forEach(c => c.solve());
    }
  }
}

// Sync to Three.js mesh
function syncClothToMesh(cloth, mesh, segmentsW) {
  const pos = mesh.geometry.attributes.position;
  cloth.particles.forEach((p, i) => {
    pos.setXYZ(i, p.pos.x, p.pos.y, p.pos.z);
  });
  pos.needsUpdate = true;
  mesh.geometry.computeVertexNormals();
}
```

### 4.4 Simple Soft Body

A soft body wraps Verlet particles around a 3D mesh's vertices.

```js
function createSoftBody(geometry) {
  const positions = geometry.attributes.position;
  const particles = [];
  const constraints = [];

  // Create particle per vertex
  for (let i = 0; i < positions.count; i++) {
    particles.push(new VerletParticle(
      positions.getX(i), positions.getY(i), positions.getZ(i)
    ));
  }

  // Create constraints from edges (use geometry index)
  const index = geometry.index.array;
  const edgeSet = new Set();
  for (let i = 0; i < index.length; i += 3) {
    const edges = [
      [index[i], index[i + 1]],
      [index[i + 1], index[i + 2]],
      [index[i + 2], index[i]],
    ];
    edges.forEach(([a, b]) => {
      const key = `${Math.min(a, b)}-${Math.max(a, b)}`;
      if (!edgeSet.has(key)) {
        edgeSet.add(key);
        constraints.push(new Constraint(particles[a], particles[b]));
      }
    });
  }

  return { particles, constraints };
}
```

### 4.5 Procedural Bone Chain (Tail/Hair)

Use inverse follow — each bone follows the one before it:

```js
function updateBoneChain(bones, dt, drag = 0.95) {
  // bones[0] is the root (driven by parent)
  for (let i = 1; i < bones.length; i++) {
    const bone = bones[i];
    const parent = bones[i - 1];

    // Store velocity
    if (!bone.userData.vel) bone.userData.vel = new THREE.Vector3();
    if (!bone.userData.prevPos) bone.userData.prevPos = bone.position.clone();

    const vel = bone.userData.vel;
    const prev = bone.userData.prevPos;

    // Verlet-style: infer velocity from position delta
    vel.subVectors(bone.position, prev).multiplyScalar(drag);
    prev.copy(bone.position);

    // Gravity
    vel.y -= 0.002;

    // Apply
    bone.position.add(vel);

    // Constrain distance to parent
    const diff = new THREE.Vector3().subVectors(bone.position, parent.position);
    const len = diff.length();
    const rest = bone.userData.restLength || 0.5;
    if (len > rest) {
      diff.multiplyScalar(rest / len);
      bone.position.copy(parent.position).add(diff);
    }
  }
}
```

---

## 5. Scene Optimization

### 5.1 Draw Call Monitoring

```js
// Check draw calls each frame
function logPerf() {
  const info = renderer.info;
  console.log({
    drawCalls: info.render.calls,
    triangles: info.render.triangles,
    geometries: info.memory.geometries,
    textures: info.memory.textures,
  });
}
// Target: < 100 draw calls for smooth 60fps
```

### 5.2 InstancedMesh for Repeated Objects

One draw call for thousands of identical objects (trees, particles, props).

```js
const geometry = new THREE.BoxGeometry(1, 1, 1);
const material = new THREE.MeshStandardMaterial({ color: 0x44aa88 });
const count = 1000;

const mesh = new THREE.InstancedMesh(geometry, material, count);
const matrix = new THREE.Matrix4();
const color = new THREE.Color();

for (let i = 0; i < count; i++) {
  matrix.setPosition(
    Math.random() * 100 - 50,
    Math.random() * 5,
    Math.random() * 100 - 50
  );
  mesh.setMatrixAt(i, matrix);

  // Optional: per-instance color
  color.setHSL(Math.random(), 0.7, 0.5);
  mesh.setColorAt(i, color);
}

mesh.instanceMatrix.needsUpdate = true;
mesh.instanceColor.needsUpdate = true;
scene.add(mesh);
```

### 5.3 BatchedMesh for Varied Geometry

When objects share a material but have different geometries, `BatchedMesh` batches them into one draw call.

```js
const batchedMesh = new THREE.BatchedMesh(
  100,       // max instances
  50000,     // max vertex count
  100000,    // max index count
  material
);

const geoId1 = batchedMesh.addGeometry(boxGeo);
const geoId2 = batchedMesh.addGeometry(sphereGeo);

// Add instances referencing different geometries
for (let i = 0; i < 50; i++) {
  const instanceId = batchedMesh.addInstance(i % 2 === 0 ? geoId1 : geoId2);
  const matrix = new THREE.Matrix4().setPosition(i * 2, 0, 0);
  batchedMesh.setMatrixAt(instanceId, matrix);
}

scene.add(batchedMesh);
```

### 5.4 Geometry Merging (Static Scenes)

For static geometry that never moves, merge into a single buffer.

```js
import { mergeGeometries } from 'three/addons/utils/BufferGeometryUtils.js';

const geometries = [];
staticMeshes.forEach(mesh => {
  const geo = mesh.geometry.clone();
  geo.applyMatrix4(mesh.matrixWorld);
  geometries.push(geo);
});

const merged = mergeGeometries(geometries);
const singleMesh = new THREE.Mesh(merged, sharedMaterial);
scene.add(singleMesh);
// Result: 1 draw call instead of N
```

### 5.5 Material Sharing

Every unique material = a separate draw call. Share materials aggressively.

```js
// BAD: new material per mesh
meshes.forEach(m => {
  m.material = new THREE.MeshStandardMaterial({ color: 0xff0000 }); // N materials
});

// GOOD: single shared material
const sharedMat = new THREE.MeshStandardMaterial({ color: 0xff0000 });
meshes.forEach(m => { m.material = sharedMat; }); // 1 material
```

### 5.6 Level of Detail (LOD)

```js
const lod = new THREE.LOD();

// High detail: < 25 units away
lod.addLevel(highPolyMesh, 0);
// Medium: 25–75 units
lod.addLevel(medPolyMesh, 25);
// Low: > 75 units
lod.addLevel(lowPolyMesh, 75);
// Optional: hide entirely beyond distance
lod.addLevel(new THREE.Object3D(), 200);

scene.add(lod);

// Update in render loop
function animate() {
  lod.update(camera);   // recalculates which level to show
  renderer.render(scene, camera);
}
```

### 5.7 Frustum Culling

Three.js automatically culls objects outside the camera frustum. Keep it enabled (default) and ensure bounding volumes are accurate.

```js
mesh.frustumCulled = true;          // default — leave enabled
skybox.frustumCulled = false;       // disable for skyboxes / full-screen quads

// Recompute bounding sphere after geometry changes
geometry.computeBoundingSphere();

// For InstancedMesh, you must set the bounding sphere/box manually
mesh.computeBoundingSphere();
```

### 5.8 three-mesh-bvh (Spatial Queries)

Accelerates raycasting by 100x+ on complex geometry. Also enables spatial queries (frustum tests, closest point, shape casting).

```bash
npm install three-mesh-bvh
```

**Setup:**

```js
import {
  computeBoundsTree, disposeBoundsTree, acceleratedRaycast
} from 'three-mesh-bvh';

// Patch Three.js prototypes (once at startup)
THREE.BufferGeometry.prototype.computeBoundsTree = computeBoundsTree;
THREE.BufferGeometry.prototype.disposeBoundsTree = disposeBoundsTree;
THREE.Mesh.prototype.raycast = acceleratedRaycast;

// Build BVH for a geometry
const geometry = model.geometry;
geometry.computeBoundsTree();

// Raycasting is now orders of magnitude faster
const raycaster = new THREE.Raycaster();
raycaster.firstHitOnly = true;   // use raycastFirst internally — much faster
const hits = raycaster.intersectObjects([model]);
```

**Async BVH generation (non-blocking):**

```js
import { GenerateMeshBVHWorker } from 'three-mesh-bvh/worker';

const worker = new GenerateMeshBVHWorker();
worker.generate(geometry).then(bvh => {
  geometry.boundsTree = bvh;
  console.log('BVH ready');
});
```

**Direct BVH spatial queries:**

```js
const bvh = geometry.boundsTree;
const invMat = new THREE.Matrix4().copy(mesh.matrixWorld).invert();

// Sphere intersection test
const sphere = new THREE.Sphere(new THREE.Vector3(0, 1, 0), 5);
sphere.applyMatrix4(invMat);
const intersects = bvh.intersectsSphere(sphere);

// Serialization (cache BVH to avoid rebuild)
const serialized = MeshBVH.serialize(bvh);
// Later:
geometry.boundsTree = MeshBVH.deserialize(serialized, geometry);
```

### 5.9 Dispose Pattern

Prevent GPU memory leaks by disposing resources when removing objects.

```js
function dispose(object) {
  object.traverse(child => {
    if (child.isMesh) {
      child.geometry.dispose();

      const materials = Array.isArray(child.material)
        ? child.material
        : [child.material];

      materials.forEach(mat => {
        // Dispose all texture properties
        for (const key of Object.keys(mat)) {
          const value = mat[key];
          if (value && value.isTexture) {
            value.dispose();
            // For ImageBitmap textures (common in glTF)
            if (value.source?.data?.close) value.source.data.close();
          }
        }
        mat.dispose();
      });
    }
  });

  object.parent?.remove(object);
}

// Monitor memory
console.log(renderer.info.memory);
// { geometries: 12, textures: 8 }
```

### 5.10 Texture Optimization

```js
// Use power-of-two textures for mipmapping
// 512x512, 1024x1024, 2048x2048

// Resize oversized textures
texture.minFilter = THREE.LinearMipmapLinearFilter;
texture.generateMipmaps = true;

// For UI/sprites that don't need mipmaps
texture.minFilter = THREE.LinearFilter;
texture.generateMipmaps = false;

// Flip-Y for non-glTF textures
texture.flipY = true;   // default; glTF sets this to false

// Compress at load time with KTX2 (see Section 8)
```

---

## 6. Lighting

### 6.1 Environment Maps with PMREMGenerator

Environment maps provide realistic reflections and ambient lighting. Use `.hdr` or `.exr` files processed through PMREMGenerator.

```js
import { RGBELoader } from 'three/addons/loaders/RGBELoader.js';

const pmremGenerator = new THREE.PMREMGenerator(renderer);
pmremGenerator.compileEquirectangularShader();

new RGBELoader().load('studio_small.hdr', (texture) => {
  const envMap = pmremGenerator.fromEquirectangular(texture).texture;

  scene.environment = envMap;      // affects all PBR materials
  scene.background = envMap;       // optional: use as skybox too

  texture.dispose();
  pmremGenerator.dispose();
});
```

**EXR format (higher precision):**

```js
import { EXRLoader } from 'three/addons/loaders/EXRLoader.js';

new EXRLoader().load('studio.exr', (texture) => {
  texture.mapping = THREE.EquirectangularReflectionMapping;
  const envMap = pmremGenerator.fromEquirectangular(texture).texture;
  scene.environment = envMap;
  texture.dispose();
});
```

**Environment-only lighting (no visible background):**

```js
scene.environment = envMap;           // lights all PBR materials
scene.background = new THREE.Color(0x000000);  // black background
// or scene.backgroundIntensity = 0;
```

### 6.2 RoomEnvironment (No HDRI File)

Generate a neutral studio-like environment map entirely in code.

```js
import { RoomEnvironment } from 'three/addons/environments/RoomEnvironment.js';

const pmremGenerator = new THREE.PMREMGenerator(renderer);
scene.environment = pmremGenerator.fromScene(
  new RoomEnvironment(), 0.04
).texture;
pmremGenerator.dispose();
```

### 6.3 LightProbe

Light probes sample incoming radiance at a point. Useful for static GI approximation.

```js
import { LightProbe } from 'three';
import { LightProbeGenerator } from 'three/addons/lights/LightProbeGenerator.js';

// Generate from a cube texture
const cubeRenderTarget = new THREE.WebGLCubeRenderTarget(256);
const cubeCamera = new THREE.CubeCamera(0.1, 1000, cubeRenderTarget);
cubeCamera.position.set(0, 1, 0);
cubeCamera.update(renderer, scene);

const probe = LightProbeGenerator.fromCubeRenderTarget(renderer, cubeRenderTarget);
scene.add(probe);
probe.intensity = 1.0;
```

**From an environment map:**

```js
// After loading HDRI
const probe = LightProbeGenerator.fromCubeTexture(cubeTexture);
scene.add(probe);
```

### 6.4 RectAreaLight

Simulates area light sources (TV screens, windows, studio softboxes). Only works with `MeshStandardMaterial` and `MeshPhysicalMaterial`.

```js
import { RectAreaLightHelper } from 'three/addons/helpers/RectAreaLightHelper.js';
import { RectAreaLightUniformsLib } from 'three/addons/lights/RectAreaLightUniformsLib.js';

// REQUIRED: initialize uniforms before creating any RectAreaLight
RectAreaLightUniformsLib.init();

const rectLight = new THREE.RectAreaLight(0xffffff, 5, 4, 2);
rectLight.position.set(0, 3, -2);
rectLight.lookAt(0, 0, 0);
scene.add(rectLight);

// Optional: helper to visualize the light
const helper = new RectAreaLightHelper(rectLight);
rectLight.add(helper);
```

> For WebGPURenderer, use `RectAreaLightTexturesLib` instead of `RectAreaLightUniformsLib`.

### 6.5 HDR Tone Mapping

```js
// Enable tone mapping for HDR → LDR conversion
renderer.toneMapping = THREE.ACESFilmicToneMapping;  // cinematic
renderer.toneMappingExposure = 1.0;

// Other options:
// THREE.LinearToneMapping        — no adjustment
// THREE.ReinhardToneMapping      — soft rolloff
// THREE.CineonToneMapping        — film-like
// THREE.AgXToneMapping           — wide-gamut (newer)
// THREE.NeutralToneMapping       — balanced (newest)
```

### 6.6 Light Baking Workflow (Blender → Three.js)

Baked lightmaps eliminate runtime lighting cost. Workflow:

1. **In Blender:** Create a second UV channel ("Lightmap Pack" unwrap)
2. **Bake:** Render > Bake > Bake Type: Diffuse (or Combined)
3. **Export:** Include the lightmap texture in glTF export
4. **In Three.js:** Apply to `material.lightMap` using UV2

```js
// Three.js uses UV2 (uv2 attribute) for lightmaps
loader.load('scene_baked.glb', (gltf) => {
  gltf.scene.traverse(child => {
    if (child.isMesh) {
      // If the model has a second UV set, Three.js maps it automatically
      // For manual lightmap application:
      const lightmapTexture = new THREE.TextureLoader().load('lightmap.png');
      child.material.lightMap = lightmapTexture;
      child.material.lightMapIntensity = 1.0;
    }
  });
  scene.add(gltf.scene);
});
```

**Important:** Lightmaps and AO maps both require a second UV set. If your geometry only has one UV, copy it:

```js
geometry.setAttribute('uv2', geometry.attributes.uv.clone());
```

### 6.7 Three-Point Lighting Setup

Classic production lighting for product shots and character presentation:

```js
// Key light — main directional light
const keyLight = new THREE.DirectionalLight(0xfff5e1, 2.0);
keyLight.position.set(5, 8, 5);
keyLight.castShadow = true;
scene.add(keyLight);

// Fill light — softer, opposite side
const fillLight = new THREE.DirectionalLight(0xc4d4ff, 0.6);
fillLight.position.set(-5, 4, 3);
scene.add(fillLight);

// Rim / back light — edge highlight
const rimLight = new THREE.DirectionalLight(0xffffff, 1.0);
rimLight.position.set(0, 4, -6);
scene.add(rimLight);

// Ambient — baseline fill so nothing is pure black
const ambient = new THREE.AmbientLight(0x404040, 0.3);
scene.add(ambient);
```

---

## 7. Shadows

### 7.1 Basic Shadow Setup

```js
// 1. Enable on renderer
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
// Options: BasicShadowMap, PCFShadowMap, PCFSoftShadowMap, VSMShadowMap

// 2. Enable on light
const light = new THREE.DirectionalLight(0xffffff, 1);
light.castShadow = true;
light.shadow.mapSize.width = 2048;
light.shadow.mapSize.height = 2048;

// 3. Configure shadow camera frustum (directional light)
light.shadow.camera.near = 0.5;
light.shadow.camera.far = 50;
light.shadow.camera.left = -15;
light.shadow.camera.right = 15;
light.shadow.camera.top = 15;
light.shadow.camera.bottom = -15;

// 4. Fix shadow acne
light.shadow.bias = -0.0005;
light.shadow.normalBias = 0.02;

// 5. Set on objects
mesh.castShadow = true;
ground.receiveShadow = true;

scene.add(light);
```

### 7.2 Variance Shadow Maps (VSM)

VSM produces softer shadows with less aliasing. Trade-off: light bleeding on thin geometry.

```js
renderer.shadowMap.type = THREE.VSMShadowMap;

const light = new THREE.DirectionalLight(0xffffff, 1);
light.castShadow = true;
light.shadow.mapSize.set(2048, 2048);
light.shadow.blurSamples = 25;    // VSM-specific: blur quality
light.shadow.radius = 4;          // VSM-specific: blur radius
```

### 7.3 Cascaded Shadow Maps (three-csm)

CSM uses multiple shadow cascades — high resolution near the camera, lower resolution far away. Essential for large outdoor scenes.

```bash
npm install three-csm
```

```js
import CSM from 'three-csm';

renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;

const csm = new CSM({
  camera: camera,
  parent: scene,
  cascades: 4,                          // number of cascade levels
  maxFar: camera.far,                   // shadow draw distance
  mode: 'practical',                    // split scheme (best default)
  shadowMapSize: 2048,                  // resolution per cascade
  lightDirection: new THREE.Vector3(1, -1, 1).normalize(),
  shadowBias: -0.0001,
  shadowNormalBias: 0.02,
  lightIntensity: 1.0,
  fade: true,                           // smooth transitions between cascades
});

// IMPORTANT: call setupMaterial on every material that receives CSM shadows
const material = new THREE.MeshStandardMaterial({ color: 0x44aa88 });
csm.setupMaterial(material);

const mesh = new THREE.Mesh(geometry, material);
mesh.castShadow = true;
mesh.receiveShadow = true;
scene.add(mesh);

// Update every frame
function animate() {
  requestAnimationFrame(animate);
  csm.update();
  renderer.render(scene, camera);
}

// On window resize, call:
csm.updateFrustums();

// Cleanup
csm.dispose();
```

**CSM configuration options:**

| Option | Default | Description |
|--------|---------|-------------|
| `cascades` | 4 | Number of shadow cascade levels |
| `mode` | `'practical'` | Split scheme: `uniform`, `logarithmic`, `practical`, `custom` |
| `shadowMapSize` | 2048 | Pixels per cascade shadow map |
| `maxFar` | `camera.far` | Maximum shadow distance |
| `fade` | false | Smooth cascade transitions |
| `lightMargin` | 200 | Z-axis headroom for shadow camera |
| `noLastCascadeCutOff` | false | Extend last cascade to infinity |

### 7.4 Contact Shadows

Soft shadows from objects near a surface, rendered as a post-process to a texture. No shadow map cost, works with any light type.

```js
import { HorizontalBlurShader } from 'three/addons/shaders/HorizontalBlurShader.js';
import { VerticalBlurShader } from 'three/addons/shaders/VerticalBlurShader.js';

// 1. Render targets
const shadowRT = new THREE.WebGLRenderTarget(512, 512);
shadowRT.texture.generateMipmaps = false;
const blurRT = new THREE.WebGLRenderTarget(512, 512);
blurRT.texture.generateMipmaps = false;

// 2. Shadow camera (orthographic, looking down)
const PLANE_SIZE = 5;
const CAMERA_HEIGHT = 0.5;
const shadowCamera = new THREE.OrthographicCamera(
  -PLANE_SIZE / 2, PLANE_SIZE / 2,
  PLANE_SIZE / 2, -PLANE_SIZE / 2,
  0, CAMERA_HEIGHT
);
shadowCamera.rotation.x = Math.PI / 2;

// 3. Depth material (converts depth to shadow)
const depthMaterial = new THREE.MeshDepthMaterial();
depthMaterial.userData.darkness = { value: 1.5 };
depthMaterial.onBeforeCompile = (shader) => {
  shader.uniforms.darkness = depthMaterial.userData.darkness;
  shader.fragmentShader = `
    uniform float darkness;
    ${shader.fragmentShader.replace(
      'gl_FragColor = vec4( vec3( 1.0 - fragCoordZ ), opacity );',
      'gl_FragColor = vec4( vec3( 0.0 ), ( 1.0 - fragCoordZ ) * darkness );'
    )}`;
};
depthMaterial.depthTest = false;
depthMaterial.depthWrite = false;

// 4. Shadow plane (shows the shadow texture)
const shadowPlane = new THREE.Mesh(
  new THREE.PlaneGeometry(PLANE_SIZE, PLANE_SIZE).rotateX(Math.PI / 2),
  new THREE.MeshBasicMaterial({
    map: shadowRT.texture,
    opacity: 0.8,
    transparent: true,
    depthWrite: false,
  })
);
shadowPlane.renderOrder = 1;
shadowPlane.scale.y = -1;
scene.add(shadowPlane);

// 5. Blur materials
const hBlur = new THREE.ShaderMaterial(HorizontalBlurShader);
const vBlur = new THREE.ShaderMaterial(VerticalBlurShader);
hBlur.depthTest = false;
vBlur.depthTest = false;

const blurPlane = new THREE.Mesh(
  shadowPlane.geometry, hBlur
);
blurPlane.visible = false;
scene.add(blurPlane);

// 6. Render pipeline (call each frame before main render)
function renderContactShadow(blurAmount = 3.5) {
  const savedBg = scene.background;
  scene.background = null;
  scene.overrideMaterial = depthMaterial;

  const savedAlpha = renderer.getClearAlpha();
  renderer.setClearAlpha(0);
  renderer.setRenderTarget(shadowRT);
  renderer.render(scene, shadowCamera);

  scene.overrideMaterial = null;

  // Two-pass blur
  blurPlane.visible = true;

  blurPlane.material = hBlur;
  hBlur.uniforms.tDiffuse.value = shadowRT.texture;
  hBlur.uniforms.h.value = blurAmount / 256;
  renderer.setRenderTarget(blurRT);
  renderer.render(blurPlane, shadowCamera);

  blurPlane.material = vBlur;
  vBlur.uniforms.tDiffuse.value = blurRT.texture;
  vBlur.uniforms.v.value = blurAmount / 256;
  renderer.setRenderTarget(shadowRT);
  renderer.render(blurPlane, shadowCamera);

  blurPlane.visible = false;

  // Restore
  renderer.setRenderTarget(null);
  renderer.setClearAlpha(savedAlpha);
  scene.background = savedBg;
}
```

### 7.5 Baked Shadow Textures

For static objects, bake shadows as textures. Zero runtime cost.

```js
// Load a pre-baked shadow texture
const shadowTexture = new THREE.TextureLoader().load('baked-shadow.png');

const shadowPlane = new THREE.Mesh(
  new THREE.PlaneGeometry(3, 3),
  new THREE.MeshBasicMaterial({
    map: shadowTexture,
    transparent: true,
    depthWrite: false,
    opacity: 0.7,
  })
);
shadowPlane.rotation.x = -Math.PI / 2;
shadowPlane.position.y = 0.01;  // just above ground to avoid z-fighting
scene.add(shadowPlane);
```

### 7.6 Shadow Performance Tuning

```js
// 1. Limit shadow-casting lights to 2–3 max
// PointLight shadows render 6 faces — very expensive!
pointLight.castShadow = false;  // prefer DirectionalLight

// 2. Disable auto-update for static scenes
renderer.shadowMap.autoUpdate = false;
renderer.shadowMap.needsUpdate = true;  // trigger manual update once

// 3. Reduce shadow map resolution for distant lights
fillLight.shadow.mapSize.set(512, 512);  // doesn't need 2048

// 4. Tighten shadow frustum
light.shadow.camera.left = -10;    // don't use -100 if your scene is 10 units
light.shadow.camera.right = 10;
// Tighter frustum = more shadow map pixels per unit = sharper shadows

// 5. Use a helper to visualize shadow camera
const helper = new THREE.CameraHelper(light.shadow.camera);
scene.add(helper);

// 6. Use CSM for large scenes instead of one big shadow map
```

---

## 8. Loading

### 8.1 GLTFLoader (Baseline)

```js
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';

const loader = new GLTFLoader();

loader.load(
  'model.glb',
  (gltf) => {
    scene.add(gltf.scene);
    // Animations: gltf.animations[]
    // Cameras:    gltf.cameras[]
    // User data:  gltf.userData
  },
  (progress) => {
    const pct = (progress.loaded / progress.total * 100).toFixed(0);
    console.log(`Loading: ${pct}%`);
  },
  (error) => console.error('Load failed:', error)
);
```

### 8.2 DRACOLoader (Geometry Compression)

Draco compresses mesh geometry (vertices, normals, UVs) by 60–90%. Requires a WASM decoder.

```js
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import { DRACOLoader } from 'three/addons/loaders/DRACOLoader.js';

const dracoLoader = new DRACOLoader();
dracoLoader.setDecoderPath('https://www.gstatic.com/draco/versioned/decoders/1.5.7/');
// Or local: dracoLoader.setDecoderPath('/draco/');

const gltfLoader = new GLTFLoader();
gltfLoader.setDRACOLoader(dracoLoader);

gltfLoader.load('model-draco.glb', (gltf) => {
  scene.add(gltf.scene);
});
```

**Creating Draco-compressed glTF:**

```bash
# Using gltf-transform CLI
npx gltf-transform draco input.glb output-draco.glb
```

### 8.3 KTX2Loader (GPU Texture Compression)

KTX2 provides GPU-native texture compression (BC7, ASTC, ETC2). Textures stay compressed in GPU memory — massive VRAM savings.

```js
import { KTX2Loader } from 'three/addons/loaders/KTX2Loader.js';

const ktx2Loader = new KTX2Loader();
ktx2Loader.setTranscoderPath('https://cdn.jsdelivr.net/npm/three/examples/jsm/libs/basis/');
ktx2Loader.detectSupport(renderer);

// Use with GLTFLoader for KTX2-compressed glTF textures
const gltfLoader = new GLTFLoader();
gltfLoader.setKTX2Loader(ktx2Loader);
gltfLoader.setDRACOLoader(dracoLoader);   // can use both

gltfLoader.load('model-ktx2.glb', (gltf) => {
  scene.add(gltf.scene);
});
```

**Standalone KTX2 texture loading:**

```js
ktx2Loader.load('texture.ktx2', (texture) => {
  material.map = texture;
  material.needsUpdate = true;
});
```

**Creating KTX2 textures:**

```bash
# Using gltf-transform CLI
npx gltf-transform ktx input.glb output-ktx2.glb --slots "baseColor,normal,emissive"

# Using toktx directly (part of KTX-Software)
toktx --t2 --encode uastc --uastc_quality 2 output.ktx2 input.png
```

### 8.4 Full Optimized Loader Setup

```js
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import { DRACOLoader } from 'three/addons/loaders/DRACOLoader.js';
import { KTX2Loader } from 'three/addons/loaders/KTX2Loader.js';
import { MeshoptDecoder } from 'three/addons/libs/meshopt_decoder.module.js';

// Draco decoder
const dracoLoader = new DRACOLoader();
dracoLoader.setDecoderPath('/draco/');

// KTX2 transcoder
const ktx2Loader = new KTX2Loader();
ktx2Loader.setTranscoderPath('/basis/');
ktx2Loader.detectSupport(renderer);

// GLTFLoader with all decoders
const loader = new GLTFLoader();
loader.setDRACOLoader(dracoLoader);
loader.setKTX2Loader(ktx2Loader);
loader.setMeshoptDecoder(MeshoptDecoder);   // alternative to Draco

loader.load('scene.glb', (gltf) => {
  scene.add(gltf.scene);
});
```

### 8.5 Progressive Loading Pattern

Show something fast, swap in full quality after.

```js
class ProgressiveLoader {
  constructor(scene, gltfLoader) {
    this.scene = scene;
    this.loader = gltfLoader;
  }

  async load(lowResUrl, highResUrl) {
    // Phase 1: Load low-res immediately
    const lowRes = await this.loadModel(lowResUrl);
    this.scene.add(lowRes);

    // Phase 2: Load high-res in background
    const highRes = await this.loadModel(highResUrl);

    // Phase 3: Swap
    this.scene.remove(lowRes);
    dispose(lowRes);   // see Section 5.9
    this.scene.add(highRes);

    return highRes;
  }

  loadModel(url) {
    return new Promise((resolve, reject) => {
      this.loader.load(url, (gltf) => resolve(gltf.scene), null, reject);
    });
  }
}

// Usage
const progressive = new ProgressiveLoader(scene, loader);
progressive.load('model-low.glb', 'model-high.glb');
```

### 8.6 Multi-Asset Preloader with Progress

```js
class AssetPreloader {
  constructor() {
    this.manager = new THREE.LoadingManager();
    this.assets = new Map();
    this.totalBytes = 0;
    this.loadedBytes = 0;

    this.manager.onProgress = (url, loaded, total) => {
      this.onProgress(loaded / total);
    };

    this.manager.onLoad = () => {
      this.onComplete(this.assets);
    };
  }

  onProgress(fraction) {}   // override
  onComplete(assets) {}     // override

  loadGLTF(key, url) {
    const loader = new GLTFLoader(this.manager);
    loader.setDRACOLoader(dracoLoader);
    loader.load(url, (gltf) => {
      this.assets.set(key, gltf);
    });
    return this;
  }

  loadTexture(key, url) {
    const loader = new THREE.TextureLoader(this.manager);
    loader.load(url, (tex) => {
      this.assets.set(key, tex);
    });
    return this;
  }

  loadHDR(key, url) {
    const loader = new RGBELoader(this.manager);
    loader.load(url, (tex) => {
      this.assets.set(key, tex);
    });
    return this;
  }
}

// Usage
const preloader = new AssetPreloader();

preloader.onProgress = (pct) => {
  document.getElementById('progress-bar').style.width = `${pct * 100}%`;
  document.getElementById('progress-text').textContent = `${(pct * 100).toFixed(0)}%`;
};

preloader.onComplete = (assets) => {
  document.getElementById('preloader-overlay').classList.add('fade-out');
  const character = assets.get('character');
  scene.add(character.scene);
};

preloader
  .loadGLTF('character', 'character.glb')
  .loadGLTF('environment', 'env.glb')
  .loadHDR('hdri', 'studio.hdr')
  .loadTexture('logo', 'logo.png');
```

### 8.7 Preloader UI (HTML/CSS)

```html
<!-- Overlay -->
<div id="preloader-overlay">
  <div id="preloader-content">
    <h2>Loading...</h2>
    <div id="progress-track">
      <div id="progress-bar"></div>
    </div>
    <span id="progress-text">0%</span>
  </div>
</div>

<style>
  #preloader-overlay {
    position: fixed;
    inset: 0;
    background: #111;
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 1000;
    transition: opacity 0.5s;
  }
  #preloader-overlay.fade-out {
    opacity: 0;
    pointer-events: none;
  }
  #progress-track {
    width: 300px;
    height: 4px;
    background: #333;
    border-radius: 2px;
    margin-top: 16px;
  }
  #progress-bar {
    width: 0%;
    height: 100%;
    background: #4af;
    border-radius: 2px;
    transition: width 0.2s;
  }
  #progress-text {
    color: #888;
    font-family: monospace;
    margin-top: 8px;
    display: block;
    text-align: center;
  }
</style>
```

### 8.8 gltf-transform CLI Reference

The `gltf-transform` CLI is the standard tool for optimizing glTF files before loading.

```bash
# Install
npm install -g @gltf-transform/cli

# Draco compression
gltf-transform draco input.glb output.glb

# Meshopt compression (alternative to Draco, often better)
gltf-transform meshopt input.glb output.glb

# KTX2 texture compression
gltf-transform ktx input.glb output.glb

# Full optimization pipeline
gltf-transform optimize input.glb output.glb

# Deduplicate materials and textures
gltf-transform dedup input.glb output.glb

# Resize textures (max 1024px)
gltf-transform resize input.glb output.glb --width 1024 --height 1024

# Combine: dedup + resize + draco + ktx
gltf-transform dedup input.glb temp.glb && \
gltf-transform resize temp.glb temp2.glb --width 1024 --height 1024 && \
gltf-transform draco temp2.glb temp3.glb && \
gltf-transform ktx temp3.glb output.glb && \
rm temp.glb temp2.glb temp3.glb

# Inspect a file
gltf-transform inspect input.glb
```

---

## Quick Reference: Key Libraries

| Library | Purpose | Install |
|---------|---------|---------|
| `camera-controls` | Smooth camera transitions, dolly, truck, orbit | `npm i camera-controls` |
| `three-csm` | Cascaded shadow maps for large scenes | `npm i three-csm` |
| `three-mesh-bvh` | BVH for fast raycasting + spatial queries | `npm i three-mesh-bvh` |
| `ik-threejs` | FABRIK + CCD inverse kinematics | `npm i ik-threejs` |
| `gsap` | Animation library (tweens, timelines, scroll) | `npm i gsap` |
| `@gltf-transform/cli` | glTF optimization (Draco, KTX2, meshopt) | `npm i -g @gltf-transform/cli` |
| `three-story-controls` | NYT scroll-driven camera narratives | `npm i three-story-controls` |

## Quick Reference: Performance Targets

| Metric | Target | How to Check |
|--------|--------|--------------|
| Draw calls | < 100 | `renderer.info.render.calls` |
| Triangles | < 500K visible | `renderer.info.render.triangles` |
| Textures in VRAM | < 200MB | `renderer.info.memory.textures` |
| Shadow maps | 2–3 lights max | Count `castShadow = true` lights |
| Frame time | < 16ms (60fps) | `stats-gl` or browser DevTools |

---

*Last updated: 2026-04-22*
