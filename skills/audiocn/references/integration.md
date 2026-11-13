# audiocn integration

Everything after the stack gate in SKILL.md. Verified Oct 2026 against
registry `https://audiocn.dev/r/registry.json` (50 items), shadcn 4.22.1,
Next 16.4.2, React 19.3.0, Tailwind v4, headless Chromium click-through.
Statements marked *(unverified)* come from docs or source reading only.

## Registry mechanics

- Index: `https://audiocn.dev/r/registry.json`. Item: `https://audiocn.dev/r/<name>.json`
  (`files[].content` is the full source; `registryDependencies` are
  namespaced `@audiocn/...` plus bare shadcn names like `button`, `badge`).
- Listed in the shadcn directory (`ui.shadcn.com/r/registries.json`) as
  `@audiocn` with URL `https://www.audiocn.dev/r/{name}.json`; the CLI adds that
  entry to `components.json` on first `add`.
- Agent docs: `https://audiocn.dev/llms.txt`, `/llms-full.txt`, and
  `<any docs url>.md`. Each component `.md` lists exactly which files and CSS
  vars the install writes and which example-only hooks it does not.
- License: MIT, © 2026 OrcDev (`license.md` — lowercase, so GitHub shows
  "Other"). The homepage's Warcraft "work work" sample is Blizzard's, not in
  the registry, never installed.
- `npm i audiocn` = unrelated placeholder (0.0.0, other maintainer). Never.

## Inventory (50 items)

Every component depends on `@audiocn/core` (`lib/audio/*`: types, decibels,
ballistics, zones, taper, bands, bar-levels, wave-line, time, frame-loop,
frame-source, history, history-playback) and its 6 audio tokens. External npm
deps are only `@base-ui/react`, `class-variance-authority`, and (blocks)
`@phosphor-icons/react`. No motion/framer-motion anywhere.

| Item | npm deps | Notable registry deps / notes |
|---|---|---|
| `level-meter` | cva | db-scale, db-readout, clip-indicator, use-reduced-motion, use-visibility |
| `db-scale` | — | use-audio-config |
| `db-readout` | — | use-frame-source |
| `clip-indicator` | base-ui | use-clip-hold |
| `bar-visualizer` | — | use-reduced-motion, use-visibility |
| `electric-bar-visualizer` | — | `electric` lib |
| `electric-waveform` | — | `electric` lib |
| `smooth-waveform` | — | |
| `live-waveform` | — | canvas, scrolling history or current frame |
| `spectrum` | — | |
| `waveform` | — | file peaks; `WaveformCanvas/Cursor/Hover/Region/Marker`; variants bars/line/mirror |
| `fader` | base-ui, cva | db-scale; tapers, detents |
| `parameter-slider` | base-ui | |
| `knob` | cva | SVG; use-audio-context |
| `pan-control` | base-ui, cva | |
| `volume-control` | base-ui | |
| `channel-toggle` | base-ui, cva | mute / solo / monitor |
| `lcd-display` | base-ui, cva | dot-matrix readout |
| `audio-device-select` | — | shadcn `select`, `button` via Base UI APIs — Base UI project only *(unverified)* |
| `channel-strip` | base-ui, cva | shadcn `badge` — answer **No** if CLI offers to overwrite a customized one |
| `mixer` | — | channel-strip |
| `audio-player` | base-ui | use-audio-player, volume-control |
| `track-list` | base-ui, cva | |
| `sound-pad` | cva | shadcn `kbd` |

Hooks: `use-frame-source`, `use-level`, `use-clip-hold`,
`use-reduced-motion`, `use-visibility`, `use-demo-signal`,
`use-audio-context` (shared context, auto-resume on gesture), `use-gain-node`,
`use-audio-analyser`, `use-audio-devices`, `use-microphone`,
`use-system-audio`, `use-audio-config`, `use-mixer`, `use-web-audio-mixer`
(ducking, monitor sends, limited master), `use-audio-player`, `use-sound`,
`use-waveform-data` (decode → cached peaks, default 512).

Blocks (all `@phosphor-icons/react` + shadcn base components, Base UI
project): `system-audio-mixer`, `mic-setup`, `system-audio-settings`,
`quick-audio-popover`, `soundboard`, `music-player`.

## Tailwind v4 install recipe (verified)

```bash
# fresh app (skip on an existing v4 + shadcn repo)
npx create-next-app@latest app --ts --tailwind --eslint --app --use-npm --yes
npx shadcn@latest init -d -y --no-monorepo          # preset base-nova (Base UI)

npx shadcn@latest add @audiocn/audio-player --dry-run
npx shadcn@latest add @audiocn/audio-player @audiocn/waveform @audiocn/use-waveform-data \
  @audiocn/bar-visualizer @audiocn/use-audio-analyser @audiocn/level-meter -y
npm run build && npx tsc --noEmit && npx eslint .
```

Result: 32 files, `@base-ui/react` added, 36 CSS vars. Build, typecheck, SSR
prerender pass; eslint needs the waveform fix below.

Wiring that was run end to end:

```tsx
"use client";
import { Waveform, WaveformCanvas, WaveformCursor, WaveformHover } from "@/components/ui/waveform";
import { BarVisualizer } from "@/components/ui/bar-visualizer";
import { LevelMeter } from "@/components/ui/level-meter";
import { useAudioPlayer } from "@/hooks/use-audio-player";
import { useWaveformData } from "@/hooks/use-waveform-data";
import { useAudioAnalyser } from "@/hooks/use-audio-analyser";

export function EpisodeScrubber({ src }: { src: string }) {
  const player = useAudioPlayer({ src });
  const waveform = useWaveformData(src);
  const analyser = useAudioAnalyser(player.element);
  return (
    <>
      <Waveform aria-label="Episode" className="h-20 text-primary"
        duration={waveform.duration} loading={waveform.status !== "ready"}
        peaks={waveform.peaks} time={player.time}
        onSeekCommitted={(t) => player.seek(t)}>
        <WaveformCanvas /><WaveformCursor /><WaveformHover />
      </Waveform>
      <button type="button" onClick={() => player.toggle()}>{player.playing ? "Pause" : "Play"}</button>
      <BarVisualizer aria-label="Output" className="h-16 text-primary" source={analyser.visual} />
      <LevelMeter aria-label="Output level" source={analyser.meter} />
    </>
  );
}
```

`<AudioPlayer src>` owns its own element; pass `player={controller}` to share
one `useAudioPlayer` between the composable player and a waveform.
Visualizer color comes from `text-*` (currentColor).

## Why Tailwind 3.4 renders broken

Tested: same page on Tailwind 3.4.19. Build passes, playback works, no console
errors — but the level meter disappears (height 0), bar visualizer collapses
into slabs, seek and volume tracks vanish. Cause, confirmed by grepping the
built CSS: v3 does not understand these, so it emits nothing.

1. **CSS-variable shorthand `utility-(--var)`** — 83 distinct uses across 19
   items. Worst offenders: `lcd-display` (18), `fader` (10), `knob` (8),
   `sound-pad` (6), `level-meter`, `pan-control`, `volume-control`,
   `waveform` (5 each), `bar-visualizer` (4). Prefixes seen: `w- h- size-
   min-w- min-h- max-w- max-h- left- bottom- gap- rounded- grid-cols-
   scale-x- origin- mask- text- bg- fill- stroke- ring- shadow- from- to-`,
   including variant forms (`before:bg-(--channel-accent)`,
   `disabled:text-(--lcd-display-ink-dim)`).
2. **Renamed utilities:** `outline-hidden` (audio-player, fader, pan-control,
   parameter-slider, volume-control), `shadow-xs` (channel-strip),
   `bg-linear-to-b` and `mask-size` (lcd-display).
3. **Theme tokens:** the CLI writes the audio colors as `@theme inline`
   (`--color-meter-ok`…). On a v3 project it instead appends a meaningless
   `.theme { --color-meter-*: … }` block and never touches
   `tailwind.config`, so `bg-meter-clip`, `bg-meter-warn`,
   `text-meter-clip-foreground` don't exist.

What a port needs (only if a v4 migration is off the table — it is
maintenance debt on every re-add):

| v4 | v3.4 equivalent |
|---|---|
| `w-(--x)`, `h-`, `left-`, `gap-`, `size-`, `min-w-`… | `w-[var(--x)]` etc. |
| `bg-(--x)`, `text-`, `fill-`, `stroke-`, `ring-`, `from-`, `to-` | `bg-[color:var(--x)]` — type hint needed, v3 can't infer color from a var |
| `shadow-(--x)` | `[box-shadow:var(--x)]` |
| `mask-(--x)`, `mask-size` | `[mask-image:var(--x)]`, `[mask-size:…]` (v3 has no mask utilities) |
| `grid-cols-(--x)` | `grid-cols-[var(--x)]` |
| `scale-x-(--x)` | `scale-x-[var(--x)]` |
| `outline-hidden` | `outline-none` |
| `shadow-xs` | `shadow-sm` |
| `bg-linear-to-b` | `bg-gradient-to-b` |
| `@theme inline` colors | add `meter-ok/warn/clip`, `channel-mute/solo/monitor` (+ `-foreground`) to `theme.extend.colors` as `var(--…)` |

*(The port table is derived from the registry source; only the breakage, not
a completed port, was verified.)* React 18 additionally blocks on
`useEffectEvent` — no CSS port fixes that.

## Token remap (design-arsenal phase 4)

The CLI writes defaults into `:root` / `.dark`:

```css
--meter-ok: oklch(0.696 0.17 162.48);   --meter-warn: oklch(0.795 0.184 86.047);
--meter-clip: var(--destructive);        --channel-mute: var(--destructive);
--channel-solo: oklch(0.795 0.184 86.047); --channel-monitor: oklch(0.623 0.214 259.815);
--meter-ok-foreground: color-mix(in oklch, var(--meter-ok), black 40%);  /* dark: = base */
```

Remap onto the palette's status scale instead of accepting the defaults
(names below are an example palette):

```css
:root {
  --meter-ok: var(--success);  --meter-warn: var(--warning);  --meter-clip: var(--destructive);
  --channel-mute: var(--destructive); --channel-solo: var(--warning); --channel-monitor: var(--info);
}
```

Then delete the self-referential lines shadcn 4.22.1 wrote into
`@theme inline` (`--meter-clip: var(--meter-clip);`,
`--channel-mute: var(--channel-mute);`, every
`--*-foreground: var(--*-foreground);`). They render fine (unlayered `:root`
wins) but are noise. Keep the `--color-meter-*: var(--meter-*)` lines.
Phase 8: check meter zones and the `-foreground` labels against both
backgrounds — warn-yellow on white is the usual failure.

## Cross-origin audio *(unverified)*

`useAudioAnalyser(element)` reroutes the element through Web Audio
(`createMediaElementSource`, cached per element). Remote audio without CORS
plays but the analyser reads zeros, and `useWaveformData` (which fetches and
decodes) fails. Fix both ends:

```tsx
const player = useAudioPlayer({ src: cdnUrl, crossOrigin: "anonymous" });
// <AudioPlayer src={cdnUrl} crossOrigin="anonymous" />
```

and the CDN must send `Access-Control-Allow-Origin` for the page origin.

## One player at a time (verified)

Not built in: two players play over each other. Drop this hook into
`hooks/use-exclusive-audio-player.ts` and use it wherever `useAudioPlayer`
was used (pass the controller to `<AudioPlayer player={…}>`). Verified:
starting player B pauses A and vice versa; passes eslint-config-next 16.

```ts
"use client";
import { useEffect, useRef, useState } from "react";
import { useAudioPlayer, type UseAudioPlayerOptions } from "@/hooks/use-audio-player";

let stopCurrent: (() => void) | null = null;

export const useExclusiveAudioPlayer = (options: UseAudioPlayerOptions) => {
  const pauseRef = useRef<() => void>(() => {});
  const [stop] = useState(() => () => pauseRef.current());
  const player = useAudioPlayer({
    ...options,
    onPlay: () => {
      if (stopCurrent && stopCurrent !== stop) stopCurrent();
      stopCurrent = stop;
      options.onPlay?.();
    },
  });
  useEffect(() => { pauseRef.current = player.pause; });
  useEffect(() => () => { if (stopCurrent === stop) stopCurrent = null; }, [stop]);
  return player;
};
```

## Waveform lint fix (verified)

eslint-config-next 16 reports 3 `react-hooks/refs` errors on one line of
`components/ui/waveform.tsx` (WaveformRegion's pointer-down handler) — a
false positive, it runs in an event handler. Fix:

```tsx
      event.currentTarget.setPointerCapture(event.pointerId);
      // eslint-disable-next-line react-hooks/refs -- event handler, not render
      dragRef.current = { end, kind, start, x: event.clientX };
```

Re-apply after any re-add. `audio-player.tsx` also draws a
`@next/next/no-img-element` warning (Artwork) — warning only.

## Autoplay and testing

- `autoPlay` defaults to false; leave it (uppbeat skill: never autoplay with
  sound). `useAudioContext` resumes the shared context on the first
  `pointerdown`/`keydown`/`touchend`.
- Headless Chromium reported the context `running` before any gesture, so
  headless tests do not prove real-browser suspension behaviour.
- `useAudioPlayer` creates `new Audio()` — no `<audio>` in the DOM. Assert on
  button labels / `player.status` / `player.playing`.
- Generate a test fixture instead of committing licensed audio:
  `ffmpeg -f lavfi -i "sine=frequency=440:duration=10" -ac 2 public/test-tone.wav`.

## Worked example: a production RocketRide app (read-only check, Oct 2026)

Its `frontend/`: Next ^16.2.10 ✓, React ^19.2.7 ✓,
motion ^13, **Tailwind ^3.4.19 ✗**, **no `components.json`**, and
`wavesurfer.js ^7.8.6` already drives `components/editor/WaveformPlayer.tsx`
(RegionsPlugin draws EDL regions).

- Product fit is strong: `waveform` + `WaveformRegion`/`WaveformMarker` maps
  onto the EDL editor; `audio-player`, `level-meter`, `bar-visualizer` fit
  playback and recording.
- Stack fit is blocked by Tailwind 3.4. **Now:** keep wavesurfer; use
  audiocn's source as an API/design reference only. **Later:** adopt as part
  of a dedicated Tailwind v4 + shadcn init migration, replacing wavesurfer in
  one pass — never two waveform engines on one screen. Hand-porting into v3
  is the last resort.

## Alternatives in the shadcn directory (not evaluated)

`@audio-ui` (audio-ui.dev, ~235 items, headless `@audio-ui/react`),
`@elevenlabs-ui` (agent + audio components; poor directory availability
Oct 2026), `@audiocn-solid` (Solid port).
