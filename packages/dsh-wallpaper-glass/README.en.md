# dsh-wallpaper-glass

Wallpaper-engine backgrounds with iOS-style liquid glass for the DSH web GUI —
a slim fork that keeps wallpaper and glass only.

Based on [elysia395/dsh-wallpaper-engine](https://github.com/elysia395/dsh-wallpaper-engine) **v1.4.2** (MIT License).

> Full documentation, the removal list and this fork's changes live in the
> **[repository root README](../../README.md)**. This file covers the package itself.

---

## What this package is

The main package, host half and browser half combined:

- **Host half** `lib/index.js` — locates the local Wallpaper Engine install
  (Steam app 431960), enumerates Scene / Video / Web / image wallpapers, and
  serves the inventory and media bytes over same-origin HTTP routes
- **Browser half** `lib/client.js` — settings panel, wallpaper layer, liquid
  glass behaviour layer, stylesheets

Wallpaper bytes and inventory routes are served by the sibling package
`dsh-wallpaper-glass-media-bridge`.

## Differences from upstream

Removed wholesale (their npm packages are no longer dependencies):

| Removed | What it was |
|---|---|
| Chat mascot / session avatars | Including the pull-cord dock |
| Click ripple / cursor trail FX | — |
| 3D parallax | — |
| Desktop-pet link | — |
| About tab | Pure info page, no settings |
| Extensions tab | Nothing left once all four source packages were gone |

Settings now has four tabs: **Library / Appearance / Playback / System**.

Several CSS gaps in upstream were also filled (settings page skeleton, spacing
hierarchy, button padding, glass-compatibility block type size). See the root
README.

## Layout

```
lib/
├── index.js            host half entry (Cordis plugin)
├── client.js           browser half bundle (built single file)
├── host-boot.js        host bootstrap and component loading
├── kernel/             host kernel (discovery / config / data dir / uploads …)
├── routes/             the few route families this package still owns
├── media/              media probing and transcode kernel
├── webwallgl/          Scene live-rendering engine
├── styles-*.js         style domains (inlined into client.js at build time)
├── glass-presets/      factory glass presets
└── types/              type declarations
```

## About this artefact

This package is a **build artefact**, not a source tree. `lib/client.js` is about
65 front-end modules inlined into a single file (~1.9 MB) and cannot be read or
edited per module. To change front-end logic, go back to the upstream source
repository.

## License

Code is released under the [MIT License](../../LICENSE), original copyright
**elysia395**. Asset licensing may differ; check upstream's asset notes before
redistributing or using commercially.

Upstream: https://github.com/elysia395/dsh-wallpaper-engine
