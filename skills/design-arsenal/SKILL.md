---
name: design-arsenal
description: Master router over this pack's design stack — maps any design need (direction, wireframes/structure, identity, icons, typography, color, components, gauges, diagrams, audio UI, glass surfaces, texture, backgrounds, motion, 3D, video, audio, logo, design-system docs) to the right skill and sequences them into a complete design-system build for an application. Use this whenever a task is bigger than one asset — "build a design system", "do a full design pass", "make this app look premium", "set up the visual identity", "redesign this whole product" — or when unsure which design skill applies, or when starting design work on a new repo and deciding what to load in what order. Also owns the glass/glassmorphism recipe (no single site-skill covers it) and the aggregated licensing pre-flight for commercial launches.
---

# Design Arsenal

A router, not a tool. Each skill in this pack owns one tool or
practice. This skill owns the map between them and the order they compose in
when the job is a whole design system rather than a single asset.

## The roster

| Need | Skill | What it yields |
|---|---|---|
| Visual direction / moodboard | `cosmos` | Reference → named decisions loop |
| Hero-section direction | `supahero` | Hero anatomy catalog + adaptation |
| "Does this read current?" | `recent-design` | Trend triage: adopt / adapt / skip |
| Case-study / showcase narrative | `pafolios` | Narrative spine for showcase pages |
| Page/app structure, wireframes | `squig` | Hand-drawn layout variations + shadcn handoff |
| Icon set choice / icon spec | `iconography` | One icon decision per project, wrapper, lucide swap |
| Logo direction (real-world refs) | `logosystem` | 1,300+ real logo systems, faceted |
| Logo generation (fast, cheap) | `logoai` | AI logo + brand kit, paid download |
| Free fonts, safely | `bestfreefonts` | Licensed fonts + next/font hosting |
| Font pairings / style database | `ui-ux-pro-max` | Curated pairing + palette data |
| Brand color → full scale | `uicolors` | Tailwind 50–950 scale from one hex |
| Chart / dashboard color | `dataviz` | Validated viz palette method |
| Gauges, dials, meters, rings | `dataviz` → `gauge-ui` | MIT registry SVG gauge primitives |
| Diagrams from Mermaid text | `pele` | Token-themed SVG diagrams, SSR or reflowing |
| Component base | `shadcn` | CLI/registry mechanics, token remap |
| Alternative component base | `arc-ui` | Calm CSS-Modules system; pick it *or* shadcn per app |
| Audio UI (players, waveforms, meters) | `audiocn` | 50 MIT registry items; Tailwind v4 + React 19.2 only |
| Design-system docs (steps, changelog, ADRs) | `mdxcn` | ASCII-framed MDX figures; specimens stay hand-built |
| Animated spectacle components | `componentry` | ~60 MIT registry components |
| Animation primitives | `motion-primitives` | Text reveals, tickers, marquees |
| Motion direction / choreography | `motionsites` | Preview → framer-motion translation |
| Designer-authored animation | `jitter` | MP4/GIF/Lottie exports + players |
| Interactive 3D | `spline` | react-spline embed or export fallback |
| SVG backgrounds (waves, blobs) | `haikei` | Generator handoff + integration |
| Texture, grain, glow, patterns | `fffuel` | ~40 generators routed by use case |
| Music / SFX for video | `uppbeat` | License tiers + audio restraint |
| Anti-slop direction + taste | `design-taste-frontend` (plugin `taste-skill`), `frontend-design` | Judgment, copy voice, finish gate |
| Company / marketing website, start to finish | `design-taste-frontend` | Whole page + real-browser gate (see below) |

## Taste Skill 2.5: how it sits in this stack

Installed as its own plugin (`taste-skill@taste-skill`, user scope, auto-updates
from `Leonxlnx/taste-skill`), not vendored here: it carries ~37 MB of kit assets
and a gate toolchain. Invoke it as `taste-skill:design-taste-frontend`.

It is no longer a style guide; it is an opinionated **website builder with a
finish gate** (Node 22+, Chrome; nine house rules such as one section per
screen, no image captions, real photographs). That decides where it fits:

- **Company, product, launch, marketing pages:** it can own the whole build.
  Treat it as the style spine; do not stack `high-end-visual-design`,
  `minimalist-ui` or `industrial-brutalist-ui` on top.
- **Repo that already has a design system** (its own tokens, specimen,
  component layer): the repo's tokens win. Do **not** let it
  copy its `system.css` / `assets` kit into `public/`. Use its `copy*.md`,
  `layout-v4.md`, `sections.md` and `photos.md` references for judgment, and run
  its gate on the built page as a check.
- **App UIs, consoles, dashboards, RocketRide apps:** not its job. Route to
  `rocketride-frontend`, `shadcn` / `arc-ui`, `dataviz` instead; borrow only its
  copy-tells lint.
- **Its gate is a free Tier-1 check** for any marketing page, whoever built it:
  `node <skill>/scripts/gate.mjs <page|url>` runs layout, copy, perf, icon and
  photo checks in headless Chrome. `loop-engineering` can gate on it. The Stop
  hook it declares only engages after `gate.mjs --start` arms a build
  (`.taste-skill/build.json`); `gate.mjs --done "<user's words>"` closes one.

The older taste-family skills still vendored here (`redesign-existing-projects`,
`high-end-visual-design`, `minimalist-ui`, `industrial-brutalist-ui`,
`brandkit`, `full-output-enforcement`) were dropped upstream in v2; they stay
as frozen MIT copies and keep their roles.

## Check the stack before writing a line of code

These skills were authored against Next 14 + framer-motion 11. Real repos drift,
and the drift is silent — the API matches, only the import path differs, so a
wrong import fails at build time rather than in review.

Verified in a production RocketRide app (Aug 2026): Next **16**, React **19**,
Tailwind **3.4**, and `motion` **v13** — imports are `from "motion/react"`, not
`"framer-motion"`. Read `package.json` first, every time. A component library
that declares `framer-motion` as its own dependency (componentry does) can
coexist, but that is two copies of the same engine — say so before installing.

## Build sequence for a full design system

Phases in order; each consumes the previous phase's output. Skip nothing
silently — say what was skipped and why.

1. **Direction** — `design-taste-frontend` + `frontend-design` first (judgment;
   in a repo with its own system, taste's references only, never its kit), then
   `cosmos` / `recent-design` for references, `supahero` for the hero,
   `pafolios` if showcase pages exist. Output: a one-page direction doc with
   named decisions (grid, type scale, spacing rhythm, color temperature,
   motion character).
1.5. **Structure** — `squig`: sketch 2–3 layout variations side by side, the
   user picks one. Skip when the structure is already decided (say so).
   Output: the chosen variation's `.squig.json` + PNG, mapped to shadcn kinds.
2. **Identity** — `logosystem` for direction, `logoai` only if a mark must be
   generated; `bestfreefonts` (or `ui-ux-pro-max` pairings) for type;
   `uicolors` to expand brand hexes into scales; `iconography` to choose the
   icon set and write the icon spec (grid, stroke, caps). Output: tokens on paper.
3. **Tokens in code** — tailwind.config.ts + CSS variables, dark mode strategy
   decided (class vs media) before any component lands.
4. **Component base** — `shadcn` (the Tailwind v3 vs v4 CLI fork is decision
   one, and `2.3.0` now needs a post-init fix), remap its tokens onto the
   palette from phase 3 — never accept defaults. **Or** `arc-ui` as the whole
   base — pick one per app: their token names collide (`--accent`,
   `--border`…), and mixing them needs `arc-ui`'s token bridge, re-run after
   every `shadcn add`. Then the
   specialists, each bridged onto the phase-3 tokens: `iconography` (execute
   the icon swap), `gauge-ui` (gauges), `pele` (diagrams, incl.
   `--pele-series-*`), `audiocn` (audio UI, Tailwind v4 repos only).
5. **Surfaces** — glass recipe below; `fffuel` for grain/noise/texture;
   `haikei` for structural backgrounds. Texture is seasoning, not structure.
6. **Motion system** — `motionsites` for choreography patterns, implemented in
   framer-motion; `motion-primitives` / `componentry` for pre-built pieces;
   `jitter` when a designer authors the animation; `arc-ui` interaction pieces
   (confirm-morph, segmented control) if Arc is the base. One easing
   vocabulary, defined once.
7. **Spectacle (optional, budgeted)** — `spline` for 3D (582KB gz runtime —
   demand it earns its LCP cost), video with `uppbeat` audio rules (never
   autoplay with sound).
7.5. **Documentation** — the design system's own docs site. Hand-built
   components carry the specimen layer (swatches, type ramp, live previews);
   `mdxcn` carries the narrative (install steps, changelog, ADRs, env tables,
   callouts); `pele` draws architecture diagrams.
8. **QA pass** — contrast on the busiest region, prefers-reduced-motion on
   every animation, bundle audit, licensing pre-flight below. Plus:
   `role="meter"` on display gauges (`gauge-ui`); the Pele parse-error check
   (`pele`); scroll before screenshotting reveal-animated pages (`mdxcn`, Arc
   counters) or frames capture blank; no autoplay on audio UI (`audiocn`).
   A human can verify the critic's spacing/grid/motion claims on the live
   page with the Squints Chrome extension (see `loop-engineering`).

## Glass recipe

No site-skill owns glassmorphism; it's pure CSS plus restraint:

```tsx
<div className="rounded-2xl border border-white/10 bg-white/[0.06]
  backdrop-blur-xl backdrop-saturate-150
  shadow-[inset_0_1px_0_0_rgba(255,255,255,0.08)]">
```

- Glass needs something behind it — a gradient, image, or `haikei`/`fffuel`
  background. Over a flat color it just reads as gray.
- Light mode: flip to `bg-black/[0.04] border-black/10`.
- Add a `fffuel` nnnoise texture at 2–4% opacity to kill banding.
- `backdrop-blur` is expensive; don't stack more than ~3 glass layers per view,
  and never animate blur radius.
- Text on glass must still pass AA against the *busiest* backdrop region.

## Licensing pre-flight (commercial launch)

- Fonts: license verified at source, OFL rename rule if subsetting
  (`bestfreefonts`).
- Logo: AI marks are a license not a copyright; trademark search done
  (`logoai`).
- Audio: tier covers the use — client work and paid ads need Business tier
  (`uppbeat`); no masters committed to the repo.
- 3D/animation: Spline watermark/plan checked; Jitter free tier watermarks.
- Components: `componentry` and shadcn are MIT — fine; `motionsites` prompts
  are paid with no published license text. `gauge-ui`, `audiocn`, `mdxcn`,
  `pele` and Arc's free tier are MIT; Arc Pro items need a paid licence.
- Icons: package + licence recorded (`iconography`); `@remixicon/react` ≥4.9.0
  is no longer Apache-2.0, Hugeicons Pro needs a licence. The Iconography
  encyclopedia reserves its text against AI use — link it, never copy it.
- Youth: every Oct-2026 addition (squig, pele, gauge-ui, audiocn, mdxcn, Arc)
  was 2–7 weeks old with one maintainer when adopted. Keep the copied source,
  pin versions, and diff before re-pulling.
- Typosquat-shaped traps: `npm i audiocn` and `pele@1.0.0` are unrelated
  packages.

## When not to use

- One asset, known tool → go straight to that skill.
- Dense work surfaces (dashboards, tables) → `rocketride-frontend` picks the
  stack; most of this arsenal is marketing-surface weaponry.
- Restraint is the arsenal's real skill: a hero carries at most one spectacle
  layer (3D *or* animated background *or* video — never two).
