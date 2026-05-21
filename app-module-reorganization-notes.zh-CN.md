# app/ 模块重组归纳说明

日期：2026-05-11
状态：`app/` 一级模块下二级、三级目录设计已确认，作为未来物理目录迁移前的设计依据

## 目的

本文档按一级模块维度归纳已经讨论确认的 `app/` 目录职责、目标结构、原项目来源、跨模块依赖边界和迁移风险。

它是 `module-structure-reorganization-design.md` 和 `module-structure-reorganization-design.zh-CN.md` 的补充材料。主设计文档记录整体结构；本文档保留 `app/` 子任务的细节，避免后续上下文压缩后丢失已经确认的设计。

## 一级职责边界

`app/` 负责用户可见的产品入口和可运行应用壳层。它承载用户直接打开、浏览、安装或交互的产品表面，但不承载后端服务实现、provider 执行逻辑、daemon 长驻进程、发布流水线或跨模块基础协议。

`app/` 可以依赖 `shared/` 中稳定的协议、运行时契约、数据类型和通用工具；可以通过清晰边界调用 `service/`、`daemon/`、`provider/` 暴露的能力；但不应把这些模块的实现细节直接内聚进应用目录。

## 已确认的二级结构

```text
app/
  client/
  website/
  docs-site/
```

- `client/`：Happier 主客户端应用，覆盖 Expo 跨端共享代码、移动端、Web 端和桌面端壳层。
- `website/`：公开产品官网、安装入口和静态营销/下载页面。
- `docs-site/`：公开文档站点应用，负责发布用户和开发者可访问的产品文档内容。

## `app/client/`

### 当前项目来源

`app/client/` 主要对应当前 `apps/ui`。当前代码是 Expo 跨端共享代码，不是单独的 mobile-only 代码。它同时包含：

- Happier 手机端 App 代码。
- Expo Web 运行入口。
- Tauri 桌面端壳层相关代码。
- 跨端共享的 React Native / Expo UI、状态、同步、会话、provider 展示面和平台适配。
- 当前 Android APK/EAS 构建配置，例如 `apps/ui/eas.json`。

因此未来重组时，`mobile/`、`web/`、`desktop/` 三类平台目录仍然需要存在，但只放平台专属实现。三端共用代码应进入统一的共享客户端结构，而不是复制到三个平台目录里。

### 目标三级结构

```text
app/client/
  entrypoints/
  routes/
  runtime/
  domains/
  ui/
  provider-surfaces/
  platforms/
  native-modules/
  assets/
  devtools/
  tooling/
```

### 职责说明

| 目录 | 职责 | 当前来源倾向 |
| --- | --- | --- |
| `entrypoints/` | 客户端启动入口、Expo 入口加载、启动前 hook。只做薄入口，不承载产品领域逻辑。 | `apps/ui` 根入口、启动文件、Expo entry 相关代码 |
| `routes/` | Expo Router 页面、layout 和路由入口。当前 `apps/ui/sources/app` 属于这里。 | `apps/ui/sources/app/**` |
| `runtime/` | App 启动编排、provider wrappers、通知、tracking、连接状态、同步 runtime wiring。 | `apps/ui/sources/runtime`、启动 wiring、全局 providers |
| `domains/` | 客户端领域能力，例如 sessions、machines、settings、messages、files、auth、voice、source control、artifacts、automations。 | 当前 `sources` 下按业务能力组织的 domain 代码 |
| `ui/` | 可复用 UI 系统，例如 components、navigation、theme、modal、text、hooks、layout。 | `apps/ui/sources/components`、theme、modal、text、navigation 等 |
| `provider-surfaces/` | provider 在客户端的展示面和配置面，例如 provider picker、图标、设置页、UI registry。 | `apps/ui/sources/agents/registry`、`apps/ui/sources/agents/providers/**` 中 UI-facing 子集 |
| `platforms/` | mobile/web/desktop 专属能力和外壳适配。 | Tauri、web-only、native/mobile-only 代码 |
| `native-modules/` | App-local Expo/native extension modules。 | `apps/ui/modules/**` |
| `assets/` | 客户端运行资产。 | `apps/ui/assets/**` |
| `devtools/` | 客户端本地开发和测试辅助，包括 UI testkit。 | `apps/ui/sources/dev/**` |
| `tooling/` | 客户端本地 build、migration、postinstall、i18n、CodeMirror、xterm、Tauri helper。 | `apps/ui/scripts/**`、客户端专属工具脚本 |

### 依赖与边界规则

- `routes/` 可以组合 `domains/`、`ui/`、`runtime/`，但应保持页面入口薄，复杂逻辑下沉到 domain 或 UI 组件。
- `domains/` 可以消费 `shared/` 协议、客户端存储和服务契约，但不直接拥有 server、daemon 或 provider 执行实现。
- `provider-surfaces/` 只负责客户端展示、选择、配置和用户交互，不承载 provider CLI/backend 运行逻辑。
- `platforms/mobile`、`platforms/web`、`platforms/desktop` 只放平台专属能力；跨端共享逻辑不应为了平台目录而拆散。
- APK 交付链路属于 `app/client` 的移动构建面；具体 provider runtime、daemon、本地服务启动仍由相应模块提供。

## `app/website/`

### 当前项目来源

`app/website/` 对应当前 `apps/website`。它是公开产品官网、安装入口和静态页面应用。

当前来源包括：

- `apps/website/package.json`、Vite/Tailwind/PostCSS 配置。
- `index*.html`、`src/main.js`、`src/styles.css`。
- `public/images/**`。
- `public/install*`、`install.sh`、`install.ps1`、`install-dev*`、`install-preview*`、`install-server*`。
- `happier-release.pub`。
- website-local tests 和 README。

### 目标三级结构

```text
app/website/
  pages/
  interactions/
  styles/
  public/
    images/
    install
    install.sh
    install.ps1
    install-dev
    install-dev.sh
    install-dev.ps1
    install-preview
    install-preview.sh
    install-preview.ps1
    install-server
    install-server.sh
    happier-release.pub
  tests/
  tooling/
  package.json
  vite.config.js
  tailwind.config.js
  postcss.config.js
  README.md
```

### 职责说明

- `pages/`：官网页面结构和静态内容入口。
- `interactions/`：官网页面交互脚本，例如安装方式选择、平台检测、复制命令等 browser-side behavior。
- `styles/`：官网样式。
- `public/`：需要稳定公开 URL 的静态资源和安装入口，尤其是 `/install*` 链接。
- `tests/`：website-local 测试。
- `tooling/`：website-local build 或静态资源辅助脚本。

### 依赖与边界规则

- `public/install*` 的 URL 兼容性是迁移风险点，物理移动时必须保留最终发布路径。
- website 可以展示安装和下载入口，但安装脚本生成、发布物同步、release pipeline 应归 `ops/release` 或相应发布模块。
- website 不承载客户端主 App 运行逻辑，也不承载 docs site 内容体系。

## `app/docs-site/`

### 当前项目来源

`app/docs-site/` 对应当前 `apps/docs`。它是公开文档站点应用，基于 Next.js 和 Fumadocs。

当前来源包括：

- `apps/docs/content/docs/**`。
- `apps/docs/src/app/**`，包含 search、LLM text、OG image、health 等路由。
- `apps/docs/src/components/**`、`src/lib/**`、`src/mdx-components.tsx`。
- `.source/**` 或 Fumadocs 生成内容。
- Next/Fumadocs/PostCSS/TypeScript 配置、tests、README。

### 目标三级结构

```text
app/docs-site/
  content/
    docs/
  src/
    app/
    components/
    lib/
    mdx-components.tsx
  generated/
  tests/
  tooling/
  package.json
  next.config.mjs
  source.config.ts
  postcss.config.mjs
  tsconfig.json
  README.md
```

### 职责说明

- `content/docs/`：发布到公开文档站点的文档内容。
- `src/app/`：Next App Router 入口。由于 Next 约定要求，物理路径可以继续保留 `src/app`。
- `src/components/`：docs site 页面组件。
- `src/lib/`：docs site 本地辅助逻辑。
- `generated/`：概念上承接 Fumadocs 生成内容；如果工具要求 `.source`，可以保持原路径并明确标记为生成物。
- `tests/`：docs-site-local 测试。
- `tooling/`：docs site 本地构建或内容生成辅助。

### 依赖与边界规则

- `app/docs-site` 只负责可发布的文档站点应用。
- 仓库内部重组设计文档、开发过程设计说明、agent 指导文档默认不进入 `app/docs-site`，除非明确决定要公开发布。
- docs site 可以引用产品文档内容和共享类型生成结果，但不应依赖 CLI、daemon、provider 或 service 的内部实现。

## 跨模块依赖方向

推荐依赖方向：

```text
app/client -> shared
app/client -> service public API contracts
app/client -> daemon public control surface
app/client -> provider UI metadata/contracts

app/website -> ops-produced public artifacts
app/website -> shared release metadata only when needed

app/docs-site -> docs content
app/docs-site -> shared generated docs inputs only when explicitly published
```

不推荐方向：

- `service/` 反向依赖 `app/`。
- `provider/` 反向依赖 `app/client` UI 实现。
- `daemon/` 反向依赖具体客户端页面或组件。
- `app/website` 直接拥有 release 生成和发布流水线。

## 后续迁移风险

- `apps/ui` 是 Expo 跨端共享应用，不能按 mobile/web/desktop 三份代码简单切开。
- `apps/ui/sources/app` 受 Expo Router 约束；如果改名或移动，需要同步路由、别名和构建配置。
- Android APK 打包链路在当前 `apps/ui` 内，未来应落在 `app/client` 的 mobile/platform/build profile 下。
- `apps/website/public/install*` 具有公开 URL 兼容性，迁移时必须先设计 URL 保持策略。
- `apps/docs/.source` 可能受 Fumadocs 工具约束，不能只按命名喜好重命名。
