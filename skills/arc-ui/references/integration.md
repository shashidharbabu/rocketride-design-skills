# Arc UI integration

Everything after deciding to use Arc. Verified Oct 2026 on two real builds:
Next 16.4.2 + React 19.3 + Tailwind v4 + shadcn (radix-nova), and Next
14.2.35 + React 18 + Tailwind 3.4 + shadcn (new-york, HSL-triplet tokens).
*(unverified)* marks anything not executed.

## 1. Registry and foundation

```jsonc
// components.json
{ "registries": { "@uiarc": "https://uiarc.dev/r/{name}.json" } }
```

```bash
npx shadcn@latest add https://uiarc.dev/r/arc-foundation.json   # tokens, once per app
npx shadcn@latest add @uiarc/<id> [@uiarc/<id> ...] -y            # or full URLs
```

- `arc-foundation` → `components/arc/foundation.css` (no item reinstalls it).
- Every motion item pulls `arc-motion-tokens` →
  `components/arc/motion-tokens.ts` + `components/arc/lib/motion-tokens.ts`.
- Internal registry deps resolve automatically (e.g. metric-card pulls
  animated-counter; hero-section pulls line-chart, sparkline, avatar,
  segmented-control, button, copy-button).
- npm deps added: `motion@^14.3.0`, `lucide-react`, per-item `@radix-ui/*`.
  Imports are `motion/react` only.
- For Arc items the CLI touched nothing else: `globals.css`,
  `tailwind.config.ts`, `components.json` diffs were empty (Tailwind v3 run).
- Discovery: `https://uiarc.dev/llms.txt`, `https://uiarc.dev/r/catalog.json`
  (whenToUse/whenNotToUse per item, Pro included),
  `https://uiarc.dev/components/<id>/markdown`
  (blocks: `/components/blocks/<id>/markdown`). Read the item's markdown for
  exact props before wiring — e.g. `segmented-control` is a **default export**;
  selection items use `value`/`onValueChange`, pickers `value`/`onChange`.
- MCP (`https://uiarc.dev/api/mcp`) requires OAuth into an Arc account even
  for free items (anonymous → 401). Not needed; registry + markdown suffice.
- Pro: `@uiarc-pro` → `https://uiarc.dev/r/pro/{name}.json` with
  `Authorization: Bearer ${ARC_PRO_TOKEN}`; 401 without a token.

## 2. Next 16.4 turbopack rule (CSS Modules dead by default)

create-next-app 16.4.2 writes this, which sends every `.css` — including
`*.module.css` — through the Tailwind loader as plain CSS. Symptom: build
"succeeds" with 12× `'global' is not recognized as a valid pseudo-class`
warnings and every Arc component renders unstyled.

```ts
// next.config.ts — default (breaks Arc)
turbopack: { rules: { "*.css": { loaders: ["@tailwindcss/turbopack"], as: "*.css" } } }
```

Fix (verified: 0 warnings, Arc styled, Tailwind utilities still work):

```ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  cacheComponents: true,
  partialPrefetching: true,
  turbopack: {
    rules: {
      "globals.css": {
        loaders: ["@tailwindcss/turbopack"],
        as: "*.css",
      },
    },
  },
};

export default nextConfig;
```

If Tailwind is imported from other global stylesheets, add a rule per file.
Alternative *(unverified)*: drop the rule and use `postcss.config.mjs` with
`@tailwindcss/postcss`.

## 3. Token collision bridge (Arc + shadcn on one app)

Arc's `:root` sets `--background`, `--foreground`, `--border`, `--accent`,
`--accent-foreground` with different meanings (Arc `--accent` = brand
colour; shadcn `--accent` = hover tint). Measured with foundation imported:

| | shadcn `bg-accent` | shadcn `border` |
|---|---|---|
| Tailwind v3, no Arc | rgb(244,244,245) | rgb(228,228,231) |
| Tailwind v3, with Arc | **transparent** (`hsl(oklch())` invalid) | rgb(113,113,122) fallback |
| Tailwind v4, with Arc | near-black (Arc accent) | Arc border |

**Fix: rename shadcn's side, never Arc's.** Arc's module CSS reads these names
directly and re-adding an item overwrites edits; `globals.css` is yours.

```bash
# Tailwind v4: globals.css holds :root, .dark and @theme inline
sed -i '' -e 's/--background/--sh-background/g' -e 's/--foreground/--sh-foreground/g' \
  -e 's/--border/--sh-border/g' -e 's/--accent/--sh-accent/g' app/globals.css
# shadcn v4 components also use raw vars in arbitrary values (button.tsx: var(--foreground))
sed -i '' -E 's/var\(--(background|foreground|border|accent)/var(--sh-\1/g' components/ui/*.tsx
# Tailwind v3: also the hsl(var(--x)) mappings
sed -i '' -e 's/--background/--sh-background/g' -e 's/--foreground/--sh-foreground/g' \
  -e 's/--border/--sh-border/g' -e 's/--accent/--sh-accent/g' tailwind.config.ts
```

(`sed -i ''` is BSD/macOS; GNU sed takes `-i` alone.) Safe because
`--color-background`, `--muted-foreground`, `--sidebar-accent` etc. don't
contain the `--name` substrings. After the rename, verified: v4 light
`bg-accent` back to oklch .97, dark tracks `.dark`; v3 back to
rgb(244,244,245) / rgb(228,228,231). Re-run the `components/ui` sed after
every `shadcn add`.

Canvas ownership: shadcn's base layer paints `body` with `--sh-background`
while Arc blocks paint `--background`; in dark they differ slightly. Pick the
owner — e.g. `--sh-background: var(--background)` in `:root` and `.dark`
*(unverified)*.

Rebranding Arc's accent: foundation ships `data-accent` = neutral, violet,
blue, green, amber, orange, coral, rose (light + dark values). For a project
brand, add your own block after foundation *(unverified)*:

```css
:root[data-accent="brand"] { --accent: …; --accent-strong: …; --accent-subtle: …; --accent-foreground: …; }
:root[data-theme="dark"][data-accent="brand"] { --accent: …; --accent-strong: …; --accent-subtle: …; }
```

Selection controls read `--control-on` (tuned per accent in foundation) —
set it too, and check white-thumb contrast in both themes.

**Arc-only app:** skip all of this; don't init shadcn, or init it and never
import its tokens.

## 4. Dark mode — set both switches

Arc: 19 rules on `:root[data-theme="dark"]`, plus 12 items with
`:global(:root[data-theme="dark"])` overrides in their modules. shadcn:
`.dark`. One toggle sets both (verified):

```ts
const root = document.documentElement;
root.dataset.theme = theme;                       // "light" | "dark"
root.classList.toggle("dark", theme === "dark");
```

With next-themes *(unverified)*: `<ThemeProvider attribute={["class", "data-theme"]}>`.
Arc has no `prefers-color-scheme` handling — resolve "system" yourself.

## 5. Fonts — foundation sets no body font

```tsx
// app/layout.tsx (Next 15/16: Geist is in next/font/google; Next 14: localFont GeistVF.woff)
const geist = Geist({ variable: "--font-geist", subsets: ["latin"] });
const inter = Inter({ variable: "--font-inter", subsets: ["latin"] });
<html className={`${geist.variable} ${inter.variable}`}>
```

```css
/* globals.css (keep it out of foundation.css — reinstall overwrites) */
body { font-family: var(--font-body); }   /* --font-body → Inter, --font-display → Geist */
```

Without it, labels, inputs and dialog descriptions render in Times
(verified); only blocks that set `font-family` themselves look right.

## 6. ArcProvider in a client component

Arc renders plain `<a>`/`<img>` unless wrapped. Passing `Link` from a server
layout fails prerender ("Functions cannot be passed directly to Client
Components"). Verified fix:

```tsx
// app/providers.tsx
"use client";
import Link from "next/link";
import { ArcProvider } from "@/components/arc/lib/arc-provider";
export function Providers({ children }: { children: React.ReactNode }) {
  return <ArcProvider link={Link}>{children}</ArcProvider>;
}
```

`arc-provider.tsx` arrives only with `avatar`, `breadcrumb` or `hero-section`
— it is not a standalone registry item. `image={Image}` accepted
*(unverified)*.

## 7. tsconfig target (Next 14 template)

The Next 14 template has no `target`, so `[...string]` in
`animated-counter.tsx:30` fails type check. Add `"target": "ES2017"` to
`compilerOptions`. Next 16 template already passes.

## 8. Missing block assets

`hero-section` fetches `/block-logos/{linear-color,raycast-color,vercel,loom-color,framer,supabase-color}.svg`
— 404, not in the registry payload. All blocks ship sample copy/data
("Lumen", "Acme Cloud"); pass your own `title`/`description`/actions or
replace the data files. Grep `components/arc/blocks` for `"/block-logos/` and
other absolute paths after every block install.

## 9. Reduced motion and a11y

- Static scan: 130/130 free items that import `motion/react` contain
  `useReducedMotion` or a `prefers-reduced-motion` query.
- Runtime (Playwright, `reducedMotion: "reduce"`): metric-card digits show
  final values within 150 ms of scroll-in.
- Overlays (dialog, popover, dropdown, tooltip, select…) are Radix-based;
  focus ring only on `:focus-visible` (pointer focus draws none — a product
  decision; keep or override per project).
- Not run: an axe audit. Counters animate on in-view, so full-page
  screenshots show "0,000" below the fold — expected, not a bug.

## 10. Free inventory (137 registry items, Oct 2026)

Deps: `motion` ×130, `lucide-react` ×84; Radix dropdown-menu ×5, dialog ×4,
popover ×4, tooltip ×2, tabs ×2, select / checkbox / switch / accordion ×1.

- **Foundation:** `arc-foundation`, `arc-motion-tokens` (`arc-skill` — don't).
- **Actions (19):** action-button, button, button-group, confirm-morph,
  context-menu, copy-button, dropdown-menu, expanding-button-group,
  floating-button-group, hold-to-confirm, icon-morph, reactions,
  split-button, swipe-actions, theme-switch (+ -eclipse/-rise/-split),
  user-menu.
- **Inputs (34):** billing-toggle, calendar, checkbox, chip-group,
  color-picker, combobox, date-picker, date-range-picker, elastic-slider,
  expanding-search, file-dropzone, inline-edit, input, mention-input,
  money-input, morph-select, multi-select, number-field, password-field,
  password-strength, phone-input, radio-cards, radio-group,
  rich-text-editor, search-field, segmented-control, select,
  shortcut-recorder, signature-pad, slider, switch, tag-input, textarea,
  time-picker.
- **Data (31):** activity-heatmap, animated-counter, avatar, avatar-group,
  badge, bar-chart, brush-chart, card, card-stack, carousel, chat-thread,
  code-block, comment-thread, donut-chart, empty-state, filter-toolbar,
  gauge, image-compare, json-viewer, line-chart, metric-card, pagination,
  ridgeline, slope-chart, sortable-data-table, sparkline, streamgraph,
  timeline, tree-view, treemap, waffle-chart.
- **Disclosure (13):** accordion, bottom-sheet, breadcrumb, dialog, drawer,
  expandable-card, expandable-text, hover-card, popover, resizable-panels,
  scroll-area, tabs, tooltip.
- **Feedback (10):** alert, announcement-bar, cookie-consent, countdown,
  progress, skeleton, stepper, toast, toast-stack, usage-meter.
- **Text (5):** in-view-title, text-morph, text-reveal, text-shimmer,
  slot-text.
- **Blocks (22):** blog-grid, changelog-feed, command-palette,
  comparison-table, contact-section, cta-section, empty-states, faq-section,
  file-upload, hero-section (centered / split / minimal), login-centered,
  logo-marquee, newsletter-signup, notification-center, otp-input,
  page-header, plan-comparison, sign-in, signup-form, site-footer,
  site-header, stats-band.

Pro adds 56 components, 72 blocks and templates (e.g. `dock`, the
"Arc CRM" template). Licence: unlimited end products incl. client work; no
redistribution as a kit/template. Check `https://uiarc.dev/r/catalog.json`
(`tier: "pro"`) before suggesting an item — never imitate Pro source.

Charts take series colour from the accent (series 1 = `--accent`); run the
**dataviz** palette check if the accent is rebranded.
