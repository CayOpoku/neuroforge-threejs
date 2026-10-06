# Review — Sweep Order and the 3D Quality Verdict

Load when reviewing a scene, a section, or a 3D site. The verdict closes every Tier 2 analysis and every review.

## Sweep order

Review as a visitor first, then as an engineer.

### 1. Visitor pass (desktop, then a real or emulated mid-range phone)

- **First 3 seconds:** does content appear immediately? Is the intro choreographed or does the canvas pop in?
- **One-idea test:** after one scroll through, can you say what the site is about and what it felt like?
- **Signature test:** is there one interaction you want to play with again?
- **Scroll feel:** smooth, native-feeling, never trapped or speed-hijacked?
- **Mobile:** re-composed, not shrunk? Touch equivalents for hover?

### 2. Craft pass

- Environment map present; lights with intent (key/rim/fill); one tone-mapping choice; colours match the brand
- Correct colour spaces (no washed-out or crushed textures)
- Materials: roughness variation, no plastic-default look on the hero
- Motion damped and frame-rate independent; no linear starts/stops; camera calm
- Type in the DOM, composed with the 3D; hierarchy clear
- Post-processing restrained; bloom selective
- Loading: real progress, designed hand-off

### 3. Engineering pass

- Stack matches the project; no second 3D runtime
- `renderer.info`: draw calls, triangles, textures within budget (`performance.md` §1)
- DPR clamped; assets compressed (Draco/meshopt, KTX2/WebP), sized to screen
- Loop paused offscreen and on hidden tab; on-demand where static
- Disposal on unmount; `renderer.info.memory` returns to baseline after navigation
- No allocations or React/Vue state updates per frame
- SSR-safe; no `window` at module scope
- `prefers-reduced-motion` handled; WebGL-unavailable fallback; content readable by screen readers; keyboard reachable CTAs
- Strict TypeScript, zero `any`; small single-purpose modules

## Finding format

```markdown
### [Severity] Title — `app/components/webgl/Hero.vue:58`
**What:** Bloom on the full scene with threshold 0.2.
**Why it matters:** Everything glows; text behind the canvas loses contrast; costs ~4 ms on mobile.
**Fix:** threshold 0.9, mipmapBlur, disable on tier ≤ 1.
```

Severity: **Critical** (crash, leak, content hidden from users/SEO, < 30fps on target) · **High** (looks broken — colour space, black renders, jank on scroll; a11y failures) · **Medium** (craft — lighting, motion, composition) · **Low** (polish, naming).

Rank by consequence. Never pad the list.

## The 3D Quality Verdict

Close with this block. Be honest with the number.

```markdown
## 3D Quality Verdict — 7/10

| Axis | Score | Note |
|---|---|---|
| Concept & art direction | 8 | Clear idea; lighting sells the product |
| Signature interaction | 6 | Explode-on-scroll planned but no hover payoff |
| Motion & scroll | 7 | Damped and smooth; camera rail overshoots on section 3 |
| Performance | 6 | 140 draw calls; hero GLB 14 MB uncompressed |
| Accessibility & content | 8 | DOM-first copy; reduced-motion missing on parallax |
| Engineering | 8 | Clean disposal; one allocation per frame in Hero |

**Biggest win available:** Compress the hero GLB (14 MB → ~2 MB) and instance the logo wall.
**Award-ready?** After the two Critical/High findings and the hover payoff.
```
