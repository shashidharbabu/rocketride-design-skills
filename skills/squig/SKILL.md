---
name: squig
description: Wireframe page and app structure with Squig (squig.sh) — Pablo Stanley's free, MIT-licensed, agent-native wireframing canvas where 155 real UI components (the shadcn/ui vocabulary plus marketing blocks, app blocks, mobile screens and full templates) render as hand-drawn rough.js sketches, driven by an agent through window.squig in the open browser tab, a local CLI, or a local MCP/HTTP companion that owns one .squig.json file and renders PNG/SVG back for inspection. It produces structure (geometry, component props, named side-by-side variations), never production React code. Key trap — it is NOT installable the usual ways: there is no npm package (npx squig 404s), no shadcn registry (/r/*.json 404s), no install script, and the hosted https://squig.sh/mcp endpoint is retired (HTTP 410); local agent use needs a persistent git clone built with Node 24+ and pnpm 10. Use this whenever someone says "wireframe this", "sketch the layout", "lo-fi mockup", "napkin sketch", "block out the page", "show me three layout directions", "compare layouts side by side", "what goes where before we pick colors", "information architecture for this page", or names squig, squig.sh or a .squig.json file; whenever design work starts on a new page or app and the structure is not yet decided (design-arsenal's Structure phase, between Direction and Identity); and whenever a chosen wireframe must be handed off to shadcn components — the kind-to-`npx shadcn add` table and the CLI/HTTP/MCP recipes live in references/integration.md.
---

# Squig

Verified Oct 2026 (repo HEAD `85f92ab`); everything was run end to end unless
marked **unverified**. A **wireframing canvas, not a component library**: real
UI components render as napkin sketches so people argue about structure, not
corner radius. Free, MIT, no account or key. Source:
`github.com/pablostanley/squig`; the workflow adapts upstream
`plugins/squig/skills/wireframe-first` (MIT © 2026 Pablo Stanley). Output is a
`.squig.json` (geometry, props, variations) plus PNG renders — it never touches
the target repo's `package.json`, so Tailwind v3/v4 and React 18/19 don't matter.

## Pick a mode

| Situation | Mode | Setup |
|---|---|---|
| User has squig.sh open and wants you to draw in it | Browser tab, `window.squig` | None — needs your access to *that* tab |
| Throwaway sketch you render and show | CLI + `render` | Persistent clone |
| Real exploration with variations, renders, history | Companion (stdio MCP or HTTP) | Persistent clone + `build:local` |

A squig.sh URL is **not** a shared canvas (per-profile storage); remote agents can't reach the loopback companion.

## Install — persistent clone (once per machine)

Recommended stable path `~/tools/squig`; the runtime and vendored fonts live
there, so never delete it while a `.squig.json` is in use.

```bash
git clone https://github.com/pablostanley/squig.git ~/tools/squig
cd ~/tools/squig
pnpm install --frozen-lockfile   # pins pnpm@10.4.1 itself; ~600 pkgs
pnpm build:local                 # next build -> out/ (the local editor)
```

Cold install can exceed a 10-min tool timeout — use `run_in_background`. "Ignored
build scripts: sharp, unrs-resolver" is harmless. Docs say Node 24; 25.6 worked.

## Run it

```bash
S="node --experimental-strip-types --disable-warning=MODULE_TYPELESS_PACKAGE_JSON --import $HOME/tools/squig/scripts/register-loader.mjs $HOME/tools/squig/scripts/squig.ts"   # $HOME, not ~ (no tilde expansion in quotes)
$S components hero                       # search the 155 kinds
$S describe pricing-block                 # props, defaults, controls
$S serve /abs/project/wireframes/home.squig.json   # editor + HTTP MCP on 127.0.0.1
```

`serve` prints `Open canvas: http://127.0.0.1:<port>/?local=1#token=…` and
`MCP: http://127.0.0.1:<port>/mcp`. **Send the user the editor URL before
drawing**; keep the process running (background task). One companion per file.

Claude Code registration — **syntax verified against `claude mcp add --help`,
not executed**:

```bash
claude mcp add squig -- node --experimental-strip-types \
  --disable-warning=MODULE_TYPELESS_PACKAGE_JSON \
  --import /abs/tools/squig/scripts/register-loader.mjs \
  /abs/tools/squig/scripts/squig.ts mcp /abs/project/wireframes/home.squig.json
```

Use absolute paths, call `node` directly (pnpm banners corrupt stdio), keep the
`--` so claude doesn't eat the node flags. Prefer stdio over
`--transport http … -H "Authorization: Bearer …"`: the HTTP token is
per-session (observed different per process). Full CLI/HTTP/MCP recipes:
`references/integration.md`.

## Workflow

1. **Read first.** `squig_local_session` → send `editorUrl`, then
   `squig_get_document`. In a browser tab: `window.squig.doc()` and match
   `window.squig.documentId()` to the invite. Preserve human edits.
2. **Catalog, don't guess.** `squig_catalog` (query or kind) for real kinds and
   props before every new kind.
3. **Draw 2–3 directions side by side** in small `squig_edit_document` batches,
   each with a visible title and its tradeoff as text, each registered as a
   `variation` (`id`, `title`, `description`, `nodeIds`). Use real copy.
4. **Inspect renders**: `squig_render_document` (`format: "png"`, optionally
   `variationId`); `squig_measure_text` for overflow. Fix clipping and spacing.
5. **User picks one.** Then `squig_export_document` → handoff; build only the
   chosen variation; map kinds to shadcn with the table below.

## Verified gotchas

1. No `npx squig`, no `shadcn add @squig/…`, no hosted MCP (`/mcp` → 410 `SQUIG_LOCAL_ONLY`).
2. CLI negative numbers need equals form: `--y=-60` (`--y -60` errors).
3. CLI `render` SVG does **not** embed fonts (falls back to Comic Sans);
   `squig_render_document` PNG embeds Patrick Hand/Geist/Source Serif. Review with the PNG.
4. API-added nodes without `seed` all get `seed: 1` (identical wobble) —
   pass a random integer `seed` per node.
5. `edit_document` deletes with `op: "delete"` (not `remove`).
   `window.squig.add()` takes node objects; use `addComponent(kind, {x, y})`.
6. Revisions are opaque; a stale one returns 409. Re-read, reconcile, retry.
7. The `#token=` in the editor URL is a bearer secret — don't paste it into
   commits, docs or shared logs. Server is loopback-only with Host/Origin checks.
8. Comments are API-only (no canvas UI): put anything the human must see in a
   text/`note` node.
9. 100 ops/batch and 1,000 nodes/add (from the OpenAPI schema); history lives
   in a sibling `.squig.json.history/` (seen on disk). 5,000 nodes, 16 MiB
   file, 50 snapshots are **unverified** (docs only).

## Handoff: squig kind → shadcn (core)

Run through the **shadcn** skill first — the `@latest` vs `@2.3.0` fork decides
what exists. Names checked against both registry indexes, Oct 2026.

| squig kind | `npx shadcn add …` |
|---|---|
| button, badge, avatar, input, textarea, select, checkbox, switch, slider, progress, card, table, tabs, dialog, tooltip, breadcrumb, pagination, accordion, carousel, calendar, skeleton, sidebar, menubar, chart | same name |
| radio → `radio-group` · divider → `separator` · dropdown → `dropdown-menu` · toast → `sonner` · otp-input → `input-otp` | renamed |
| navbar → `navigation-menu` + `sheet` · command-menu → `command` · combobox → `popover` + `command` · date-picker → `popover` + `calendar` · data-table → `table` + `@tanstack/react-table` | composed |
| hero, feature-*, pricing-block, footer, other marketing blocks | none — bespoke (`supahero`, `componentry`) |

Full table incl. v4-only kinds (`kbd`, `spinner`, `button-group`,
`input-group`, `field`, `empty`, `combobox`): `references/integration.md`.

## When to skip squig

- Structure is already decided (approved Figma, existing page being restyled,
  a brief that fixes the layout) — go straight to Identity/Tokens.
- Single-component or single-asset tasks (a button, a background, a logo).
- No local Node/pnpm and no browser-tab access — don't fake it with a
  substitute canvas; say what access is needed.

## Position in design-arsenal

Phase **Structure**, between Direction (taste-skill, cosmos, supahero) and
Identity (logosystem, bestfreefonts, uicolors). Direction says *how it should
feel*; squig fixes *what goes where*; Identity and Tokens then dress the chosen
variation. Squig's sketch ink and fonts are wireframe chrome — never carry them
into the production palette.
