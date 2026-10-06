# Vanilla three.js — Architecture

A structure that stays clean from a hero section to a full experience. API details: `fundamentals.md` and the other API references.

## Layout

```
src/webgl/
  Experience.ts      ← owns renderer, scene, camera, loop, resize, dispose
  Resources.ts       ← loaders (GLTF + Draco/KTX2/Meshopt), progress, cache
  world/
    Hero.ts          ← one class per section/object group: create(), update(dt, state), dispose()
    Environment.ts   ← lights, environment map, fog
  scroll.ts          ← Lenis + ScrollTrigger → normalised progress state
  quality.ts         ← GPU tier, DPR, reduced-motion flags
  debug.ts           ← lil-gui, only when ?debug is in the URL
```

One `Experience` instance per canvas. Sections never create their own renderer or loop — they receive `update(delta, state)` from the experience.

## The shell

```ts
import * as THREE from "three";

export class Experience {
  renderer: THREE.WebGLRenderer;
  scene = new THREE.Scene();
  camera: THREE.PerspectiveCamera;
  private clock = new THREE.Clock();
  private observer: IntersectionObserver;
  private updaters = new Set<(dt: number) => void>();

  constructor(private canvas: HTMLCanvasElement) {
    this.renderer = new THREE.WebGLRenderer({ canvas, antialias: true, alpha: true, powerPreference: "high-performance" });
    this.renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
    this.renderer.toneMapping = THREE.AgXToneMapping;
    this.renderer.outputColorSpace = THREE.SRGBColorSpace;

    this.camera = new THREE.PerspectiveCamera(35, 1, 0.1, 100);
    this.camera.position.set(0, 0, 6);

    this.resize();
    window.addEventListener("resize", this.resize);

    // pause when the canvas is offscreen
    this.observer = new IntersectionObserver(([entry]) => (entry.isIntersecting ? this.start() : this.stop()));
    this.observer.observe(canvas);
    document.addEventListener("visibilitychange", this.onVisibility);
  }

  onUpdate(fn: (dt: number) => void) {
    this.updaters.add(fn);
    return () => this.updaters.delete(fn);
  }

  private tick = () => {
    const dt = Math.min(this.clock.getDelta(), 0.1);
    for (const fn of this.updaters) fn(dt);
    this.renderer.render(this.scene, this.camera);
  };

  start = () => this.renderer.setAnimationLoop(this.tick);
  stop = () => this.renderer.setAnimationLoop(null);
  private onVisibility = () => (document.hidden ? this.stop() : this.start());

  private resize = () => {
    const { clientWidth: w, clientHeight: h } = this.canvas;
    this.camera.aspect = w / h;
    this.camera.updateProjectionMatrix();
    this.renderer.setSize(w, h, false);
  };

  dispose() {
    this.stop();
    this.observer.disconnect();
    window.removeEventListener("resize", this.resize);
    document.removeEventListener("visibilitychange", this.onVisibility);
    // disposeScene() from performance.md §9
    disposeScene(this.scene);
    this.renderer.dispose();
  }
}
```

Notes:
- Size from the canvas's CSS box (`setSize(w, h, false)`), so CSS controls layout and there's no layout shift.
- `Clock` works everywhere; newer three releases also ship a `Timer` class — use it if the installed version has it and the project prefers it.
- `alpha: true` lets the page background show through — useful for layering DOM and canvas.

## Rules

- Every class that creates GPU resources has `dispose()`, and its owner calls it.
- No `new THREE.Vector3()` (or any allocation) inside `update()` — preallocate and reuse; GC pauses cause jank.
- State flows one way: scroll/pointer → plain state object → `update()` reads it. Sections never read the DOM per frame.
- Debug GUI (`lil-gui`) only behind a flag; tuned values get copied back into code as constants.

## Mounting in a framework without TresJS/R3F

Nuxt/Vue: create the `Experience` in `onMounted` on a template ref canvas, `dispose()` in `onBeforeUnmount`, wrap the component in `<ClientOnly>`. Next/React: same in a `useEffect` with cleanup in a `"use client"` component. Keep one persistent canvas in the root layout if the 3D spans pages.
