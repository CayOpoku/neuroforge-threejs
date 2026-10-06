# Motion and Scroll

How motion is choreographed on an award-level 3D site. Stack-specific wiring is in `vanilla.md`, `tresjs.md`, `r3f.md`.

## Contents

1. Motion principles for 3D
2. Damping and frame-rate independence
3. Smooth scroll — Lenis + GSAP
4. Scroll-driven timelines
5. Camera rails
6. Cursor and pointer
7. Page and section transitions
8. Reduced motion
9. Anti-patterns

---

## 1. Motion principles for 3D

- **Never linear** for anything that starts or stops. Ease or damp.
- **Weight:** large objects move slower and settle longer than small ones.
- **Follow-through:** secondary objects lag the primary by a few frames of damping.
- **Camera calm, objects alive:** a slow camera with lively details reads cinematic; a busy camera reads nauseating.
- **One focus per moment** — the thing that moves is the thing the eye goes to.

## 2. Damping and frame-rate independence

Per-frame constants (`x += 0.01`, `lerp(a, b, 0.1)`) run twice as fast on a 120 Hz screen. Use time.

```js
import * as THREE from "three";

// exponential damping toward a target — same feel at 60, 120, or 144 Hz
// lambda ≈ 4 soft, 8 snappy, 12+ tight
current.x = THREE.MathUtils.damp(current.x, target.x, 6, delta);
```

For vectors and quaternions, damp each component or use `maath/easing` (`damp3`, `dampQ`, `dampE`, `dampC`) — common in R3F, framework-agnostic.

Clamp `delta` (e.g. `Math.min(delta, 0.1)`) so a backgrounded tab doesn't teleport everything on return.

## 3. Smooth scroll — Lenis + GSAP

Lenis smooths native scroll (keeps the scrollbar, find-in-page, and accessibility); GSAP ScrollTrigger maps scroll to animation. Wire them to one ticker:

```js
import Lenis from "lenis";
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";

gsap.registerPlugin(ScrollTrigger);

const lenis = new Lenis({ autoRaf: false });
lenis.on("scroll", ScrollTrigger.update);
gsap.ticker.add((time) => lenis.raf(time * 1000));
gsap.ticker.lagSmoothing(0);
```

- Destroy on unmount: `lenis.destroy()`, kill ScrollTriggers (`ScrollTrigger.getAll().forEach(t => t.kill())` or scope with `gsap.context()` and `ctx.revert()`).
- GSAP's plugins (ScrollTrigger, SplitText, etc.) are free to use in current GSAP releases — check the installed version.
- Reduced motion → skip Lenis entirely and keep native scroll.

## 4. Scroll-driven timelines

Drive 3D from a **normalised progress value**, not directly from pixels:

```js
const state = { progress: 0 };

gsap.timeline({
  scrollTrigger: {
    trigger: "#features",
    start: "top top",
    end: "+=300%",
    pin: true,
    scrub: 1,          // 1s of smoothing behind the scrollbar
  },
}).to(state, { progress: 1, ease: "none" });

// in the render loop: map progress → scene state (camera, explode amount, material uniforms)
```

- `scrub: true` locks to the scrollbar; `scrub: 1` adds a weighted catch-up that feels physical.
- Keep the timeline `ease: "none"` and shape motion in the mapping (`gsap.parseEase("power2.inOut")(p)` or your own curves), so one scroll position always means one scene state.
- Pin with care on mobile; test with real touch scrolling.
- Use `ScrollTrigger.refresh()` after the canvas or images change layout.

Alternatives: drei `ScrollControls` + `useScroll()` in R3F (scroll inside the canvas — fine for single-page experiences, weaker for long DOM content); a plain `scroll` listener + IntersectionObserver for simple cases.

## 5. Camera rails

For guided fly-throughs, move the camera along a curve and look at a second curve or a target:

```js
const path = new THREE.CatmullRomCurve3([
  new THREE.Vector3(0, 1, 8),
  new THREE.Vector3(3, 1.5, 4),
  new THREE.Vector3(0, 2, 0),
]);
const look = new THREE.Vector3();

function updateCamera(p, delta) {
  const t = THREE.MathUtils.clamp(p, 0, 1);
  const target = path.getPointAt(t);
  camera.position.x = THREE.MathUtils.damp(camera.position.x, target.x, 5, delta);
  camera.position.y = THREE.MathUtils.damp(camera.position.y, target.y, 5, delta);
  camera.position.z = THREE.MathUtils.damp(camera.position.z, target.z, 5, delta);
  look.lerpVectors(lookA, lookB, t);
  camera.lookAt(look);
}
```

Author keyframes as data (position, target, fov per section) so the storyboard maps one-to-one onto code.

## 6. Cursor and pointer

- Normalise pointer to `[-1, 1]` and damp toward it — never apply raw mouse deltas.
- Subtle parallax: rotate the hero group by `pointer.x * 0.15` rad, damped.
- Cursor-reactive shaders: pass the pointer (or a raycast hit point) as a uniform, damped.
- Raycast only against what needs it (`raycaster.layers`, a simplified proxy mesh, or BVH via `three-mesh-bvh`) — raycasting a 500k-triangle mesh every pointermove is a frame-killer.
- Touch has no hover: map the signature interaction to drag, tap, or scroll.

## 7. Page and section transitions

- Keep one persistent canvas across route changes (Nuxt layout / Next root layout) and transition the scene, rather than tearing down and rebuilding WebGL per page.
- Transition out: animate the camera/uniforms to an "exit" state, swap content, animate in. GSAP timelines with `onComplete` hooked into the router's transition hooks.
- Never leave the old page's objects in the scene — dispose or reuse.

## 8. Reduced motion

```js
const reduceMotion = window.matchMedia("(prefers-reduced-motion: reduce)").matches;
```

- No smooth-scroll library, no scrubbed camera moves, no auto-rotation, no parallax.
- Keep a static, well-composed frame per section (cross-fade between them).
- Listen for changes (`matchMedia(...).addEventListener("change", ...)`).

## 9. Anti-patterns

- `lerp(a, b, 0.1)` per frame (frame-rate dependent).
- Scroll speed changed or hijacked; scrollbar hidden.
- Camera shake, constant orbiting, or roll that makes people sick.
- Raycasting every mesh on every pointermove.
- ScrollTriggers and Lenis never destroyed on route change — doubles on every navigation.
