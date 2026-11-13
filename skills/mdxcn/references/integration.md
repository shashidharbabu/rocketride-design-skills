# mdxcn integration: verified fixes and traps

Verified Oct 2026 on two builds:
- Next 16.4.2 / React 19.3 / Tailwind v4 / shadcn 4.22.1, with Turbopack
- Next 14.2.35 / React 18 / Tailwind 3.4.1

Anything not exercised is marked **(unverified)**.

## 1. CLI install: the broken re-export (Tailwind v4, shadcn 4.x)

The registry source imports `@/registry/default/<item>/<file>`, and the items declare no `target`. The CLI therefore flattens the files by type:

| Registry file | Lands at |
|---|---|
| `graph-frame/graph-frame.tsx` | `components/ui/graph-frame.tsx` |
| `graph-frame/graph-markdown.ts`, `graph-motion.ts`, `graph-clock.ts` | `lib/` |
| `<item>/<item>.tsx` | `components/<item>.tsx` |
| `mdx/mdx.tsx` | `lib/mdx.tsx` |
| `mdx/mdx-footnotes.tsx` | `components/mdx-footnotes.tsx` |

The CLI rewrites `import … from` correctly but **not `export … from`**. Two re-exports in `components/ui/graph-frame.tsx` end up as `@/components/graph-frame/graph-markdown`, a path that doesn't exist. `next build` then fails with `Module not found: Can't resolve '@/components/graph-frame/graph-markdown'`.

Fix, then check:

```bash
perl -pi -e 's#@/components/graph-frame/graph-markdown"#@/lib/graph-markdown"#g' components/ui/graph-frame.tsx
grep -rnE 'from "@/(registry/default|components/[a-z-]+/[a-z-]+)"' components lib   # expect no output
npx tsc --noEmit
```

Comment lines that mention `@/registry/default` (for example the docstring in `annotate.tsx`) are harmless.

Two more things to know:
- **Every item re-ships the shared `graph-frame` files.** Older CLIs stop on an "already exists, overwrite?" prompt when several items go in one `add`. Pass `-o` **only** before you have customised `graph-frame.tsx`.
- **The first `add` once hit a 10 s connect timeout to mdxcn.dev.** A plain retry worked.

## 2. Many items, Comark or Knap: use the verbatim layout, never `all.json`

`npx shadcn add https://mdxcn.dev/r/all.json` produced **60 TypeScript errors**. Generic file names (`lib/index.ts`, `lib/filters.ts`, `lib/props.ts`, `lib/frame.ts` …) collide, and relative imports break.

**What builds:** write each item's `files[].content` to its declared `registry/default/**` path. The source's own `@/registry/default/...` imports then resolve through the stock `"@/*": ["./*"]` tsconfig alias, and imports match the upstream docs exactly. A minimal fetch-and-write script (Node 18+, run from the project root):

```js
// node mdxcn-verbatim.mjs <slug> [<slug> ...]
import { mkdirSync, writeFileSync } from "node:fs";
import { dirname, resolve, sep } from "node:path";
const root = process.cwd();
for (const slug of process.argv.slice(2)) {
  const item = await (await fetch(`https://mdxcn.dev/r/${slug}.json`)).json();
  for (const f of item.files) {
    if (!f.path.startsWith("registry/default/")) throw new Error(f.path);
    const out = resolve(root, f.path);
    if (!out.startsWith(root + sep)) throw new Error("escapes project: " + f.path);
    mkdirSync(dirname(out), { recursive: true });
    writeFileSync(out, f.content);
  }
  console.log(slug, item.files.length, "files; deps:", item.dependencies ?? []);
}
```

How the full set behaved:
- **`all.json` omits `registry/default/graph-knap/markdown.ts`.** Verbatim `all` alone gave 1 error. Running `all` **plus `graph-knap`** built clean: 66 files, about 520 KB of source.
- **`knap` appears only in a docstring**, so the build passed without it. Install the `knap` package only if you actually wire the Knap filters (unverified beyond the build).
- **The verbatim route skips the registry's `cssVars` and `css`.** Copy the variables and utilities from §3 or §4 by hand. Without them the frames have no dashed border. (Inferred: the verbatim build was tested in a project that already had the CSS.)
- **You also add `motion` yourself:** `npm i motion`.

## 3. Tailwind v3 port

Which shadcn CLI to run on a v3 project is the **shadcn** skill's decision. Whatever writes the files, check all of the following.

**a. Import paths.** shadcn@2.3.0 mangled nearly every cross-file import, for example `@/components/graph-frame/graph-frame` and `@/components/callout/callout`. Map them to where the files actually landed:

| Wrong | Right |
|---|---|
| `@/components/graph-frame/graph-frame` | `@/components/ui/graph-frame` |
| `@/components/graph-frame/graph-markdown`, `@/registry/default/graph-frame/graph-markdown` | `@/lib/graph-markdown` |
| `@/components/graph-frame/graph-motion` | `@/lib/graph-motion` |
| `@/components/<x>/<x>` (callout, quote, terminal) | `@/components/<x>` |
| `@/components/mdx/mdx-footnotes` | `@/components/mdx-footnotes` |

The verbatim layout from §2 avoids this whole class of bug.

**b. Colours.** The v3 CLI writes the `:root` / `.dark` variables but **silently drops** the `@theme` mapping. Add the mapping to `tailwind.config.ts` under `theme.extend.colors`. The values are full `oklch()` colours, so **no `hsl()` wrapper**. Opacity modifiers like `/50` won't work on these, and none of the components use them (verified by grep).

```ts
'graph-accent': 'var(--graph-accent)', 'graph-accent-2': 'var(--graph-accent-2)',
'graph-accent-3': 'var(--graph-accent-3)', 'graph-frame': 'var(--graph-frame)',
'graph-muted': 'var(--graph-muted)', 'graph-faint': 'var(--graph-faint)',
'contrast-14': 'var(--contrast-14)', 'contrast-23': 'var(--contrast-23)',
'contrast-45': 'var(--contrast-45)', 'contrast-70': 'var(--contrast-70)',
```

**c. Utilities.** The registry's `@utility` blocks are v4-only and are also **silently dropped**. Port them into `globals.css`:

```css
@layer utilities {
  .graph-frame {
    background-image:
      repeating-linear-gradient(to right, var(--graph-frame) 0 2px, transparent 2px 7px),
      repeating-linear-gradient(to bottom, var(--graph-frame) 0 2px, transparent 2px 7px),
      repeating-linear-gradient(to right, var(--graph-frame) 0 2px, transparent 2px 7px),
      repeating-linear-gradient(to bottom, var(--graph-frame) 0 2px, transparent 2px 7px);
    background-repeat: repeat-x, repeat-y, repeat-x, repeat-y;
    background-position: 0 0, 100% 0, 0 100%, 0 0;
    background-size: 100% 1px, 1px 100%, 100% 1px, 1px 100%;
  }
  .graph-rule { height: 1px; background-image: repeating-linear-gradient(to right, var(--graph-frame) 0 2px, transparent 2px 7px); }
  .graph-rule-y { width: 1px; background-image: repeating-linear-gradient(to bottom, var(--graph-frame) 0 2px, transparent 2px 7px); }
}
```

**d. TypeScript target.** Next 14's default tsconfig has no `target`. `annotate.tsx` spreads a `Set`, so the build fails with `Type 'Set<number>' can only be iterated…`. Fix it with `"target": "ES2017"` in `compilerOptions`.

**e. Base tokens.** On the test project, `shadcn@2.3.0 init` wrote **oklch** values but kept `hsl(var(--x))` in `tailwind.config`. In dark mode that left no page background, made swatches disappear, and drew dashed lines through every frame title (see §6). This is a shadcn problem, owned by the **shadcn** skill; mdxcn only exposes it. If `bg-background` renders transparent, check this first.

Once a–e were done, the v3 build matched v4 in light and dark.

## 4. Remap the theme onto the design system

Install puts the 10 variables at the end of `:root` and `.dark` (oklch defaults: a blue accent and neutral greys). Point them at the design system's tokens; doing that is the whole theming API:

```css
:root  { --graph-accent: var(--brand-600); --graph-frame: color-mix(in oklch, var(--ink) 28%, transparent); }
.dark  { --graph-accent: var(--brand-400); --graph-frame: var(--line-strong); }
```

(The token names above are examples. `color-mix` support is unverified in older targets.)

- **Fonts:** the frames use `font-mono text-sm`, so set the system's mono family on Tailwind's `--font-mono` / `fontFamily.mono`.
- **Contrast:** `--graph-muted` defaults to the **same** value in both themes. Contrast-check it against the dark background.

## 5. MDX wiring

```bash
npm i @next/mdx @mdx-js/loader @mdx-js/react @types/mdx remark-gfm
```

Next 16 with Turbopack (`next.config.ts`). **Plugins must be strings**, because Turbopack can't serialize imported functions:

```ts
import createMDX from "@next/mdx";
const withMDX = createMDX({ options: { remarkPlugins: ["remark-gfm"] } });
export default withMDX({ pageExtensions: ["ts", "tsx", "md", "mdx"] /* …existing config */ });
```

Next 14 with webpack (`next.config.mjs`) takes the imported function: `import remarkGfm from "remark-gfm"` → `remarkPlugins: [remarkGfm]`.

`mdx-components.tsx` at the project root. This file runs on the server and needs no `"use client"`; the figures inside are client components.

```tsx
import type { MDXComponents } from "mdx/types";
import { withMdxcn } from "@/lib/mdx";              // verbatim layout: "@/registry/default/mdx/mdx"
import { Callout } from "@/components/callout";
import { Steps } from "@/components/steps";
export function useMDXComponents(components: MDXComponents): MDXComponents {
  return withMdxcn({ ...components, Callout, Steps }); // register every parent tag you use
}
```

How it behaved:
- **Rendering:** `page.mdx` routes prerendered as static (○) on both builds.
- **`withMdxcn` options:** `{ alerts, quotes, terminals, footnotes }`, all on by default. Turn off any you style yourself.
- **Upgrades:** `> [!TIP]` and ```` ```console ```` were upgraded on the test page. Plain fences (```` ```ts ````) stay plain `<pre>`, with no highlighting.

## 6. Title patches vs glass, gradient and texture

`components/ui/graph-frame.tsx` hides the dashed border behind the `[ TITLE ]` and the corner glyphs by painting them with **`bg-background`**. The title is `absolute … bg-background px-2.5`.

- **On a glass, gradient, image or grain surface:** each title becomes an opaque rectangle.
- **If `--background` is broken or transparent:** the dashed line runs straight through the title. Both builds showed this in testing.

Options, in order:
1. **Put figures only on solid `--background` sections.** This is the default for docs pages.
2. **Solid panel in a different colour:** redefine the variable on the wrapper so the patches match:
   ```tsx
   <div className="bg-[var(--panel)] [--background:var(--panel)]">…figures…</div>
   ```
   (Unverified. It should cascade on both v4 and v3 configs, since both resolve `bg-background` through `var(--background)`.)
3. **True glass:** edit `graph-frame.tsx` so the border breaks around the title instead of being painted over, for example by splitting the top edge into two `graph-rule` segments either side of the title. You own the source; record the change in the decisions log. (Unverified.)

Never fix it by giving the title `bg-transparent` alone. That reproduces the struck-through title.

## 7. QA: scroll before you screenshot

Every figure animates in with motion `whileInView` (`initial="hidden"`, `viewport.once`). In a full-page capture taken without scrolling, **11 figure elements stayed at opacity 0**: blank frames. Scrolling through first left 0 at opacity 0. The same applies to print and PDF export (inferred).

```js
const h = await page.evaluate(() => document.body.scrollHeight);
for (let y = 0; y <= h; y += 300) { await page.evaluate((v) => scrollTo(0, v), y); await page.waitForTimeout(150); }
await page.evaluate(() => scrollTo(0, 0)); await page.waitForTimeout(800);
await page.screenshot({ path, fullPage: true });
```

Alternative (unverified): `page.emulateMedia({ reducedMotion: "reduce" })` should render everything at once, because all 8 tested components call `useReducedMotion()` and skip the hidden initial state. Use that as the design-critic default if it holds.

## 8. Annotate: inline code breaks the notes

In `<Annotate>`, a note like ``1. Becomes the `bg-graph-accent` utility.`` renders as stacked lines: "Becomes the" / `bg-graph-accent` / "utility." The cause is that `itemParts()` treats each inline element after the first as a body block.
- Keep the notes as **plain text**.
- Or fix it in `annotate.tsx` by passing the whole `<li>` children rather than `[head, ...body]` (unverified).
- Callout, Steps and Spec render inline code correctly.

## 9. Minor

- **`graph-scroll-x` is undefined.** It is used 9 times (Terminal, Annotate, GraphTable …) but no registry CSS defines it. Narrow viewports may overflow. Add `.graph-scroll-x { overflow-x: auto; }` if they do (unverified as to the intended styling).
- **Weight:** the test route (6 figures, Next 14) was 59.4 kB of page JS and 147 kB first load. `graph-markdown.ts` alone is 22 KB of source.
- **Copying from upstream:** the docs and `https://mdxcn.dev/llms.txt` give import paths as `@/registry/default/...`. That is correct only for the verbatim layout. Rewrite the paths after a CLI install.
- **Other public endpoints:** a no-auth read API (`/openapi.json`, `/api/v1/components`, 1000 GET/h) and `https://mdxcn.dev/llms.txt`. No MCP server exists.
