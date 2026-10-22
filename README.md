# PlanStar

> 中文文档：[README.zh-CN.md](./README.zh-CN.md)

PlanStar（企划星）is a self-hosted knowledge workspace, built on the Anytype
renderer. It runs in a web browser and as an Electron desktop app, and talks to
a self-hosted [PlanStar API](https://github.com/uifj/PlanStar_server) backend,
which lives in its own repository.

## Screenshots

| Boot — star ignition | Second page — Ink Paper |
| --- | --- |
| ![PlanStar boot badge](docs/screenshots/boot-splash.png) | ![PlanStar paper splash](docs/screenshots/paper-splash.png) |

The start-up runs as two screens: a self-contained boot badge (a four-point
gradient star with an orbiting ring, glow and twinkle), then an "Ink Paper"
splash that unfolds into a document while the account/vault status loads
beneath it.

## Features

- Knowledge workspace client: spaces, objects, blocks, data views, relations
  and a graph view, rendered by the Anytype-derived component layer.
- Runs twice from one codebase: in the browser (`entry.web.tsx` injects an
  `electronMock`) and as a native Electron shell (`entry.tsx`), with the same
  renderer.
- Email/password sign-in through the backend's Better Auth service.
- Realtime collaboration and a publish/sharing surface, backed by the API.
- Desktop capabilities behind feature flags: auto-update (electron-updater + a
  manifest served by the API), crash/feedback intake, and a log uplink.
- Component catalogue and stories in Storybook.

## Tech Stack

| Category | Technology |
| --- | --- |
| Framework | React 18 |
| State | MobX (`mobx`, `mobx-react-lite`) |
| Routing | React Router v5 |
| Editor | TipTap |
| Graph | PixiJS |
| Animation | Framer Motion |
| Drag & drop | `@dnd-kit/*` |
| Virtualization | `react-virtualized` |
| Code highlighting | PrismJS |
| PDF | `react-pdf` |
| Styling | Frozen `legacy.css` + a Tailwind CSS v4 theme/utilities layer (`src/styles/desktop.css`) for new surfaces |
| Desktop | Electron + `electron-builder` + `electron-updater` |
| Build | Vite (`@vitejs/plugin-react-swc`) |
| Component docs | Storybook |
| Test | Vitest |
| Lint / format | Biome |
| Language | TypeScript |

## Backend

This client expects the [PlanStar API](https://github.com/uifj/PlanStar_server)
to run separately. Create `.env` in this directory and point the client at it:

```bash
VITE_API_BASE_URL=http://localhost:8000/api
```

At boot the client resolves its API base in this order: the `?server=` URL
parameter, the address stored by a previous boot, `VITE_API_BASE_URL`, then the
page's own origin plus `/api`. An origin-only value gains the `/api` suffix, so
`?server=http://HOST:PORT` works without writing `/api`.

## Getting Started

Requirements: Node.js ≥ 24, pnpm ≥ 11 (`engines`/`packageManager` in
`package.json`), and the PlanStar API running locally.

```bash
pnpm install

# .env: VITE_API_BASE_URL=http://localhost:8000/api

pnpm dev
```

The web app runs at `http://localhost:3030`.

### Desktop (Electron)

```bash
pnpm dev:electron    # build the renderer + main, then launch Electron
pnpm build:desktop   # web bundle + esbuild main/preload
pnpm start:electron  # run the packaged shell against the built output
pnpm dist:mac        # electron-builder --mac --arm64 (unsigned rehearsal)
```

The desktop shell needs the renderer build present, or the main process exits
with an error. See [docs/desktop/](https://github.com/uifj/PlanStar_server/tree/main/docs/desktop)
for the shell design and
[docs/desktop/release.md](https://github.com/uifj/PlanStar_server/blob/main/docs/desktop/release.md)
for publishing.

## Scripts

| Command | Description |
| --- | --- |
| `pnpm dev` | Start the local Vite dev server (web) |
| `pnpm build` | Build the web bundle |
| `pnpm preview` | Preview the production build |
| `pnpm build:shared` | Build the `desktop-shared` IPC contract package |
| `pnpm build:electron` | Bundle the Electron main + preload with esbuild |
| `pnpm build:desktop` | `build` + `build:electron` |
| `pnpm dev:electron` / `pnpm start:electron` | Run the desktop shell |
| `pnpm dist:mac` | Package the macOS (arm64) desktop build |
| `pnpm storybook` | Component catalogue on :6006 |
| `pnpm typecheck` | TypeScript checks (app + electron configs) |
| `pnpm test` | Vitest suite |
| `pnpm check` / `pnpm check:fix` | Biome lint/format |
| `pnpm lint:gate` | Biome over `electron`, `packages`, and the check scripts |
| `pnpm check:desktop-parity` / `check:desktop-contract` / `check:desktop-feed-contract` | Desktop bridge/feed contracts |
| `pnpm verify:desktop-feed` | Verify the updater feed against the API |

## Architecture

```text
src/
  html/index.html   Desktop boot template (inline boot badge: star ignition)
  ts/
    entry.tsx        Electron renderer entry
    entry.web.tsx    Browser entry (injects electronMock, then mounts App)
    app.tsx          Root component: stores, router, theme, boot lifecycle
    component/       The renderer's component layer (pages, blocks, menus, popups)
    model/ store/    Domain models and MobX stores
    interface/       Shared types/contracts
    lib/             Runtime helpers (desktop bridge, api, utils)
    hook/ workers/   Hooks and web workers (e.g. the graph adapter)
  styles/        legacy.css (frozen) + desktop.css (Tailwind v4, the entry for new surfaces)
  img/           Icons, logo wordmarks (incl. the PlanStar star mark)
  json/          i18n strings and constants
dist/index.web.html   Web boot template (same inline badge; Vite entry)
electron/
  main.ts main/      Main process (window, IPC command bus, updater, logs)
  preload.ts preload/ Preload bridge
  server.ts          Loopback server for the shell
packages/
  desktop-shared/    Cross-process IPC contract shared by main, preload and renderer
```

The renderer is desktop-agnostic: in the browser `entry.web.tsx` installs an
`electronMock` so `window.Electron` and the desktop facade resolve to no-ops,
keeping one codebase for both targets.

The splash screens span both layers: the boot badge is plain DOM inlined in the
HTML templates (`src/html/index.html` for the desktop shell, `dist/index.web.html`
for the web entry) and is torn down by `app.tsx`; the second, Ink Paper screen is
React-rendered into `#root-loader` under `app.tsx`'s `rootSplash` class.

## Quality Checks

```bash
pnpm typecheck
pnpm check
pnpm test
```

## Production Deployment

This directory builds two artifacts — the web bundle served by Caddy, and the
Electron desktop shell. The surrounding stack is documented in the API
repository:

- [deploy/README.md](https://github.com/uifj/PlanStar_server/blob/main/docs/deploy/README.md) — the single-host runbook.
- [deploy/troubleshooting.md](https://github.com/uifj/PlanStar_server/blob/main/docs/deploy/troubleshooting.md) — failure modes.
- [desktop/release.md](https://github.com/uifj/PlanStar_server/blob/main/docs/desktop/release.md) — publishing a desktop release.

The images are the API [Dockerfile](https://github.com/uifj/PlanStar_server/blob/main/Dockerfile)
and this directory's `Dockerfile` (web), orchestrated by
[deploy/podman/](https://github.com/uifj/PlanStar_server/tree/main/deploy/podman).
