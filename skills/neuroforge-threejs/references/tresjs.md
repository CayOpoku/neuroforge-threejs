# TresJS (Vue / Nuxt)

Declarative Three.js for Vue — the natural fit for NeuroForge Nuxt projects. TresJS moves fast between majors (v5 at the time of writing): **check `node_modules/@tresjs/core/package.json` and the docs at docs.tresjs.org before using any signature below.**

Packages: `@tresjs/core` (renderer + components), `@tresjs/cientos` (helpers: controls, loaders, staging, abstractions), `@tresjs/post-processing`, `@tresjs/nuxt` (Nuxt module, auto-imports, client-only handling).

## Setup in Nuxt

```bash
npx nuxi module add @tresjs/nuxt   # suggest; don't run unprompted
```

The module registers the Tres components and handles client-only rendering. Without the module, wrap the canvas component in `<ClientOnly>`.

## The canvas

```vue
<script setup lang="ts">
import { TresCanvas } from "@tresjs/core";
import { AgXToneMapping, SRGBColorSpace } from "three";
</script>

<template>
  <div class="hero-canvas">
    <TresCanvas
      :tone-mapping="AgXToneMapping"
      :output-color-space="SRGBColorSpace"
      :dpr="[1, 2]"
      clear-color="#0B0B0F"
    >
      <TresPerspectiveCamera :position="[0, 0, 6]" :fov="35" />
      <HeroScene />
    </TresCanvas>
  </div>
</template>
```

- Every Three.js class is available as `Tres<ClassName>` — `<TresMesh>`, `<TresMeshStandardMaterial>`, `<TresDirectionalLight>`; constructor args via `:args`.
- Size the canvas with CSS on its wrapper (absolute/fixed full-bleed behind DOM content).
- Prop names follow Three.js properties in kebab-case; verify renderer props (`dpr`, `tone-mapping`, `render-mode`, `clear-color`, `shadows`) against the installed version.

## Per-frame logic

```vue
<script setup lang="ts">
import { shallowRef } from "vue";
import { useLoop } from "@tresjs/core";
import { MathUtils, type Mesh } from "three";

const hero = shallowRef<Mesh>();
const pointer = { x: 0, y: 0 }; // updated from a pointermove listener, normalised -1..1

const { onBeforeRender } = useLoop();

onBeforeRender(({ delta }) => {
  if (!hero.value) return;
  const dt = Math.min(delta, 0.1);
  hero.value.rotation.y = MathUtils.damp(hero.value.rotation.y, pointer.x * 0.3, 5, dt);
  hero.value.rotation.x = MathUtils.damp(hero.value.rotation.x, -pointer.y * 0.15, 5, dt);
});
</script>

<template>
  <TresMesh ref="hero">
    <TresTorusKnotGeometry :args="[1, 0.32, 256, 32]" />
    <TresMeshPhysicalMaterial :roughness="0.15" :metalness="0.2" :clearcoat="1" />
  </TresMesh>
</template>
```

- `useLoop()` must be called inside a component rendered **within** `<TresCanvas>`.
- The callback context includes `delta`, `elapsed`, `renderer`, `camera`, `scene`, `invalidate()` (on-demand mode) — see the docs.
- Use `shallowRef` for Three objects — deep reactivity on a Three.js object is expensive and pointless.
- **Never put per-frame values in reactive state** that re-renders Vue templates. Mutate the Three object in the loop.

## Models, environment, controls (cientos)

cientos provides the common helpers — model loading (a GLTF composable and a `GLTFModel` component, with Draco support), `Environment`, `OrbitControls`, `ContactShadows`, `Float`, `Html`, `Text3D`, `Stars`, `Sparkles`, and more. Their signatures changed between cientos majors (e.g. composables returning reactive `state` vs. awaited objects) — read the installed version's docs before writing against them.

Pattern that holds across versions:
- Load once, reuse — don't load the same GLB in two components.
- Wrap async-loaded content in `<Suspense>` when the API is `await`-based.
- Compress models first (`performance.md` §3).

## Scroll

GSAP + Lenis drive a plain (non-reactive) progress object; the Tres loop reads it — `motion-and-scroll.md` §3–4. Create and destroy them in the component that owns the section (`onMounted` / `onBeforeUnmount` with `gsap.context()` → `ctx.revert()`).

## Post-processing

`@tresjs/post-processing` wraps pmndrs `postprocessing` effects (bloom, DOF, noise, vignette, …) as components inside an effect composer component. Component names differ between versions — check the docs. Keep to ≤ 3 effects desktop, ≤ 1 mobile.

## Performance in Tres

- `render-mode="on-demand"` + `invalidate()` for scenes that only change on interaction.
- Instancing: `<TresInstancedMesh :args="[geometry, material, count]">` and set matrices in `onMounted`/the loop.
- Persistent canvas across pages: put `<TresCanvas>` in a Nuxt layout and swap scene components per route.
- Dispose anything you created with `new` yourself in `onBeforeUnmount`.

## Nuxt specifics

- SSR: never touch `window`, `matchMedia`, or WebGL outside `onMounted`/client-only code.
- Assets in `public/models/`, `public/draco/`, `public/basis/`.
- Pair with **neuroforge-nuxt** for the app around the canvas.
