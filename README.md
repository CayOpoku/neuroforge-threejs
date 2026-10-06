# NeuroForge Three.js

A senior creative-developer skill for award-level 3D websites. Persona, an analysis-first workflow (concept → art direction → scroll storyboard → code), look-dev and motion standards, a 60fps performance budget, and a full Three.js API reference — for vanilla three, TresJS (Vue/Nuxt), and React Three Fiber. Written for coding agents, structured so an agent loads only what the task needs.

Part of the NeuroForge family, alongside [neuroforge-nuxt](https://github.com/cayopoku/neuroforge-nuxt), [neuroforge-nest](https://github.com/cayopoku/neuroforge-nest), and [neuroforge-remotion](https://github.com/cayopoku/neuroforge-remotion).

---

## Installation

Install using [skills](https://github.com/vercel-labs/skills):

```bash
# GitHub shorthand
npx skills add cayopoku/neuroforge-threejs

# Install globally (available across all projects)
npx skills add cayopoku/neuroforge-threejs --global

# Install for specific agents
npx skills add cayopoku/neuroforge-threejs -a claude-code -a cursor
```

### Supported Agents

Claude Code · OpenCode · Codex · Cursor · Antigravity · Roo Code

---

## What it does

| Layer | What the agent gets |
|---|---|
| **Workflow** | Severity tiers, a Diagnose mode for render bugs, `neuroforge/` memory files split into `project/`, `direction/`, `scenes/`, a scroll-storyboard format, and two approval gates before code |
| **Stack** | Detects vanilla / TresJS / R3F from `package.json` and follows that stack's idioms — never mixes runtimes |
| **Art direction** | Concept-first, references, look-dev (environment, key/rim/fill, tone mapping, colour space), materials, type + 3D composition, the signature interaction, the loading moment, mobile re-composition |
| **Motion** | Frame-rate-independent damping, Lenis + GSAP ScrollTrigger, scroll-driven timelines, camera rails, cursor reactions, page transitions, reduced motion |
| **Performance** | Budgets, gltf-transform / Draco / meshopt / KTX2 pipeline, instancing, adaptive quality, render-on-demand, pausing, disposal, WebGPU |
| **Review** | Visitor → craft → engineering sweep and a scored 3D Quality Verdict |
| **API reference** | Ten Three.js references by CloudAI-X: fundamentals, geometry, materials, lighting, textures, animation, loaders, shaders, post-processing, interaction |

---

## Usage

> "Build an immersive scroll-driven hero for our headphone launch — the GLB is in public/models."

> "Add a TresJS 3D section to the Nuxt landing page that reacts to the cursor."

> "Our R3F site drops to 20fps on iPhone — fix it."

Invoking the skill with no task (`/neuroforge-threejs`) runs a full audit of the project's 3D.

---

## Token Budget

| Layer | Size | When it loads |
|---|---|---|
| `SKILL.md` | ~14KB | On skill activation — the only file always in context |
| `references/<craft>.md` | 5–12KB each | On demand — workflow, art direction, motion, performance, stack, review, debugging |
| `references/<api>.md` | 12–18KB each | On demand — one or two API references per task |

---

## Structure

- `skills/neuroforge-threejs/` — the installable skill. `skills add` copies this directory and everything under it; nothing at the repo root ships.
  - `SKILL.md` — entry point: persona, hard stops, triage tiers, stack detection, the NeuroForge order, the award-level bar, references table
  - `references/`
    - NeuroForge layer: `workflow.md`, `art-direction.md`, `motion-and-scroll.md`, `performance.md`, `vanilla.md`, `tresjs.md`, `r3f.md`, `review.md`, `debugging.md`
    - API reference (CloudAI-X): `fundamentals.md`, `geometry.md`, `materials.md`, `lighting.md`, `textures.md`, `animation.md`, `loaders.md`, `shaders.md`, `postprocessing.md`, `interaction.md`
- `metadata.json` — version, authors, abstract, upstream

No build step, no dependencies. Markdown all the way down.

---

## Contributing

1. API corrections go in the matching `references/<api>.md` — verify against [threejs.org/docs](https://threejs.org/docs/) and note the three.js version.
2. Craft and workflow changes go in the NeuroForge layer files; keep `SKILL.md` a router — anything not needed on every task belongs in `references/`.
3. Add any new reference to the table in `SKILL.md` in the same commit.

---

## Acknowledgments

- The Three.js API references originated in [CloudAI-X/threejs-skills](https://github.com/CloudAI-X/threejs-skills) (MIT) — thanks to [CloudAI-X](https://github.com/CloudAI-X) for the original corpus.
- [Three.js](https://threejs.org/), [TresJS](https://tresjs.org/), and [React Three Fiber](https://r3f.docs.pmnd.rs/) — the libraries this skill builds on.
- Structure follows the NeuroForge family ([neuroforge-nuxt](https://github.com/cayopoku/neuroforge-nuxt), [neuroforge-nest](https://github.com/cayopoku/neuroforge-nest)).
- Compatible with [skills](https://github.com/vercel-labs/skills) for installation across coding agents.
