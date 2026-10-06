# Debugging Three.js

Load before your second fix attempt. Diagnose mode rules apply: at most five files, two likely causes, one cheapest check, then ask.

## The cheapest checks

- `console.log(renderer.info.render, renderer.info.memory)` — is anything being drawn? Is memory growing?
- Add `new THREE.AxesHelper(2)` and a `MeshNormalMaterial` box at the origin — is the camera pointing at anything?
- Swap the suspect material for `MeshBasicMaterial({ color: "hotpink" })` — geometry problem or lighting/material problem?
- Toggle one thing at a time: post-processing off, shadows off, DPR 1.

## Symptom → likely causes

| Symptom | Most likely | Then check |
|---|---|---|
| Black screen, no errors | Camera inside/behind the object or `near`/`far` wrong | Canvas has 0 height (CSS); nothing added to the scene; render never called |
| Model is black | No lights and no environment for a PBR material | Normals inverted/missing; `metalness: 1` with no env map |
| Colours washed out / too pale | Colour texture not marked `SRGBColorSpace`, or post-processing missing output/colour-space pass | Double tone mapping (renderer + composer) |
| Colours too dark / crushed | Data texture (normal/roughness) marked sRGB, or `toneMappingExposure` low | Light intensities from an old version (physically based units since r155) |
| Z-fighting / flicker on surfaces | `near` plane too small vs `far` | Coplanar faces; use `polygonOffset` or move geometry |
| Shadow acne / stripes | Shadow bias | `shadow.bias` small negative, `shadow.normalBias` ~0.02 |
| Shadows missing | `renderer.shadowMap.enabled`, `castShadow`, `receiveShadow` | Shadow camera frustum doesn't cover the objects |
| Transparent objects disappear/sort wrong | Transparency sorting | `depthWrite: false`, `renderOrder`, alpha test instead of blend |
| Low fps | DPR / fill-rate (try DPR 1) | Draw calls (`renderer.info`), shadows, post-processing, raycasting per move |
| Jank on scroll | Allocations per frame (GC), layout reads per frame | Shader compile on first view → `renderer.compile` after load |
| Memory grows on navigation | Missing disposal | Loops, ScrollTriggers, Lenis, listeners not removed; second canvas created |
| Two canvases / doubled animation in dev | HMR or StrictMode double-mount without cleanup | Effect/onMounted cleanup missing |
| `window is not defined` / `document is not defined` | Three/GSAP code running during SSR | Module-scope access; component not client-only |
| GLB fails to load | Wrong path (`public/` → `/models/x.glb`) | Draco/KTX2/Meshopt decoder not set or decoder path wrong |
| Model loads but invisible | Scale (cm vs m — 100× off) | Placed far from origin; camera `far` too small |
| WebGL context lost | GPU memory exhausted (huge textures) or too many contexts | Listen for `webglcontextlost`; reduce texture sizes; one renderer per page |
| Interaction hits wrong object | Raycaster using stale camera/pointer, or invisible meshes still raycastable | `layers`, `raycast = () => {}` on helpers |

## Rules while debugging

- Instrument before you guess. One log, read it, decide.
- Two fixes that didn't move the symptom → stop. Say what you know and the one thing that would tell you.
- Never mask a symptom you haven't explained (e.g. raising light intensity to hide a colour-space bug).
- API doubt → check the installed version (`node_modules/three/package.json` → `version`) against threejs.org docs and the migration guide.
