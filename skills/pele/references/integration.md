# Pele integration reference

Verified Oct 2026 against pele 1.6.0, Next 16.4.2 (Turbopack), React 19.2.0,
Tailwind 3.4.19, TypeScript 5.7. Each block is marked **verified** (built,
served, checked in headless Chromium) or **unverified** (taken from the docs
or inferred). Live API: `https://pele.run/api.md`, `https://pele.run/theming.md`.

## 1. Token bridge (CSS)

Set the bridge on `.pele`, which is the class on every diagram's root `<svg>`.
Swapping colours costs nothing: on the dark-mode toggle the SVG markup stayed
byte-identical while the computed node fill changed (verified).

### shadcn on Tailwind v3 (`shadcn@2.3.0`, bare HSL triplets): verified

```css
/* globals.css: after the shadcn :root / .dark token blocks */
.pele {
  --pele-bg:          hsl(var(--background));
  --pele-fg:          hsl(var(--foreground));
  --pele-muted:       hsl(var(--muted-foreground));
  --pele-line:        hsl(var(--foreground));      /* edges + arrowheads */
  --pele-surface:     hsl(var(--card));            /* node fill */
  --pele-surface-alt: hsl(var(--muted));           /* subgraph fill */
  --pele-border:      hsl(var(--border));
  --pele-accent:      hsl(var(--primary));         /* NOT --accent (a hover surface) */
  --pele-critical:    hsl(var(--destructive));     /* gantt crit, kanban high */
  --pele-error:       hsl(var(--destructive));     /* mount() error text */
  --pele-radius:      var(--radius);
  --pele-font:        var(--font-sans);            /* same family you pass to render() */
  --pele-font-mono:   var(--font-mono);

  /* Charts are NOT covered by the vars above. Unset = default blue/orange. */
  --pele-series-1: hsl(var(--chart-1));
  --pele-series-2: hsl(var(--chart-2));
  --pele-series-3: hsl(var(--chart-3));
  --pele-series-4: hsl(var(--chart-4));
  --pele-series-5: hsl(var(--chart-5));
  /* shadcn stops at chart-5: choose 6-8 yourself (dataviz skill), or they
     fall back to Pele defaults #b279a2 / #eeca3b / #9d755d. */
  --pele-series-6: hsl(var(--chart-6, 280 30% 55%));
  --pele-series-7: hsl(var(--chart-7, 45 80% 55%));
  --pele-series-8: hsl(var(--chart-8, 25 25% 45%));
}
```

Because the bridge reads the tokens through `var()`, the existing `.dark { ... }`
token overrides flow through on their own. You do not need a second `.pele`
block for dark mode.

### shadcn on Tailwind v4 (`shadcn@latest`, full oklch values): unverified

Same mapping without the `hsl()` wrapper, because v4 tokens are complete colours:

```css
.pele {
  --pele-bg: var(--background);
  --pele-fg: var(--foreground);
  --pele-accent: var(--primary);
  --pele-series-1: var(--chart-1);
  /* ...same pattern for the rest */
}
```

### A bespoke DS (`--ds-*` or `cv-` style tokens): verified

Map by meaning, the same way, e.g. `--pele-surface: var(--ds-surface)`.

### Contrast rules (verified by eye on screenshots)

- Pie slice labels and some shape labels use `--pele-bg` as their text colour.
  Check every series colour against `--pele-bg` in **both** modes.
- Hex values in diagram source (`style`, `classDef`, `linkStyle`) are frozen and
  do not follow the theme. Instead, target nodes from CSS:

```css
.pele-node[data-id="motion"] :is(rect, path, polygon) { fill: hsl(var(--primary)); }
.pele-node[data-id="motion"] .pele-label { fill: hsl(var(--primary-foreground)); }
```

The class hooks are `pele-<type>` on the root, plus `pele-node`, `pele-edge`,
`pele-edge-label`, `pele-cluster`, `pele-label`, `pele-marker`, `pele-title` and
`pele-icon`.

## 2. Server `render()`: React Server Component (verified)

Ships 0 KB of Pele JavaScript. Rendering happens at build or request time, and
in Node, label widths come from Pele's built-in width table because there is no
canvas.

```tsx
// components/diagram.tsx  (no 'use client')
import { render, PeleError } from 'pele';

const FONT = "var(--font-sans), ui-sans-serif, system-ui, sans-serif"; // match --pele-font

export function Diagram({ source, id, maxWidth }: { source: string; id: string; maxWidth?: number }) {
  try {
    const { svg } = render(source, {
      idPrefix: id,          // unique per diagram on the page (a11y title/desc ids)
      fontFamily: FONT,      // the same family as --pele-font, or labels overflow
      maxWidth,              // e.g. 360 to force a phone-safe layout
    });
    return <div className="w-full" dangerouslySetInnerHTML={{ __html: svg }} />;
  } catch (e) {
    if (!(e instanceof PeleError)) throw e;
    // the marker attribute is what the CI check greps for (section 6)
    return <pre data-pele-error="" className="text-destructive">{e.message}</pre>;
  }
}
```

Notes:
- Gantt "today" markers use the current time. Pass `now: new Date(...)` to make
  static builds deterministic (verified option).
- `render()` returns `{ type, svg, width, height, links }` (verified).
- React escapes `{e.message}` as text. That matters because error messages
  contain diagram source (see security.md).

## 3. Client `mount()`: responsive diagrams (verified)

```tsx
'use client';
import { useEffect, useRef } from 'react';
import { mount, register } from 'pele/core';
import flowchart from 'pele/diagrams/flowchart';
import sequence from 'pele/diagrams/sequence';

register(flowchart, sequence); // only the types this page renders

export function LiveDiagram({ source }: { source: string }) {
  const ref = useRef<HTMLDivElement>(null);
  useEffect(() => {
    if (!ref.current) return;
    const d = mount(ref.current, source, {
      zoom: false,                       // default 'auto' overlays +/- on the diagram
      onError: (err) => console.error(err), // mount() never throws; it draws the error in the container
    });
    return () => d.destroy();
  }, [source]);
  // a block element whose width does NOT depend on the diagram
  return <div ref={ref} className="block w-full" />;
}
```

- `Mounted` exposes `update(text)`, `refresh()` (call it after changing fonts or
  moving the container), `destroy()`, `error`, `zoom` and `result` (verified
  from the `.d.ts`).
- `mount()` reads `--pele-font` / `--pele-font-mono` from CSS and redraws once
  pending web fonts load (docs).
- Measured client cost, gzipped: core+pie 18.8 KB; core+pie+flowchart 50.4 KB.
  Never import the root `pele` in a client file, because it pulls in every
  type (about 251 KB gz).
- `pele/lazy` is unverified: `renderAsync(source)` loads each type on first
  use, and `await load('flowchart')` preloads one.
- `onError?: (error: unknown) => void` and `sanitize?: (svg: string) => Node`
  are in `MountOptions` (verified from the `.d.ts`). `sanitize` must return a
  DOM `Node`, not a string. Neither callback was exercised at runtime.

## 4. Choosing between them: maxWidth and reflow

| Diagram | Use |
|---|---|
| Vertical (`TD`), sequence, small pie or gantt | Server `render()` |
| Wide `LR` flowchart, architecture, C4, block on a responsive page | Client `mount()` |
| Wide and must be zero-JS | Server `render(…, { maxWidth: 360 })`, then check it at desktop too |

Measured at a 390 px viewport, using the same `flowchart LR` with 10 nodes:
- server `render()` with no `maxWidth`: viewBox 1265×152, displayed at 292×35 px. Unreadable.
- client `mount()`: re-laid out to 290×729 px, vertical, readable.

`autoDirection` (default `true`) switches a layout to vertical below
`directionBreakpoint` (default 640). Pass `autoDirection: false` to keep the
source direction. `parse()` once and `render(model, { maxWidth })` at several
widths if you build your own responsive SSR (docs, unverified).

## 5. Fonts

- With server `render()`, pass `fontFamily` and set the **same** family on
  `--pele-font`. Nodes are sized from the measured family. If the browser
  paints a different one, labels overflow (docs).
- With `next/font`, expose the variable (`--font-sans`) and use it in both places.
- A custom `measurer: { width(text, size, style) }` is supported (docs, unverified).

## 6. Parse-error check (CI): verified

A broken diagram does **not** fail `next build`. The page ships an error block
instead. Verified trigger: an unquoted `@` in a label, `B[shadcn@2.3.0]`, throws
`PeleError` code `syntax`; quoting it as `B["shadcn@2.3.0"]` fixes it. Add both
checks below.

**a. Source check.** Covers `.mmd` files and ` ```mermaid ` fences in `.md`/`.mdx`:

```js
// scripts/check-diagrams.mjs  ->  "check:diagrams": "node scripts/check-diagrams.mjs ."
import { readFileSync, readdirSync, statSync } from 'node:fs';
import { join, extname } from 'node:path';
import { render, PeleError } from 'pele';

const skip = new Set(['node_modules', '.next', '.git', 'dist', 'out']);
const files = [];
(function walk(dir) {
  for (const name of readdirSync(dir)) {
    if (skip.has(name)) continue;
    const p = join(dir, name);
    if (statSync(p).isDirectory()) walk(p);
    else if (['.mmd', '.md', '.mdx'].includes(extname(p))) files.push(p);
  }
})(process.argv[2] ?? '.');

let failures = 0, count = 0;
for (const file of files) {
  const text = readFileSync(file, 'utf8');
  const sources = extname(file) === '.mmd' ? [text]
    : [...text.matchAll(/```mermaid\n([\s\S]*?)```/g)].map((m) => m[1]);
  for (const src of sources) {
    count++;
    try { render(src); } catch (e) {
      if (!(e instanceof PeleError)) throw e;
      failures++;
      console.error(`${file}: [${e.code}] ${e.message.split('\n')[0]}`);
    }
  }
}
console.log(`pele: ${count} diagram(s) checked, ${failures} failed`);
process.exit(failures ? 1 : 0);
```

Verified output: `content/sample.mdx: [syntax] Parse error on line 2:` followed
by `pele: 3 diagram(s) checked, 1 failed`, exit 1.

**b. Build-output check.** Covers diagrams defined as strings in TS. It relies
on the `data-pele-error` marker from section 2, and only sees statically
prerendered routes:

```bash
! grep -rl 'data-pele-error' .next/server/app --include='*.html'
```

Verified: exit 1 with the broken label, exit 0 after quoting it.

## 7. Limits and security (docs)

- Default limits: 50,000 source characters (`limit`), 4,000,000 SVG characters
  (`outputLimit`), 5,000 flowchart/agentflow edges (`maxEdges`). For a time
  limit, render in a worker you can terminate.
- Verified in the probe: an HTML label containing `<img onerror>` produced no
  `<img>`, `onerror` or `<script>` in the SVG. A `click A "javascript:..."`
  produced no links, and `click A callback` was ignored.
- Allowed link schemes default to http, https, mailto and tel (`linkSchemes`);
  image schemes default to http and https (`imageSchemes`). Links get
  `rel="noopener"`.
- An `icons` resolver's markup is inserted **unfiltered**. Return markup only
  from a trusted set, and `null` for unknown names.
- Never let diagram authors supply `options` or `options.config`. To run your
  own sanitizer, pass `sanitize` to `mount()`.

## 8. Diagram types

All built-in Mermaid 12.1.0 types (from `compatibility.md`): flowchart/graph,
swimlane-beta, sequenceDiagram, classDiagram, stateDiagram(-v2), erDiagram,
gantt, gitGraph, pie, mindmap, kanban, timeline, journey, quadrantChart,
xychart, requirementDiagram, sankey, architecture-beta, C4*, block,
agentflow-beta, railroad-*, treeView-beta, cynefin-beta, wardley-beta,
eventmodeling, usecase-beta, venn-beta, ishikawa, packet, radar-beta, treemap,
info.

Rendered in testing: flowchart, sequence, gantt, pie. The rest are unverified
here; preview them at `https://pele.run/examples/<type>` or in the
`https://pele.run/playground`. Only ZenUML and external plugins are unsupported
(`supports('zenuml')` returns `false`, verified).
