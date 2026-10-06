# NeuroForge Three.js — Tier 2 Workflow

The full protocol for a new 3D site or section, a scroll story, a configurator, a stack migration, or an audit. Tier 0 and Tier 1 never touch this file.

## Contents

1. Activation sequence
2. The `neuroforge/` folder
3. `00-answers.md`
4. Domain 1 — `project/`
5. Domain 2 — `direction/`
6. Domain 3 — `scenes/` (the scroll storyboard)
7. Approval gates
8. Building
9. Closing the loop — prune and handoff

---

## 1. Activation sequence

1. Open your reply with `Activating NeuroForge Three.js analysis...` as its own first line — the developer's signal the protocol engaged.
2. **Detect the stack** from `package.json` (SKILL.md → Detect the stack). Note `three`, `@tresjs/*`, `@react-three/*`, `gsap`, `lenis`, `postprocessing` versions.
3. **Create `neuroforge/`** with `project/`, `direction/`, `scenes/`; add `neuroforge/` to `.gitignore` (create it if missing). Mandatory, no permission needed.
4. **Inventory `neuroforge/` before reading code.** Read every existing file, verify its claims against the code, report a status table (`current` / `stale` / `superseded`). Don't re-derive what was decided; don't re-report what was fixed.
5. **Scan** — structure before detail: where the canvas mounts, the loop, the scene graph, assets in `public/` (sizes in MB!), shaders, post-processing, scroll libs, cleanup paths.
6. Write `project/` + `direction/` → present → **wait** → `scenes/` → present → **wait for "Proceed"**.
7. Track work in the IDE's native task list. **Never** a `task.md`, `todo.md`, `plan.md`, or `checklist.md` inside `neuroforge/`; no native list → inline checklist in the reply.

## 2. The `neuroforge/` folder

```
neuroforge/
  00-answers.md      ← settled answers (append-only log)
  project/           ← technical audit
  direction/         ← concept, references, look-dev, interaction model
  scenes/            ← section-by-section scroll storyboard
```

- **Analysis only** — no executable code.
- **Supersede, never destroy** — version a replaced file (`02-v2-look-dev.md`), say so. No archive folders, no `-old` suffixes. Current, or proposed for deletion.
- **Small files, one concern each.**

## 3. `00-answers.md`

```markdown
## Brand
- Ink #0B0B0F, accent #C8FF00, display "PP Neue Montreal". (2026-10-06)

## Stack
- Nuxt 4 + @tresjs/nuxt; GSAP + Lenis already installed. (2026-10-06)

## Decisions
- Hero model is the client's GLB (12 MB raw) — compress, don't remodel. Settled. (2026-10-06)

## Preferences
- Dev server always on :3000 — never start it. (2026-10-06)
```

One dated line per entry, append-only, never pruned. Ask in chat, log the answer — never a queue of open questions.

## 4. Domain 1 — `project/`

| File | Holds |
|---|---|
| `01-project-overview.md` | Stack + versions, where 3D mounts, routing, SSR boundaries. Amend in place across tasks — never overwrite |
| `02-scene-graph.md` | Scenes, cameras, objects, materials, lights, loops — and who owns disposal |
| `03-assets.md` | Every model/texture/HDR: format, size on disk, resolution, compression, where it loads |
| `04-performance.md` | Measured or estimated draw calls, triangles, texture memory, DPR, fps on desktop and a mid phone |
| `05-a11y-and-content.md` | What content is DOM vs canvas, reduced-motion handling, fallback, keyboard paths |
| `06-gaps.md` | Delta from this skill's standards, with file:line |

## 5. Domain 2 — `direction/`

| File | Holds |
|---|---|
| `01-concept.md` | The one idea, the feeling, the audience, the conversion goal, 3–5 reference sites and what to take from each |
| `02-look-dev.md` | Palette, environment/HDR, key/rim/fill plan, tone mapping, materials, post-processing look, type pairing with 3D |
| `03-interaction-model.md` | Scroll behaviour, cursor/hover reactions, the **signature interaction**, mobile/touch equivalents, reduced-motion behaviour |
| `04-loading.md` | Asset budget, preload order, the preloader design, the intro choreography |

Content guidance: `references/art-direction.md`, `references/motion-and-scroll.md`.

## 6. Domain 3 — `scenes/` (the scroll storyboard)

`01-storyboard.md` maps the whole page first:

```markdown
| # | Section | Scroll | Camera | 3D on screen | DOM on screen | Transition out |
|---|---|---|---|---|---|---|
| 0 | Intro | load | dolly 8 → 5 | product in silhouette, rim light sweeps | logo, "Scroll" hint | light ramps up |
| 1 | Hero | 0–100vh | orbit −20° → 0° | product, hero lit | H1 + CTA | product scales down, moves right |
| 2 | Features | 100–400vh (pinned) | rail along 3 stops | product rotates to each feature; exploded parts | one feature card per stop | parts reassemble |
| 3 | Proof | 400–500vh | static | instanced logos wall reacts to cursor | testimonials | fade to footer colour |
| 4 | CTA | 500vh+ | slow push | product, final hero light | CTA, footer | — |

**Signature interaction:** Section 2 — the product explodes into parts on scroll; hovering a part highlights it and pins its card.
**Mobile:** sections 2–3 unpinned; camera framed 30% wider; no DOF; instanced wall halved.
**Reduced motion:** no explode; crossfade between three static product renders.
```

Add one file per complex section (pinned sequences, shader-heavy effects, configurators) with: scroll range, camera keyframes, object states per keyframe, easing, DOM elements and their timing, the perf notes, and the mobile/reduced-motion variants.

## 7. Approval gates

Two, never more:
1. **After `project/` + `direction/`** — "Is this the idea and the look?"
2. **After `scenes/`** — "Proceed?" No code before this.

After "Proceed", execute the approved plan end to end; don't re-ask per section, don't silently expand it.

**Arriving already approved.** If the developer hands you an approved plan in the message and no `neuroforge/` files exist, don't re-run the analysis — build. Record the plan in `scenes/01-storyboard.md` and settled facts (brand, stack, asset sizes) in `00-answers.md` so the next session starts from it.

## 8. Building

- Asset pipeline first (compress models/textures — `references/performance.md`), then the canvas shell (renderer, resize, loop, disposal, reduced-motion flag), then look-dev on the hero, then sections in storyboard order.
- After each section: check it in the browser at desktop and a mobile viewport, glance at `renderer.info` and fps, tick it in the task list.
- A section that drifts from its storyboard row → update the row in the same step.

## 9. Closing the loop — prune and handoff

1. Verify every file you wrote exists with the expected content.
2. Promote anything durable into `project/01-project-overview.md` or `00-answers.md`.
3. Close with the **3D Quality Verdict** (`references/review.md`).
4. **Propose a prune** of `neuroforge/` — a list with one reason per file; delete only what is approved. `00-answers.md` is never pruned.

**Context getting heavy?** Offer a handoff note in the OS temp directory, never in the repo: skills to load, `neuroforge/` files to read, sections done and remaining, the dev port. No secrets.
