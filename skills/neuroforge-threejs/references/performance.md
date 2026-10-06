# Performance

Smoothness is part of the design. Target: **60fps on a mid-range phone**, no jank on scroll, no layout shift when the canvas mounts.

## Contents

1. Budgets
2. Measure first
3. Asset pipeline
4. Draw calls and geometry
5. Textures and memory
6. Rendering cost
7. Adaptive quality
8. Render on demand and pausing
9. Disposal
10. WebGPU

---

## 1. Budgets

Starting points — adjust to measured devices.

| Metric | Desktop | Mobile |
|---|---|---|
| Draw calls (`renderer.info.render.calls`) | < 100 | < 50 |
| Triangles on screen | < 1M | < 300k |
| Texture memory | < 256 MB | < 128 MB |
| First-view 3D payload (compressed) | 3–5 MB | 2–3 MB |
| Pixel ratio | `min(devicePixelRatio, 2)` | ≤ 1.5 |
| Post-processing passes | ≤ 3 | ≤ 1 (or none) |
| Shadow-casting lights | 1 | 0–1 (prefer baked/contact) |

## 2. Measure first

- `renderer.info` — calls, triangles, geometries, textures. Log it once per second while developing.
- `stats-gl` or `r3f-perf` (R3F) / Tres devtools for fps and GPU time.
- Chrome Performance panel with CPU 4× slowdown; a real mid-range Android over USB.
- Spector.js for what the GPU actually does per frame.

Never optimise blind — name the bottleneck first (CPU: too many objects/draw calls, JS per frame; GPU: fill-rate, overdraw, shader cost, resolution; memory: textures).

## 3. Asset pipeline

Compress every model before it ships:

```bash
# Meshopt geometry + WebP textures, resized — a good default
npx @gltf-transform/cli optimize in.glb out.glb --compress meshopt --texture-compress webp --texture-size 2048

# Draco instead of meshopt
npx @gltf-transform/cli optimize in.glb out.glb --compress draco --texture-compress webp

# KTX2 (GPU-compressed textures — biggest memory win; needs KTX-Software installed)
npx @gltf-transform/cli optimize in.glb out.glb --compress meshopt --texture-compress ktx2
```

R3F: `npx gltfjsx model.glb --transform` compresses and generates a typed JSX component in one step.

Loader setup for compressed assets (vanilla — see `loaders.md` for full detail):

```js
import { GLTFLoader } from "three/addons/loaders/GLTFLoader.js";
import { DRACOLoader } from "three/addons/loaders/DRACOLoader.js";
import { KTX2Loader } from "three/addons/loaders/KTX2Loader.js";
import { MeshoptDecoder } from "three/addons/libs/meshopt_decoder.module.js";

const draco = new DRACOLoader().setDecoderPath("/draco/");
const ktx2 = new KTX2Loader().setTranscoderPath("/basis/").detectSupport(renderer);
const gltf = new GLTFLoader().setDRACOLoader(draco).setKTX2Loader(ktx2).setMeshoptDecoder(MeshoptDecoder);
```

Copy the decoder/transcoder files from `node_modules/three/examples/jsm/libs/{draco,basis}/` into `public/` (or use a pinned CDN) — and note the versions in `project/03-assets.md`.

Other rules:
- Texture resolution matches on-screen size: a 400 px element doesn't need 4K maps.
- HDR environments: 1K–2K `.hdr` (or prefiltered `.ktx2`/`.env`), not 8K.
- Bake lighting/AO into textures for static scenes — the cheapest "premium" there is.

## 4. Draw calls and geometry

- **Instancing** (`InstancedMesh`, drei `<Instances>`, Tres instancing) for anything repeated — one draw call for thousands.
- **Merge** static meshes sharing a material (`BufferGeometryUtils.mergeGeometries`).
- **Share materials and geometries** — never create them per object or per frame.
- **LOD** (`THREE.LOD`, drei `<Detailed>`) for large scenes.
- Particles: `Points` or instanced quads with a shader; animate in the vertex shader, not by updating attributes on the CPU every frame.
- Frustum culling is on by default — don't disable it without reason; set correct bounding spheres on custom/animated geometry.

## 5. Textures and memory

- KTX2/Basis textures stay compressed on the GPU (≈4–8× less memory than PNG/JPG/WebP, which decode to raw RGBA).
- Power-of-two not required in WebGL2, but mipmaps and compression behave best with it.
- Pack channels: AO/roughness/metalness in one ORM texture (glTF does this).
- Reuse render targets; size them to need (half-res for blur/bloom).

## 6. Rendering cost

- **Pixel ratio is the biggest lever** — fill-rate scales with its square. Clamp it.
- `antialias: true` is costly at high DPR; at DPR ≥ 2 you often don't need it; with post-processing use SMAA/FXAA in the composer instead.
- Shadows: one light, tight `shadow.camera` bounds, `shadow.mapSize` 1024 (2048 max), `castShadow` only on what needs it, `shadowMap.autoUpdate = false` + `needsUpdate = true` for static scenes.
- Transparency and transmission are expensive (overdraw, extra passes) — limit them to the hero.
- Post-processing: prefer the `postprocessing` library (pmndrs) which merges effects into fewer passes; keep bloom selective and low-res.
- Shaders: move work from fragment to vertex where possible; avoid branching and loops with dynamic bounds; precompute in JS what's constant per frame.
- Pre-compile to avoid first-interaction hitches: `renderer.compile(scene, camera)` (or `compileAsync`) after loading.

## 7. Adaptive quality

```js
import { getGPUTier } from "detect-gpu";

const { tier, isMobile } = await getGPUTier();
// tier 0–1: static fallback or matcap-only, no post; tier 2: reduced; tier 3: full
```

- R3F: drei `<PerformanceMonitor>` + `<AdaptiveDpr>` to step DPR/effects down when fps drops.
- Decide quality tiers in `direction/02-look-dev.md` — what each tier keeps and drops.

## 8. Render on demand and pausing

- Static scene that only changes on interaction → render on demand: vanilla renders in the controls `change` handler; R3F `frameloop="demand"` + `invalidate()`; Tres `render-mode="on-demand"` + `invalidate()`.
- Pause the loop when the canvas leaves the viewport (`IntersectionObserver`) and when the tab is hidden (`visibilitychange`). Vanilla: `renderer.setAnimationLoop(null)`; R3F: switch `frameloop` to `"never"`; Tres: stop/pause the loop from `useLoop()` (check the installed API).

## 9. Disposal

GPU memory is not garbage-collected. On unmount / route change:

```js
function disposeScene(root) {
  root.traverse((obj) => {
    if (obj.geometry) obj.geometry.dispose();
    const mats = Array.isArray(obj.material) ? obj.material : obj.material ? [obj.material] : [];
    for (const m of mats) {
      for (const value of Object.values(m)) {
        if (value && value.isTexture) value.dispose();
      }
      m.dispose();
    }
  });
}

disposeScene(scene);
composer?.dispose();
controls?.dispose();
renderer.setAnimationLoop(null);
renderer.dispose();
```

R3F and Tres dispose objects they created declaratively when unmounted; anything you created imperatively (`new THREE.*` in a hook, loaders' cached textures you no longer use, render targets) is still yours to dispose. Watch `renderer.info.memory` across navigations — it should return to baseline.

## 10. WebGPU

- `import * as THREE from "three/webgpu"` and `new THREE.WebGPURenderer()` (call `await renderer.init()` before the first render, or use `setAnimationLoop`). It falls back to a WebGL2 backend where WebGPU is unavailable.
- Materials are written with **TSL** (`three/tsl`) node materials; GLSL `ShaderMaterial` and `onBeforeCompile` do not carry over.
- `EffectComposer` is WebGL-only; WebGPU uses the node-based post-processing pipeline — check the class name for the installed release.
- Choose WebGPU for compute-heavy work (large particle sims, GPU compute) or new projects that accept TSL. Keep WebGL for projects with existing GLSL/composer investment. Don't migrate mid-project without a Tier 2 plan.
