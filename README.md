# Claude Design Skills

**28 UI/UX and design skills for [Claude Code](https://claude.com/claude-code) — the ones that survived.**

This is a curated pack, not a dump. Each skill here was tried on real projects and kept because it changed the output. The dead weight is not in this repo.

The pack covers the whole path from *"I don't know what this should look like"* to *"the fonts are licensed, the palette passes contrast, and the hero doesn't look like every other AI-built page."*

---

## Install

Skills live in `.claude/skills/`. Drop in the whole pack or cherry-pick.

**Whole pack, one project:**

```bash
git clone https://github.com/shashidharbabu/claude-design-skills.git
mkdir -p .claude/skills
cp -R claude-design-skills/skills/* .claude/skills/
```

**Whole pack, every project (global):**

```bash
cp -R claude-design-skills/skills/* ~/.claude/skills/
```

**One skill:**

```bash
cp -R claude-design-skills/skills/taste-skill .claude/skills/
```

Claude picks skills up automatically from their frontmatter `description` — no registration step. Verify with `/skills` in Claude Code.

> **A note on loading everything at once.** Skill descriptions are always in context; skill bodies are not. 28 skills is a large routing surface. If you only build landing pages, install the ~8 you'll actually use. `design-arsenal` exists to route between them when you do install the lot.

---

## Start here

If you install nothing else, install these three:

| Skill | Why |
|---|---|
| **[design-arsenal](skills/design-arsenal/)** | The router. Maps any design need to the right skill and sequences them into a full design-system build. Read this first — it's the map to everything below. |
| **[taste-skill](skills/taste-skill/)** | The anti-slop enforcer. Audit-first on redesigns, strict pre-flight, and a long list of the specific tells that make a page read as templated. |
| **[ui-ux-pro-max](skills/ui-ux-pro-max/)** | The database. 84 styles, 192 palettes, 74 font pairings, 98 UX guidelines, 25 chart types across 22 stacks — queryable, not vibes. |

---

## The full map

### 🧭 Direction & judgment
*Deciding what good looks like before anything gets built.*

| Skill | Use when |
|---|---|
| [design-arsenal](skills/design-arsenal/) | The job is bigger than one asset — "build a design system", "full design pass". Also owns the glassmorphism recipe and the licensing pre-flight. |
| [taste-skill](skills/taste-skill/) | Landing pages, portfolios, redesigns. Infers the right direction from the brief instead of defaulting to purple gradients. |
| [frontend-design](skills/frontend-design/) | New UI or reshaping existing UI — aesthetic direction, typography, avoiding templated defaults. |
| [ui-ux-pro-max](skills/ui-ux-pro-max/) | Any concrete lookup: palette, font pairing, product-type pattern, UX rule, motion preset, chart type. |
| [soft-skill](skills/soft-skill/) | You want it to feel *expensive*. Exact fonts, spacing, shadows, card structure — and blocks the defaults that read as cheap. |
| [redesign-skill](skills/redesign-skill/) | An existing site needs upgrading without breaking it. Audits first, identifies generic AI patterns, then fixes. |
| [output-skill](skills/output-skill/) | Claude keeps truncating with `// ... rest of the code`. Bans placeholder patterns and enforces complete generation. |

### 🎨 Style systems
*Pick a committed look instead of averaging every look.*

| Skill | The look |
|---|---|
| [minimalist-skill](skills/minimalist-skill/) | Clean editorial. Warm monochrome, typographic contrast, flat bento grids, muted pastels. No gradients, no heavy shadows. |
| [brutalist-skill](skills/brutalist-skill/) | Swiss print × military terminal. Rigid grids, extreme type-scale contrast, analog degradation. Declassified-blueprint energy. |

### 🔭 Reference mining
*Stealing structure, not pixels. Each skill turns a gallery into named decisions.*

| Skill | Source | Yields |
|---|---|---|
| [cosmos](skills/cosmos/) | [cosmos.so](https://cosmos.so) | Moodboard → named decisions (grid, type scale, spacing rhythm, color temperature) |
| [recent-design](skills/recent-design/) | [recent.design](https://recent.design) | "Does this read current?" — trend triage: adopt / adapt / skip |
| [supahero](skills/supahero/) | [supahero.io](https://supahero.io) | Hero anatomy decomposed into reusable structure |
| [pafolios](skills/pafolios/) | [pafolios.com](https://pafolios.com) | Case-study and showcase narrative spine |
| [motionsites](skills/motionsites/) | [motionsites.ai](https://motionsites.ai) | Motion choreography patterns → real `motion/react` code |
| [logosystem](skills/logosystem/) | [logosystem.co](https://logosystem.co) | 1,300+ real logo systems, faceted — plus systematic lockup/clearspace implementation |

### 🪪 Identity — logo, brand, type, color

| Skill | Use when |
|---|---|
| [brandkit](skills/brandkit/) | Generating brand-guideline boards, logo systems, identity decks, visual-world presentations. |
| [logoai](skills/logoai/) | Need a logo fast and cheap ([logoai.com](https://logoai.com)) — plus the ownership/licensing reality check before commercial launch. |
| [bestfreefonts](skills/bestfreefonts/) | Sourcing genuinely free fonts ([bestfreefonts.com](https://bestfreefonts.com)) and self-hosting via `next/font/local`. **License verification is non-negotiable here.** |
| [uicolors](skills/uicolors/) | One brand hex → full Tailwind 50–950 scale ([uicolors.app](https://uicolors.app)), wired into config and contrast-checked for AA. |

### 🧩 Components & implementation

| Skill | Use when |
|---|---|
| [shadcn](skills/shadcn/) | Any `npx shadcn` command, `components.json`, token remapping, third-party registries. **Read before running the CLI on a Tailwind v3 project** — the v3/v4 fork is the first decision. |
| [componentry](skills/componentry/) | Spectacle backgrounds, kinetic typography, scroll set-pieces — ~60 MIT components via the shadcn registry. |
| [motion-primitives](skills/motion-primitives/) | Text reveals, number tickers, marquees, morphing dialogs, scroll reveals. Also: when *not* to animate. |
| [react-best-practices](skills/react-best-practices/) | Performance. 40+ rules from Vercel Engineering — waterfalls, bundle size, data fetching. |

### 🖼️ Assets — background, texture, 3D, motion, audio

| Skill | Use when |
|---|---|
| [haikei](skills/haikei/) | Generated SVG backgrounds ([haikei.app](https://haikei.app)) — waves, blobs, blurry gradients, low-poly, dividers. |
| [fffuel](skills/fffuel/) | Noise, grain, mesh gradients, holographic, neon glow, patterns ([fffuel.co](https://fffuel.co)) — ~40 generators routed by use case. |
| [spline](skills/spline/) | Interactive 3D heroes ([spline.design](https://spline.design)) — plus the honest cost (~582KB gzipped runtime) and when to use a video export instead. |
| [jitter](skills/jitter/) | Designer-authored animation ([jitter.video](https://jitter.video)) — MP4/GIF/Lottie exports and picking the right player. |
| [uppbeat](skills/uppbeat/) | Music/SFX for demo videos ([uppbeat.io](https://uppbeat.io)) — license tiers, and why the answer for websites is usually a restrained no. |

---

## Common routes

**"Make this look better" (no other information)**
→ `design-arsenal` to route → `taste-skill` to audit → `recent-design` to check it reads current

**Greenfield landing page**
→ `cosmos` (direction) → `supahero` (hero anatomy) → `bestfreefonts` + `uicolors` (type & color) → `shadcn` (base) → `motion-primitives` (life) → `taste-skill` (pre-flight)

**Existing app looks dated**
→ `redesign-skill` (audit) → `recent-design` (triage) → `soft-skill` (raise the ceiling) → `react-best-practices` (don't regress perf)

**Brand identity from zero**
→ `logosystem` (references) → `logoai` *or* a real designer (mark) → `brandkit` (guidelines) → `uicolors` + `bestfreefonts` (tokens)

**Page feels flat**
→ `fffuel` (texture/grain) or `haikei` (background) → `motion-primitives` (motion) → `spline` (only if 3D genuinely earns 582KB)

---

## Notes on the tool skills

Roughly half of these wrap external web tools (Cosmos, Haikei, fffuel, Spline, Jitter, Uppbeat, LogoAI, UI Colors…). Those tools are **browser-based and human-driven** — no API, no CLI. The skill's job is not to automate them; it's to know which tool fits the need, tell you exactly what to configure, and then handle the genuinely hard part: integrating the output into Next.js correctly.

Each of those skills also carries the licensing truth for its tool. That's deliberate — "is this font/track/logo actually safe to ship commercially" is the question that bites late, and `design-arsenal` aggregates the pre-flight.

**Stack assumption:** integration code targets **Next.js 14 + Tailwind 3.4 + motion v13 (`motion/react`)**. The design judgment is stack-agnostic; the code snippets are not. `ui-ux-pro-max` covers 22 stacks.

**One dangling reference:** `design-arsenal` routes charting to `dataviz`, which ships with Claude Code rather than living here.

---

## Licenses & attribution

This pack aggregates skills from several authors. **Each skill keeps its own license file where one was provided** — check `skills/<name>/LICENSE` before redistributing.

| Skills | Author | License |
|---|---|---|
| `taste-skill`, `soft-skill`, `redesign-skill`, `minimalist-skill`, `brutalist-skill`, `brandkit`, `output-skill` | [Leonxlnx](https://github.com/leonxlnx) | MIT |
| `frontend-design` | Anthropic | Apache 2.0 (see `LICENSE.txt`) |
| `react-best-practices` | Vercel Engineering | See `skills/react-best-practices/` |
| `ui-ux-pro-max` | Community | No license file shipped upstream — verify before redistributing |
| All tool skills (`cosmos`, `haikei`, `fffuel`, `spline`, `jitter`, `uppbeat`, `logoai`, `logosystem`, `uicolors`, `bestfreefonts`, `shadcn`, `componentry`, `motion-primitives`, `motionsites`, `supahero`, `pafolios`, `recent-design`, `design-arsenal`) | This repo | MIT — see [LICENSE](LICENSE) |

The tool skills document third-party services but contain no third-party code. The services themselves (Cosmos, Haikei, Uppbeat, LogoAI, …) have their own terms — the skills tell you what they are, but **verify before shipping commercially**.

If you authored something here and want the attribution corrected or the skill removed, open an issue.

---

## Contributing

The bar for adding a skill: it has to change the output on a real project, not just describe a tool. Open an issue with what it did that the existing 28 didn't.
