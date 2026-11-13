# Integration — wrapper, imports, shadcn swap

Verified Oct 2026: React 19.3.1, TypeScript strict, Vite 6 build. The Next.js
specifics were read from installed Next 14.2.5 and 16.2.10 sources.

## 1. The `<Icon>` wrapper — one place that owns size, stroke and a11y

Save as `components/ui/icon.tsx`. It was type-checked under `strict` with
lucide, tabler, iconoir and heroicons components passed without casts. It was
rendered against all four plus hugeicons, and every library produced the same
width, height, stroke-width and aria output.

```tsx
import type { ComponentType, SVGProps } from "react";
import { cn } from "@/lib/utils";

export type IconComponent = ComponentType<SVGProps<SVGSVGElement>>;

const SIZES = { xs: 14, sm: 16, md: 20, lg: 24, xl: 32 } as const;
export type IconSize = keyof typeof SIZES | number;

export interface IconProps
  extends Omit<SVGProps<SVGSVGElement>, "ref" | "width" | "height" | "strokeWidth"> {
  icon: IconComponent;
  size?: IconSize;
  /** Accessible name. Omit for decorative icons (the default). */
  label?: string;
  /** Line weight in screen px. Held constant at every size. */
  strokeWeight?: number;
}

export function Icon({ icon: Glyph, size = "md", label, strokeWeight = 1.5, className, ...rest }: IconProps) {
  const px = typeof size === "number" ? size : SIZES[size];
  const strokeWidth = (strokeWeight * 24) / px; // 24-unit viewBox → constant on-screen weight
  return (
    <Glyph
      width={px}
      height={px}
      strokeWidth={strokeWidth}
      aria-hidden={label ? undefined : true}
      aria-label={label}
      role={label ? "img" : undefined}
      focusable="false"
      className={cn("shrink-0", className)}
      {...rest}
    />
  );
}
```

Verified behaviour:
- `size="sm"` (16px) at `strokeWeight` 1.5 renders `stroke-width="2.25"`, which
  is a constant 1.5 screen px. `xl` (32px) at weight 2 renders `1.5`.
- Passing `label` drops `aria-hidden` and sets `role="img"` and
  `aria-label`. This also holds for Heroicons, whose own `title` prop leaves
  `aria-hidden="true"` on the element.
- Iconoir's `1.5em` default and Heroicons' missing default size are both
  overridden by the explicit width and height.

Rules:
- An icon-only button gets its name on the **button**, not on the icon:
  `<button aria-label="Close"><Icon icon={X} /></button>`.
- Colour comes from `currentColor`. Set it with `text-*` on the icon or on
  its parent, never with a hex value inside the SVG.
- Size tokens are design tokens. If the system's type scale changes, change
  `SIZES` here, not at call sites.

### Adapters for sets that don't fit `IconComponent`

```tsx
// Hugeicons: icons are data, rendered through one component.
import { HugeiconsIcon } from "@hugeicons/react";
import { Home01Icon } from "@hugeicons/core-free-icons";
export const HomeGlyph = (p: SVGProps<SVGSVGElement>) => <HugeiconsIcon icon={Home01Icon} {...(p as any)} />;
```

- **Phosphor** is fill-based, so `strokeWeight` does nothing. Control weight
  with `weight="light"` and so on. Use the `@phosphor-icons/react/ssr` subpath
  (verified to exist) inside Server Components. The context-based
  `IconContext` defaults apply to client components; that last point is
  inferred and was not tested.
- **Remix** is fill-based too: choose `Ri*Line` or `Ri*Fill` by name.

## 2. Named imports only — the bundle rule

Measured with Vite 6 and the same 12 icons from 4 libraries:

| Import style | JS output |
|---|---|
| `import * as Lucide from "lucide-react"` + `Lucide[name]` lookup | **7,635 KB** (1,212 KB gzip) |
| `import { House, Search, … } from "lucide-react"` | **262 KB** (78 KB gzip), mostly React itself |

A namespace import with a dynamic lookup defeats tree-shaking, because the
bundler can't know which icons are used. Never build an icon picker by
looking names up in a namespace. If you truly need dynamic icons (a CMS that
stores icon names), build an explicit map of the allowed icons.

On Next.js, `optimizePackageImports` already covers `lucide-react`,
`@tabler/icons-react` and `@heroicons/react/*` by default (verified in 14.2.5
and 16.2.10). **Add any other set yourself:**

```js
// next.config.js
experimental: { optimizePackageImports: ["@phosphor-icons/react", "iconoir-react", "@hugeicons/core-free-icons"] }
```

The `react-best-practices` skill has the general barrel-import rule. This is
the icon-specific part of it.

## 3. Swapping lucide out of shadcn

`components.json` has an `iconLibrary` field, and the CLI has
`shadcn migrate icons`. **What the CLI supports depends on the CLI version,
and the version depends on your Tailwind major** (see the `shadcn` skill for
that fork):

| CLI | Libraries `migrate icons` supports |
|---|---|
| `shadcn@latest` (4.22.1, Tailwind v4) | lucide, tabler, hugeicons, phosphor, remixicon, radix |
| `shadcn@2.3.0` (Tailwind v3.4) | **lucide ↔ radix only** |

Iconoir and Heroicons are supported by **neither** CLI.

### Tailwind v4 projects

```bash
npx shadcn@latest migrate icons --from lucide --to tabler          # components/ui only
npx shadcn@latest migrate icons "app/**/*.tsx" --from lucide --to tabler   # app code too
```

What it does (read from the 4.22.1 source; not run against a live repo):
- Installs the target packages. It does **not** uninstall `lucide-react`.
- Fetches the icon name map from the registry (`icons/index.json`), so it
  needs network access.
- Rewrites imports only in `components/ui` unless you pass a path or glob.
- Sets `iconLibrary` in `components.json` **only when run without a path**.
  Run it once without a path, then again with globs for app code.
- Leaves icons that have no mapping untouched and prints them as
  "Skipped N icons". Fix those by hand, then `grep -r "lucide-react"` until
  only intentional uses remain.

After the migration, `shadcn add` transforms new components to the
configured `iconLibrary`. The transformer is present in both 2.3.0 and
4.22.1; the 4.x add-time behaviour is inferred from the 2.3.0 source.

### Tailwind v3.4 projects (shadcn@2.3.0), or Iconoir / Heroicons anywhere

The migration is a manual codemod:
1. Set `"iconLibrary"` in `components.json` explicitly. **2.3.0 defaults a
   missing field to `radix` for the `new-york` style** (verified in source),
   which silently changes what `add` writes.
2. List the lucide icons in use: `grep -rhoE "import \{[^}]+\} from \"lucide-react\"" components app`.
3. Map each name by hand. Names don't line up across sets: lucide `X` is
   iconoir `Xmark`, heroicons `XMarkIcon` and tabler `IconX`; lucide
   `ChevronRight` is iconoir `NavArrowRight`. Then switch call sites to
   `<Icon icon={…} />`.
4. Remove `lucide-react` only once grep is clean **and** no
   motion-primitives component still imports it (its install line includes
   `lucide-react`).

## 4. QA checklist

- [ ] One family per surface: grep `package.json` for more than one icon set
      and justify each one.
- [ ] Every icon goes through `<Icon>`, or a documented adapter.
- [ ] Every icon-only control has an accessible name on the control itself.
- [ ] Named imports only. Non-default sets are listed in `optimizePackageImports`.
- [ ] 16px icons still read at 1× DPR: screenshot them, don't assume.
- [ ] The licence line is in the design-arsenal phase 8 pre-flight (check Remix ≥ 4.9.0).
