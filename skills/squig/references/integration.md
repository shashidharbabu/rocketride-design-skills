# Squig — integration reference

Verified Oct 2026 against repo HEAD `85f92ab` on macOS, Node 25.6, pnpm 10.4.1
(auto-selected via `packageManager`). Items marked **unverified** were read in
docs or source but not run.

## 1. Facts sheet

| | |
|---|---|
| Site | https://squig.sh (hosted canvas, browser storage only) |
| Source | https://github.com/pablostanley/squig — MIT © 2026 Pablo Stanley |
| Agent docs | `/llms.txt`, `/llms-full.txt`, `/openapi.json` (Squig Local Agent API 2.0.0) |
| Not available | npm `squig` / `@squig/cli` (404), `/registry.json`, `/r/*.json` (404), install script (404), hosted `/mcp` (410 `SQUIG_LOCAL_ONLY`) |
| Upstream skill | `plugins/squig/skills/wireframe-first/SKILL.md` (Codex plugin format; no Claude Code manifest) |
| Own stack (sealed in the clone) | Next 16.2, React 19.2, Tailwind v4, shadcn 4 `base-nova` on `@base-ui/react`, rough.js, Phosphor, zustand, sharp, resvg |
| Pricing | Free. Hosted collaboration retired; old online canvases have read-only recovery at `/connect` |

## 2. CLI (all verified unless noted)

```bash
S="node --experimental-strip-types --disable-warning=MODULE_TYPELESS_PACKAGE_JSON \
  --import $HOME/tools/squig/scripts/register-loader.mjs $HOME/tools/squig/scripts/squig.ts"
# Use $HOME, not ~ — tilde does not expand inside double quotes.
# From inside the clone, `pnpm squig <cmd>` is equivalent (not for stdio MCP).

$S components [query]                       # one line per kind: kind, name, category, default size
$S describe <kind>                          # JSON: defaults, controls, keywords, size
$S new  F.squig.json --name "Home" [--force]
$S add  F.squig.json navbar --x 0 --y 0 --w 1200 --id nav [--props '{"ctaLabel":"Start"}']
$S text F.squig.json "Direction A: pricing-led" --x 0 --y=-60 --size 28 --bold --id title
$S shape F.squig.json rect --x 0 --y 0 --w 200 --h 80 [--fill none|light|strong|paper] [--dashed]
$S arrow F.squig.json --from nav --to hero [--style straight|elbow|curved]   # unverified
$S set  F.squig.json hero --patch '{"w":1200}'                               # unverified
$S ls F.squig.json ; $S validate F.squig.json
$S render F.squig.json --out F.svg [--transparent]   # SVG; fonts NOT embedded
$S serve F.squig.json [--port N]                     # editor + HTTP MCP
$S mcp   F.squig.json                                # stdio MCP + editor on a random port
```

Negative numbers: always `--x=-40`. CLI-added nodes get random seeds.

## 3. HTTP tools (companion)

`POST http://127.0.0.1:<port>/api/v1/tools/<name>` with
`Authorization: Bearer <token>` (token = the `#token=` fragment of the editor
URL) and a JSON body. Without the token → 401.

| Tool | Body (minimum) | Notes |
|---|---|---|
| `documents` | `{}` | returns `id` (e.g. `local_…`), `revision`, `editorUrl` |
| `get_document` | `{documentId}` | full document + `revision` |
| `catalog` | `{query?}` / `{kind?}` | empty → compact index (155); query adds defaults + controls |
| `edit_document` | `{documentId, revision, operations:[…]}` | ≤100 ops; returns new revision + changed nodes; stale → 409 |
| `render_document` | `{documentId, format:"png"\|"svg", variationId?}` | PNG as `{mimeType, base64, bounds}`; fonts embedded |
| `measure_text` | `{documentId, nodeIds?:[…]}` | lineCount, requiredWidth/Height, overflowX/Y, missingGlyphs; text nodes only (an unknown key is ignored and all text nodes are measured) |
| `export_document` | `{documentId}` | `document` + `handoff` {variations, comments, instruction} |
| `history` / `restore` | `{documentId}` / `{documentId, revision, …}` | restore unverified |
| `comment` / `resolve_comment` / `replace_document` | see `/openapi.json` | unverified |

Verified `edit_document` payload (adds a direction and registers it):

```json
{"documentId":"local_…","revision":3129813381656875,"operations":[
  {"op":"add","nodes":[
    {"id":"b-title","type":"text","text":"Direction B: feature-led","x":1400,"y":-60,"w":600,"h":40,"fontSize":28,"bold":true,"seed":48213},
    {"id":"b-nav","type":"component","kind":"navbar","x":1400,"y":0,"w":1200,"h":56,"seed":90211},
    {"id":"b-hero","type":"component","kind":"hero-minimal","x":1400,"y":80,"w":1200,"h":240,"seed":11873},
    {"id":"b-feat","type":"component","kind":"feature-grid","x":1400,"y":360,"w":1200,"h":420,"seed":70452}]},
  {"op":"variation","id":"feature-led","title":"Feature led",
   "description":"Lead with capabilities, pricing later.","nodeIds":["b-title","b-nav","b-hero","b-feat"]}]}
```

Op names in the OpenAPI schema (all 19): `add`, `update`, `delete`,
`duplicate`, `group`, `ungroup`, `detach`, `align`, `distribute`, `tidy`,
`spacing`, `spacing_resize`, `reorder`, `flip`, `rename`, `look`, `note`,
`variation`, `remove_variation` (only `add`, `variation` and `delete`
exercised). Component props go under `props`
(see `describe`); omitted props take defaults.

## 4. MCP

Stdio (verified: initialize, tools/list, prompts/list, resources/list,
`squig_local_session`): 13 tools — `squig_local_session`, `squig_catalog`,
`squig_documents`, `squig_get_document`, `squig_edit_document`,
`squig_replace_document`, `squig_history`, `squig_restore`, `squig_comment`,
`squig_resolve_comment`, `squig_export_document`, `squig_measure_text`,
`squig_render_document`; prompt `wireframe-first`; resource
`squig://guides/wireframing`. Same JSON as HTTP.

HTTP MCP at `http://127.0.0.1:<port>/mcp` (verified: initialize 200 with
Bearer, 401 without).

Claude Code (syntax verified against `claude mcp add --help`, **not executed**):

```bash
# stdio — preferred; Claude Code launches the process and the editor with it
claude mcp add squig -- node --experimental-strip-types \
  --disable-warning=MODULE_TYPELESS_PACKAGE_JSON \
  --import /Users/<you>/tools/squig/scripts/register-loader.mjs \
  /Users/<you>/tools/squig/scripts/squig.ts mcp /abs/project/wireframes/home.squig.json

# http — only for an already-running `serve`; token changes per process
claude mcp add --transport http squig http://127.0.0.1:<port>/mcp \
  -H "Authorization: Bearer <token>"
```

Default scope is `local` (this project, this user). One server = one file; for
a different canvas, re-register with a new path. Don't run a `serve` and an MCP
process on the same file.

## 5. Browser tab (`window.squig`, verified on hosted squig.sh via Playwright)

Keys: `version, documentId, doc, serialize, load, add, addComponent, addText,
addShape, addArrow, update, remove, group, tidy, spacing, resizeSpaced, toFront,
toBack, select, selection, zoomToFit, zoomTo, bounds, components, describe, svg`.

```js
window.squig.documentId()                         // match the invite before editing
window.squig.components().length                  // 155
window.squig.addComponent("login", { x: -200, y: -230 })   // returns new id
window.squig.addText("Option A: social-first", { x: -200, y: -290, fontSize: 24 })
window.squig.zoomToFit()
```

`add()` takes node objects (passing a kind string throws "not a node squig can
draw"). WebMCP tool names (`squig_read_canvas`, `squig_add_component`, …) are
documented but **unverified** here. Edits share the tab's undo and autosave.

## 6. Component inventory (155 kinds, verified)

- **Basic**: button badge avatar input textarea select checkbox radio switch slider progress
- **Display**: card image divider paragraph table tabs dialog dropdown toast alert tooltip breadcrumb pagination chart
- **More**: icon-button button-group split-button fab chip chip-group form-field search-input otp-input file-upload date-picker combobox input-group stepper-input color-swatch-row radio-group checkbox-group toggle-group rating slider-range accordion carousel kbd timeline tree-view list-item code-block calendar skeleton spinner card-stat card-profile card-media card-pricing card-testimonial card-product card-blog card-notification menubar context-menu command-menu nav-tabs-pill bottom-nav stepper stat data-table kv-list video-player audio-player image-grid map
- **Nav**: navbar sidebar
- **Marketing blocks**: hero hero-minimal cta-block feature-grid feature-split feature-list testimonial-block logo-cloud pricing-block faq stats-band newsletter banner announcement-bar footer team-grid awards gallery about-block blog-list blog-post-header article-body changelog contact-form-block breadcrumb-header section-header
- **App blocks**: usage-meter invoice-list account-block billing-block notification-list activity-feed comments inbox-list kanban-board calendar-block file-browser search-results onboarding-checklist empty-block settings-block profile-header command-palette-block ai-input ai-chat ai-prompt-suggestions ai-response ai-agent-card ai-thinking order-summary cart checkout product-detail product-grid payment-methods app-shell landing-page chat-screen inbox-screen profile-screen
- **Mobile**: mobile-feed mobile-list mobile-detail mobile-settings mobile-onboarding
- **Templates**: login signup settings dashboard pricing feed empty-state
- **Extras**: icon sticky frame callout spec

Looks (`document.look`): theme `internet-blue|riso-red|terminal-green|plum|marigold|graphite`,
paper `white|subtle|shaded`, font `hand|sans|serif`, `grid` bool.

## 7. Full handoff table: squig kind → shadcn

Registry names checked Oct 2026 against `ui.shadcn.com/r/index.json`
(`@latest`, Tailwind v4) and `v3.shadcn.com/r/index.json` (`@2.3.0`, Tailwind v3).
"v4 only" = absent from the v3 index — on Tailwind 3.4 build it from the
listed fallback.

| squig kind | shadcn (`npx shadcn@<fork> add …`) | v3 fallback |
|---|---|---|
| button, icon-button, fab | `button` | — |
| button-group, split-button | `button-group` (v4 only) | `button` ×N + `dropdown-menu` |
| badge, chip | `badge` | — |
| chip-group, toggle-group | `toggle-group` | — |
| avatar | `avatar` | — |
| input, search-input | `input` | — |
| input-group, stepper-input | `input-group` (v4 only) | `input` + `button` |
| form-field | `field` (v4 only) / `form` | `form` + `label` |
| textarea | `textarea` | — |
| select | `select` (or `native-select`, v4 only) | `select` |
| checkbox, checkbox-group | `checkbox` | — |
| radio, radio-group | `radio-group` | — |
| switch | `switch` | — |
| slider, slider-range | `slider` | — |
| progress, usage-meter | `progress` | — |
| otp-input | `input-otp` | — |
| combobox | `combobox` (v4 only) | `popover` + `command` |
| date-picker | `popover` + `calendar` | same |
| calendar, calendar-block | `calendar` | — |
| file-upload | none — bespoke (input type=file + `button`) | — |
| card, card-* | `card` | — |
| table | `table` | — |
| data-table | `table` + `@tanstack/react-table` (docs pattern) | same |
| tabs, nav-tabs-pill | `tabs` | — |
| dialog | `dialog` (`sheet`/`drawer` on mobile) | — |
| dropdown | `dropdown-menu` | — |
| context-menu | `context-menu` | — |
| menubar | `menubar` | — |
| command-menu, command-palette-block | `command` (+ `dialog`) | — |
| toast | `sonner` | — |
| alert | `alert` / `alert-dialog` | — |
| tooltip | `tooltip` (`hover-card` for rich) | — |
| breadcrumb, breadcrumb-header | `breadcrumb` | — |
| pagination | `pagination` | — |
| accordion, faq | `accordion` | — |
| carousel, gallery | `carousel` | — |
| divider | `separator` | — |
| skeleton | `skeleton` | — |
| spinner | `spinner` (v4 only) | bespoke |
| kbd | `kbd` (v4 only) | bespoke `<kbd>` |
| code-block | `code-block` (v4 only) | bespoke |
| empty-state, empty-block | `empty` (v4 only) | bespoke |
| list-item, kv-list | `item` (v4 only) | bespoke |
| chart, stat, card-stat, stats-band | `chart` (recharts) / `card` | — |
| navbar | `navigation-menu` + `sheet` | — |
| sidebar, app-shell | `sidebar` | — |
| bottom-nav, mobile-* | bespoke + `drawer` | — |
| ai-chat, chat-screen, ai-response | `message`, `message-scroller`, `bubble` (v4 only) — **unverified** components | bespoke |
| hero, hero-minimal, cta-block, feature-*, pricing-block, testimonial-block, logo-cloud, newsletter, footer, team-grid, blog-*, changelog, banner, announcement-bar | no shadcn primitive — bespoke; see `supahero`, `componentry`, `motion-primitives` | — |
| login, signup, settings, dashboard, pricing, checkout, kanban-board, inbox-*, file-browser | compose from rows above (`card`, `field`/`form`, `input`, `button`, `tabs`, `table`) | — |
| sticky, callout, spec, frame | wireframe annotation — do not build | — |

## 8. Handoff discipline

`export_document.handoff.instruction` is literally: *"Build only the direction
explicitly chosen by the user. Preserve content, hierarchy and layout; choose
production styling separately."* Follow it: carry over order, hierarchy, copy
and relative sizes; take colors, fonts, radii and motion from the Identity /
Tokens phases, never from squig's ink theme or Patrick Hand.

## 9. Troubleshooting

| Symptom | Cause / fix |
|---|---|
| `fetch https://squig.sh/mcp` → 410 | Hosted MCP retired; run the local companion |
| `Option '--y' argument is ambiguous` | Use `--y=-60` |
| Render text in Comic Sans | You rendered with the CLI; use `squig_render_document` PNG |
| 409 Revision conflict | Re-read `get_document`, reconcile, resend with new revision |
| 400 "No matching discriminator" | Wrong `op` name (e.g. `remove` → `delete`) |
| 401 | Missing/old bearer token; re-read the editor URL from the running process |
| Install hangs or times out | Cold pnpm store; run in background, it completes |
| "can't resolve @base-ui/react" in the clone | Stale `node_modules`; `pnpm install` (the clone's `.npmrc` sets `verify-deps-before-run=install`) |
