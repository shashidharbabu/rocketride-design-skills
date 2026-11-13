---
name: mdxcn
description: Install and use mdxcn (mdxcn.dev): a free, MIT, very young (v0.0.1, Aug 2026) library by Keshav Bagaade under the community "Shadcn Labs" org, which is not affiliated with shadcn. It has 51 shadcn-registry items that draw ASCII-styled "figures" for MDX. Each figure is a dashed monospace frame with a [ TITLE ] on the top edge, filled by plain Markdown written inside the tag. The set includes Callout, Steps, Terminal, Annotate, Changelog, Decision, Env, Endpoint, Faq, Keys, and about 34 Graph* data figures such as Table, Spec, Timeline, Flow and Diff. A withMdxcn() wrapper turns GitHub alerts, console fences and footnotes into framed versions, and the same figures exist as fenced ASCII blocks for READMEs. It is NOT a docs-site kit: there are no tabs, no syntax highlighting, no colour swatches, no props tables and no live previews, so hand-built specimen components still carry those. Key trap: install single items with the shadcn CLI after registering the namespace (`npx shadcn@latest registry add @mdxcn=https://mdxcn.dev/r/{name}.json`), never all.json. The CLI flattens the files and breaks re-export paths; all.json gives 60 type errors and is missing a file. Use this whenever a task involves mdxcn.dev or an @mdxcn/ install command; documenting a design system or writing its docs pages; a changelog page, ADR or decision record, install steps, an env-var table or API endpoint doc in MDX; callouts or admonitions, GitHub alerts or terminal blocks in MDX; an ASCII figure for a README or PR; "framed figure", GraphTimeline, GraphFlow, Comark or Knap; or making a write-up scan faster with a figure. Install fixes, Tailwind v3 port and surface/QA traps are in references/integration.md, and the figure chooser is in references/chooser.md.
---

# mdxcn

Verified Oct 2026 by building it twice:
- Next 16.4 / React 19 / Tailwind v4 / shadcn 4.22
- Next 14.2 / React 18 / Tailwind 3.4

Light and dark screenshots were checked on both.

mdxcn is **copy-in source distributed through the shadcn registry**. There is no npm package.

- **Maker:** Keshav Bagaade (@kshvbgde).
- **Repo:** `github.com/shadcn-labs/mdxcn`. "Shadcn Labs" says it is not endorsed by or affiliated with shadcn.
- **License and maturity:** MIT, free, no auth, about 7 weeks old at verification. Expect churn; you own the copied source.

**The only npm dependency is `motion`**, which the CLI installs. It matches our `motion/react` stack.

**Implicit requirements:**
- a shadcn-initialised project: `cn` in `@/lib/utils`, plus `bg-background`, `text-foreground` and `text-destructive`
- `remark-gfm`, for tables, task lists and footnotes inside the tags

It does **not** need next-mdx-remote, fumadocs, contentlayer, shiki or rehype-pretty-code. Any host with an `mdx-components.tsx` works. Fumadocs and Nextra maps are claimed to work through `withMdxcn` (unverified).

## Where it sits

This skill is design-arsenal's **Documentation phase**:
- **mdxcn carries the narrative docs layer:**
  - install Steps
  - Env tables
  - Changelog
  - Decision (ADRs)
  - Spec and Table token references
  - Callouts
  - Terminal sessions
- **Hand-built components carry the specimen layer:** swatch grids, type ramps, live component previews, props tables.

mdxcn has none of those specimen pieces. Don't stretch it to fake them.

## Install (verified path)

CLI version policy (Tailwind v3 vs v4, which `shadcn@` to run) belongs to the **shadcn** skill. Read it first.

1. **Register the namespace once.** It is not in shadcn's built-in index; unregistered, the CLI fails with `Unknown registry "@mdxcn"`.
   ```bash
   npx shadcn@latest registry add @mdxcn=https://mdxcn.dev/r/{name}.json
   ```
2. **Add single items** (names in `references/chooser.md`). Full URLs also work: `https://mdxcn.dev/r/<slug>.json`.
   ```bash
   npx shadcn@latest add @mdxcn/callout @mdxcn/steps @mdxcn/graph-table @mdxcn/mdx
   ```
3. **Run the post-install fix and check** (`references/integration.md` §1):
   - The CLI flattens the files into `components/`, `components/ui/graph-frame.tsx` and `lib/graph-*.ts`.
   - It leaves 2 `export … from` lines pointing at a path that doesn't exist, which breaks `next build`.
   - Patch them, then run `npx tsc --noEmit`.
   - **Upstream docs say imports land under `@/registry/default`. With the CLI they don't.** Import from where the files actually landed: `@/components/callout`, `@/lib/mdx`.
4. **Wire MDX** (`references/integration.md` §5):
   - `@next/mdx` with remark-gfm. On Turbopack, the plugin must be the **string** `"remark-gfm"`.
   - `mdx-components.tsx` returns `withMdxcn({...components, Callout, Steps, …})`. Register each parent tag once.
5. **Need many items, or Comark/Knap?** Use the verbatim `registry/default/**` layout (§2), not `all.json`.

## Writing with it

Write Markdown **inside** the tag. Don't pass array props when the data is prose.

```mdx
<Steps title="ADOPT">

1. Copy the variables

   Paste the `:root` and `.dark` blocks into `globals.css`.

2. **Map them in the theme**

   Bold marks the current step.

</Steps>

> [!WARNING]
> withMdxcn turns this GitHub alert into a framed Callout. GitHub still renders it natively.
```

The grammar is shared by every figure:

| Markdown | Meaning |
|---|---|
| `**bold**` | now / chosen / total, drawn in the accent |
| `*italic*` | next / rejected, dimmed |
| `- label: value` | one row |
| `x — note` | a caption |
| a table whose last row is `**Total**` | the table gets a footer |
| a code line ending `// (1)` | an Annotate marker |

House rules, from upstream:
- At most two figures per section, with prose between them.
- Titles are 1–2 uppercase words.
- One accent by default.
- Don't restyle the frame and don't add motion; it fades in by itself.

## Theming

Install adds 10 oklch variables on `:root` and `.dark`:
- `--graph-accent`, `--graph-accent-2`, `--graph-accent-3`
- `--graph-frame`, `--graph-muted`, `--graph-faint`
- `--contrast-14`, `--contrast-23`, `--contrast-45`, `--contrast-70`

On Tailwind v4 it also adds `@theme inline` mappings and 3 `@utility` blocks (`graph-frame`, `graph-rule`, `graph-rule-y`).

**Remap `--graph-accent` and `--graph-frame` onto the design system's tokens** before the first page ships. Everything else about the look is fixed: `font-mono text-sm`, the dashed frame, `[ TITLE ]`. Dark mode uses the shadcn `.dark` class (verified).

## When NOT to use

- **Specimen content** (swatches, type ramps, component previews with code tabs). Build those by hand.
- **Real charts.** mdxcn charts are low-fidelity glyphs; the **dataviz** skill owns charts.
- **Glass, gradient or textured surfaces.** Each frame title sits on a solid `bg-background` patch that hides the border (§6). Put figures on plain `--background` sections only, or patch the frame.
- **Hosts without MDX, Comark or Knap**, such as a dependency-free hand-written `lib/markdown.tsx`. Use the official fenced-ASCII versions from `https://mdxcn.dev/llms.txt` instead of JSX.
- **A brand direction that fights a monospace terminal look.** It fits brutalist or industrial systems (**brutalist-skill**) and looks out of place on soft or editorial ones.

## Non-negotiables

Details are in `references/integration.md`.

- Never `add all.json` through the CLI.
- Always run `tsc --noEmit` after any mdxcn add.
- On Tailwind v3, port the dropped colours, utilities and TS target by hand (§3). The CLI drops them **silently**.
- **QA screenshots must scroll the page first.** Every figure fades in with `whileInView`, so a static capture or a PDF shows blank frames (§7).
- Keep Annotate notes to plain text. Inline code breaks a note into separate lines (§8).
- Read the copied source after install; you own it. Re-adding with `-o` overwrites your token remap, so diff first (**shadcn** skill).
