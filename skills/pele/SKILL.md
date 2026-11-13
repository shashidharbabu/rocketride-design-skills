---
name: pele
description: Render Mermaid diagrams as token-themed SVG with Pele (pele.run) — a free, MIT-licensed, zero-dependency TypeScript library by kepano / the Obsidian team (github.com/obsidianmd/pele) that turns Mermaid syntax into static SVG themed entirely through `--pele-*` CSS variables, covering every diagram type built into Mermaid 12.1.0 (flowchart, sequence, class, state, ER, gantt, git, pie, mindmap, timeline, xychart, sankey, C4, architecture, block, agentflow, wardley, radar, treemap and more; not ZenUML), with server-side `render()` (zero client JS) and client `mount()` (container-width reflow, zoom), in Next 14–16 + React 18–19 + Tailwind 3.4 or v4 + motion v13 (`motion/react`). Trap: the npm name `pele` has an unrelated 2020 publish (`pele@1.0.0`, a LavaMoat experiment) and the real library shipped 14 releases in its first 12 days — always pin an exact current version (`pele@1.6.0` verified). Use this whenever a task involves Mermaid in a React/Next app, "render a mermaid diagram", "replace mermaid.js / mermaid is too heavy", an architecture, flowchart, sequence, gantt, ER, C4, state, sankey or pipeline / agent-flow diagram on a docs, landing, case-study, MDX or blog page, "make the diagrams match the design system", dark-mode diagrams, SSR or static SVG diagrams, a `pre.mermaid` block that needs rendering, a pele.run link, or `import ... from 'pele'`. Token bridge, render-vs-mount rule, per-type imports, and the CI parse-error check live in references/integration.md.
---

# Pele

Verified Oct 2026 (pele 1.6.0 on Next 16.4.2 + React 19.2 + Tailwind 3.4.19,
built, served and screenshotted with Playwright).

Pele is **a Mermaid-to-SVG renderer, not a component library**. It replaces
`mermaid.js`: same diagram language, no runtime dependencies, and the output is
plain SVG whose colours, fonts and radius come from `--pele-*` CSS variables.
That makes diagrams **consumers of your token layer**: map the design system's
tokens once and every diagram follows light/dark with no re-render.
Built by kepano (Steph Ango) for Obsidian; MIT (`LICENSE`: "Copyright (c) 2026
kepano" — GitHub's API mislabels it "Other"). Free, no account.

- **Young and fast-moving**: repo created 2026-10-05, 1.0.1 → 1.6.0 in 12 days.
  Pin exactly; re-run the parse check (below) on every upgrade.
- **Name trap**: `pele@1.0.0` (2020, kumavis) is an unrelated package.
- Docs are served as Markdown for agents — fetch these instead of guessing:
  `https://pele.run/api.md`, `/theming.md`, `/compatibility.md`,
  `/security.md`, `/examples.md`, `/index.md`. There is no `llms.txt`, no shadcn
  registry, no CLI, no MCP server.

## Install

```bash
npm install pele@1.6.0 --save-exact
```

Verified: 0 dependencies, 0 peer dependencies, no install scripts. ESM only
(`"type":"module"`), `engines.node >=20`, types included. No React, Tailwind
or animation-engine coupling, so the Tailwind v3/v4 and `motion` vs
`framer-motion` questions do not arise. (Next 14 / React 18 / Tailwind v4:
unverified, expected to work — nothing in the package depends on them.)

## What exists

| Entry | Use | Client gz (measured) |
|---|---|---|
| `pele` | `render`, `mount`, `parse`, `detectType`, `supports`, `PeleError`, all 33 types | ~251 KB if shipped to the client — don't |
| `pele/core` + `pele/diagrams/<type>` | `register(flowchart, pie)` then `mount` / `render` | core+pie 18.8 KB; +flowchart 50.4 KB |
| `pele/lazy` | `renderAsync`, `load('flowchart')` — per-type on demand (unverified) | — |
| `pele/global` | `<script>` global for `pre.mermaid` (docs only, unverified) | — |

Server-rendered diagrams ship **0 KB** of Pele JS (verified).

## The one decision: server `render()` or client `mount()`

- **Server `render()`** (React Server Component, `dangerouslySetInnerHTML`):
  zero client JS, static. **Does not reflow** — a wide `flowchart LR` measured
  1265×152 and shrank to 292×35 px at a 390 px viewport, unreadable. Use it for
  diagrams that are narrow, vertical (`TD`), or rendered with a mobile-safe
  `maxWidth`.
- **Client `mount()`** (`'use client'`, `useEffect`, `destroy()` on cleanup):
  reads the container width, re-renders on resize, switches wide diagrams to
  vertical under 640 px (`autoDirection`; same flowchart became 290×729 px on
  the phone), re-measures after web fonts load. Use it for wide or important
  diagrams on responsive pages. Import from `pele/core` + per-type files.

Code for both, the token bridge, and the CI check: `references/integration.md`.

## Theming — map tokens, never accept defaults

Pele's defaults are neutral grey plus Tableau-like series colours. Bridge the
design system's tokens onto `.pele` (shadcn v3 HSL mapping verified; v4/oklch
form is the same minus `hsl()`, unverified). Three things people miss:

1. **`--pele-series-1..8` are separate** from the 9 base colours. Unset, pie /
   git / xychart / journey colours stay default blue-orange. shadcn's
   `--chart-1..5` cover 1–5; choose 6–8 yourself. Chart colour judgment
   (palette, contrast, ordering) belongs to **dataviz**.
2. Slice and some shape labels use `--pele-bg` as text colour — series colours
   must contrast with the background **in both modes**.
3. Map `--pele-accent` to shadcn `--primary`, not `--accent` (shadcn's
   `--accent` is a hover surface, not a brand hue).

## When to use

- Architecture, data-flow, sequence, ER, state, C4 diagrams on docs, landing,
  case-study (**pafolios**) or MDX pages.
- Pipeline and agent diagrams — `agentflow-beta` / `architecture-beta` fit
  RocketRide pipelines (types verified in docs, not rendered here).
- Replacing `mermaid` (~1.6 MB gz per Pele's own benchmark) anywhere.

## When NOT to use

- **Real dashboard charts** with interaction, tooltips, live data — that is a
  charting library's job; **dataviz** owns the form and colour decisions.
  Pele's pie / xychart are fine inside prose, not as a product surface.
- **Decorative SVG** — waves, blobs, grain: **haikei** / **fffuel**.
- **Diagrams authored by untrusted users** without reading `security.md`:
  keep `options` server-controlled, show errors via `textContent`, supply
  icons only from a trusted set.
- **ZenUML or Mermaid plugins** — unsupported (`supports('zenuml') === false`).

## Anti-patterns (each one observed in testing)

- Pinning `pele@1.0.0` or a caret range on a 2-week-old library.
- Shipping the `pele` root entry in a client component — use `pele/core` +
  per-type imports.
- Hex colours in `style` / `classDef` lines: they stay fixed in dark mode
  (verified). Style via CSS on `.pele-node[data-id="x"]` instead.
- Trusting a green build: an unquoted `E[shadcn@2.3.0]` throws `PeleError`
  (`syntax`) and the build still passes, so the page silently shows an error.
  Quote labels containing `@` and run the parse check from the reference
  in CI.
- Server `render()` of a wide `LR` diagram on a responsive page with no
  `maxWidth`.
- Several server-rendered diagrams on one page without a unique `idPrefix` each
  (`mount()` handles this itself).
- Leaving `mount()`'s default `zoom: 'auto'` on — its +/− buttons sat on top of
  the last node in testing. Pass `zoom: false` or pad the container.

## Where it sits

**design-arsenal phase 4 (component base)**: like shadcn, a consumer of the
phase-3 tokens, remapped before the first diagram lands. Phase 8 (QA): series
contrast in both modes, 390 px check on every wide diagram, parse check green.
Owners next door: **dataviz** (chart form, palette, series colour), **shadcn**
(the token names being bridged), **pafolios** (where case-study diagrams go).
