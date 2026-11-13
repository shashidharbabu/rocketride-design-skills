# Icon sets — verified facts (Oct 2026)

Every number here came from `npm view` or from rendering the package with
`react-dom/server` under React 19.3.1 (18 Oct 2026). Re-run `npm view <pkg>
version license peerDependencies` before you quote a version: these move.

## Stroke-based sets (all work with the `<Icon>` wrapper)

| Package | Ver | License | React peer | Icons¹ | Default box | Default stroke | Stroke prop | `aria-hidden` default | shadcn migrate (4.x) | Next default `optimizePackageImports` |
|---|---|---|---|---|---|---|---|---|---|---|
| `lucide-react` | 1.59.0 | ISC | 16.5–19 | ~1,890 | 24px | 2 | `strokeWidth` (+`absoluteStrokeWidth`) | `true`; removed when `aria-label` is passed | yes (`lucide`) | yes |
| `@tabler/icons-react` | 3.49.0 | MIT | ≥16 | ~6,240 (1,058 *Filled*) | 24px | 2 | `stroke` (documented); `strokeWidth` also renders | none | yes (`tabler`) | yes |
| `iconoir-react` | 7.12.1 | MIT | 18 ‖ 19 | 1,673 | **1.5em** | **1.5** | `strokeWidth` | none | **no** | **no** |
| `@heroicons/react/24/outline` | 2.2.0 | MIT | ≥16 | 324 | **none (fills its parent)** | **1.5** | `strokeWidth` | `true`, **even with `title`** | **no** | yes |
| `@hugeicons/react` + `@hugeicons/core-free-icons` | 1.1.10 / 4.3.5 | MIT / MIT | ≥16 | 6,025 `*Icon` exports² | 24px | **1.5** | `strokeWidth` on `<HugeiconsIcon icon={…}>` | none | yes (`hugeicons`) | **no** |

## Fill-based sets (no stroke to tune)

| Package | Ver | License | Icons¹ | Default box | Weight control | `aria-hidden` default | shadcn migrate (4.x) | Next default |
|---|---|---|---|---|---|---|---|---|
| `@phosphor-icons/react` | 2.1.10 (last publish May 2025) | MIT | ~1,510 | **1em**, viewBox 256 | `weight`: thin/light/regular/bold/fill/duotone | none | yes (`phosphor`) | **no** |
| `@remixicon/react` | 4.9.0 | **Remix Icon License 1.0** (see below) | 3,223 (1,538 *Line*) | 24px | Line vs Fill variants | none | yes (`remixicon`) | **no** |
| `@radix-ui/react-icons` | 1.3.2 | MIT | 318 | **15px, viewBox 15** | none | none | yes (`radix`) | **no** |

¹ Distinct component identities at runtime. Aliases are deduplicated except where noted.
² Hugeicons icons are data objects, so aliases were not deduplicated.

## Licence notes

- **Remix Icon changed licence at 4.9.0** (Jan 2026). Versions up to 4.8.0 are
  Apache-2.0. Remix Icon License 1.0 still allows free commercial use,
  modification and use in apps and UI kits. It **forbids selling or
  redistributing the icons as a standalone icon pack, library or font**.
  That's fine for a product UI. It's a blocker if you're building a
  design-system package for resale. The core `remixicon` package's npm
  metadata still says Apache-2.0, so read the LICENSE file, not the npm
  metadata.
- **Hugeicons**: the free package is MIT. A Pro tier exists. Never import
  from a Pro package without a purchased licence. (Pro pricing and terms were
  not verified in this session.)
- Lucide (ISC), Tabler, Iconoir, Heroicons, Phosphor and Radix (MIT) all
  need a licence notice kept in redistributed source. App use needs nothing
  beyond that.

## Fit notes (judgement, not measurement)

- **Pairs with lucide without a visible seam:** Tabler. Both use a 24px box,
  round caps and joins, and stroke 2, and both are first-class in shadcn
  migrate. Also the largest catalogue here.
- **Native 1.5px stroke, lighter and quieter:** Iconoir, Heroicons outline,
  Hugeicons. Heroicons only has 324 icons, so check coverage against your
  feature list before committing.
- **Several weights from one family:** Phosphor. Pick one weight per project
  and don't mix weights.
- **Radix** is drawn on a 15px grid. Don't mix it with 24px sets in the same
  toolbar.

## Re-verify quickly

```bash
npm view lucide-react @tabler/icons-react iconoir-react version license   # one at a time
grep -o '"lucide-react"\|"@tabler/icons-react"' node_modules/next/dist/server/config.js
```

Next's default optimised list was read from `next/dist/server/config.js` in
14.2.5 and 16.2.10. Both include `lucide-react`, `@tabler/icons-react` and
`@heroicons/react/{20/solid,24/outline,24/solid}`. Neither includes
Phosphor, Iconoir, Hugeicons, Remix or Radix icons.
