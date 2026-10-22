# PlanStar

> 英文文档：[README.md](./README.md)

PlanStar（企划星）是一款基于 Anytype 渲染器构建的自托管知识工作台。它既能在 Web
浏览器中运行，也能作为 Electron 桌面应用安装，后端对接自托管的
[PlanStar API](https://github.com/uifj/PlanStar_server)，后端位于独立仓库。

## 截图

| 启动 —— 聚光成星 | 第二页 —— 纸页 Ink Paper |
| --- | --- |
| ![PlanStar 启动徽标](docs/screenshots/boot-splash.png) | ![PlanStar 纸页启动屏](docs/screenshots/paper-splash.png) |

启动分两屏：先是自包含的启动徽标（四角渐变星，配旋转星轨环、光晕与星芒），随后是
"纸页 Ink Paper" 启动屏 —— 一叠纸页展开成一张文档，账户/空间状态在其下方实时加载。

## 功能

- 知识工作台客户端：空间、对象、块、数据视图、关系与图谱视图，由源自 Anytype 的
  组件层渲染。
- 一套代码、两种运行：浏览器（`entry.web.tsx` 注入 `electronMock`）与原生 Electron
  壳层（`entry.tsx`），共用同一渲染器。
- 经后端 Better Auth 的邮箱/密码登录。
- 实时协作与公开分享，由 API 支撑。
- 以特性开关控制的桌面能力：自动更新（electron-updater + 由 API 提供的 manifest）、
  崩溃/反馈接入、日志上行。
- Storybook 中的组件清单与用例。

## 技术栈

| 类别 | 技术 |
| --- | --- |
| 框架 | React 18 |
| 状态 | MobX（`mobx`、`mobx-react-lite`） |
| 路由 | React Router v5 |
| 编辑器 | TipTap |
| 图谱 | PixiJS |
| 动画 | Framer Motion |
| 拖拽 | `@dnd-kit/*` |
| 虚拟滚动 | `react-virtualized` |
| 代码高亮 | PrismJS |
| PDF | `react-pdf` |
| 样式 | 冻结的 `legacy.css` + 面向新表面的 Tailwind CSS v4 主题/工具类层（`src/styles/desktop.css`） |
| 桌面 | Electron + `electron-builder` + `electron-updater` |
| 构建 | Vite（`@vitejs/plugin-react-swc`） |
| 组件文档 | Storybook |
| 测试 | Vitest |
| 校验 / 格式化 | Biome |
| 语言 | TypeScript |

## 后端

本客户端需另行运行 [PlanStar API](https://github.com/uifj/PlanStar_server)。
在本目录创建 `.env` 并指向后端：

```bash
VITE_API_BASE_URL=http://localhost:8000/api
```

客户端启动时按以下优先级解析 API 基址：`?server=` URL 参数 → 上次启动存储的地址 →
`VITE_API_BASE_URL` → 页面自身 origin 加 `/api`。仅含 origin 的值会自动补上 `/api`，
因此 `?server=http://HOST:PORT` 无需手写 `/api`。

## 快速开始

前置：Node.js ≥ 24、pnpm ≥ 11（见 `package.json` 的 `engines`/`packageManager`），
以及本地运行中的 PlanStar API。

```bash
pnpm install

# .env: VITE_API_BASE_URL=http://localhost:8000/api

pnpm dev
```

Web 应用运行于 `http://localhost:3030`。

### 桌面（Electron）

```bash
pnpm dev:electron    # 构建渲染器 + 主进程，随后启动 Electron
pnpm build:desktop   # Web 产物 + esbuild 打包 main/preload
pnpm start:electron  # 针对构建产物运行桌面壳层
pnpm dist:mac        # electron-builder --mac --arm64（未签名演练）
```

桌面壳层要求渲染器产物已存在，否则主进程会报错退出。壳层设计见
[docs/desktop/](https://github.com/uifj/PlanStar_server/tree/main/docs/desktop)，
发布流程见
[docs/desktop/release.md](https://github.com/uifj/PlanStar_server/blob/main/docs/desktop/release.md)。

## 脚本

| 命令 | 说明 |
| --- | --- |
| `pnpm dev` | 启动本地 Vite 开发服务器（Web） |
| `pnpm build` | 构建 Web 产物 |
| `pnpm preview` | 预览生产构建 |
| `pnpm build:shared` | 构建 `desktop-shared` IPC 契约包 |
| `pnpm build:electron` | 用 esbuild 打包 Electron main + preload |
| `pnpm build:desktop` | `build` + `build:electron` |
| `pnpm dev:electron` / `pnpm start:electron` | 运行桌面壳层 |
| `pnpm dist:mac` | 打包 macOS（arm64）桌面构建 |
| `pnpm storybook` | 在 :6006 启动组件清单 |
| `pnpm typecheck` | TypeScript 检查（app + electron 两套配置） |
| `pnpm test` | Vitest 测试 |
| `pnpm check` / `pnpm check:fix` | Biome 校验/格式化 |
| `pnpm lint:gate` | 对 `electron`、`packages` 与检查脚本运行 Biome |
| `pnpm check:desktop-parity` / `check:desktop-contract` / `check:desktop-feed-contract` | 桌面桥接/发布契约 |
| `pnpm verify:desktop-feed` | 针对 API 校验自动更新 feed |

## 架构

```text
src/
  html/index.html   桌面启动模板（内联启动徽标：聚光成星）
  ts/
    entry.tsx        Electron 渲染进程入口
    entry.web.tsx    浏览器入口（注入 electronMock 后挂载 App）
    app.tsx          根组件：stores、router、主题、启动生命周期
    component/       渲染器组件层（页面、块、菜单、弹窗）
    model/ store/    领域模型与 MobX stores
    interface/       共享类型/契约
    lib/             运行时助手（桌面桥接、api、utils）
    hook/ workers/   Hooks 与 Web Worker（如图谱适配器）
  styles/        legacy.css（冻结）+ desktop.css（Tailwind v4，新表面入口）
  img/           图标、字标（含 PlanStar 星标）
  json/          i18n 文案与常量
dist/index.web.html   Web 启动模板（同一内联徽标；Vite 入口）
electron/
  main.ts main/      主进程（窗口、IPC 命令总线、更新、日志）
  preload.ts preload/ 预加载桥接
  server.ts          壳层的 loopback 服务器
packages/
  desktop-shared/    main、preload、renderer 共用的跨进程 IPC 契约
```

渲染器与桌面无关：在浏览器里 `entry.web.tsx` 安装 `electronMock`，使
`window.Electron` 与桌面 facade 解析为空操作，从而一套代码覆盖两种目标。

两屏启动分属两层：启动徽标是内联在 HTML 模板里的纯 DOM（桌面壳层用
`src/html/index.html`，Web 入口用 `dist/index.web.html`），由 `app.tsx` 负责拆除；
第二屏纸页 Ink Paper 由 React 渲染到 `#root-loader`，并带 `app.tsx` 的 `rootSplash`
类名。

## 质量检查

```bash
pnpm typecheck
pnpm check
pnpm test
```

## 生产部署

本目录构建两种产物 —— 由 Caddy 服务的 Web 包，以及 Electron 桌面壳层。周边部署栈的
文档位于 API 仓库：

- [deploy/README.md](https://github.com/uifj/PlanStar_server/blob/main/docs/deploy/README.md) —— 单机运行手册。
- [deploy/troubleshooting.md](https://github.com/uifj/PlanStar_server/blob/main/docs/deploy/troubleshooting.md) —— 故障排查。
- [desktop/release.md](https://github.com/uifj/PlanStar_server/blob/main/docs/desktop/release.md) —— 发布桌面版本。

镜像为 API 的 [Dockerfile](https://github.com/uifj/PlanStar_server/blob/main/Dockerfile)
与本目录的 `Dockerfile`（Web），由
[deploy/podman/](https://github.com/uifj/PlanStar_server/tree/main/deploy/podman) 编排。
