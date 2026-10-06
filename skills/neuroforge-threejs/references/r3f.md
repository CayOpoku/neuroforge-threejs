# React Three Fiber (React / Next)

Declarative Three.js for React. Packages: `@react-three/fiber`, `@react-three/drei` (helpers), `@react-three/postprocessing`, `maath` (damping/easing math). R3F's major tracks React's (v9 ↔ React 19) — **check installed versions before using a signature.**

## The canvas

```tsx
"use client";
import { Canvas } from "@react-three/fiber";
import { Environment } from "@react-three/drei";
import * as THREE from "three";

export function HeroCanvas() {
  return (
    <Canvas
      dpr={[1, 2]}
      camera={{ position: [0, 0, 6], fov: 35 }}
      gl={{ antialias: true, toneMapping: THREE.AgXToneMapping, outputColorSpace: THREE.SRGBColorSpace }}
      className="hero-canvas"
    >
      <Environment preset="studio" />
      <HeroScene />
    </Canvas>
  );
}
```

Next.js: the canvas is a client component; import it with `dynamic(() => import("./HeroCanvas"), { ssr: false })` where it would otherwise render on the server. Keep one persistent canvas in the root layout when the 3D spans routes.

## Per-frame logic

```tsx
import { useRef } from "react";
import { useFrame } from "@react-three/fiber";
import { easing } from "maath";
import type { Mesh } from "three";

export function HeroScene() {
  const hero = useRef<Mesh>(null);

  useFrame((state, delta) => {
    if (!hero.current) return;
    // state.pointer is normalised -1..1
    easing.dampE(hero.current.rotation, [-state.pointer.y * 0.15, state.pointer.x * 0.3, 0], 0.25, delta);
  });

  return (
    <mesh ref={hero}>
      <torusKnotGeometry args={[1, 0.32, 256, 32]} />
      <meshPhysicalMaterial roughness={0.15} clearcoat={1} />
    </mesh>
  );
}
```

- **Never `setState` in `useFrame`.** Mutate refs. React re-renders at 60fps destroy performance.
- No allocations inside `useFrame` — hoist vectors to module scope or `useMemo`.
- `useFrame` is forbidden inside Remotion's `<ThreeCanvas>` — see **neuroforge-remotion**.

## drei essentials

| Need | drei |
|---|---|
| Models | `useGLTF` (+ `useGLTF.preload`), or generate a typed component with `npx gltfjsx model.glb --transform` |
| Lighting | `Environment`, `Lightformer`, `ContactShadows`, `AccumulativeShadows`, `Stage` |
| Materials | `MeshTransmissionMaterial`, `MeshReflectorMaterial`, `shaderMaterial` helper |
| Text | `Text` (SDF, crisp), `Text3D` — but body copy stays in the DOM |
| DOM in 3D | `Html`, `View` (multiple scenes in one canvas tied to DOM elements) |
| Scroll | `ScrollControls` + `useScroll` (in-canvas scroll) |
| Perf | `PerformanceMonitor`, `AdaptiveDpr`, `Instances`, `Merged`, `Detailed`, `Bvh`, `Preload` |
| Loading | `useProgress` for a real preloader |
| Motion | `Float`, `PresentationControls`, `CameraControls` |

## Suspense and loading

Wrap async content in `<Suspense fallback={null}>` inside the canvas; drive the DOM preloader from `useProgress()` outside it. Preload the hero model at module scope (`useGLTF.preload("/models/hero.glb")`).

## Scroll

- Long DOM pages: GSAP + Lenis outside the canvas writing a progress ref; `useFrame` reads it — `motion-and-scroll.md` §3–4.
- Single-screen experiences: drei `ScrollControls` + `useScroll().offset`.

## Post-processing

```tsx
import { EffectComposer, Bloom, Vignette, Noise } from "@react-three/postprocessing";

<EffectComposer multisampling={0}>
  <Bloom mipmapBlur luminanceThreshold={0.9} intensity={0.6} />
  <Noise opacity={0.03} />
  <Vignette darkness={0.4} />
</EffectComposer>
```

Effects merge into few passes; still keep ≤ 3 desktop, ≤ 1 mobile. High `luminanceThreshold` keeps bloom selective.

## Performance in R3F

- `frameloop="demand"` + `invalidate()` for scenes that only change on interaction.
- `<PerformanceMonitor onDecline={...}>` to step DPR/effects down.
- Instancing via `<Instances>`/`<instancedMesh>`.
- R3F disposes declaratively created objects on unmount; dispose imperatively created ones yourself (`useEffect` cleanup).
- `r3f-perf` during development.
