# gauge-ui integration

Everything after the decision to use a gauge. That decision itself belongs to the
**dataviz** skill. Verified Oct 2026 against `gauge-ui.dev/r/gauge.json` and
`github.com/thordursk/gauge-ui` at commit `bace7dc` (2026-10-15). Items marked
**[unverified]** were read from source or the README but not exercised.

## Registry mechanics (verified)

- **Index:** `https://gauge-ui.dev/r/registry.json`. It holds one item, `gauge`, of type
  `registry:ui`, with categories `data-display` and `charts`.
- **Payload:** `https://gauge-ui.dev/r/gauge.json`. It has 15 `files[]` with full source,
  `registryDependencies: ["utils"]`, no npm `dependencies`, and a `docs` string that the
  CLI prints after install.
- **Install targets** are written as `@components/gauge/<file>`:
  - shadcn 4.x expands that to `components/gauge/`.
  - `shadcn@2.3.0` does **not** expand it (fix 1 below).
- **Files:** `index.ts gauge.tsx inset.tsx context.tsx arcs.tsx ticks.tsx text.tsx
  needle.tsx dot.tsx tooltip.tsx control.tsx math.ts utils.ts transition.ts
  use-animated-value.ts`, about 2,700 LOC. Every component file is `"use client"`.
- **Import:** always from the barrel, `import { Gauge, ... } from "@/components/gauge"`.
- **Runtime:** needs only `react`, `react-dom` and `cn()`. On shadcn 4.x, the `utils`
  item resolves `cn()` to the npm package `cn` (shadcn-ui/cn) — that comes from shadcn,
  not gauge-ui. Under 2.3.0 `cn()` stays clsx + tailwind-merge.
- **Tailwind classes** the source relies on all generated correctly on Tailwind 3.4:
  `overflow-visible`, `tabular-nums`, `sr-only`, `font-sans/mono/rounded`, the tooltip's
  `animate-in fade-in-0 zoom-in-95 bg-foreground text-background` (via
  `tailwindcss-animate` on v3 or `tw-animate-css` on v4), and the control's
  `has-[:focus-visible:not([data-pointer])]:ring-2`.
- **License:** MIT, (c) 2026 Thordur. Commercial use is unrestricted.

## Install, both routes

```bash
# By URL (any CLI version that accepts URLs)
npx shadcn@latest add https://gauge-ui.dev/r/gauge.json

# By namespace (shadcn 3+; verified on 4.22.1)
#   components.json → "registries": { "@gauge-ui": "https://gauge-ui.dev/r/{name}.json" }
npx shadcn@latest add @gauge-ui/gauge
npx shadcn@latest add @gauge-ui/gauge --dry-run   # preview files before writing
npx shadcn@latest add @gauge-ui/gauge --diff      # before any re-pull: you own the source
```

## Tailwind v3 / React 18 recipe: two fixes (three under the 2.3.0 CLI)

Verified on Next 14.2.35, React 18.3.1, Tailwind 3.4, `shadcn@2.3.0`, where
build and render pass after all three fixes. Version policy and the state of
`shadcn@2.3.0 init` are in the **shadcn** skill.

### Fix 1: file placement — avoid it by installing with `@latest`

Re-verified Oct 2026 on a fresh Next 14.2 / Tailwind 3.4 repo initialised with
`shadcn@2.3.0`: **`npx shadcn@latest add https://gauge-ui.dev/r/gauge.json -y`
lands all 15 files in `components/gauge/`**, skips the identical `lib/utils.ts`,
and leaves `tailwind.config.ts`, `globals.css`, `components.json` and
`package.json` untouched. That's the route the **shadcn** skill now prescribes
for third-party items on v3; fixes 2 and 3 still apply.

Only if you're stuck on the 2.3.0 CLI: `shadcn@2.3.0 add <url> -y` stops at a "utils.ts already exists, overwrite?" prompt even
with `-y`, which hangs non-interactive runs. Run it with `-o`: the existing
`lib/utils.ts` is replaced by an identical file. The 15 files then land in a **literal
`./@components/gauge/` folder at the repo root**.

```bash
npx shadcn@2.3.0 add https://gauge-ui.dev/r/gauge.json -y -o
mkdir -p components && git mv @components/gauge components/gauge && rmdir @components
```

Alternative: skip the CLI and copy each `files[].content` from `/r/gauge.json` into
`components/gauge/`.

### Fix 2: `useEffectEvent` shim (React 18, 19.0, 19.1)

`tooltip.tsx` imports `useEffectEvent`, which only exists in React 19.2+.
- Because `index.ts` re-exports the tooltip, **tsc and `next build` fail even if no
  tooltip is rendered**.
- The error is `Module "react" has no exported member 'useEffectEvent'`.

Replace the import block at the top of `components/gauge/tooltip.tsx` with:

```tsx
import {
  useCallback,
  useEffect,
  useLayoutEffect,
  useRef,
  useState,
  useSyncExternalStore,
  type CSSProperties,
  type ReactNode,
} from "react"
import { createPortal } from "react-dom"

/* ADAPTED for React 18 / 19.0-19.1: upstream imports React 19.2's
   useEffectEvent. Ref-backed shim with the same contract (stable identity,
   always calls the latest closure; only call it from effects/handlers). */
// eslint-disable-next-line @typescript-eslint/no-explicit-any
function useEffectEvent<T extends (...args: any[]) => any>(fn: T): T {
  const ref = useRef(fn)
  useLayoutEffect(() => {
    ref.current = fn
  })
  return useCallback(((...args) => ref.current(...args)) as T, [])
}
```

After the fix, `useEffectEvent` is used once (`const place = useEffectEvent(...)`). On
React 18 the tooltip worked with the shim: hover showed the portalled bubble with the
right value, and there were no console errors. Re-apply the fix after every re-pull.
Remove it when the repo reaches React 19.2.

### Fix 3: token colours on v3, including the dot halo

What matters is the **token format in `globals.css`, not the Tailwind version**. Check
`--background` first:

- **Bare HSL channels** (`--background: 0 0% 100%`, classic shadcn v3 and most existing
  v3 repos): a bare `var(--x)` is not a colour, so wrap it in `hsl()`.
- **Full colour values** (`--background: oklch(1 0 0)`, which is Tailwind v4 *and* what
  today's `shadcn@2.3.0 init` writes into fresh v3 repos): `var(--x)` is already a colour.
  `hsl(oklch(...))` is invalid, and the default halo already works.

```tsx
// lib/tokens.ts: one helper, so every gauge colour goes through the token system
export const tok = (name: string) => `hsl(var(--${name}))`   // bare HSL channel tokens
// export const tok = (name: string) => `var(--${name})`     // full oklch/hex tokens

<GaugeArc color={tok("chart-2")} />
<GaugeDot halo={4} haloColor={tok("background")} />   // REQUIRED on v3
```

`GaugeDot`'s default `haloColor` is the hard-coded `"var(--background)"`. With bare-HSL
tokens it computes to `rgb(0,0,0)`, giving a black blob around the thumb in light mode; in dark
mode the bug is hidden. Everything else defaults to `currentColor`:
- the track draws at opacity 0.15;
- dim ticks use `fadeColor()`, which is `color-mix(in oklch, ...)`.

So an untouched gauge inherits the parent's `text-*` on both Tailwind versions.

## Accessibility

### Display gauges: add `role="meter"` yourself

`<Gauge>` spreads extra props onto its `<svg>`, which has no role or `<title>` of its
own. The following was verified in the browser: the role and values reached the
accessibility tree.

```tsx
export const meterProps = (label: string, value: number, min: number, max: number, spoken: string) => ({
  role: "meter" as const,
  "aria-label": label,
  "aria-valuenow": Math.round(value),   // the TARGET value, not the animated frame
  "aria-valuemin": min,
  "aria-valuemax": max,
  "aria-valuetext": spoken,             // "711 documents per minute"
})

<Gauge value={throughput} min={0} max={1000} transition
  {...meterProps("Ingest throughput", throughput, 0, 1000, `${Math.round(throughput)} documents per minute`)}>
```

For a range value (`[from, to]`), `role="meter"` does not fit. Use `role="img"` with an
`aria-label` that states both ends **[unverified with a screen reader]**. `GaugeValue` and
`GaugeText` are `<text>` inside the SVG. Screen readers may read them in addition to the
meter; add `aria-hidden` to decorative ones **[unverified]**.

### GaugeControl: slider semantics and keyboard

`GaugeControl` wraps a gauge and supplies the accessible interaction:
- **Single value:** the wrapper is `role="slider"` with `aria-valuenow/min/max`,
  `aria-valuetext` (from `valueText`) and `tabIndex=0`.
- **Range:** the wrapper is `role="group"` (`aria-label=label`) with two `sr-only`
  `role="slider"` thumbs, named by `thumbLabels`. Each thumb's min/max is clamped to the
  other end. Verified: "SLO low" 0–380 and "SLO high" 120–500.
- **Keys** (from source; arrows verified, ArrowRight ×5 moved 120 → 125):

| Key | Effect |
|---|---|
| `ArrowUp`, `ArrowRight` | +`step` (default 1) |
| `ArrowDown`, `ArrowLeft` | −`step` |
| `PageUp`, `PageDown` | ±10% of the domain |
| `Home`, `End` | `min` / `max` |
| `Enter`, `Space` | `onPress()` if given, for a dial that is also a switch |

- **Pointer:** drag round the ring, or with `knob` turn the face. On a range, `knob`
  moves the whole band and keeps its width.
- **Focus:** `focus-visible` shows a `ring-ring` ring, and pointer focus does not show
  it.
- **Tooltip:** `GaugeTooltip` stays open during focus and press, so it works on keyboard
  and touch **[source; touch unverified]**.

```tsx
<GaugeControl value={slo} onChange={(v) => setSlo(v as [number, number])}
  min={0} max={500} startAngle={40} endAngle={320}        // must match the inner Gauge
  label="Latency SLO" thumbLabels={["SLO low", "SLO high"]}
  valueText={[`${slo[0]} ms`, `${slo[1]} ms`]}>
  <Gauge value={slo} min={0} max={500} startAngle={40} endAngle={320} transition>
    <GaugeTrack width={14} />
    <GaugeArc width={14} color={tok("chart-2")} />
    <GaugeDot end="from" radius={10} />
    <GaugeDot radius={10} halo={4} haloColor={tok("background")} />
    <GaugeValue decimals={0} fontSize={30} weight="semibold" />
  </Gauge>
</GaugeControl>
```

### Reduced motion (verified)

`use-animated-value.ts` checks `matchMedia("(prefers-reduced-motion: reduce)")` and snaps.
With `initialValue={0}` and a spring, the first frames read:
- **no-preference:** `0 71 242 362 482 557` (the sweep);
- **reduce:** `0 640 640 640` (immediate).

No extra work is needed. Don't add a second motion layer, such as motion-primitives'
`AnimatedNumber`, on top of a gauge, because it would not inherit this gate.

## SSR (verified)

- All files are client components. A page using them prerenders as static, with no
  hydration warnings, as long as the first render is deterministic: seed values, and
  start intervals in `useEffect`.
- `initialValue` is what the server HTML shows. Omit it where first paint must show the
  real reading.
- Import gauges from a client component. Pass plain numbers down from server
  components.

## Pattern: system-monitor dashboard (RocketRide pipeline monitoring)

Modelled on upstream's `components/examples/monitor-dashboard.tsx` (the
"Monitor" example on gauge-ui.dev), mapped onto pipeline metrics. Keep the
dataviz rule in front: dials only where the metaphor is real.

| Card | Metric | Why a dial (or not) | Primitives |
|---|---|---|---|
| Throughput | docs/min vs lane capacity | capacity + redline | `GaugeZones` + zone-coloured `GaugeArc` + `GaugeNeedle` + `GaugeTooltip` |
| Worker CPU | busy %, split user/system | share of a whole | `GaugeStack` weights + `GaugeTicks` + mono `GaugeValue` |
| Per-worker | small multiples, one per worker | identical scale, scan for outliers | `GaugeZones` + `GaugeArc`, `font="mono"` |
| Memory + swap | GB of limit, swap as inner ring | capacity, two related limits | `GaugeStack` + `GaugeInset` |
| Queue depth | jobs waiting | segmented level | `GaugeTickFill color={(v) => v > redline ? tok("destructive") : tok("primary")}` |
| Latency SLO | p50–p95 band, editable target | target range | `[from, to]` value, optionally inside `GaugeControl` |
| Error rate over time, per-node tables | not dials | dataviz → line chart / table | none |

Upstream conventions worth copying:
- **One arrival spring for the whole board:**
  `const ARRIVE = { type: "spring", visualDuration: 1, bounce: 0 }`. That is a calm
  settle, not a bounce, for monitoring.
- **Named colour roles, not hues.** Upstream uses `MUTED = "var(--muted-foreground)"`,
  `ALERT = "var(--foreground)"` and `CLEAR = "transparent"`. Monochrome dials with alert
  emphasis read better than rainbow zones; take the alert colour from dataviz.
- **Pause updates off-screen.** Upstream gates its `setInterval(step, 1500)` on a
  `visible` flag from an in-view hook. For live data, gate the subscription the same way.
- **Partial sweeps get `fit="content"`.** It keeps half dials from reserving an empty
  lower half; `cap="butt"` keeps segment ends crisp.
- **Wire live data at the edge.** Feed values from the RocketRide client's events or a
  polling hook. Gauges only take numbers.

Minimal card shell (verified layout):

```tsx
<section className="rounded-xl border bg-card p-4 text-card-foreground">
  <h2 className="text-sm font-medium">Throughput</h2>
  <p className="text-xs text-muted-foreground">docs/min, ingest lane</p>
  <div className="mx-auto mt-2 aspect-square w-full max-w-[220px]">
    <Gauge value={v} min={0} max={1000} startAngle={40} endAngle={320} transition
      {...meterProps("Throughput", v, 0, 1000, `${Math.round(v)} docs per minute`)}>
      <GaugeTrack width={12} />
      <GaugeZones zones={zones} width={4} offset={14} />
      <GaugeArc width={12} color={zoneColor(zones, v, "currentColor")} />
      <GaugeTicks count={20} />
      <GaugeTickLabels count={5} fontSize={18} color={tok("muted-foreground")} />
      <GaugeNeedle style="pointer" />
      <GaugeHub />
      <GaugeValue decimals={0} y={70} fontSize={34} weight="semibold" />
      <GaugeTooltip label="Throughput" unit=" docs/min" decimals={0} />
    </Gauge>
  </div>
</section>
```

## Gotchas (verified unless marked)

- **`font="rounded"`** emits `font-rounded`. Define a `rounded` font family in the
  Tailwind theme, or it falls back silently **[fallback unverified]**.
- **`fadeColor()` needs `color-mix(in oklch)`**, which means a 2023+ browser (Chrome
  111+, Safari 16.2+) **[unverified]**.
- **Bundle cost:** the 4-gauge demo route was 18.2 kB of route JS on the v3 build.
- **Local test servers:** other agents' dev servers share localhost. Bind tests to
  `127.0.0.1` on an unusual port.
