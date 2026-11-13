---
name: iconography
description: Decide and wire a project's UI icon system once — choose the icon set (lucide, Tabler, Iconoir, Heroicons, Phosphor, Hugeicons, Remix, Radix) from verified facts (license, count, default size, stroke prop, aria default, shadcn-migrate and Next optimizePackageImports support), write a one-paragraph icon spec in precise vocabulary (grid, base size, stroke weight, caps/joins, corner radius, optical size), normalise every set through one typed <Icon> wrapper (size tokens, constant stroke, aria-hidden vs label), swap lucide out of shadcn with `shadcn migrate icons` (and know when that command can't help), keep bundles tree-shaken, and judge when to commission custom icons instead. Use this whenever someone asks "which icon set", "replace lucide", "swap shadcn icons", "icons look generic / too thick / off-brand", "match icons to our font", "1.5 stroke icons", "outline vs filled", "Phosphor vs Tabler vs Heroicons", "icon library for React/Next", "custom icon set", "icon style guide", "icon audit", "icons are blowing up the bundle", or an iconography.co link appears — and whenever another design skill's icon rule (lucide required vs lucide discouraged) conflicts with the project. This skill is the single place the per-project icon decision is made.
---

# Iconography

Icons are a system decision. Make it once per project, early, then enforce it
mechanically. This skill owns three things: **which set**, **the spec it must
satisfy**, and **how it is wired** (wrapper, imports, shadcn swap).
Verified Oct 2026 against npm, rendered output (React 19.3.1) and the shadcn
4.22.1 / 2.3.0 sources. Anything not tested is marked *(unverified)*.

## Policy: this skill decides; other skills' icon lines are defaults

Several skills carry a one-line icon default that suits their own context.
`shadcn` and `motion-primitives` assume `lucide-react`. `taste-skill`,
`soft-skill`, `minimalist-skill` and `redesign-skill` steer away from stock
lucide to avoid a templated look. Both positions are reasonable, and they
conflict. Resolve the conflict here, record the decision in the project's
direction doc (`icons: <set> @ <weight>, <size scale>`), and that record
overrides every skill's default for that project. Lucide is a legitimate
choice when the brief is neutral or speed matters, especially when it's
re-weighted through the wrapper.

## Position in design-arsenal

- **Phase 2 (Identity):** choose the set and write the spec. The icon stroke
  should sit next to the type weight; a 1.5px stroke reads best beside a
  regular or medium sans.
- **Phase 4 (Component base):** run the shadcn swap right after init and
  token remap, then add `<Icon>` before any feature code lands.
- **Phase 8 (QA / licensing):** add one line, `Icons: <pkg>@<ver> — <licence>`.
  Check Remix ≥ 4.9.0 and Hugeicons Pro.

## 1. Choose the set (summary — full table in `references/sets.md`)

| Need | Pick | Why (verified) |
|---|---|---|
| Keep shadcn feel, zero risk | `lucide-react` 1.59 (ISC) | ~1,890 icons, 24px, stroke 2. shadcn and Next optimisation are both built in. |
| Lucide-compatible but larger, or a filled set too | `@tabler/icons-react` 3.49 (MIT) | ~6,240 icons, 24px, stroke 2, 1,058 *Filled*. Supported by `migrate icons` and Next. |
| Lighter 1.5px line, mid-size catalogue | `iconoir-react` 7.12 (MIT) | 1,673 icons, 1.5 stroke. Not supported by shadcn migrate or Next's defaults. Defaults to 1.5em. |
| 1.5px line, Tailwind-family look, small app | `@heroicons/react` 2.2 (MIT) | Only 324 outline icons. Has no default size. |
| Several weights from one family | `@phosphor-icons/react` 2.1 (MIT) | ~1,510 icons, fill-based, 6 weights. Last publish May 2025. |
| Huge free catalogue at 1.5 stroke | `@hugeicons/*` (MIT free tier) | 6,025 exports, rendered through `<HugeiconsIcon>`. Pro is a separate licence. |

Pick **one family per product surface**. A second set is allowed only for a
named gap (brand or payment logos, say), and that exception gets documented.
Fill-based sets (Phosphor, Remix, Radix) can't be stroke-tuned, so pick a
weight instead. Radix is drawn on a 15px grid, so don't mix it with 24px sets.

## 2. Write the spec (one paragraph, before any code)

Use these terms with these meanings. The definitions are this skill's own;
for illustrated depth, follow the links to The Iconography Company's
encyclopedia (`https://encyclopedia.iconography.co/#view=directory&term=<Term>`).
**That site reserves rights against AI training and text-and-data mining**
(its robots.txt blocks AI crawlers, and it sends `noai` and
`tdm-reservation: 1`). Link to it for humans. Never scrape it, quote its
definitions, or vendor its `entries.json`.

- **Base size / frame:** the canvas the glyph is drawn on, almost always
  24×24 for UI sets. Smaller sizes are scaled versions, not separate drawings.
  ([Base size](https://encyclopedia.iconography.co/#view=directory&term=Base+size), [Frame](https://encyclopedia.iconography.co/#view=directory&term=Frame))
- **Live area / padding:** the inset the drawing stays inside, usually 1–2
  units, so icons align optically with text. ([Padding](https://encyclopedia.iconography.co/#view=directory&term=Padding), [Key line](https://encyclopedia.iconography.co/#view=directory&term=Key+line))
- **Stroke weight:** the line thickness in canvas units. Decide it in screen
  px; the wrapper converts. ([Stroke weight](https://encyclopedia.iconography.co/#view=directory&term=Stroke+weight))
- **Caps and joins:** the shape of line ends and corners. Every set in this
  skill uses round for both, so mixing in a miter set will show.
  ([End cap](https://encyclopedia.iconography.co/#view=directory&term=End+cap), [Join](https://encyclopedia.iconography.co/#view=directory&term=Join))
- **Corner radius:** the rounding on rectangles inside icons. It should echo
  the UI's `--radius` character (sharp UI, tight icon corners).
  ([Corner radius](https://encyclopedia.iconography.co/#view=directory&term=Corner+radius))
- **Optical size:** small icons need relatively heavier lines and less
  detail to read. Keep on-screen stroke constant, and drop detailed glyphs
  below 16px. ([Optical corrections](https://encyclopedia.iconography.co/#view=directory&term=Optical+corrections), [Visual weight](https://encyclopedia.iconography.co/#view=directory&term=Visual+weight))
- **Fill vs outline:** outline is the default. Use filled only for a
  *selected/active* state, never decoratively. ([Fill](https://encyclopedia.iconography.co/#view=directory&term=Fill), [Outline](https://encyclopedia.iconography.co/#view=directory&term=Outline))

Template: *"Icons: Tabler, 24px base, 1.5px on-screen stroke at all sizes,
round caps and joins, sizes 16/20/24 only, outline by default, filled only
for active nav, decorative unless the control has no text."*

## 3. Wire it (code in `references/integration.md`)

- **One `<Icon>` wrapper** (`components/ui/icon.tsx`) owns size tokens,
  constant stroke (`strokeWeight * 24 / px`), `aria-hidden` by default, and
  `label` → `role="img"` + `aria-label`. It was type-checked strict against
  lucide, tabler, iconoir and heroicons, and render-tested with hugeicons too.
- **Named imports only.** A namespace import with a dynamic lookup bundled
  7.6 MB; named imports bundled 262 KB (same icons, Vite 6). Next optimises
  lucide, tabler and heroicons by default. Add any other set to
  `optimizePackageImports`.
- **shadcn swap.** `npx shadcn@latest migrate icons --from lucide --to tabler`
  works for lucide, tabler, hugeicons, phosphor, remixicon and radix. On the
  Tailwind 3.4 CLI (`shadcn@2.3.0`) it only supports lucide ↔ radix.
  **Iconoir and Heroicons are never supported**, so those swaps are a manual
  codemod. Set `iconLibrary` explicitly: 2.3.0 defaults a missing field to
  `radix` for new-york.

## 4. When to commission custom icons

Commission (a studio such as iconography.co, or an in-house designer) when:
- the product's core metaphors don't exist in any set (domain objects or
  proprietary features);
- icons *are* the brand surface (consumer apps, marketplaces, app icons);
- you need more than ~30 bespoke glyphs to fill gaps; or
- the visual direction demands a personality no stock set has.

Otherwise extend a stock set: draw missing glyphs on its grid with its stroke
and caps, and keep them in `components/icons/` behind the same wrapper.
Budget, timeline and pricing for commissions were *(unverified)*: the studio
publishes no prices.

## When not to use

- The project already has a documented icon decision. Follow it, and use
  the wrapper section only.
- Logos and brand marks belong to `logosystem` / `logoai`. A per-concept
  icon lookup belongs to `ui-ux-pro-max` (its data is Phosphor-only).
- Illustrations and spot art are outside an icon system.

## Anti-patterns

- Mixing two stroke sets in one toolbar, or mixing 24px sets with Radix's 15px.
- Emoji or hand-rolled inline SVG standing in for system icons.
- Icon-only buttons without an accessible name on the button.
- Hex colours inside SVGs. Icons inherit `currentColor`.
- `import * as Icons` anywhere in shipped code.
- Running `migrate icons` and then uninstalling `lucide-react` before grep is
  clean (motion-primitives components may still import it).
- Shipping `@remixicon/react` ≥ 4.9.0 in a product whose icons are the thing
  being sold.
