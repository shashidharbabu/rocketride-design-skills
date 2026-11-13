# RocketRide Design Skills

**34 UI/UX and design skills for [Claude Code](https://claude.com/claude-code). The ones that survived real projects.**

This is a curated pack, not a dump. Each skill was tried on real work and kept because it changed the output. It covers the whole path from *"I don't know what this should look like"* to *"the fonts are licensed, the palette passes contrast, and the hero doesn't look like every other generated page."*

---

## Install

**As a plugin (recommended, updates with the repo):**

```
/plugin marketplace add shashidharbabu/rocketride-design-skills
/plugin install rocketride-design@rocketride-design-skills
```

**Copy in, one project:**

```bash
git clone https://github.com/shashidharbabu/rocketride-design-skills.git
mkdir -p .claude/skills
cp -R rocketride-design-skills/skills/* .claude/skills/
```

**One skill:** `cp -R rocketride-design-skills/skills/<name> .claude/skills/`

Claude picks skills up from their frontmatter `description`; there is no registration step. Check with `/skills`.

**Also install Taste Skill.** The pack routes to it but does not bundle it: v2 is about 39 MB with its own browser checker, and it updates fastest from its author's repo.

```
/plugin marketplace add Leonxlnx/taste-skill
/plugin install taste-skill@taste-skill
```

> **On loading everything at once.** Skill descriptions are always in context; bodies load only when a skill fires. 34 skills is a wide routing surface. If you only build landing pages, install the handful you use. `design-arsenal` routes between them when you install the lot.

---

## Start here

| Skill | Why |
|---|---|
| **[design-arsenal](skills/design-arsenal/)** | The router. Maps any design need to the right skill and sequences them into a full design-system build. Read it first. |
| **Taste Skill** (separate plugin) | Builds a company or marketing site end to end, then checks it in a real browser before calling it done. In a repo with its own design system, use its rules and checker, not its kit. |
| **[ui-ux-pro-max](skills/ui-ux-pro-max/)** | The database: styles, palettes, font pairings, UX rules and chart types across 22 stacks. Queryable, not vibes. |

---

## The full map

### Direction and judgment

| Skill | Use when |
|---|---|
| [design-arsenal](skills/design-arsenal/) | The job is bigger than one asset: "build a design system", "full design pass". Also owns the glass recipe and the licensing pre-flight. |
| [frontend-design](skills/frontend-design/) | New UI or reshaping existing UI: aesthetic direction, typography, avoiding templated defaults. |
| [ui-ux-pro-max](skills/ui-ux-pro-max/) | Any concrete lookup: palette, font pairing, product-type pattern, UX rule, motion preset, chart type. |
| [redesign-skill](skills/redesign-skill/) | An existing site needs upgrading without breaking it. Audits first, names the generic patterns, then fixes. |
| [output-skill](skills/output-skill/) | Claude keeps truncating with `// ... rest of the code`. Bans placeholders and enforces complete output. |

### Style systems

Pick one committed look instead of averaging every look. Never stack two.

| Skill | The look |
|---|---|
| [soft-skill](skills/soft-skill/) | Agency polish. Exact fonts, spacing, shadows and card structure that feel expensive. |
| [minimalist-skill](skills/minimalist-skill/) | Clean editorial. Warm monochrome, typographic contrast, flat bento grids. |
| [brutalist-skill](skills/brutalist-skill/) | Swiss print meets military terminal. Rigid grids, extreme type-scale contrast. |

### Structure and reference mining

Borrow structure, not pixels. Each skill turns a gallery or canvas into named decisions.

| Skill | Source | Yields |
|---|---|---|
| [squig](skills/squig/) | [squig.sh](https://squig.sh) | Hand-drawn wireframe variations of a page before any styling |
| [cosmos](skills/cosmos/) | [cosmos.so](https://cosmos.so) | Moodboard to named decisions: grid, type scale, spacing rhythm, color temperature |
| [recent-design](skills/recent-design/) | [recent.design](https://recent.design) | "Does this read current?" Trend triage: adopt, adapt, skip |
| [supahero](skills/supahero/) | [supahero.io](https://supahero.io) | Hero anatomy broken into reusable structure |
| [pafolios](skills/pafolios/) | [pafolios.com](https://pafolios.com) | Case-study and showcase narrative spine |
| [motionsites](skills/motionsites/) | [motionsites.ai](https://motionsites.ai) | Motion choreography patterns turned into `motion/react` code |
| [logosystem](skills/logosystem/) | [logosystem.co](https://logosystem.co) | 1,300+ real logo systems, faceted, plus lockup and clearspace rules |

### Identity: logo, brand, type, color, icons

| Skill | Use when |
|---|---|
| [brandkit](skills/brandkit/) | Generating brand-guideline boards, logo systems, identity decks. |
| [logoai](skills/logoai/) | You need a logo fast and cheap ([logoai.com](https://logoai.com)), plus the ownership check before commercial use. |
| [bestfreefonts](skills/bestfreefonts/) | Sourcing genuinely free fonts ([bestfreefonts.com](https://bestfreefonts.com)) and self-hosting them. License verification is non-negotiable. |
| [uicolors](skills/uicolors/) | One brand hex to a full 50 to 950 scale ([uicolors.app](https://uicolors.app)), contrast-checked for AA. |
| [iconography](skills/iconography/) | Choosing a project's icon set once (lucide, Tabler, Phosphor and others), then writing the icon spec and wrapper. |

### Components and implementation

| Skill | Use when |
|---|---|
| [shadcn](skills/shadcn/) | Any `npx shadcn` command, `components.json`, token remapping, third-party registries. Read before running the CLI on a Tailwind v3 project. |
| [arc-ui](skills/arc-ui/) | A calm, neutral product UI base as an alternative to shadcn, from [uiarc.dev](https://uiarc.dev). |
| [componentry](skills/componentry/) | Spectacle backgrounds, kinetic type, scroll set-pieces: about 60 MIT components via the shadcn registry. |
| [motion-primitives](skills/motion-primitives/) | Text reveals, number tickers, marquees, morphing dialogs. Also: when *not* to animate. |
| [gauge-ui](skills/gauge-ui/) | Gauges, dials, meters and rings as composable SVG ([gauge-ui.dev](https://gauge-ui.dev)). |
| [audiocn](skills/audiocn/) | Audio UI: players, waveforms, meters ([audiocn.dev](https://audiocn.dev)). |
| [pele](skills/pele/) | Mermaid diagrams rendered as token-themed SVG ([pele.run](https://pele.run)). |
| [mdxcn](skills/mdxcn/) | Design-system docs in MDX: steps, changelogs, decision records ([mdxcn.dev](https://mdxcn.dev)). |
| [react-best-practices](skills/react-best-practices/) | Performance rules from Vercel Engineering: waterfalls, bundle size, data fetching. |

### Assets: background, texture, 3D, motion, audio

| Skill | Use when |
|---|---|
| [haikei](skills/haikei/) | Generated SVG backgrounds ([haikei.app](https://haikei.app)): waves, blobs, gradients, dividers. |
| [fffuel](skills/fffuel/) | Noise, grain, mesh gradients, glow, patterns ([fffuel.co](https://fffuel.co)), routed by use case. |
| [spline](skills/spline/) | Interactive 3D heroes ([spline.design](https://spline.design)), plus the honest runtime cost and when a video export is better. |
| [jitter](skills/jitter/) | Designer-authored animation ([jitter.video](https://jitter.video)): MP4, GIF and Lottie exports and the right player. |
| [uppbeat](skills/uppbeat/) | Music and SFX for demo videos ([uppbeat.io](https://uppbeat.io)): license tiers, and why websites usually need none. |

---

## Common routes

**"Make this look better" (no other information):**
`design-arsenal` to route, then `redesign-skill` to audit, then `recent-design` to check it reads current.

**Greenfield marketing site:**
`cosmos` (direction), `squig` (structure), then Taste Skill for the build and its browser check.

**Greenfield app UI:**
`cosmos`, `squig`, `bestfreefonts` and `uicolors` (type and color), `iconography`, `shadcn` or `arc-ui` (base), `motion-primitives` (life).

**Existing app looks dated:**
`redesign-skill` (audit), `recent-design` (triage), `soft-skill` (raise the ceiling), `react-best-practices` (keep it fast).

**Brand identity from zero:**
`logosystem` (references), `logoai` *or* a real designer (mark), `brandkit` (guidelines), `uicolors` and `bestfreefonts` (tokens).

**Page feels flat:**
`fffuel` or `haikei` (texture, background), `motion-primitives` (motion), `spline` only if 3D earns its weight.

---

## Notes on the tool skills

About half of these wrap external web tools (Cosmos, Haikei, fffuel, Spline, Jitter, Uppbeat, LogoAI, UI Colors). Those tools are browser-based and human-driven. The skill's job is not to automate them; it is to know which tool fits, how to hand off, and how to wire the result into code.

Each of those skills also carries the licensing facts for its tool, because "is this font, track or logo actually safe for commercial use" is the question that bites late. `design-arsenal` collects them into one pre-flight.

**Stack assumption:** integration code targets Next.js 14 to 16, Tailwind 3.4 and `motion/react`. The design judgment is stack-agnostic; the snippets are not. `ui-ux-pro-max` covers 22 stacks.

`design-arsenal` routes charting to `dataviz`, which comes with Claude Code rather than living here.

---

## Licenses and attribution

This pack collects skills from several authors. **Each skill keeps its own license file where one was provided.** Check `skills/<name>/` before redistributing.

| Skills | Author | License |
|---|---|---|
| `soft-skill`, `redesign-skill`, `minimalist-skill`, `brutalist-skill`, `output-skill` | [Leonxlnx](https://github.com/Leonxlnx/taste-skill) (from Taste Skill v1, which has since folded them into one skill) | MIT |
| `brandkit` | [Leonxlnx](https://github.com/Leonxlnx/imagegen-skills) | MIT |
| `frontend-design` | [Anthropic](https://github.com/anthropics/skills) | Apache 2.0 (see `LICENSE.txt`) |
| `react-best-practices` | [Vercel](https://github.com/vercel-labs/agent-skills) | See `skills/react-best-practices/` |
| `ui-ux-pro-max` | [Next Level Builder](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill) | MIT |
| All tool skills (`cosmos`, `squig`, `haikei`, `fffuel`, `spline`, `jitter`, `uppbeat`, `logoai`, `logosystem`, `uicolors`, `bestfreefonts`, `iconography`, `shadcn`, `arc-ui`, `componentry`, `motion-primitives`, `gauge-ui`, `audiocn`, `pele`, `mdxcn`, `motionsites`, `supahero`, `pafolios`, `recent-design`, `design-arsenal`) | This repo | MIT |

The tool skills document third-party services and libraries but contain no third-party code. Those services have their own terms; the skills say what they are, but verify before commercial use.

If you authored something here and want the attribution corrected or the skill removed, open an issue.

---

## Updating

Third-party skills are synced from their upstream repos; the rest are maintained here. Last sync: November 2026 (`frontend-design`, `ui-ux-pro-max`, `brandkit`, and the final pre-retirement versions of the Taste Skill v1 skills).
