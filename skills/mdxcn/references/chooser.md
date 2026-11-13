# mdxcn figure chooser

Adapted from the upstream mdxcn agent skill (`https://mdxcn.dev/skill.md` and `https://mdxcn.dev/skill/recipes.md`), © 2026 Keshav Bagaade, MIT License, `github.com/shadcn-labs/mdxcn`.

Reviewed Oct 2026. Changes from upstream:
- install advice replaced with this skill's verified path (upstream says `add all.json`; don't)
- import paths corrected for CLI installs
- a design-system docs map added

## Pick the host first

| Host | What to write | Source |
|---|---|---|
| MDX page with `mdx-components.tsx` | Markdown **inside** the tag (`<GraphTimeline>…</GraphTimeline>`) | docs page `.mdx` tab, `https://mdxcn.dev/docs/<slug>` |
| README, GitHub, Linear, Slack, PR comment, Notion | The **official fenced ASCII**. Swap labels, keep the frame | `https://mdxcn.dev/llms.txt` → `## MDX`, or the docs `.md` tab |
| Comark app (plain `.md` rendered by the app) | a `::graph-*` block with YAML props | `llms.txt` → `## Comark` (needs the `graph-comark` item) |
| Knap template (data → Markdown) | a `{{ value \| graph_* }}` filter | `llms.txt` → `## Knap` (needs `graph-knap` + `knap`) |
| Dependency-free renderer (e.g. a hand-written `lib/markdown.tsx`) | Fenced ASCII only | as for README |

Rules for each host:
- Never paste JSX into a host that can't import it.
- Never paste `::graph-*` into GitHub or Linear.
- Never draw your own ASCII; copy the official fence.
- **These have no fenced ASCII:** Flow, Plot, Activity, Heatmap, Calendar, Timer, Countdown, Frame. On README-type hosts, pick another figure or skip.

## Design-system docs map (this pack's addition)

| Docs section | Figure | Note |
|---|---|---|
| Installing the system / a component | `Steps` (bold = current step) | `Terminal` for the command and its output |
| Required env / config | `Env` | bold marks required |
| Token reference: one token | `GraphSpec` (`- label: value`) | the swatch itself is hand-built |
| Token reference: a role table | `GraphTable` (GFM table; `**Total**` row = footer) | numbers right-align automatically |
| Release notes | `Changelog` (`- added: …`, `- fixed: …`) | `GraphDiff` for before → after values |
| Why we chose X (ADR) | `Decision` (`**chosen** — why`, `*rejected* — why`) | `GraphScore` if options were rated |
| Caveat, deprecation, breaking change | `Callout type="warning"` or a GitHub `> [!WARNING]` | the alert form also reads well on GitHub |
| A code sample with 2–3 lines to explain | `Annotate` (`// (1)` markers) | plain-text notes only (integration.md §8) |
| Keyboard shortcuts | `Keys` | |
| An API the system exposes | `Endpoint` | |
| Questions people ask | `Faq` (`### question` + answer) | |
| Migration progress between versions | `GraphMeter`, then `GraphKpi` | |
| Component status / roadmap | `GraphBoard`, then `GraphCheck` | |

Not covered, so build them by hand: colour swatch grids, type ramps, spacing scales, live component previews, tabbed code/preview, typed props tables.

## Chooser by what the writing is (upstream)

| The writing is | Use |
|---|---|
| A path or a refactor | `GraphFlow`, then `GraphTimeline` |
| An incident / postmortem | `GraphTimeline`, then `GraphUptime` |
| Pick A vs B | `GraphCompare`, then `GraphRank` if size matters |
| What a PR changed | `GraphDiff`, then `GraphSlope` |
| Overlapping work this week | `GraphGantt`, then `GraphStat` |
| A migration in flight | `GraphMeter`, then `GraphKpi` |
| Nested files / org | `GraphTree` |
| An RFC or a launch list | `GraphSheet`, then `GraphCheck` |
| A decision or an ADR | `Decision`, then `GraphScore` |
| A roadmap or a sprint | `GraphBoard`, then `GraphCheck` |
| A code sample to explain | `Annotate` |
| An agent session | `Chat`, then `GraphDiff` |
| Setup: env vars, an API | `Env`, `Endpoint` |
| A procedure | `Steps` |
| A caveat or a warning | `Callout` |
| Questions people ask | `Faq` |

## Chooser by data shape (upstream)

| The data is | Use | Not |
|---|---|---|
| A handful of numbers, no axis | `GraphSpark` | Plot |
| A series that needs a y-scale | `GraphPlot` | Spark |
| One fill from 0 to 1 | `GraphMeter` | Bullet |
| Actual vs a target | `GraphBullet` | Meter |
| Parts of a whole | `GraphStack` (or `GraphWaffle`, ~100 cells) | a pie |
| A short ranked list | `GraphRank` | Bars |
| A small filled/empty grid | `GraphCells` | Waffle, Activity |
| Two small histograms | `GraphBars` | Rank |
| One headline + a trend | `GraphKpi` | Stat |
| Two to four numbers, no trend | `GraphStat` | KPI |
| Before → after numbers | `GraphSlope` | Bars |
| Elapsed time / time left | `GraphTimer` / `GraphCountdown` | |
| Status per day | `GraphUptime` | Activity, Heatmap |
| Daily counts over months | `GraphActivity` | Calendar |
| One month, a few marks | `GraphCalendar` | Activity |
| A labelled intensity grid | `GraphHeatmap` | Matrix |
| Exact numbers on both axes | `GraphMatrix` | Heatmap |
| A running total | `GraphWaterfall` | Stack |
| Steps that drop off | `GraphFunnel` | Flow |
| Rows of numbers | `GraphTable` | Sheet |
| A grouped table with section titles | `GraphSheet` | Table, Spec |
| A punch list `[x]` / `[ ]` | `GraphCheck` | Timeline |
| From / bill-to / line items | `GraphInvoice` | Table |
| A label/value sheet | `GraphSpec` | Stat, Sheet |
| Ratings out of 5 or 10 | `GraphScore` | Bullet |
| A shell session | `Terminal` | a plain fence |
| One release | `Changelog` | Diff |

Skip `GraphFrame` unless you're assembling a custom figure. For real charts with axes, legends and interaction, use the **dataviz** skill instead.

## Slugs and exports

- **Install slug:** the kebab name, for example `graph-timeline`, `callout`, `mdx`.
- **Export:** PascalCase, for example `GraphTimeline`, `Callout`, `withMdxcn`.
- **Where the file lives:**

  | Install route | Location |
  |---|---|
  | CLI | `@/components/<slug>`; the frame primitive is at `@/components/ui/graph-frame` |
  | Verbatim layout | `@/registry/default/<slug>/<slug>` |

Don't invent a barrel file.

## Markdown grammar (upstream, shared by every figure)

| Write | Means |
|---|---|
| `**bold**` | now / chosen / the total, drawn in the accent |
| `*italic*` | next / rejected / an aside, dimmed |
| `- label: value` | one row (the label ends at the first `: `) |
| `x — note` | a side note or caption, muted |
| `a → b → c` (or `->`) | a path (Flow, Slope) |
| `- [x]` / `- [ ]` | a box; nesting = sub-tasks |
| blank line + indented paragraph | the item's body (Steps, Timeline, Spec, Chat) |
| `### heading` | a section, a column (Board) or a question (Faq) |
| a GFM table, last row `**Total**` | data plus a footer |
| a fence + `// (1)` | marked line; item 1 of the list after the fence explains it |
| `~~old~~ new` | a rewrite (Diff) |
| `ok*40` | a run of forty values |

Full table: `https://mdxcn.dev/docs/grammar`.

## Rules (upstream)

- Decide whether a figure earns its place. One sentence needs no figure; a path, a sequence, a matrix or a diff does.
- At most **two** figures per section, with 1–3 sentences of prose between them. Don't lead with the figure, and never make a gallery.
- Titles: 1–2 words, uppercase, no punctuation. Labels: lowercase and plain.
- Palette: one accent by default. `palette="duo"` or `"multi"` only when a second or third series needs it. Never on Table, Sheet, Invoice, Spec, Stat, Tree or Frame.
- Don't restyle the frame (no extra borders, rounded cards or new corner marks) and don't add motion (loops, pulses, CSS animation). It already fades in.
- Don't draw SVG, Mermaid, Recharts or canvas in place of a figure. Don't invent APIs or props: check the prop types in the copied source.
