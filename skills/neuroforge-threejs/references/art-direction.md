# Art Direction — What Makes a 3D Site Award-Level

Choices, not APIs. For the API behind each choice see `lighting.md`, `materials.md`, `textures.md`, `postprocessing.md`.

## Contents

1. Concept first
2. References
3. Look-dev: light, environment, tone
4. Materials
5. Composition with type
6. The signature interaction
7. The loading moment
8. Mobile composition
9. Anti-patterns

---

## 1. Concept first

Write the one idea as a feeling and a metaphor before any geometry:

| Brief | Idea | 3D expression |
|---|---|---|
| Battery that lasts | Endless | An object in perpetual, slow, frictionless motion |
| Calm productivity app | Stillness | Soft daylight, fog, slow drift, matte materials |
| Fintech for speed | Precision + speed | Glass and chrome, crisp rim light, fast damped snaps |
| Creative studio portfolio | Play | Cursor-reactive physics, bold colour, playful overshoot |

If you can't fill the right column, you don't have a concept yet — ask the developer.

## 2. References

Collect 3–5 references (Awwwards SOTD, FWA, studio sites the developer likes) and write for each **what specifically to take**: "the way the camera eases into each section", "the matte clay look", "type that overlaps the model". Never "make it like X" — name the device.

## 3. Look-dev: light, environment, tone

Lighting sells "premium" more than polygon count.

- **Environment first.** An HDR environment (`RGBELoader`/`HDRLoader` or a `RoomEnvironment` via `PMREMGenerator`) gives every PBR material believable reflections. Studio HDRs for products; outdoor HDRs for organic scenes. Set `scene.environment`; show it as `scene.background` only if the art direction wants it.
- **Then shape with lights:** a key light with a clear direction, a rim light separating the object from the background, a low fill. Three lights with intent beat ten without.
- **Shadows:** one shadow-casting light, tight shadow camera frustum, soft (`PCFSoftShadowMap` or VSM), or a baked/contact shadow under the hero — cheaper and often prettier.
- **Tone mapping:** pick one for the whole site — `AgXToneMapping` (filmic, handles bright colours gracefully), `ACESFilmicToneMapping` (punchy, shifts saturated colours), `NeutralToneMapping` (closest to true product colour; best for e-commerce). Tune `toneMappingExposure` once.
- **Colour space:** `renderer.outputColorSpace = SRGBColorSpace`; colour textures `SRGBColorSpace`; data textures (normal/roughness/metal/AO) stay linear. Washed-out or crushed colours are almost always this.
- **Atmosphere:** fog matched to the background colour gives depth for free. A subtle vignette and grain in post finish the frame.

## 4. Materials

- **Fewer, better materials.** One hero material that gets the craft; everything else supports it.
- `MeshStandardMaterial` for most things; `MeshPhysicalMaterial` when you need clearcoat (car paint, lacquer), transmission (glass), sheen (fabric), or iridescence — physical is costlier, use it on the hero only.
- Matcaps (`MeshMatcapMaterial`) give a stylised, very cheap, very consistent look — great for illustrative sites and mobile.
- Roughness variation (a roughness map, even a subtle noise) is the difference between "CG" and "real".
- Custom shaders when the concept needs something materials can't do — dissolves, flow, noise-driven displacement, cursor reactions (`shaders.md`). Prefer extending a built-in material (`onBeforeCompile` / TSL) over writing lighting from scratch.

## 5. Composition with type

- Type is DOM (hard stop 1) — big, editorial, from the brand's display face.
- Layer: background canvas → type → foreground 3D element overlapping the type (a second canvas layer or careful z-order) creates depth that reads as "designed".
- Frame the hero off-centre against the copy on desktop; rule-of-thirds or a strong centre axis — chosen, not default.
- Leave negative space. A single object with room to breathe beats a crowded scene.
- Keep the camera's field of view narrow (25–40°) for products — less distortion, more "photographed".

## 6. The signature interaction

Every award-level site has one thing visitors play with and remember. Plan it in `direction/03-interaction-model.md`:

- **Cursor-reactive material** — displacement, refraction, or colour bleeding toward the pointer.
- **Scroll-driven assembly** — a product explodes into parts and reassembles as you scroll.
- **Fly-through transition** — the camera travels through an object or logo into the next section/page.
- **Physical play** — objects you can drag, throw, or knock that settle believably.
- **Particle morph** — a point cloud that forms the logo, the product, then the next shape.

One, executed perfectly, with a touch equivalent on mobile and a reduced-motion fallback.

## 7. The loading moment

- Total first-view payload budget ≈ **3–5 MB** (compressed). Lazy-load anything below the fold.
- Preloader tied to real progress (`LoadingManager` / cientos/drei progress helpers) — no fake timers.
- The preloader hands off into an **intro choreography** (camera dolly, light ramp, type reveal) — the reveal is the first impression.
- Show DOM content immediately; the 3D can fade in when ready. Never block reading on WebGL.

## 8. Mobile composition

- Re-frame: wider camera or object moved/scaled so it doesn't fight the copy in portrait.
- Drop the expensive tier: DOF, SSAO, physical transmission, real-time shadows → baked/contact shadows, fewer instances, DPR ≤ 1.5.
- Hover → tap or gyroscope (ask for permission on iOS) or auto-motion.
- Don't pin long scroll sections on mobile unless tested; native scroll feel matters more there.

## 9. Anti-patterns

- A spinning model on a gradient with no idea behind it.
- Default grey lighting, no environment map, no tone mapping choice.
- Everything glows (bloom threshold too low).
- Copy rendered as 3D text in the canvas (inaccessible, unindexed, blurry).
- Scroll-jacking that changes the scroll speed or traps the visitor.
- A 20 MB uncompressed GLB and 4K textures for a 400 px element.
- Five post-processing effects stacked "for look".
- Desktop composition shrunk onto a phone.
