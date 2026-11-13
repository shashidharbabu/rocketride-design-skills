---
name: audiocn
description: Install and integrate audiocn (audiocn.dev) — a free, MIT-licensed shadcn registry of ~50 audio UI items for React by OrcDev (github.com/audiocn/ui): audio player, file waveform with seek/regions/markers, level meters, bar/spectrum/electric visualizers, live waveform, faders, knobs, pan, mute/solo toggles, channel strips, a full Web Audio mixer, sound pads, mic/device pickers, plus the Web Audio hooks that drive them (shared AudioContext, analyser, gain, microphone, system-audio capture, waveform peaks) — copied in via `npx shadcn@latest add @audiocn/<name>`, built on Base UI and canvas/rAF with no motion/framer-motion dependency. HARD GATE: Tailwind v4 + React 19.2+ only — on Tailwind 3.4 it builds green and renders broken (meters vanish, bars collapse), and on React 18 the hooks fail (`useEffectEvent`). TRAP: `npm i audiocn` installs an unrelated placeholder package — audiocn arrives only through the shadcn registry. Use this whenever a task involves audiocn.dev or an @audiocn/ install, an audio or podcast/episode player, a waveform seek bar or waveform regions/markers, a VU/level/dB meter, an audio visualizer or spectrum, voice-activity bars, a mic check or input-device picker, a mixer/fader/knob/channel strip, a soundboard or sound pads, replacing wavesurfer.js, or deciding whether an audio product repo can take these components at all. Inventory, deps, token remap, the v3 port list, crossOrigin, single-player coordination, and the waveform lint fix live in references/integration.md.
---

# audiocn

audiocn is a **free, MIT-licensed shadcn registry of audio UI for React**
(© 2026 OrcDev; co-built with fortysevenfx; repo `github.com/audiocn/ui`).
The CLI copies TypeScript source into your repo — meters, visualizers,
controls, a player, a mixer, and the Web Audio hooks behind them. It is
**UI for audio**, not a sound-effects library: nothing here plays click
sounds. Verified Oct 2026 by a real install, build, and headless click-through.

- **Distribution:** listed in the official shadcn registry directory as
  `@audiocn` → `https://www.audiocn.dev/r/{name}.json`. `add @audiocn/<name>`
  works without editing `components.json` first; the CLI writes the
  `registries` entry itself.
- **Agent docs:** `audiocn.dev/llms.txt`, `/llms-full.txt` (~300 KB), and any
  docs page with `.md` appended (e.g. `/docs/components/waveform.md`) returns
  install steps + full API. Fetch the `.md` before writing props.
- **Trap:** the npm package `audiocn` (0.0.0, a different maintainer) is an
  unrelated placeholder. Never `npm install audiocn`.
- **Young project** (repo created 2026-09-30, near-daily feature commits).
  The copied source is yours; diff before any re-add.

## Stack gate — check before the first command

Read `package.json` and `components.json` first.

| Repo | Verdict |
|---|---|
| Tailwind **v4** + React **≥ 19.2** + shadcn 4.x | Works. Verified: Next 16.4.2, React 19.3, shadcn 4.22.1 — build, SSR prerender, playback, zero console errors. |
| Tailwind **3.4** (any React) | **Do not install.** Builds green, renders broken — 83 distinct v4-only `x-(--var)` utilities across 19 items silently drop. Migrate to v4 first, or do the documented port (references/integration.md). Verified. |
| React **18** | No — hooks import `useEffectEvent` (React 19.2). Inferred from source + docs, not run. |
| Radix-based shadcn project | Components work (CLI adds `@base-ui/react` beside Radix). The 6 blocks and `audio-device-select` need a Base UI project (per docs, unverified). |

CLI version policy (which `shadcn` to run on which Tailwind) belongs to the
**shadcn** skill. audiocn-specific fact: only a current v4-era CLI can resolve
its namespaced registry dependencies — older CLIs fail looking for
`@audiocn/core` on ui.shadcn.com (verified).

## Install

```bash
npx shadcn@latest add @audiocn/level-meter --dry-run   # see files, deps, CSS vars first
npx shadcn@latest add @audiocn/audio-player @audiocn/waveform @audiocn/use-waveform-data
```

Every item pulls `@audiocn/core` (13 files in `lib/audio/`) and 6 audio
tokens with `-foreground` pairs into `globals.css`. Doc examples import hooks
the component install does not bring (`use-audio-player`,
`use-waveform-data`, `use-demo-signal`) and a site-only `@/lib/docs/use-demo-audio`
— add the hooks explicitly; replace the demo import with your own `src`.

## Pick the right item

| Need | Item(s) |
|---|---|
| Play an episode/track | `audio-player` (composable: Play, Seek, Time, Volume, Rate, Loop, Skip, Prev/Next, Artwork, Title) |
| Waveform seek bar, edit regions | `waveform` + `use-waveform-data` (+ `WaveformRegion`, `WaveformMarker`) |
| Meter a signal | `level-meter` (peak/RMS, zones, hold, clip), `db-readout` |
| Visualize sound | `bar-visualizer`, `spectrum`, `live-waveform`, `smooth-waveform`; `electric-*` for spectacle |
| Mic check / device pick | `mic-setup` block, `audio-device-select`, `use-microphone` |
| Mixing desk | `system-audio-mixer` block (installs most of the library) |

Feed meters and visualizers with a frame source:
`useAudioAnalyser(player.element | MediaStream | AudioNode)` → `.meter` /
`.visual`. They paint on rAF without re-rendering React.

## When NOT to use

- **Tailwind 3.4 repos** without a v4 migration plan (gate above).
- **A working wavesurfer.js integration** you aren't rebuilding — don't put
  two waveform engines on one screen.
- **Background music on a marketing page** — that is a restraint and
  licensing question for **uppbeat**, not a component question.
- **A bare `<audio controls>` is enough** (single clip, no seek UI) — 0 KB.

## Non-negotiables

Details and code in `references/integration.md`.

- **No autoplay with sound.** `autoPlay` defaults to false — keep it there
  (uppbeat's rule). The shared AudioContext resumes on the first
  pointerdown/keydown/touchend; never try to start audio on load.
- **Remap the six audio tokens** (`--meter-ok/warn/clip`,
  `--channel-mute/solo/monitor`) onto the phase-3 palette — design-arsenal
  **phase 4**. Clean the self-referential `@theme inline` lines the CLI adds.
- **Cross-origin audio needs `crossOrigin="anonymous"`** plus CORS headers,
  or the analyser reads silence. Inferred from API/source, not reproduced.
- **One player at a time** is not built in — two players play over each
  other. Use the `useExclusiveAudioPlayer` snippet (verified).
- **Lint:** eslint-config-next 16 fails `components/ui/waveform.tsx` on
  `react-hooks/refs` (false positive in a pointer handler). Apply the
  one-line disable (verified).
- **QA — design-arsenal phase 8:** check `prefers-reduced-motion` (bar
  visualizer and level meter honor it via `useReducedMotion`; verify any
  custom canvas), meter-zone contrast on both themes, keyboard seek on
  waveform/seek sliders, and that nothing makes sound before a gesture.
- Tests can't find the media element in the DOM: `useAudioPlayer` creates
  `new Audio()`. Assert on UI state or the controller instead.
