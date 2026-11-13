---
name: gauge-ui
description: Install and integrate gauge-ui (gauge-ui.dev): MIT-licensed, free, composable SVG gauge primitives for React by Thordur (github.com/thordursk/gauge-ui). It covers speedometers, dials, meters, ring/donut progress, tick-fill bars, compass and clock faces, low–high range gauges, nested dials and draggable knob/slider gauges. It ships as one copy-in shadcn registry item (`npx shadcn@latest add https://gauge-ui.dev/r/gauge.json`, or `@gauge-ui/gauge` via a components.json namespace) with zero npm deps and its own spring/tween engine (no motion or framer-motion). Caveats: it requires React 19.2+, because `tooltip.tsx` imports `useEffectEvent`, so React 18 and 19.0/19.1 fail the build. On React 18 / Tailwind v3 it needs two fixes, both in references/integration.md: a useEffectEvent shim and token colours that match the repo's token format, including the dot halo (install with `shadcn@latest add` even on v3). Display gauges carry no ARIA, so you add role="meter" yourself. Use this whenever a task involves gauge-ui.dev or an @gauge-ui install, a gauge, speedometer, tachometer, dial, meter, radial or ring progress, capacity or redline indicator, compass/heading or time-of-day dial, an SLO/target-range dial, a rotary knob or circular slider, or a system-monitor / pipeline-health dashboard (RocketRide pipeline monitoring). Also use it to decide whether a dial is the right form at all; defer that judgment and all colour choices to the dataviz skill.
---

# gauge-ui

Verified Oct 2026: installed, built, rendered and probed on Next 16 / React 19.3 /
Tailwind v4 and on Next 14 / React 18 / Tailwind 3.4. Items marked **[unverified]** were
read from source or docs but not exercised.

gauge-ui is a **set of SVG gauge building blocks** (it is not a measurement tool).
The CLI copies 15 files into `components/gauge/`, and from then on you own them.
- **Maker and license:** Thordur ("Doddi", `thordursk`), MIT, free, no pro tier.
- **Maturity:** the repo was created 2026-09-27 and has a single maintainer. The API
  is still growing (ranges, GaugeStack and TickFill landed in mid-Oct 2026).
- **Studio:** the site has a visual Studio (`gauge-ui.dev/studio`) that generates the
  TSX, plus 7 example dashboards. The Monitor dashboard is the closest model for
  RocketRide.
- **Not available:** there is no npm package, `llms.txt` or MCP server. The README is
  the API doc.

## Defer first: form and colour belong to dataviz

**dataviz** (a built-in skill) owns the judgment calls: which chart form, the palette,
contrast and legend rules. This skill only implements the radial-meter case. Read
dataviz before choosing a gauge.

- **Prefer a bar/bullet chart unless the dial metaphor carries meaning.** A dial earns
  its space when the reading has a physical or cyclic sense: capacity against a limit,
  a redline, heading, time of day, a knob the user turns. A row of eight dials for
  eight unrelated KPIs should be bars.
- One hero dial per card. Small multiples of identical dials, such as per-core CPU, are
  fine. A wall of mixed dials is not.
- Colours come from dataviz-validated tokens. Pass them as strings. Never hand-pick hues
  per gauge.

## Install

CLI version policy belongs to the **shadcn** skill; read it before the first install.
Short version: install gauge-ui with `shadcn@latest add` on **both** Tailwind v4 and
v3 repos (re-verified on v3: clean placement, no config edits). Never `2.3.0 add`.

```bash
# Tailwind v4 / shadcn 4.x: verified drop-in, zero edits
npx shadcn@latest add https://gauge-ui.dev/r/gauge.json
# or register once in components.json → "registries": { "@gauge-ui": "https://gauge-ui.dev/r/{name}.json" }
npx shadcn@latest add @gauge-ui/gauge
```

- **Tailwind v3 / React 18:** follow the 3-fix recipe in `references/integration.md`.
  Do not skip it. The build fails without fix 2.
- **Registry:** `https://gauge-ui.dev/r/registry.json` (1 item, `gauge`), whose only
  registry dependency is `utils` (`cn()`).

## Mental model

`<Gauge value min max startAngle endAngle>` owns the domain and geometry. Every child
reads them from context.
- Angles are measured clockwise from six o'clock.
- Defaults: `startAngle={40} endAngle={320}` for a speedometer; `0/360` for a ring;
  `90/270` for a half dial (add `fit="content"`).

| Need | Primitives |
|---|---|
| Dial with redline | `GaugeTrack` + `GaugeZones` + `GaugeArc color={zoneColor(...)}` + `GaugeTicks` + `GaugeTickLabels` + `GaugeNeedle` + `GaugeHub` |
| Ring progress | `GaugeTrack` + `GaugeArc` + `GaugeValue` |
| Split fill (user/system, wired/app/cache) | `GaugeStack parts={[{weight, color}]}` |
| Segmented "LED" meter | `GaugeTickFill count length color={(v)=>...}` |
| Low–high band | `value={[from, to]}` + `GaugeArc` + two `GaugeDot` (one `end="from"`) |
| Secondary reading inside | `GaugeInset value x y scale` (all primitives work inside) |
| User sets the value | wrap in `GaugeControl` (slider/knob, keyboard, two thumbs for ranges) |
| Hover detail | `GaugeTooltip label unit decimals` (portalled, shadcn-tooltip styled) |

- **Animation:** `transition` on `<Gauge>`, either `true` for the default spring or
  `{ type: "spring", visualDuration, bounce }` / `{ type: "tween", duration, ease }`.
  The spring vocabulary mirrors motion's. Align it with the project's single easing
  vocabulary (design-arsenal phase 6).
- **Reduced motion:** honoured automatically (it snaps).

## Non-negotiables

Code for each is in `references/integration.md`.

- **Add meter ARIA to every display gauge.** `<Gauge>` renders a bare `<svg>`. Spread
  `role="meter" aria-label aria-valuenow aria-valuemin aria-valuemax aria-valuetext` onto
  it, using the *target* value, never the animated frame.
- **Pass colours as tokens:**
  - Tailwind v4: `"var(--chart-1)"`.
  - Tailwind v3: `"hsl(var(--chart-1))"`.
  - On v3, also pass `haloColor="hsl(var(--background))"` to `GaugeDot` whenever `halo`
    is set. The default `var(--background)` renders **black** on v3 tokens.
- **Give `<Gauge>` a sized parent.** It is `h-full w-full`. Wrap it in
  `aspect-square w-full max-w-*`, or use `fit="content"` for partial sweeps.
- **Repeat the geometry on `GaugeControl`.** It needs the same `min/max/startAngle/endAngle`
  as its inner `<Gauge>`; they are separate props and are not read from context.
- **Keep the first render deterministic.** All files are `"use client"`. They SSR-prerender
  cleanly only if the first render is deterministic: seed live data, start intervals in
  `useEffect`. `initialValue` makes the server HTML show the initial value, not the
  reading. Omit it where first paint must be accurate.
- **Pause live updates off-screen**, as upstream's Monitor example does.
- **Run `npx shadcn@latest add @gauge-ui/gauge --diff` before re-pulling.** The source is
  yours and upstream moves fast.

## When NOT to use

- **dataviz says bar or bullet.** Most single KPIs read faster as a number plus a
  sparkline or bullet.
- **Time series or comparisons across categories.** Use charts.
- **React < 19.2 when you cannot carry a local shim.** Pick a different approach rather
  than fork-drift.
- **A shadcn `chart` RadialBarChart is already on the page and a plain ring is all you
  need.** Don't add a second system for one ring. Use gauge-ui once you need a needle,
  ticks, zones, a range or a control.

## Overlap

- **dataviz:** owns form, colour and a11y judgment. gauge-ui is the radial
  implementation.
- **shadcn:** owns CLI and registry mechanics.
- **motion-primitives:** `AnimatedNumber` overlaps `GaugeValue`. Inside a gauge, use the
  gauge's own `transition`.
- **componentry:** no overlap; it is decoration, and this is data.
