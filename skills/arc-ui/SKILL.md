---
name: arc-ui
description: Install and integrate Arc (uiarc.dev, "Arc UI") — Elia Kuratli's agent-oriented React library of 112 free MIT components + 22 free blocks (56 components / 72 blocks more behind Arc Pro, $129/yr or $199 lifetime), distributed as copy-in source through a shadcn registry (`@uiarc/<id>` → https://uiarc.dev/r/{name}.json) and styled with CSS Modules + its own oklch CSS-variable token layer (not Tailwind classes), animated with `motion/react` — into Next.js 14–16, React 18–19, Tailwind v3.4 or v4 repos, alone or beside shadcn/ui. Use this whenever a task mentions uiarc, uiarc.dev, Arc UI, an `@uiarc/` or `@uiarc-pro/` install, `components/arc/foundation.css` or `@/components/arc/...` imports; whenever an app wants a calm, neutral, Linear-like product UI base as an alternative to shadcn; or whenever it needs a finished product-UI interaction piece — confirm-in-place / hold-to-confirm / morphing delete button, sliding segmented control, KPI metric cards with rolling numbers, icon morph, expanding button group, swipe row actions, ridgeline / streamgraph / waffle / slope / treemap charts, or settings / pricing / dashboard / sign-in blocks with motion. Also use it before letting Arc's own vendor skill near a repo (don't — see below). Install pre-flight, the token-collision bridge with shadcn, and every verified fix live in references/integration.md.
---

# Arc UI (uiarc.dev)

Verified Oct 2026 by real installs and builds (Next 16.4 / React 19 / Tailwind
v4 and Next 14.2 / React 18 / Tailwind 3.4), Playwright screenshots, and a
static scan of all 137 free registry items. Anything not run is marked
*(unverified)*.

Arc is a **complete design system, not a shadcn skin**. Components are
`"use client"` TSX + `.module.css`, reading Arc's own tokens (`--surface`,
`--text-secondary`, `--accent`, `--space-*`, `--radius-control`,
`--font-display` / `--font-body`) from one `foundation.css`. Motion is
`motion/react` only (pulls `motion@^14.3`; zero framer-motion), with shared
springs in `arc-motion-tokens` (`snappy` presses, `smooth` panels, `morph`
shared highlights). License: MIT for free items (© 2026 Elia Kuratli,
`github.com/kuratlielia/arc-library`, a mirror synced from a private repo —
solo maker, young project; you own the copied files). Pro items install only
from the token-gated `@uiarc-pro` registry — never reconstruct Pro source.

## Where it sits in design-arsenal

- **Phase 4 (component base) — alternative to shadcn. Pick one base per app.**
  Arc as base: let Arc own tokens and dark mode; use shadcn only if bridged.
  shadcn as base: Arc pieces are guests and need the token bridge.
- **Phase 6 (motion system)** — cherry-pick interaction pieces; borrow
  `arc-motion-tokens` as a ready "one easing vocabulary".
- vs **componentry**: complementary — componentry is landing spectacle
  (WebGL, kinetic type); Arc is product-UI interaction + data viz.
- vs **motion-primitives**: Arc's text-morph / text-shimmer /
  animated-counter are finished, styled components; motion-primitives are
  primitives you style. Don't ship both for the same job.

## Do NOT install the vendor's `arc-skill`

`npx shadcn add https://uiarc.dev/r/arc-skill.json` drops a skill named `arc`
into `.claude/skills/arc/` (verified: project root, not `~`). Don't. Its
description triggers on generic asks ("settings page, pricing section,
dashboard, landing page, form, table, dialog…") so it hijacks routing from
taste-skill / frontend-design, and it enforces Arc's house copy and style
rules (sentence case only, no eyebrows, no uppercase, no em dashes, weights
400/500 only, no gradients or glows). Those are one brand's choices; project
direction owns them. Read its files on the web
(`https://uiarc.dev/r/skills/arc/<file>.md`) as reference only.

## Install (verified)

```bash
# components.json: "registries": { "@uiarc": "https://uiarc.dev/r/{name}.json" }
npx shadcn@latest add https://uiarc.dev/r/arc-foundation.json   # once per app
npx shadcn@latest add @uiarc/segmented-control @uiarc/confirm-morph -y
```

Which shadcn CLI version to run (especially on Tailwind v3) is the **shadcn**
skill's call — read it before the first Arc install. Files land in
`components/arc/<id>/`; Arc files import each other relatively (no tsconfig
alias needed). Then, once: `import "@/components/arc/foundation.css"` in the
root layout **after** `globals.css`.

## Pre-flight — every item verified to break or bite

1. **Next 16.4 create-next-app config kills all Arc styling.** Its default
   turbopack rule routes `*.css` (modules too) through
   `@tailwindcss/turbopack`; CSS Modules stop being modules. Narrow the rule
   key to `"globals.css"` (exact config in the reference).
2. **Token collision with shadcn.** Arc's `:root` redefines `--background`,
   `--foreground`, `--border`, `--accent`, `--accent-foreground`. Tailwind v3:
   `bg-accent` goes transparent. v4: shadcn's hover tint becomes Arc's brand
   accent. Fix = rename shadcn's five tokens to `--sh-*` (reference).
3. **Dark mode is `data-theme="dark"` on `<html>` only** — no `.dark`, no
   `prefers-color-scheme`. Beside shadcn, toggle both together.
4. **No body font.** Load Geist as `--font-geist`, Inter as `--font-inter`,
   and add `body { font-family: var(--font-body); }` or component text
   inherits browser serif.
5. **`<ArcProvider link={Link}>` must live in a `"use client"` file** —
   in a server layout the build fails. (Only shipped with avatar,
   breadcrumb, hero-section.)
6. **Next 14 template tsconfig needs `"target": "ES2017"`** (animated-counter
   spreads a string; type check fails otherwise).
7. **Blocks reference unshipped assets** — hero-section requests
   `/block-logos/*.svg` (404). Replace sample data/logos before shipping.

## Worth using from the free tier

Product interactions: `confirm-morph`, `hold-to-confirm`, `segmented-control`,
`metric-card` (+`animated-counter`), `icon-morph`, `expanding-button-group`,
`floating-button-group`, `swipe-actions`, `copy-button`, `inline-edit`,
`morph-select`, `password-strength`, `money-input`. Data viz: `ridgeline`,
`streamgraph`, `waffle-chart`, `slope-chart`, `treemap`, `activity-heatmap`,
`sparkline`, `line-chart`, `sortable-data-table`. Blocks: `command-palette`,
`plan-comparison`, `notification-center`, `sign-in`, `stats-band`. Full
list and per-item deps in the reference.

## Reduced motion

All 130 motion-using free items ship a reduced-motion branch (static scan;
metric-card verified at runtime — digits jump to final values). Still check
any block you compose around them, per design-arsenal phase 8.

## When NOT to use

- The app is already a tuned shadcn system and you need one generic
  primitive (dialog, select, tabs) — use shadcn's; two primitive sets is debt.
- The direction wants loud brand expression (eyebrows, uppercase, glow) as
  the base — Arc's neutrals fight it; use Arc only for isolated pieces.
- Landing spectacle → componentry; static backgrounds → haikei / fffuel.
- Pro-only items — don't buy Pro on spec; the free tier covers most needs.
