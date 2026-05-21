# Happier 模块结构重组设计

日期：2026-05-10
状态：一级模块、CLI 二级/三级结构、provider 二级结构、provider/catalog 三级结构、provider/cross-provider 三级结构、provider/managed-tools 三级结构、provider/provider-families 三级结构、provider-id 三级原则、shared/protocol 三级结构、shared/agent-domain 三级结构、shared/mcp-domain 三级结构、shared/agent-runtime 三级结构、shared/transfers 三级结构、shared/connection-supervisor 三级结构、shared/release-runtime 三级结构已复核确认；尚未授权物理迁移

## 目的

本文档记录 Happier 仓库未来进行真实物理目录重组时，已经确认的一级模块结构。当前内容刻意限制在设计约束和所有权规则上，不代表已经授权移动文件。

目标是让开发者更容易理解仓库结构，同时降低重组后出现 import 断裂、依赖倒置、职责混杂的风险。

## 当前结构基线

当前仓库主要围绕以下结构组织：

- `apps/`：可运行应用和编排型 package，例如 UI、CLI、server、stack scripts、docs site、website、bootstrap。
- `packages/`：共享库，例如 protocol、agent runtime/domain 逻辑、transfers、release runtime、connection supervision、native modules、relay server、测试套件。
- 根目录支撑目录：配置、CI、Docker、Dagger、脚本、文档、工具专属目录、agent 指导文件。

当前布局可以工作，但顶层混合了多个不同概念：产品界面、共享运行时 package、provider 适配器、开发者文档、AI-agent 指导文档、仓库运维能力。

## 已确认的一级模块

目标一级结构为：

```text
app/
service/
cli/
daemon/
provider/
agents/
shared/
tests/
ops/
docs/
```

这些名称表达的是所有权和依赖方向，而不只是文件类型。

## 一级模块拆分子任务文档

目录重组的详细讨论按一级模块拆分记录，避免上下文压缩后丢失已经确认的设计：

- `app/`：`app-module-reorganization-notes.zh-CN.md`
- `service/`：`service-module-reorganization-notes.zh-CN.md`
- `cli/`：`cli-boundary-reorganization-design-notes.zh-CN.md`
- `provider/`：`provider-module-reorganization-notes.zh-CN.md`
- `shared/`：`shared-module-reorganization-notes.md`、`shared-module-reorganization-notes.zh-CN.md`
- 子任务总索引：`module-reorganization-subtask-index.zh-CN.md`

## 模块职责

### `app/`

负责用户可见的产品入口。

预期范围：

- 移动端、桌面端、Web 应用 UI。
- 当 website 和 docs site 以应用形式交付时，它们的运行时。
- provider 能力在 UI 中的组合和展示，但不包含 provider 执行逻辑。
- 产品级交互流程、页面、导航和展示。

当前可能来源：

- `apps/ui`
- `apps/website`
- `apps/docs`

已确认的二级目录：

```text
app/
  client/
  website/
  docs-site/
```

`app/client/` 负责主产品跨端客户端。它对应当前 `apps/ui` workspace，并包含当前从该 workspace 构建出来的手机 App、Web App 和 Tauri 桌面壳。

`app/client/` 已确认的三级目录：

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

`entrypoints/` 负责很薄的客户端启动入口，包括 Expo 入口装载和 Router 装载前的安装钩子。

`routes/` 负责 Expo Router 页面、layout 和 route files。它对应当前 `apps/ui/sources/app` router root。如果未来真实物理目录改名，必须在同一次修改中同步更新 app-local 配置里的 Expo Router root。

`runtime/` 负责应用运行时编排，包括 boot、provider wrappers、notifications、tracking、connectivity、sync runtime wiring。

`domains/` 负责客户端业务域，例如 sessions、machines、settings、messages、files、auth、voice、source control、artifacts、automations。

`ui/` 负责可复用 UI 系统，例如 components、navigation、theme、modal、text、hooks、layout。轻量的平台同位实现文件如果表达的是同一个 UI 抽象，可以继续和该 UI 抽象放在一起。

`provider-surfaces/` 负责 provider 在客户端里的 UI 表达，包括 provider pickers、icons、settings plugins、UI registries、presentation behavior。它不负责 provider runtime execution、CLI backends 或 provider protocol truth sources。

`platforms/` 负责比同位组件变体更重的平台专属能力：

```text
platforms/
  mobile/
  web/
  desktop/
```

`native-modules/` 负责 app-local Expo/native extension modules。

`assets/` 负责客户端运行时资源。

`devtools/` 负责 client-local 开发和测试辅助，包括 UI testkits、dev-only utilities、debugging helpers。

`tooling/` 负责 client-local 构建、迁移、postinstall、i18n、CodeMirror、xterm、Tauri 辅助脚本。通用仓库运维仍归 `ops/`。

`app/website/` 负责公开官网/营销站。它对应当前 `apps/website` workspace。

`app/website/` 是 Happier 的公开静态网站。它负责营销首页、release/prerelease 首页变体、官网交互、公开品牌资源，以及由网站托管的公开安装入口。

当前来源：

- `apps/website/package.json`
- `apps/website/vite.config.js`
- `apps/website/tailwind.config.js`
- `apps/website/postcss.config.js`
- `apps/website/index.html`
- `apps/website/index.prerelease.html`
- `apps/website/index.release.html`
- `apps/website/src/main.js`
- `apps/website/src/styles.css`
- `apps/website/public/images/**`
- `apps/website/public/install*`
- `apps/website/public/happier-release.pub`
- `apps/website/tests/**`
- `apps/website/README.md`

已确认的目标结构：

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

`pages/` 负责静态网站入口页面和首页变体。它对应当前 `index.html`、`index.prerelease.html`、`index.release.html`。

`interactions/` 负责官网专属浏览器行为，例如主题切换、移动端菜单、复制安装命令按钮、toast 通知、平滑滚动、渐进 reveal、feature tabs。它对应当前 `src/main.js`。

`styles/` 负责官网专属样式，例如 Tailwind input CSS 和自定义官网 CSS。它对应当前 `src/styles.css`，以及 website-local Tailwind/PostCSS 配置。

`public/images/` 负责公开官网图片、logo、badge、favicon、官网截图。它们不是 `app/client/` 的客户端运行时资源。

当前提交在 `public/` 下的 `install*` 文件和 `happier-release.pub` 是由网站发布的安装器产物。它们由 website 托管，但生成和同步职责属于 release/operations tooling。除非 release 和 installer 兼容性方案明确保证 `/install`、`/install.sh`、`/install.ps1` 等现有公开端点不变，否则不要把这些文件移动到会改变公开 URL 的位置。

`tests/` 负责 website-local 测试，目前包括 release homepage contract checks。

`tooling/` 预留给 website-local 工具，例如未来首页变体选择 helper。通用 release、deploy、installer-generation 逻辑属于 `ops/`，不属于 `app/website/`。

允许依赖：

- Vite、Tailwind、PostCSS、浏览器 API。
- 公开静态资源。
- 由 `ops/` 复制或同步过来的 release-published installer artifacts。

禁止依赖：

- 不 import `app/client/`。
- 不 import 具体 `service/`、`cli/` 或 `daemon/` 实现。
- 不拥有 installer generation logic。
- 不拥有 docs-site content/runtime。
- 不作为 provider runtime 或 provider protocol 的 source of truth。

`app/docs-site/` 负责文档站运行时。它对应当前 `apps/docs` workspace。人类编写的文档如果不绑定 docs-site 应用运行时，仍然可以属于顶层 `docs/` 模块。

`app/docs-site/` 是 Happier 的公开文档站应用。它负责把 MDX 文档内容通过 Next/Fumadocs 运行时发布成文档站，包括搜索路由、LLM 文本路由、Open Graph 图片路由、健康检查、布局配置，以及文档站专属 UI 组件。

当前来源：

- `apps/docs/package.json`
- `apps/docs/next.config.mjs`
- `apps/docs/source.config.ts`
- `apps/docs/tsconfig.json`
- `apps/docs/postcss.config.mjs`
- `apps/docs/content/docs/**`
- `apps/docs/src/app/**`
- `apps/docs/src/components/**`
- `apps/docs/src/lib/**`
- `apps/docs/src/mdx-components.tsx`
- `apps/docs/README.md`

已确认的目标结构：

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

`content/docs/` 负责公开发布的文档内容。它对应当前 `apps/docs/content/docs/**`，包括应由文档站发布的产品文档、开发者文档、provider 文档、protocol 文档、部署文档、安全文档、发布文档和法律文档。

仓库内部设计记录、临时迁移方案、开发过程工作笔记应继续归顶层 `docs/` 模块，除非明确决定把它们发布到文档站。

`src/app/` 负责 Next App Router 路由树。它对应当前 `apps/docs/src/app/**`，包括 catch-all 文档页面、`/api/search`、`/health`、`/llms.txt`、`/llms-full.txt`、`/llms.mdx` 和 Open Graph 图片路由。由于这是 Next App Router 的框架约定，物理目录应保持 `src/app/`，除非同一次迁移中同步修改并验证框架配置和构建行为。

`src/components/` 负责文档站专属 React 组件，例如复制或打开 LLM-ready 文档内容的 page actions。这些组件可以使用文档站页面上下文，但不是共享应用组件系统。

`src/lib/` 负责文档站支撑代码，例如 Fumadocs source loader、共享 layout options、URL helpers 和小型工具函数。

`src/mdx-components.tsx` 负责文档站的 MDX 组件注册。

`generated/` 是 Fumadocs 生成产物的概念归属，例如当前 `.source`。如果 Fumadocs 要求物理路径必须是 `.source/`，则保留该路径，并在设计中明确它是生成物，而不是人工维护源码。

`tests/` 预留给 docs-site-local 测试。如果现有 CI 或 release contract 验证的是部署/发布行为，而不是 docs-site 内部行为，则仍可以归更大的 `tests/` 或 `ops/` 模块。

`tooling/` 预留给 docs-site-local 工具，例如未来链接检查、内容索引生成、MDX 迁移 helper。通用 deploy、release 和 pipeline orchestration 属于 `ops/`。

允许依赖：

- Next、React、Fumadocs、MDX、Tailwind/PostCSS，以及 docs-site browser/server API。
- `content/docs/` 下的公开文档内容。
- 必要时依赖 `shared/` 中稳定的品牌、资源或类型契约。
- `ops/` 可以构建或部署 docs site。

禁止依赖：

- 不拥有通用仓库设计记录，除非这些记录明确要作为公开文档发布。
- 不 import 具体 `app/client/` 实现。
- 不 import 具体 `service/`、`cli/` 或 `daemon/` 实现。
- 不作为 provider runtime 或 provider protocol 的 source of truth。
- 不拥有 release/deploy orchestration logic，该职责属于 `ops/`。

明确非目标：

- `apps/bootstrap` 默认不属于 `app/`。它暴露 `hsetup` 命令，并包含 system/bootstrap tasks，因此后续应在 `cli/`、`daemon/` 或 `ops/` 下再分类。

允许依赖：

- `shared/`
- 来自 `provider/` 的 provider capability 和 UI adapter surface
- 来自 `service/` 的 service API contract

禁止依赖：

- `app/` 不能被 `shared/`、`provider/`、`cli/` 或 `service/` 反向依赖。
- `app/` 不能拥有 backend/provider 执行策略。

### `service/`

负责后端服务行为。

预期范围：

- HTTP/WebSocket API。
- 数据库访问和迁移。
- 认证和授权。
- 对象存储、远程协作、relay 类服务、服务端集成。
- 服务端 jobs 和定时工作流。

当前可能来源：

- `apps/server`
- `packages/relay-server`

已确认的二级目录：

```text
service/
  api-server/
  server-runner/
```

`service/api-server/` 负责后端服务实现。它对应当前 `apps/server` workspace。在真实物理目录重组时，它应继续作为一个完整的服务 workspace 处理，避免过早把当前服务实现横向拆碎。

`api-server/` 预期范围：

- 服务端入口和启动编排。
- HTTP API 和 WebSocket/Socket.IO 实时 API。
- 认证、授权、pairing、OAuth、服务端身份相关行为。
- 服务端业务域，例如 sessions、account state、sharing、presence、automations、artifacts、activity、feed、key-value state、social features、retention、server-side feature flags。
- 数据库、blob/file storage、Redis、queues、locks、cache、sequence、private-file storage adapters。
- 服务端 workers、定时任务、retention processing、presence workers、metrics、observability、diagnostics。
- full/light server flavors，包括 SQLite/PGlite/local-file 行为。
- Prisma schema、不同数据库 provider 的 schema 变体、migrations、generated Prisma clients。
- server-local testkit、Vitest 配置、integration/db-contract 测试配置，以及 server-local 开发或迁移脚本。
- 描述该服务自身的 server-local deployment descriptors。跨模块 release/deploy orchestration 仍归 `ops/`。

当前 `api-server/` 来源：

- `apps/server/sources/**`
- `apps/server/prisma/**`
- `apps/server/generated/**`
- `apps/server/scripts/**`
- `apps/server/deploy/**`
- `apps/server/.env.example`
- `apps/server/.env.dev`
- `apps/server/vitest.config.ts`
- `apps/server/vitest.integration.config.ts`
- `apps/server/vitest.dbcontract.config.ts`
- `apps/server/package.json`
- `apps/server/tsconfig.json`
- `apps/server/README.md`

`service/api-server/` 已确认的三级目录：

```text
service/api-server/
  entrypoints/
  runtime/
  http/
  realtime/
  domains/
  auth/
  storage/
  workers/
  integrations/
  observability/
  config/
  flavors/
  database/
  generated/
  testkit/
  tooling/
  deployment/
```

`entrypoints/` 负责服务进程入口，例如 full/light server main。它只设置最外层运行模式，并把控制权交给 `runtime/`，不承载业务逻辑。

`runtime/` 负责启动、关闭、角色选择和服务生命周期装配。它可以把 HTTP、realtime、workers、storage、observability、config、flavors 连接起来，但不拥有业务域规则。

`http/` 负责 Fastify HTTP routes、route registration、request/response 边界、HTTP validation、rate limits、HTTP auth guards。业务规则应归 `domains/` 或 `auth/`。

`realtime/` 负责 Socket.IO、rooms、event routing、socket auth policy 和实时传输行为。Presence 业务状态归 `domains/presence`，实时投递归这里。

`domains/` 负责服务端业务域，例如 sessions、account state、sharing、presence、automations、artifacts、activity、feed、key-value state、social features、feature flags、retention、pets、changes。

`auth/` 负责 authentication、authorization、pairing、terminal auth、OAuth、keyless auth、account auth、identity policy，以及其他服务端身份规则。

`storage/` 负责运行时存储适配，例如 DB client wrappers、blob/file storage、Redis、queues、locks、cache、sequence、private files。它不能依赖 HTTP route 细节。

`workers/` 负责后台 workers、长驻循环、定时服务端工作，以及 `SERVER_ROLE=worker` 进程行为。它负责调度和生命周期；具体业务算法应留在对应 `domains/` 区域。

`integrations/` 负责服务端外部集成，例如 GitHub webhooks、Tailscale URL inference、ElevenLabs/voice service integration，以及其他服务端外部系统。Provider runtime 默认不属于这里，除非该集成明确是 server-side provider hook。

`observability/` 负责 metrics、Sentry、diagnostics、logging 支撑。

`config/` 负责 server env/config parsing、backend selection，以及供 `runtime/` 消费的 resolved config objects。它不直接执行业务行为。

`flavors/` 负责 server runtime flavors，例如 full/light，包括 SQLite、PGlite、local-file defaults 和 flavor-specific setup behavior。

`database/` 负责 Prisma schema、不同数据库 provider 的 schema variants、migrations、schema synchronization contracts。它是服务端持久化契约，不替代 `shared/protocol`。

`generated/` 负责生成代码，例如 Prisma generated clients。它是生成产物，不是人工维护源码。

`testkit/` 负责 server-local 测试 helpers 和 harnesses。它不能成为生产 runtime 依赖。

`tooling/` 负责 server-local 开发、迁移、schema sync、generated client、runtime build、validation scripts。通用 release 和 pipeline orchestration 属于 `ops/`。

`deployment/` 负责描述该服务自身的 deployment descriptors。跨模块 deployment 或 release orchestration 属于 `ops/`。

`api-server/` 内部依赖方向：

```text
entrypoints -> runtime
runtime -> http / realtime / workers / storage / config / flavors / observability
http -> auth / domains / storage / realtime
realtime -> auth / domains
workers -> domains / storage / observability
domains -> storage / shared / provider server hooks
auth -> storage / shared
storage -> database / generated / shared
```

内部硬性规则：

- `storage/` 不能依赖 `http/` route 细节。
- `domains/` 不能依赖 HTTP 或 Socket.IO 传输细节。
- `testkit/` 不能成为生产依赖。
- `tooling/` 和 `deployment/` 不能拥有 release 或 pipeline orchestration。
- Provider-specific execution policy 属于 `provider/`，除非它是通过 provider-owned contract 消费的明确 server-side provider hook。

`service/server-runner/` 负责已发布 server runner / distribution wrapper。它对应当前 `packages/relay-server` workspace。它不是后端服务业务实现，而是负责为目标平台解析、下载、校验并启动合适的 Happier server binary。

`server-runner/` 预期范围：

- `happier-server`、`relay-server` 这类 runner CLI 入口。
- 目标平台和 release asset 解析。
- checksum 和 minisign 校验。
- 通过 shared release runtime primitives 执行 verified download 和 extraction orchestration。
- runner assets、runner-local tests、runner packaging helpers。

当前 `server-runner/` 来源：

- `packages/relay-server/bin/**`
- `packages/relay-server/src/**`
- `packages/relay-server/assets/**`
- `packages/relay-server/scripts/**`
- `packages/relay-server/package.json`

它在整个项目中的作用：

`server-runner/` 负责把已发布的 release artifacts 连接到用户本地运行环境。它是 Happier Server 的轻量启动器，不是后端服务本体。

运行时它大致执行：

1. 解析 runner 调用参数。
2. 判断当前操作系统和 CPU 架构。
3. 找到匹配的 Happier server release artifact。
4. 下载并通过 checksum 和 minisign signature 校验 artifact。
5. 解压 artifact 到本地缓存。
6. 可选下载 UI web bundle。
7. 启动真正的 Happier server binary。

它与其他模块的关系：

```text
ops/ 构建并发布 server release artifacts
shared/release-runtime/ 提供可复用的下载、校验、解压原语
service/server-runner/ 消费这些原语以获取并启动 server binary
service/api-server/ 是 runner 启动的后端服务本体
daemon/ 或 stack/self-host tooling 可以调用 runner 管理本地服务
app/client/ 连接正在运行的 api-server
```

`service/server-runner/` 已确认的三级目录：

```text
service/server-runner/
  entrypoints/
  runtime/
  invocation/
  targets/
  release-assets/
  verification/
  assets/
  tooling/
  package.json
```

`entrypoints/` 负责 runner 命令入口，例如 `happier-server` 和 `relay-server`。它接收 CLI 调用，并把控制权交给 `runtime/`。

`runtime/` 负责 runner 主执行流程：解析调用、解析目标平台、解析 release artifacts、下载、校验、解压、缓存，并 spawn 真正的 server binary。它是启动服务的编排层，不是服务端业务逻辑。

`invocation/` 负责命令参数解析和调用配置，包括 channel、server tag、UI tag、是否携带 UI bundle，以及透传给 server binary 的 positional arguments。

`targets/` 负责本机目标解析：操作系统、CPU 架构、server 可执行文件名、runner cache root。Windows、macOS、Linux、x64、arm64 的兼容性规则属于这里。

`release-assets/` 负责从 GitHub release metadata 中选择 release artifact，包括 server binary artifact 和可选 UI web artifact。它消费 release-runtime 的 asset helpers，而不是复制通用 artifact resolution 行为。

`verification/` 负责 runner-facing 校验边界，包括 checksum 和 minisign signature。通用 checksum、minisign、download、extraction 实现仍归 `shared/release-runtime`。

`assets/` 负责随 runner package 分发的静态资源，例如用于校验的 Happier release public key。

`tooling/` 负责 runner-local packaging 和 prepack helpers，例如为已发布 runner package bundling workspace dependencies。通用 release、deploy、pipeline orchestration 属于 `ops/`。

这个小型 runner 的测试可以继续与被测模块同位，不必现在单独建立 `tests/` 目录。如果后续 runner 出现共享测试辅助面，再引入 `testkit/`。

`server-runner/` 内部依赖方向：

```text
entrypoints -> runtime
runtime -> invocation / targets / release-assets / verification / assets
release-assets -> shared/release-runtime
verification -> shared/release-runtime
tooling -> shared/release-runtime / workspace bundling helpers
```

内部硬性规则：

- `server-runner/` 不能 import `service/api-server/` 源码。
- `server-runner/` 不能拥有 server 业务逻辑。
- 通用 release download、checksum、minisign、extraction 行为属于 `shared/release-runtime`。
- `server-runner/` 可以消费 `shared/release-runtime`，但 `shared/` 不能反向依赖 `server-runner/`。
- `tooling/` 不能拥有跨模块 release 或 deploy orchestration。

明确非目标：

- `packages/protocol` 归 `shared/protocol`；`service/` 实现 protocol contracts，但不拥有它们。
- `packages/transfers` 归 `shared/transfers`。
- `packages/connection-supervisor` 归 `shared/connection-supervisor`。
- `packages/release-runtime` 归 `shared/release-runtime`；server runner 消费它。
- `apps/stack` 中的 service lifecycle commands、self-host orchestration、本地服务管理脚本应归 `daemon/` 或 `ops/`，不归 `service/`，除非某个脚本明确是 server-local implementation tooling。
- `apps/bootstrap` 不是 service module。它后续应按 setup/bootstrap 命令所有权再分类。

允许依赖：

- `shared/`
- 必要时依赖 provider contract 或服务端 provider hook。

禁止依赖：

- 不依赖 UI 实现。
- 不依赖 CLI 命令实现。
- 不依赖 daemon lifecycle implementation。
- 不依赖 `ops/` 内部实现。
- 不把 `tests/` 作为生产运行时输入。
- 不拥有 provider-specific executable behavior，除非该行为明确属于服务端，并且位于 provider-owned interface 后面。

### `cli/`

负责命令行用户入口和命令行运行支撑。它把用户输入的命令转换为对 `daemon/`、`provider/`、`service/`、`shared/` 契约的调用，但不应长期拥有这些模块的核心实现。

预期范围：

- `happier`、`happier-dev`、`happier-mcp`、`hsetup`、`hstack` 命令面。
- 命令解析和命令路由。
- CLI 本地环境处理。
- 面向用户的本地命令，例如 sessions、providers、setup、diagnostics、development workflows。
- 只对命令执行有意义的 CLI 公共工具。

已确认的二级结构：

```text
cli/
  happier-cli/
  setup-cli/
  stack-cli/
  command-runtime/
```

二级目录职责：

- `happier-cli/`：Happier 主 CLI 产品面。负责 `happier`、`happier-dev`、`happier-mcp` 命令入口、command registry、help、参数解析、JSON/TTY 输出和命令层编排。它可以暴露 daemon、provider、session、server、service、MCP、auth、diagnostics 等命令，但底层 daemon/provider/service 实现应归对应 owner。
- `setup-cli/`：setup/bootstrap CLI 面，主要对应当前 `apps/bootstrap` 和 `hsetup` 入口。负责 setup 命令流程和 system tasks 的交互式执行入口。共享 task schema 与可复用 system-task 原语应归 `shared/`。
- `stack-cli/`：`hstack` 命令 facade 和 stack-local 命令体验，主要对应当前 `apps/stack` 中面向用户的命令入口层。负责 stack 命令解析、help 和 delegation。build、release、mobile、daemon lifecycle、service lifecycle、self-host orchestration 等能力如果不是纯命令 facade，应拆到各自 owner。
- `command-runtime/`：多个 CLI 产品面共享的 CLI-only 命令执行支撑。包括 command context、输出渲染、TTY/JSON helper、prompt helper、CLI-only path/env helper、bin wrapper helper、命令错误格式化、只对 CLI 命令执行有意义的 network/proxy helper、平台 console hardening。该命名刻意窄于 generic runtime，不应吸收 first-party runtime、managed tools、daemon runtime 或 provider runtime。

截至 2026-05-14 的 CLI 总体复核结论：

- 当前已确认的 `cli/` 结构成立。基于当前仓库结构，不需要新增第五个 CLI 二级目录。
- `cli/` 是连接层，不是 daemon、provider、service、app、shared runtime 或 ops pipeline 的实现 owner。它的长期职责是 command entrypoint、命令体验、命令层编排，以及 CLI-only command runtime 支撑。
- `command-runtime/` 必须保持窄边界。它不能演变成 generic shared utils、first-party runtime owner、managed tools owner、daemon runtime owner 或 provider runtime owner。
- 依赖闸口是目录设计的一部分，而不是实现细节：`happier-cli/module-adapters/`、`setup-cli/task-adapters/`、`stack-cli/*-commands/` 必须调用 owner public contracts，不能直接 import daemon、service、provider、app 或 ops 深层实现。
- 当前 `apps/cli/src` 下的 `backends/`、`daemon/`、`agent/`、`api/`、`scm/`、`workspaces/`、`transfers/`、`pets/`、`promptAssets/`、`promptRegistries/`、`capabilities/`、`cloud/`、`installables/` 等源码族，物理迁移前必须逐项 inventory 并映射到 owner module。不能因为它们当前位于 CLI package 中，就整体搬进目标 `cli/`。
- 真实物理迁移还必须单独保护 package 边界：`bin`、`exports`、`files`、bundled workspace dependencies、prepack/bundling scripts、生成的 `dist` 或 `package-dist` 产物，以及对应 test/typecheck lanes。

`cli/setup-cli/` 已确认的三级目录：

```text
cli/setup-cli/
  bin-entrypoints/
  command-router/
  system-task-runner/
  interactive-io/
  task-registry/
  setup-workflows/
  task-adapters/
  remote-bootstrap/
  secure-access/
  packaging/
  help/
  testkit/
```

`setup-cli/` 是 `hsetup` 命令产品面。它负责 setup 用户流程、system task 的 CLI 执行入口、交互 IO、hsetup task registry 组装，以及面向 setup 的 adapter。它不能变成 daemon、service、provider、first-party runtime、release 或 installer implementation 的 owner。

`bin-entrypoints/` 负责 `hsetup` 可执行入口包装和启动 fallback。`command-router/` 负责 hsetup 命令解析和分发，包括 `hsetup system-tasks run [--spec-json <json>]`。`system-task-runner/` 负责 system task 执行的 CLI 壳：stdin 或 flag 输入、`SystemTaskSpec` 解析、task id 生成、取消信号、registry 执行、JSONL event/result 输出和 exit code 映射。`interactive-io/` 负责 prompt IO、abortable readline、敏感字段 redaction 后的 event 输出和交互答案解析。`task-registry/` 负责 hsetup task-kind 注册和依赖组装。`setup-workflows/` 负责 setup 产品旅程，例如 `setup.thisComputer.v1`。`task-adapters/` 负责通过 public command 或模块 contract 薄适配 auth、server selection、daemon service install/start/status、relay configuration 等能力。`remote-bootstrap/` 负责 remote SSH bootstrap 的命令面编排和 host-trust 交互。`secure-access/` 负责 setup-facing secure access 流程，例如 Tailscale setup task。`packaging/` 负责 hsetup-local binary build 和 dist wrapper 生成。`help/` 负责 hsetup usage/help。`testkit/` 负责 setup-cli 专属 fake IO、registry fixture 和 task-runner 测试 helper。

允许依赖方向：

```text
cli/setup-cli/
  -> cli/command-runtime/
  -> shared/system-tasks
  -> shared/first-party-runtime
  -> daemon public service/auth/server contracts
  -> service/server-runner public relay/runtime contracts
  -> provider public install/runtime hooks
```

禁止依赖方向：

```text
daemon/ service/ provider/ shared/
  -> cli/setup-cli/

cli/setup-cli/
  -> daemon/service/provider deep implementation
  -> ops/release or installer pipeline implementation
```

当前代码迁移映射：

```text
apps/bootstrap/bin/hsetup.mjs
apps/bootstrap/src/bin/writeDistExecutableWrapper.ts
  -> cli/setup-cli/bin-entrypoints/

apps/bootstrap/src/bin/hsetup.ts
  -> cli/setup-cli/command-router/
  -> cli/setup-cli/system-task-runner/
  -> cli/setup-cli/interactive-io/

apps/bootstrap/src/systemTasks/registry.ts
  -> cli/setup-cli/task-registry/

apps/bootstrap/src/systemTasks/kinds/setupThisComputer.ts
  -> cli/setup-cli/setup-workflows/

apps/bootstrap/src/systemTasks/localDaemonCli.ts
apps/bootstrap/src/systemTasks/happierCli.ts
apps/bootstrap/src/systemTasks/localFirstPartyCommand.ts
  -> cli/setup-cli/task-adapters/

apps/bootstrap/src/systemTasks/remoteSshBootstrapTasks.ts
apps/bootstrap/src/ssh/**
  -> cli/setup-cli/remote-bootstrap/

apps/bootstrap/src/systemTasks/kinds/secureAccessTailscale.ts
apps/bootstrap/src/integrations/tailscale/**
  -> cli/setup-cli/secure-access/

apps/bootstrap/scripts/**
  -> cli/setup-cli/packaging/ 或 release-wide 时归 ops/packaging

packages/protocol/src/systemTasks/**
packages/cli-common/src/systemTasks/**
  -> shared/system-tasks

packages/cli-common/src/firstPartyRuntime/**
  -> shared/first-party-runtime
```

`cli/stack-cli/` 已确认的三级目录：

```text
cli/stack-cli/
  bin-entrypoints/
  command-router/
  global-options/
  stack-context/
  stack-management/
  stack-runtime-commands/
  worktree-commands/
  component-commands/
  auth-context/
  host-service-commands/
  remote-setup-commands/
  provider-commands/
  maintainer-commands/
  integrations/
  happier-passthrough/
  help/
  testkit/
```

`stack-cli/` 是 `hstack` 命令产品面。它负责 stack-local 命令 facade、命令路由、stack context 解析、stack 用户工作流、help，以及面向 owner module 的命令层 delegation。它不能变成 app build、mobile packaging、daemon lifecycle、service runtime、provider install/runtime policy、release pipeline、remote execution primitive 或 self-host implementation 的 owner。

命令 facade 目录统一使用 `*-commands/` 后缀，避免目录名称暗示其拥有底层实现。`stack-runtime-commands/` 负责 `start`、`dev`、`stop`、`logs`、`tui` 命令面，不负责 server、daemon、Expo 或 UI runtime。`worktree-commands/` 负责 `hstack wt ...` 命令体验，不负责 source-control domain。`component-commands/` 负责 build、lint、typecheck、test、mobile、dev-client、EAS 命令 facade；真正 app/mobile/build 实现应归 `app/client/` 或 `ops/`。`host-service-commands/` 负责 daemon、service、Tailscale、self、self-host 命令 facade；daemon、service、relay、secure-access 逻辑归各自 owner。`remote-setup-commands/` 负责 remote setup 命令面；可复用 SSH 和 remote install 原语归 `shared/` 或 `ops/`。`provider-commands/` 负责 `hstack providers ...` 命令面；provider resolution 和 install policy 归 `provider/`。`maintainer-commands/` 负责 setup-from-source、contrib、PR review/setup、monorepo、import、migrate、CI、pack 等维护者工作流命令 facade；持久 release、packaging、review sandbox、migration 实现归 `ops/` 或 shared tooling。`happier-passthrough/` 负责 `hstack happier <args...>` 和 repo-local Happier CLI 包装。

允许依赖方向：

```text
cli/stack-cli/*-commands/
  -> cli/stack-cli/stack-context
  -> cli/command-runtime
  -> owner public contracts

cli/stack-cli/
  -> cli/command-runtime/
  -> shared/
  -> app/client public dev/build contracts
  -> service public runner/server contracts
  -> daemon public control contracts
  -> provider public install/runtime hooks
  -> ops public workflow entrypoints
```

禁止依赖方向：

```text
daemon/ service/ app/ provider/ ops/ shared/
  -> cli/stack-cli/

cli/stack-cli/
  -> daemon/service/app/provider/ops deep implementation

cli/stack-cli/*-commands/
  -> app/client internal dev scripts
  -> daemon internal service manager
  -> service internal server runner
  -> provider internal install/runtime policy
  -> ops internal release pipeline
```

当前代码迁移映射：

```text
apps/stack/bin/hstack.mjs
apps/stack/bin/happier.mjs
apps/stack/bin/localBundledWorkspacePreflight.mjs
  -> cli/stack-cli/bin-entrypoints/

apps/stack/scripts/utils/cli/cli_registry.mjs
apps/stack/scripts/happier_main.mjs routing portions
  -> cli/stack-cli/command-router/

apps/stack/bin/hstack.mjs global flag, sandbox, re-exec portions
apps/stack/scripts/utils/env/** stack-specific portions
  -> cli/stack-cli/global-options/

apps/stack/scripts/utils/stack/**
apps/stack/scripts/utils/paths/** stack-specific portions
  -> cli/stack-cli/stack-context/

apps/stack/scripts/stack.mjs
apps/stack/scripts/stack/**
  -> cli/stack-cli/stack-management/

apps/stack/scripts/run.mjs
apps/stack/scripts/dev.mjs
apps/stack/scripts/stop.mjs
apps/stack/scripts/logs.mjs
apps/stack/scripts/tui.mjs
  -> cli/stack-cli/stack-runtime-commands/

apps/stack/scripts/worktrees.mjs
apps/stack/scripts/utils/git/worktrees*.mjs
apps/stack/scripts/utils/worktrees/**
  -> cli/stack-cli/worktree-commands/ 或可复用时归 shared/source-control

apps/stack/scripts/build.mjs
apps/stack/scripts/lint.mjs
apps/stack/scripts/typecheck.mjs
apps/stack/scripts/test_cmd.mjs
apps/stack/scripts/mobile*.mjs
apps/stack/scripts/eas.mjs
  -> cli/stack-cli/component-commands/ 作为命令 facade
  -> app/client 或 ops/mobile 承载实现

apps/stack/scripts/auth.mjs
apps/stack/scripts/utils/auth/**
  -> cli/stack-cli/auth-context/

apps/stack/scripts/daemon_cmd.mjs
apps/stack/scripts/service.mjs
apps/stack/scripts/tailscale.mjs
apps/stack/scripts/self_host.mjs
apps/stack/scripts/self.mjs
  -> cli/stack-cli/host-service-commands/ 作为命令 facade
  -> daemon/service/shared 承载实现

apps/stack/scripts/remote_cmd.mjs
apps/stack/scripts/utils/remote/**
  -> cli/stack-cli/remote-setup-commands/ 作为命令 facade
  -> shared/remote-execution 或 shared/system-tasks 承载原语

apps/stack/scripts/providers_cmd.mjs
  -> cli/stack-cli/provider-commands/

apps/stack/scripts/setup.mjs
apps/stack/scripts/contrib.mjs
apps/stack/scripts/setup_pr.mjs
apps/stack/scripts/review_pr.mjs
apps/stack/scripts/review.mjs
apps/stack/scripts/monorepo.mjs
apps/stack/scripts/import.mjs
apps/stack/scripts/migrate.mjs
apps/stack/scripts/ci.mjs
apps/stack/scripts/pack.mjs
  -> cli/stack-cli/maintainer-commands/ 作为命令 facade
  -> ops/dev-workflows 或 ops/packaging 承载实现

apps/stack/scripts/menubar.mjs
apps/stack/scripts/completion.mjs
apps/stack/extras/**
  -> cli/stack-cli/integrations/ 作为命令 facade
  -> 可复用 assets 归 owner module

apps/stack/scripts/happier.mjs
apps/stack/scripts/happier_main.mjs
apps/stack/scripts/repo_local.mjs
apps/stack/scripts/repo_cli_activate.mjs
  -> cli/stack-cli/happier-passthrough/

apps/stack/.claude/**
apps/stack/.cursor/**
apps/stack/.edison/**
apps/stack/.pal/**
  -> agents/、docs/ 或 ops workflow configuration；不归 cli/stack-cli/
```

`cli/command-runtime/` 已确认的三级目录：

```text
cli/command-runtime/
  context/
  argv/
  output/
  errors/
  prompts/
  terminal/
  environment/
  paths/
  process/
  network/
  bin-wrappers/
  console/
  testkit/
```

`context/` 负责共享 command execution context、stdio/TTY metadata、command invocation metadata。具体产品的 command registry 仍归对应 CLI 产品面，例如 `happier-cli/routing/`。

`argv/` 负责共享 argv parsing 和 normalization 原语，例如 flag readers、special command parsing helper、generic argument shapes。产品级 routing 仍归 `happier-cli/`、`setup-cli/` 或 `stack-cli/`。

`output/` 负责共享 JSON envelope、TTY/JSON rendering helper、progress output、table/list rendering、稳定命令输出原语。某个命令领域专属输出模型可以留在 owner 命令领域内部，例如 `happier-cli/commands/sessions/output/`。

`errors/` 负责 command-level error normalization、exit-code mapping、用户可读错误包装和 debug-output policy。领域错误应由 owner module 定义，这里只做 CLI 展示适配。

`prompts/` 负责共享 CLI prompts，例如 confirm、input、secret input、multiple-choice prompts。

`terminal/` 负责命令行 terminal 支撑，例如 terminal runtime flags、terminal metadata、attach planning、headless/tmux command support。PTY 和 session runtime behavior 归 daemon/provider/session owner，不归这里。

`environment/` 负责 CLI-only environment handling，包括 environment sanitization、nested-session environment cleanup、CLI release-channel/env helper。Daemon/service runtime config parsing 归对应 owner。

`paths/` 负责 CLI-only path helper，例如 home-path expansion/display、path shape helper、CLI invocation path parsing。跨模块 path contract 归 `shared/`。

`process/` 负责短生命周期 command execution helper，例如 command existence checks、streaming command runner、Windows command invocation resolution。长驻 supervision、daemon process management、service supervisor 不属于这里。

`network/` 负责 CLI-only network helper，例如 proxy resolution、no-proxy matching、HTTP client proxy installation、socket proxy helper。如果某个 network primitive 稳定被 app/server/daemon 复用，应迁到 `shared/network/`。

`bin-wrappers/` 负责 bin wrapper helper、spawn current CLI helper、invoker name resolution、wrapper invocation compatibility logic。Package publishing、release artifact construction、workspace bundling 归 `ops/` 或 package-local tooling owner。

`console/` 负责平台 console hardening，例如 Windows UTF-8 code page setup、stdout/stderr best-effort writing、console write guards。

`testkit/` 负责 `command-runtime/` 自身测试 helper 和 fixture。

`command-runtime/` 的依赖方向：

```text
happier-cli/ setup-cli/ stack-cli/
  -> command-runtime/
  -> shared/
```

`command-runtime/` 可以依赖 `shared/` 和外部库，但不能依赖 `app/`、`service/`、`daemon/`、`provider/`、`ops/`，也不能依赖某个 CLI 产品面下的具体 commands。

以下能力不应放入 `command-runtime/`：

- Provider managed tools、provider installation、provider resolution。
- First-party runtime installation、release payload、managed component runtime。
- Daemon runtime、长驻 process supervision、daemon state、service supervisor。
- Relay host engine 或 server-runner behavior。
- System tasks 和 remote bootstrap task implementations。
- Release/package/workspace bundling、component artifact construction。
- MCP server/runtime/resource/provider-detection behavior。
- 只是因为 CLI 当前 import 了某个 generic utility，就把它放入 command-runtime；跨模块 utility 应归 `shared/`，owner-specific utility 应贴近 owner。

具体 CLI lifecycle 行为，例如 auto-update notice、runtime re-exec、self-update command wrapping，归 `happier-cli/cli-lifecycle/`。`command-runtime/` 只能拥有这些 lifecycle surface 可复用的 rendering/error primitive。

`cli/happier-cli/` 已确认的三级目录：

```text
cli/happier-cli/
  entrypoints/
  routing/
  commands/
  agent-commands/
  mcp-surface/
  module-adapters/
  help/
  cli-lifecycle/
  testkit/
```

`entrypoints/` 负责用户可执行的 Happier CLI 入口，包括 `happier`、`happier-dev` 和 `happier-mcp`。它应保持很薄，只做启动期 argv 规范化、启动期加固和进入 routing 的交接。

`routing/` 负责顶层参数规范化、命令分发、command registry wiring、command surface manifest、根级 `--help`/`--version`、默认 provider 命令兜底，以及 provider 命令透传判断。它只做路由，不实现 daemon、provider 或 service 行为。

`commands/` 负责普通用户可见 Happier 命令 facade。它包含命令局部参数解析、help、JSON/TTY 输出选择，以及 sessions、auth、machines、servers、daemon、service、connections、profiles、diagnostics、installation、notifications、relay、self、tools 等命令层编排。底层 daemon/provider/service 实现仍归对应 owner。

`cli/happier-cli/commands/` 已确认的四级目录：

```text
cli/happier-cli/commands/
  sessions/
  auth/
  machines/
  servers/
  daemon/
  service/
  connections/
  profiles/
  diagnostics/
  installation/
  notifications/
  relay/
  self/
  tools/
```

`sessions/` 负责会话类命令 facade，例如 `session`、`sessions`、`attach`、`resume`、`send`、`wait`、`history`、`run`、`delegate`、`plan`、`review` 和 voice-agent session 命令入口。Session runtime、PTY、handoff runtime 和 provider session execution 不属于 commands。

`auth/` 负责 login、logout、pairing、approval、request、status、wait、server-scoped auth cleanup 等命令 facade。Token persistence、daemon auth state、service auth API 和 auth protocol schema 归对应 owner。

`machines/` 负责 machine 命令 facade 和展示行为。Machine ownership、daemon-local machine state、registration、sync 行为归 daemon/shared owner。

`servers/` 负责 server 管理命令 facade，例如 add、list、select、reachable URL flow、server-selection UX 和 self-heal prompts。它不负责 API server 实现。

`daemon/` 负责 daemon 命令 facade，例如 start、stop、status、takeover、ownership-conflict 展示、daemon service list/status 入口。Daemon supervision 和长驻进程状态归 `daemon/`。

`service/` 负责本地 service 控制命令 facade，以及 service repair 的命令入口。本地 service lifecycle implementation 归 `daemon/`；服务运行时归 `service/api-server/` 或 `service/server-runner/`。

`connections/` 负责 connect 命令 facade、连接目标解析、auth intent resolution 和目标 service selection 的 CLI 行为。共享连接协议和 transport primitives 不属于 commands。

`profiles/` 负责 profile 命令 facade，例如 profile list 和当前 profile 展示。Profile source of truth、provider profile schema 和 account settings persistence 归对应 owner。

`diagnostics/` 负责 doctor、bug-report、capabilities、repair report、健康检查输出和诊断型清理命令 facade。具体 owner 的诊断检查应通过 owner contract 暴露，并由这里聚合渲染。

`installation/` 负责 `happier install ...` 命令 facade 和安装结果展示。Provider installation source of truth、managed runtime、binary-safe install plan 归 provider/runtime owner。

`notifications/` 负责 push notification 命令 facade，例如 `notify`。

`relay/` 负责 relay 命令 facade，例如 relay host、status、warnings、local server binary version 展示。Relay server implementation、runner download/start 行为和 release asset resolution 不属于 commands。

`self/` 负责 CLI self 命令 facade，例如 self、self-update、self migrate、version-gated migration 入口。底层 CLI lifecycle 行为归 `cli-lifecycle/`；release/publish pipeline 归 `ops/`。

`tools/` 负责 Happier tools 命令 facade，例如 tools list/call 和 session-bound tool call 参数。Tool catalog、built-in tool runtime 和 MCP tool resolution 归对应 owner。

通用命令输出 helper 不属于 `commands/`。共享 JSON envelope、TTY/JSON rendering、tables、error formatting 归 `cli/command-runtime/output/`。某个命令领域专属输出模型可以放在对应命令领域内部，例如 `commands/sessions/output/`。

`agent-commands/` 负责 agent/provider 子命令的 CLI command-hook 层。它注册 provider command surface，并适配 help/output 行为；provider-specific execution、runtime policy、install/detection、direct session、fork、resume 等能力归 `provider/<agent>/`。

`cli/happier-cli/agent-commands/` 已确认的四级目录：

```text
cli/happier-cli/agent-commands/
  registry/
  invocation/
  session-start/
  passthrough/
  help/
  testkit/
```

`agent-commands/` 属于 Happier CLI 产品面。它负责把 provider 暴露为用户可见子命令，例如 `happier codex`、`happier claude`、`happier opencode`，但不能拥有 provider-specific 行为。`registry/` 负责把 provider catalog entry 和 `cliSubcommand` 挂到 Happier command registry。`invocation/` 负责 lazy-load 并调用 provider command hook，同时适配 `CommandContext`、错误和 command-runtime 输出行为。`session-start/` 负责 provider session 启动的通用 CLI 流程，包括共享参数解析、account settings bootstrap、profile overlay、daemon autostart coordination、runner lock 和通用错误处理。`passthrough/` 负责 provider CLI info passthrough，例如原生 `--help`、`--version` 请求判断。`help/` 负责 provider 命令分组 help 和 provider 列表渲染。`testkit/` 只放这个 CLI hook 层的测试 helper 和 fixture。

provider-owned 行为仍归 `provider/<agent>/`：provider-specific command behavior、ACP/runtime/app-server/MCP client、auth spec、detect、capability contribution、daemon spawn hook、direct session、attach、fork、resume、install/runtime policy、prompt、permission、tool 和 provider metadata shaping。

依赖方向：

```text
cli/happier-cli/routing/
  -> cli/happier-cli/agent-commands/registry

cli/happier-cli/agent-commands/
  -> cli/command-runtime/
  -> cli/happier-cli/module-adapters/
  -> provider/catalog or shared/agent-domain
  -> provider/<agent> public hooks
```

禁止依赖方向：

```text
provider/<agent>/
  -> cli/happier-cli/agent-commands/

provider/<agent>/
  -> cli/happier-cli/commands/*
```

provider CLI hook contract 应放在中立的 provider catalog 或 shared agent-domain 位置，而不是放在 `agent-commands/` 内，避免 provider 为了暴露 hook 反向 import Happier CLI 产品模块。

当前代码迁移映射：

```text
apps/cli/src/cli/commandRegistry.ts buildAgentCommandRegistry
  -> cli/happier-cli/agent-commands/registry/

apps/cli/src/cli/runBackendSessionCliCommand.ts
  -> cli/happier-cli/agent-commands/session-start/

apps/cli/src/cli/providerCliPassthrough.ts
  -> cli/happier-cli/agent-commands/passthrough/

apps/cli/src/backends/<provider>/cli/command.ts
  -> provider/<agent>/cli-surface/commandHook

apps/cli/src/backends/<provider>/cli/detect.ts
apps/cli/src/backends/<provider>/cli/auth/**
apps/cli/src/backends/<provider>/cli/capability.ts
  -> provider/<agent>/detection/
  -> provider/<agent>/auth/
  -> provider/<agent>/capabilities/

apps/cli/src/backends/<provider>/acp/**
apps/cli/src/backends/<provider>/directSessions/**
apps/cli/src/backends/<provider>/daemon/**
apps/cli/src/backends/<provider>/mcp/**
apps/cli/src/backends/<provider>/runtime/**
  -> provider/<agent>/ 对应 runtime 能力目录
```

`mcp-surface/` 负责 Happier CLI 暴露给用户和外部 MCP Host 的 MCP 操作面。它覆盖 `happier mcp ...`、`happier-mcp*.mjs`、外部 MCP Host 启动、managed MCP servers 命令 facade，以及通用 stdio bridge/launcher 暴露层。它是 CLI 产品面，不是 provider-specific MCP adapter、session-agent MCP runtime、共享 MCP tool/resource contract 的 owner。

`cli/happier-cli/mcp-surface/` 已确认的四级目录：

```text
cli/happier-cli/mcp-surface/
  bin-entrypoints/
  command-router/
  external-server/
  managed-servers/
  stdio-launchers/
  stdio-bridges/
  composition/
  help/
  testkit/
```

`bin-entrypoints/` 负责 `happier-mcp*.mjs` 这类 MCP 可执行入口包装；只做 Node/runtime entrypoint 准备、启动期加固和委派，不写 provider 实现行为。`command-router/` 负责 `happier mcp` 命令路由和 usage 行为。`external-server/` 负责 `happier mcp serve`，包括 CLI 参数、credential/machine context、account-settings bootstrap、stdio-safe startup，以及交给 external MCP server factory。`managed-servers/` 负责用户配置的 MCP servers 的 CLI facade，包括 list/add/bind/unbind/detect/test。`stdio-launchers/` 和 `stdio-bridges/` 负责通用 CLI-facing stdio MCP launcher/bridge surface。`composition/` 负责 MCP surface 的依赖组装，不能演变成通用 utils 垃圾桶。`help/` 负责 MCP command help 和分组。`testkit/` 负责 MCP surface 测试 fixture、命令 harness 和 helper。

`mcp-surface/` 不能吸收 provider-specific MCP 逻辑。Codex、Claude、OpenCode、Gemini 等 provider 的 MCP client、detect、config merge、spawn resolution、tool-name policy 和 provider-specific bridge 应归 `provider/<agent>/mcp/` 或 provider 等价 public capability 目录。session-scoped MCP runtime，例如 per-session MCP server、session-agent HTTP/stdio bridge，应按最终 owner 归 `daemon/` 或 `shared/agent-runtime/`。共享 MCP schema、resource/tool registration contract、server config record、transport-independent normalization 应归 `shared/mcp-domain` 或 `shared/agent-domain`。

允许依赖方向：

```text
cli/happier-cli/routing/
  -> cli/happier-cli/mcp-surface/command-router

cli/happier-cli/mcp-surface/
  -> cli/command-runtime/
  -> cli/happier-cli/module-adapters/
  -> shared/mcp-domain or shared/agent-domain
  -> provider/catalog public MCP hooks
```

禁止依赖方向：

```text
provider/<agent>/mcp/
  -> cli/happier-cli/mcp-surface/

daemon/
  -> cli/happier-cli/mcp-surface/

shared/
  -> cli/happier-cli/mcp-surface/

cli/happier-cli/mcp-surface/
  -> provider/<agent>/mcp/* deep implementation
  -> daemon/session internals
```

当前代码迁移映射：

```text
apps/cli/bin/happier-mcp*.mjs
  -> cli/happier-cli/mcp-surface/bin-entrypoints/

apps/cli/src/cli/commands/mcp.ts
  -> cli/happier-cli/mcp-surface/command-router/

apps/cli/src/cli/commands/mcp/serve.ts
  -> cli/happier-cli/mcp-surface/external-server/

apps/cli/src/cli/commands/mcp/servers/**
  -> cli/happier-cli/mcp-surface/managed-servers/

apps/cli/src/cli/commands/mcp/deps.ts
  -> cli/happier-cli/mcp-surface/composition/

apps/cli/src/mcp/bridges/remoteMcpStdioBridge.ts
apps/cli/src/mcp/launchers/stdioMcpServerLauncher.ts
  -> cli/happier-cli/mcp-surface/stdio-bridges/
  -> cli/happier-cli/mcp-surface/stdio-launchers/

apps/cli/src/mcp/createHappierMcpServer.ts
apps/cli/src/mcp/startHappyServer.ts
  -> daemon/ 或 shared/agent-runtime/ session MCP runtime

apps/cli/src/backends/<provider>/mcp/**
apps/cli/src/backends/codex/happyMcpStdioBridge.ts
apps/cli/src/backends/codex/codexMcpClient.ts
  -> provider/<agent>/mcp/

apps/cli/src/mcp/server/registerHappierMcpBuiltInTools.ts
apps/cli/src/mcp/resources/registerHappierMcpResources.ts
apps/cli/src/mcp/happierMcpToolCatalog.ts
  -> shared/mcp-domain 或 shared/agent-domain
```

`module-adapters/` 负责从 CLI 产品面到其他模块的薄适配层，例如 daemon control client、service/server selection client、provider command client。它可以转换 CLI 参数、调用模块 owner 暴露的 contract、映射错误、适配输出，但不能成为目标模块的实现 owner。

`cli/happier-cli/module-adapters/` 已确认的四级目录：

```text
cli/happier-cli/module-adapters/
  sessions/
  daemon/
  service/
  server/
  auth/
  machines/
  connections/
  profiles/
  relay/
  diagnostics/
  installation/
  notifications/
  capabilities/
  terminal/
  shared/
```

`module-adapters/` 是 `commands/` 和 owner module 之间的依赖闸口。`commands/*` 应优先依赖这些 adapter，而不是直接 import daemon、service、provider、session、API、terminal、persistence 等深层内部实现。每个 adapter 面向一个 owner 能力边界暴露稳定的 CLI-facing API：负责规范化命令输入，把 typed request 传给 owner module，把 owner error 映射成 command-runtime 的 error/output 形态，并返回适合 command rendering 的数据结构。

adapter 目录按 owner-facing capability 切分，而不是按技术 helper 类型切分。`sessions/` 适配 session 查询、创建、发送、停止、等待、历史和 execution-run 操作。`daemon/` 适配 daemon start/stop/status/restart/takeover/ownership/service-list。`service/` 适配 background service install/uninstall/status/repair。`server/` 适配 server profile、server selection、本地或远端 server 管理。`connections/` 适配 OAuth、connected service 和 credential persistence。`terminal/` 适配 attach、tmux、Windows Terminal、console focus 等 CLI command execution 专属终端行为。`capabilities/` 只适配 capability query，真实 capability registry 仍归 owner module。`shared/` 只放 adapter 内部共享类型和小型组合 helper，不能变成新的通用 utils 垃圾桶。

允许依赖方向：

```text
cli/happier-cli/commands/*
  -> cli/happier-cli/module-adapters/*
  -> daemon/ service/ provider/ agents/ shared/

cli/happier-cli/module-adapters/*
  -> cli/command-runtime/*
  -> shared/*
```

禁止依赖方向：

```text
daemon/ service/ provider/ shared/
  -> cli/happier-cli/module-adapters/

cli/happier-cli/module-adapters/*
  -> cli/happier-cli/commands/*
  -> cli/happier-cli/routing/*
```

`module-adapters/` 不能吞掉 `agent-commands/` 或 `mcp-surface/`。provider 命令 hook 和 provider-specific 命令行为归 `agent-commands/` 与 `provider/<agent>/`；CLI-facing MCP launch/bridge 入口归 `mcp-surface/`；provider-specific MCP adapter 归 `provider/`；跨模块 MCP contract 归 `shared/`。

`help/` 负责 root help、跨命令 help material、命令分组和跨命令 surface 文档。只对某个命令有意义的局部 help 可以继续贴近该命令。

`cli-lifecycle/` 负责 CLI 自身生命周期能力，例如版本展示、auto-update notice、runtime re-exec、self-update 命令包装。release pipeline、跨模块 packaging、deploy orchestration 仍归 `ops/`。

`testkit/` 负责 Happier 主 CLI 产品面专属的测试 helper 和 fixture。跨 package 测试工具应归 `tests/` 或 shared testkit 位置，不应塞进 `happier-cli/`。

当前可能来源：

- `apps/cli`
- `packages/cli-common`
- `apps/bootstrap`
- `apps/stack` 中面向用户的命令 facade 部分

CLI 边界混杂分析：

- `cli-boundary-reorganization-design-notes.zh-CN.md`

允许依赖：

- `shared/`
- 来自 `provider/` 的 provider CLI backend surface。
- 调用本地服务时依赖 daemon control interface。

禁止依赖：

- 不依赖 app UI 实现。
- 当 provider-specific 行为可以放在 provider-owned module 中时，不应放在 generic CLI core 中。
- 不在 CLI command core 中承载 daemon 长驻进程实现。
- 不在 CLI command core 中承载 service/api-server 实现。
- 不在 CLI command core 中拥有 release/deploy pipeline。

### `daemon/`

负责本地长驻进程行为和本地服务生命周期。

预期范围：

- 本地 daemon 生命周期。
- service install、uninstall、enable、disable、restart、status、logs。
- 本地状态文件、端口、进程监督、运行时清理。
- 启动或停止 Happier 本地环境的 local wrapper scripts。

当前可能来源：

- `apps/stack/scripts` 中的相关部分。
- `apps/cli` 中相关 daemon/runtime 部分。
- `scripts/local`

允许依赖：

- `shared/`
- daemon 命令通过 CLI 暴露时，可依赖 CLI-local command entrypoint。
- 只能通过 provider-owned interface 依赖 provider runtime。

禁止依赖：

- 不在 daemon orchestration 中内嵌 provider policy。
- 不依赖 app UI 实现。

### `provider/`

负责外部 agent/provider 系统的运行时集成。

预期范围：

- Codex、Claude、OpenCode、Gemini、Kimi、Qwen、Copilot 等 provider adapter。
- CLI backend 实现。
- Provider capability 声明。
- Provider-owned protocol adapter 和 projection。共享 wire/schema/API
  contracts 仍归 `shared/protocol`。
- Provider UI behavior adapter。
- Provider contract tests 和 provider-specific fixtures。

当前可能来源：

- `apps/cli/src/backends/*`
- `apps/ui/sources/agents/providers`
- `apps/ui/sources/agents/registry`
- `apps/ui/sources/agents/backendCatalog`
- `packages/cli-common/src/providers`
- `packages/agents/src/providers` 中 provider-specific 的部分
- `packages/agents/src/providerSettings` 中的 provider settings definitions 和
  registries，按 catalog/domain/provider owner 拆分
- `packages/agents/src/sessionControls` 中 provider-specific 的部分
- `packages/protocol/src/providers` 中经过 shared protocol contract 与
  executable provider policy 分类后的 provider-owned 候选内容
- 描述 provider 行为的 provider-focused docs，例如 feature matrices。

来源分类规则：

- `packages/protocol` 仍然拥有共享 protocol contracts。位于
  `packages/protocol/src/providers/**` 下的 provider wire/schema/API contracts
  应随 `shared/protocol` 迁移，不能直接整体搬入 `provider/`。
- 当前靠近 protocol/provider packages 的 provider-specific executable behavior、
  install policy、manifest data 或 family adapter behavior，必须先分类，再按 owner
  进入 `provider/<providerId>/`、`provider/catalog/`、`provider/managed-tools/`
  或 `provider/provider-families/`。
- `packages/agents` 不能作为整体一次性搬迁。如果它同时包含 provider-agnostic
  agent domain/runtime primitives 和 provider-specific definitions，应先按 owner
  拆分：前者进入 `shared/agent-domain` 或 `shared/agent-runtime`，后者进入
  `provider/`。

已确认的二级目录：

```text
provider/
  catalog/
  cross-provider/
  managed-tools/
  provider-families/
  claude/
  codex/
  opencode/
  gemini/
  auggie/
  qwen/
  kimi/
  kilo/
  kiro/
  pi/
  copilot/
  customAcp/
```

`catalog/` 负责 provider registry 和派生产物。它聚合 provider entries、settings registries、UI/provider registry surfaces、provider id/schema catalogs，但不拥有具体 provider runtime implementation。

`cross-provider/` 负责多个 provider 共同需要的 provider-domain 原语，例如 hook types、provider capability helpers、model capability normalization、direct-session/fork hook contracts、provider-agnostic adapter utilities。它是 `provider/` 内部的跨 provider 公共能力层，不是仓库一级 `shared/`。

`managed-tools/` 负责 provider CLI resolution 和 installation support，包括 managed Node/PNPM runtime helpers、provider CLI path resolution、vendor recipe planning、managed package installation、GitHub release download/extraction，以及 binary-safe runtime guards。

`provider-families/` 负责相关 provider family 的共享行为，例如 OpenCode-family providers 或 catalog-defined ACP providers。它用于避免一个具体 provider 直接 import 另一个具体 provider。

`codex/`、`claude/`、`opencode/`、`customAcp/` 等具体 provider 目录负责该 provider 的 runtime、CLI adapter、auth、cloud connect、direct-session、MCP、settings、UI adapter、provider-owned protocol adapter/projection、fixtures 和 provider-local tests。共享 protocol wire/schema/API contracts 仍归 `shared/protocol`。`customAcp/` 先保留 camelCase，以匹配当前持久化 `AgentId`，避免额外 migration alias。

`provider/catalog/` 已确认的三级结构：

```text
provider/catalog/
  entry-contracts/
  registry/
  identifiers/
  capability-index/
  hook-resolvers/
  settings-index/
  backend-targets/
  installables/
  ui-projections/
  derived-artifacts/
  validation/
  testkit/
```

`provider/catalog/` 是连接各 provider 的薄入口层，只通过公开 manifest 和公开 hook 连接 provider。它不包含单个 provider 的特殊 runtime 接入方式。

- `entry-contracts/` 定义 catalog entry、manifest entry、hook 类型和 capability 声明结构。
- `registry/` 聚合 `provider/<providerId>/manifest/` 暴露的 entry，并提供 provider 查询 API。
- `identifiers/` 管理 provider id、alias、默认 provider、CLI subcommand 映射和持久化 id 兼容规则。
- `capability-index/` 从 manifest 与 hook presence 派生 provider 能力矩阵。
- `hook-resolvers/` 负责 lazy hook 查询、缓存、缺失 hook 错误和通用 fallback。
- `settings-index/` 聚合 provider settings 定义，并校验 settings key owner。
- `backend-targets/` 管理通用 backend target reference、profile/flavor 索引和查询 helper。
- `installables/` 声明 provider installable key 与 provider owner。可执行安装逻辑归 `provider/managed-tools/`。
- `ui-projections/` 构建 UI 消费的纯数据投影，不放 React screen 或 provider 特殊 UI 行为。
- `derived-artifacts/` 放从 registry 派生出的 protocol、CLI、UI 稳定产物。
- `validation/` 校验 catalog 一致性、id 唯一、hook/capability 对齐、settings key 冲突和 installable owner。
- `testkit/` 放 catalog 测试 fixtures 和 manifest builders。

允许依赖：

```text
provider/catalog/registry       -> provider/<providerId>/manifest/
provider/catalog/hook-resolvers -> provider/catalog/registry
provider/catalog/*              -> provider/catalog/entry-contracts
provider/catalog/*              -> provider/cross-provider/contracts
```

禁止依赖：

```text
provider/catalog/ -> provider/<providerId>/runtime/
provider/catalog/ -> provider/<providerId>/cli-adapter/
provider/catalog/ -> provider/<providerId>/auth/
provider/catalog/ -> provider/<providerId>/sessions/
provider/catalog/ -> provider/<providerId>/ui-adapter/
provider/catalog/ -> provider/managed-tools executable installers
provider/catalog/ -> app/client screens
```

`provider/cross-provider/` 已确认的三级结构：

```text
provider/cross-provider/
  contracts/
  direct-sessions/
  forking/
  session-metadata/
  session-capabilities/
  execution-runs/
  model-capabilities/
  cli-adapter-primitives/
  ui-adapter-contracts/
  permissions/
  diagnostics/
  testkit/
```

`provider/cross-provider/` 是 provider 域内部的跨 provider 公共能力层。它存放跨 provider
复用、但仍然属于 provider 集成语义的原语。它不是仓库一级 `shared/`，也
不是 provider catalog，更不是具体 provider 实现目录。

- `contracts/` 负责 provider 共享类型契约、hook shape 和基础 capability
  声明原语。它可以描述 provider hook 长什么样，但 provider 注册、查询和
  hook 解析仍归 `provider/catalog/`。
- `direct-sessions/` 负责跨 provider direct-session 操作契约、transcript、
  page、candidate、activity 结构，以及 provider-agnostic environment merge
  helper。具体 direct-session 行为仍归 `provider/<providerId>/sessions/`
  或 provider 自己的 direct-session 能力目录。
- `forking/` 负责 provider fork 契约，例如 ACP continuation handler、
  provider-native fork point 和 fork dispatch result shape。它不执行具体
  provider 的 fork 行为。
- `session-metadata/` 负责通用 provider session metadata update helper、
  provider session id metadata 处理，以及 metadata merge/update 原语。
- `session-capabilities/` 负责 provider-agnostic session capability 归一化，
  例如 resume、attach、follow、fork、direct-session 支持结果，供 CLI、UI、
  daemon 或 catalog projection 消费。
- `execution-runs/` 负责 provider-agnostic execution-run adapter 原语和简单
  backend factory。它不能变成 CLI command routing，也不能放具体 provider
  runtime。
- `model-capabilities/` 负责模型能力归一化，例如 context window token 解析
  和稳定的 model capability shape。
- `cli-adapter-primitives/` 负责 provider CLI adapter 可复用的小型基础件，
  例如通用 terminal display factory。只有当这些 helper 是 provider adapter
  原语而不是 Happier command surface 时，才放在这里；深度依赖 CLI command
  routing 的内容应归 `cli/`。
- `ui-adapter-contracts/` 只负责 data-only 的 UI adapter 契约，例如 provider
  settings、local auth、install banner 和 UI-facing provider behavior 声明。
  React component、screen、theme 和 provider-specific UI 行为仍归 `app/client`
  或 `provider/<providerId>/ui-adapter/`。
- `permissions/` 负责 provider 共享 permission request source、permission
  capability contract 和权限归一化 helper。具体 provider policy 仍归具体
  provider。
- `diagnostics/` 负责标准化 provider diagnostics、warnings、capability
  mismatch report 和 runtime diagnostic result shape。它输出结构化诊断结果，
  不负责 UI 渲染。
- `testkit/` 负责 provider-shared fixtures、typed builders、fake provider
  hooks 和 contract test helpers。它不能成为 production runtime dependency。

允许依赖：

```text
provider/catalog/*           -> provider/cross-provider/contracts
provider/<providerId>/*      -> provider/cross-provider/*
provider/provider-families/* -> provider/cross-provider/*
provider/managed-tools/*     -> provider/cross-provider/contracts or model-capabilities
provider/cross-provider/*            -> shared/ public contracts and primitives
```

禁止依赖：

```text
provider/cross-provider/ -> provider/catalog/
provider/cross-provider/ -> provider/<providerId>/
provider/cross-provider/ -> provider/provider-families/
provider/cross-provider/ -> provider/managed-tools executable installers
provider/cross-provider/ -> app/client screens or React UI components
provider/cross-provider/ -> cli/happier-cli command routing
provider/cross-provider/ -> daemon implementation internals
provider/cross-provider/ -> service concrete storage/API implementation
```

当前代码来源映射建议：

```text
apps/cli/src/backends/directSessions/providerOps.ts
  -> provider/cross-provider/direct-sessions/

apps/cli/src/backends/forking/**
  -> provider/cross-provider/forking/

apps/cli/src/backends/modelCapabilities/contextWindowTokens.ts
  -> provider/cross-provider/model-capabilities/

apps/cli/src/backends/shared/createProviderSessionIdMetadataUpdater.ts
  -> provider/cross-provider/session-metadata/

apps/cli/src/backends/shared/createSimpleExecutionRunBackendFactory.ts
  -> provider/cross-provider/execution-runs/

apps/cli/src/backends/shared/createProviderTerminalDisplay.tsx
  -> provider/cross-provider/cli-adapter-primitives/；仅当它保持为薄 provider-adapter
     primitive，否则应留在 cli/command-runtime/

apps/ui/sources/agents/providers/shared/* data contracts
  -> provider/cross-provider/ui-adapter-contracts/

apps/ui/sources/agents/providers/shared/* React/theme/rendering code
  -> app/client provider UI integration 或 provider/<providerId>/ui-adapter/

packages/agents/src/providers/providerCliRuntime.ts provider-specific data
  -> provider/catalog/installables/ 或 provider/managed-tools/，不归
     provider/cross-provider/
```

`provider/managed-tools/` 已确认的三级结构：

```text
provider/managed-tools/
  contracts/
  installable-resolvers/
  provider-cli-resolution/
  provider-cli-launch/
  install-plans/
  installers/
  managed-runtimes/
  release-assets/
  tool-store/
  tool-status/
  updates/
  diagnostics/
  testkit/
```

`provider/managed-tools/` 负责 provider 相关外部工具的解析、安装、托管运行时支撑、状态、更新检查和 binary-safe 启动保障。它不是 provider runtime，不是 catalog，也不是 CLI command surface。它的职责是把 catalog/provider manifest 中声明的 installable 转换为可解析、可安装、可启动、可诊断、可更新的真实工具。

- `contracts/` 负责 managed-tool 类型契约，例如 install mode、resolution source、install result、tool status 和 managed runtime availability。它必须限制在 managed-tools 内部契约，不能替代 `provider/cross-provider/contracts/`。
- `installable-resolvers/` 把 `provider/catalog/installables/`、provider manifest 和 provider CLI runtime 声明解析成 managed-tools 可执行的工具定义。它不是第二套 installable registry。
- `provider-cli-resolution/` 负责 provider CLI 路径解析：显式 override、system PATH、known candidates、managed install path 和 source preference。它回答“用哪个 CLI 路径”。
- `provider-cli-launch/` 把 resolved CLI path 转成 spawn-safe launch spec，包括 JavaScript runtime wrapper、shebang 检查、`.js`/`.mjs` 入口处理、command/args 组合。它回答“如何安全启动该工具”。
- `install-plans/` 负责 dry-run 和确认安全的安装计划：平台、安装模式、admin requirement、安装命令和结构化 plan output。它不能执行安装。
- `installers/` 负责真正执行安装。四级目录应拆为 `vendor-recipes/`、`managed-packages/` 和 `github-release-binaries/`。vendor recipe 默认必须需要确认；managed package 必须通过 managed runtime/tooling；GitHub release binary 必须完成下载、校验、解压和 staged 落位。
- `managed-runtimes/` 负责 Happier 自管运行时前置条件，例如 managed JavaScript runtime 和 managed pnpm。这里是 binary-safe 边界，保证没有系统 `node`、`npm`、`npx`、`pnpm`、`yarn`、`bunx` 时 first-party runtime path 仍能工作。
- `release-assets/` 负责 provider/tool-specific release asset 选择、digest 要求和 managed-tool 落位适配。通用 archive extraction、checksum、release download 原语在可跨模块复用时应下沉到 `shared/release-runtime`。
- `tool-store/` 负责 `$HAPPIER_HOME/tools/**` 布局、`current/next` staged install、scratch dir、lock、install-state file 和 install log path。
- `tool-status/` 负责 managed tool 状态：installed status、resolved path、source kind、installed version、last install log、last background check timestamp。显式命名为 `tool-status/` 是为了避免吸收 provider、daemon 或 session status。
- `updates/` 负责 latest-version check、version comparison、background auto-update eligibility、throttling 和 update result recording。它和 `tool-status/` 分离，避免查询状态隐含更新副作用。
- `diagnostics/` 负责结构化 managed-tool 诊断，例如 invalid override、missing runtime prerequisites、unavailable CLI、checksum verification failure、install failure、update failure。UI/CLI 文案渲染仍归消费层。
- `testkit/` 负责 fake releases、fake provider CLI、fake tool stores、install plan fixtures 和 managed-runtime test helpers。它不能成为 production dependency。

允许依赖：

```text
provider/managed-tools/ -> provider/catalog/installables declarations
provider/managed-tools/ -> provider/cross-provider/contracts
provider/managed-tools/ -> shared/release-runtime
provider/managed-tools/ -> shared/first-party-runtime or shared/system primitives
provider/<providerId>/cli-adapter -> provider/managed-tools/provider-cli-resolution
provider/<providerId>/runtime -> provider/managed-tools/provider-cli-launch
cli/installation commands -> provider/managed-tools public install APIs
daemon/status hooks -> provider/managed-tools/tool-status
```

禁止依赖：

```text
provider/catalog/ -> provider/managed-tools executable installers
provider/managed-tools/ -> app/client screens
provider/managed-tools/ -> cli/happier-cli command routing
provider/managed-tools/ -> provider/<providerId>/runtime implementation
provider/managed-tools/ -> service concrete storage/API implementation
provider/managed-tools/ -> ops release pipeline
```

当前代码来源映射建议：

```text
packages/cli-common/src/providers/resolution.ts
apps/cli/src/runtime/managedTools/providerCliResolution.ts
  -> provider/managed-tools/provider-cli-resolution/

apps/cli/src/runtime/managedTools/requireProviderCliLaunchSpec.ts
  -> provider/managed-tools/provider-cli-launch/

packages/cli-common/src/providers/install.ts
apps/cli/src/runtime/managedTools/invokeProviderCliInstall.ts
  -> provider/managed-tools/install-plans/ 和 provider/managed-tools/installers/

packages/cli-common/src/providers/managedJavaScriptRuntime.ts
packages/cli-common/src/providers/managedPnpm.ts
apps/cli/src/runtime/managedTools/pnpm/managedPnpm.ts
  -> provider/managed-tools/managed-runtimes/

packages/cli-common/src/providers/downloadGitHubReleaseAsset.ts
packages/cli-common/src/providers/extractGitHubReleaseAsset.ts
packages/cli-common/src/providers/codexRelease.ts
apps/cli/src/runtime/managedTools/providers/*Release.ts
  -> provider/managed-tools/release-assets/

apps/cli/src/capabilities/deps/gh.ts
apps/cli/src/capabilities/deps/codexAcp.ts
  -> provider/managed-tools/installers/、tool-status/ 和 updates/

packages/agents/src/providers/providerCliRuntime.ts
  -> provider/catalog/installables/、provider/<providerId>/manifest/ 和
     provider/managed-tools/contracts/
```

Codex release asset resolver、GitHub CLI installer、Codex ACP installer 这类 tool-specific adapter 可以归入 `provider/managed-tools/`，前提是它们是 installable/tool adapter。它们不能 import 具体 provider runtime implementation，也不能变成 provider 业务行为。

`provider/provider-families/` 已确认的三级结构：

```text
provider/provider-families/
  contracts/
  acp-runtime/
  built-in-acp/
  configured-acp/
  opencode-compatible/
  diagnostics/
  testkit/
```

`provider/provider-families/` 负责 provider family 的共享行为：多个 provider id 或动态 provider 机制因为共享协议、启动模式、transport profile 或 runtime pattern 而复用的代码。它不是第二套 catalog，不替代 `provider/cross-provider/`，也不能用来隐藏某个具体 provider 的实现。

判断规则：跨 provider 的通用契约、归一化和轻量原语进入 `provider/cross-provider/`；因为同一协议族、runtime family、transport profile 或启动模型而共享的实现进入 `provider/provider-families/`。

- `contracts/` 负责 family 层内部契约，例如 family factory 参数、family capability override、transport profile hook 和 family diagnostic result shape。它不能声明 provider catalog。
- `acp-runtime/` 负责共享 ACP 协议/runtime family，例如 ACP backend 创建、ACP runtime 构造、bridge/update/history 处理、permission mapping、spawn、stdout/stderr stream 处理和 transport extension points。provider-specific transport policy 仍归 provider-owned 目录，通过 family contract 接入。
- `built-in-acp/` 负责实现形态为 generic ACP 的内置 provider builder，例如 catalog-defined ACP entry/backend 构造。provider identity 和 public manifest output 仍归 `provider/<providerId>/manifest/`。
- `configured-acp/` 负责 account settings 或用户配置出来的 ACP backend materialization，包括动态 backend config、launch environment materialization、configured runtime 创建和 configured ACP CLI command delegation。产品层面的 `customAcp/` 仍是 provider facade，不是 family 实现本身。
- `opencode-compatible/` 负责 OpenCode-compatible family 行为，目前最明确的是 OpenCode 与 Kilo 共享的 native permission env/ruleset mapping。OpenCode server runtime、Kilo transport、provider-specific stderr parsing、auth、session metadata 和 UI behavior 仍归具体 provider。
- `diagnostics/` 负责 family 层诊断，例如 ACP handshake shape、transport profile resolution、factory inputs 和 family runtime lifecycle checks。具体 provider 的 auth/model/install 诊断仍归具体 provider 或 `provider/managed-tools/`。
- `testkit/` 负责 family 层 fixtures 和 harness，例如 fake ACP server、subprocess harness、family manifest fixtures 和 transport-profile test doubles。它不能成为 production dependency。

允许依赖：

```text
provider/<providerId>/acp -> provider/provider-families/acp-runtime
provider/<providerId>/manifest -> provider/provider-families/built-in-acp builders
provider/customAcp/settings -> provider/provider-families/configured-acp contracts
provider/opencode|kilo -> provider/provider-families/opencode-compatible
provider/provider-families/* -> provider/cross-provider/contracts
provider/provider-families/* -> provider/managed-tools public launch/resolution APIs
provider/catalog -> provider/<providerId>/manifest outputs
```

禁止依赖：

```text
provider/provider-families/* -> provider/<providerId> concrete implementation
provider/provider-families/* -> provider/catalog registry internals
provider/provider-families/* -> cli/happier-cli command routing
provider/provider-families/* -> app/client screens
provider/cross-provider/ -> provider/provider-families
provider/<providerId> -> provider/<otherProviderId>
```

当前代码来源映射建议：

```text
apps/cli/src/agent/acp/AcpBackend.ts
apps/cli/src/agent/acp/createAcpBackend.ts
apps/cli/src/agent/acp/runtime/createAcpRuntime.ts
apps/cli/src/agent/acp/bridge/**
apps/cli/src/agent/acp/updates/**
apps/cli/src/agent/acp/history/**
apps/cli/src/agent/acp/permissions/**
apps/cli/src/agent/acp/acpSpawn.ts
  -> provider/provider-families/acp-runtime/

apps/cli/src/agent/acp/catalog/createCatalogDefinedAcpEntry.ts
apps/cli/src/agent/acp/catalog/createCatalogDefinedAcpBackend.ts
apps/cli/src/agent/acp/catalog/createCatalogDefinedCliDetect.ts
apps/cli/src/agent/acp/catalog/auth/**
packages/agents/src/acp.ts
packages/protocol/src/providers/kiro/acpCatalog.ts
  -> shared/protocol/providers/kiro/，如果它保持为 wire/schema/API contract
  -> provider/provider-families/built-in-acp/，仅限分类后确认属于 family builder
     或 materialization behavior 的部分

apps/cli/src/agent/acp/catalog/configured/**
  -> provider/provider-families/configured-acp/

apps/cli/src/backends/openCodeFamily/**
  -> provider/provider-families/opencode-compatible/
```

设计说明：`provider/provider-families/` 应暴露供具体 provider 消费的 factory 和 hook，不能 import 具体 provider implementation。当前 catalog-defined ACP transport resolver 在物理迁移时应拆分：family 层定义 transport profile contract，Kiro 等 provider-owned manifest 或 hook 提供具体 transport implementation。

`provider/<providerId>/` 已确认的三级原则：

```text
provider/<providerId>/
  manifest/
  runtime/
  cli-adapter/
  auth/
  connected-services/
  daemon-hooks/
  sessions/
  local-control/
  mcp/
  acp/
  execution-runs/
  prompt-assets/
  models/
  settings/
  ui-adapter/
  protocol/
  diagnostics/
  testkit/
  <provider-owned-feature>/
```

这是一组能力菜单，不是强制模板。只有 `manifest/` 建议每个 provider 都存在。其他目录必须在 provider 实际拥有对应能力时才创建。Codex、Claude、OpenCode 这类复杂 provider 可以保留更深的 provider-owned structure；轻量 ACP/CLI provider 应保持小结构，不为了对齐而创建空壳目录。逐个 provider 的细粒度结构设计暂缓到整体物理骨架搭建后再展开。

允许依赖：

- `shared/`
- provider 内部支撑目录，例如 `provider/cross-provider/`、`provider/catalog/`、`provider/managed-tools/`、`provider/provider-families/`

允许消费者：

- `app/`
- `cli/`
- `daemon/`
- `service/`
- `tests/`

禁止依赖：

- Provider modules 不能依赖 app screens、CLI command routing、service concrete storage 或 test packages。
- Provider modules 不能彼此直接依赖。共享的 provider family 行为应放到 `provider/provider-families/` 或 `provider/cross-provider/`，而不是让一个 provider import 另一个 provider。

### `agents/`

负责 AI agents 和开发工具的指导材料。它是一级模块，并且有意与运行时 provider 代码分离。

预期范围：

- Codex 指导文档。
- Claude 指导文档。
- Cursor 指导文档。
- 共享 agent policies、prompts、operating procedures、tool-use guidance。
- 面向 agent/tool 消费的产品配置说明，但前提是这些内容不是运行时代码。

当前可能来源：

- `AGENTS.md`
- `CLAUDE.md`
- `.claude`
- `.cursor`
- `skills`
- 为 agent/tool 行为编写的开发者指导文档，而不是 application runtime behavior。

允许依赖：

- 可以通过文档概念性引用所有模块。
- 没有生产 import 依赖。

禁止依赖：

- 生产运行时代码不能 import `agents/`。
- `agents/` 不能成为 provider runtime adapters 的归属地。Provider runtime 属于 `provider/`。

### `shared/`

负责可复用的产品/运行时原语和跨模块契约。

预期范围：

- Protocol contracts。
- 不属于 tool guidance 的 agent runtime/domain primitives。
- Transfer protocols。
- Release runtime helpers。
- Connection supervision。
- Shared native modules。
- Shared schemas、catalogs、provider-agnostic domain logic。

当前可能来源：

- `packages/protocol`，包括 `packages/protocol/src/providers/**` 下的共享 provider
  protocol wire/schema/API contracts。
- `packages/agents` 中 provider-agnostic 的部分，目标结构中需要重命名，以避免和新的一级 `agents/` 指导模块冲突。
- `packages/transfers`
- `packages/release-runtime`
- `packages/connection-supervisor`
- `packages/audio-stream-native`
- `packages/sherpa-native`

当前 `packages/agents` 已确认的命名方向：

- 移到 `shared/agent-runtime` 或 `shared/agent-domain` 下。
- 如果该 package 主要暴露可执行的 agent runtime behavior，优先使用 `shared/agent-runtime`。
- 如果该 package 主要暴露 provider-agnostic domain models、settings、permissions、session-control concepts，优先使用 `shared/agent-domain`。
- 如果该 package 同时混合 provider-agnostic domain primitives 和 provider-specific definitions，不能整体移动，必须先按 owner 拆分再选择目标路径。

`shared/protocol/` 已确认的三级结构：

```text
shared/protocol/
  core/
  rpc/
  sessions/
  accounts/
  security/
  features/
  workspaces/
  source-control/
  mcp/
  actions/
  tools/
  prompts/
  providers/
  system-tasks/
  diagnostics/
  generated/
  testkit/
```

`shared/protocol/` 负责 app、service、CLI、daemon、provider 等模块共同消费的最低层 wire/schema/API contracts。它主要对应当前 `packages/protocol` workspace。目标结构按 protocol domain 切分，而不是按当前文件分布机械搬迁，方便开发者根据所消费的产品契约定位 owner。

- `core/` 负责基础协议原语，例如 identifiers、timestamps、result envelopes、pagination、generic error shapes 和小型共享 contract helpers。它不能变成归属不清的业务域兜底目录。
- `rpc/` 负责 transport-neutral RPC request/response contracts、socket RPC contracts、daemon/server RPC shapes 和共享 RPC error contracts。
- `sessions/` 负责 session、direct-session、session-control、session-message、session-metadata、session-authoring、fork、rollback、replay、execution-run 等 session 相关 protocol contracts。
- `accounts/` 负责 account、面向 auth 的 account state、pairing、profile、social/account relationship，以及不属于具体 service auth implementation 的 user-scoped protocol contracts。
- `security/` 负责协议层 auth、approval、crypto/encryption envelope、permission request shape 和 security policy contracts。Runtime crypto、secret storage、service/provider enforcement 仍归各自 owner。
- `features/` 负责 feature catalogs、feature decisions、capability payloads、compatibility gates 和共享 feature policy contracts。
- `workspaces/` 负责 workspace、machine、host、ownership、transfer 和 workspace location contracts。source-control 专属契约归 `source-control/`。
- `source-control/` 负责 SCM repository、branch、worktree、stash、pull request、path-scope、policy、capability 和 error-code contracts。
- `mcp/` 负责共享 MCP server、resource/tool registration、auth mode、selection、settings 和 transport-independent MCP normalization contracts。Provider-specific MCP client/spawn/config behavior 仍归 `provider/`。
- `actions/` 负责共享 action、automation、voice action、sent-from、checklist 和 user-intent contracts，前提是它们是协议层 records，而不是 UI 或 command implementation。
- `tools/` 负责共享 tool schema、tool naming、tool metadata、alias、sub-agent-family 和 tool-v2 protocol contracts。
- `prompts/` 负责 prompt、prompt-library、prompt-asset、structured-message 和 LLM-task protocol contracts，前提是它们是共享契约，而不是 provider prompt behavior。
- `providers/` 负责 provider wire/schema/API contracts、provider IDs，以及需要跨模块共享的 provider-facing protocol records。它不负责 provider runtime、install policy、managed-tool behavior 或 provider-specific executable defaults。
- `system-tasks/` 负责 system task schemas、task state、task execution protocol records 和 bootstrap/setup task contracts。具体 setup CLI flows 仍归 `cli/setup-cli/`。
- `diagnostics/` 负责 bug report、server diagnostics、capability diagnostics、health/status payloads 和协议层 diagnostic records。
- `generated/` 负责需要 check in 或作为 package artifact 消费的 generated protocol artifacts。人工维护源码不能放在这里。
- `testkit/` 负责 protocol fixtures、contract builders、compatibility fixtures 和 package-local test helpers。生产模块不能依赖它。

`shared/protocol/` 依赖规则：

```text
shared/protocol/<domain> -> shared/protocol/core
shared/protocol/<domain> -> 该 package 拥有的外部 schema/validation libraries
app/ cli/ daemon/ service/ provider/ -> shared/protocol public contracts
```

禁止依赖：

```text
shared/protocol/ -> app/
shared/protocol/ -> cli/
shared/protocol/ -> daemon/
shared/protocol/ -> service/
shared/protocol/ -> provider/
shared/protocol/ -> agents/
shared/protocol/ -> tests/
shared/protocol/ -> ops/
```

真实物理迁移时的目录准入规则：

- 不一次性创建所有目标空目录。只有当前 protocol 源码或已批准 contract move 有明确 owner 时，才创建对应目录。
- 一个 contract 只有在被多个一级模块消费，或属于稳定发布的 protocol surface 时，才进入 `shared/protocol/`。
- `packages/protocol/src/providers/**` 下的 provider material 必须先分类再迁移。共享 provider wire/schema/API contracts 留在 `shared/protocol/providers/`；executable provider policy、runtime defaults、installables、manifests、family adapter behavior 迁到 `provider/`。
- `core/` 只允许放多个 protocol domains 共同使用的窄原语。已经有明确 protocol domain 的业务概念必须放到对应 domain。
- `security/` 只允许放协议层 security contracts。Runtime enforcement、storage 或 cryptographic implementation 应归拥有它们的 service、app、daemon 或 provider module。
- `generated/` 和 `testkit/` 必须和生产 contract ownership 保持隔离。

`shared/agent-domain/` 已确认的三级结构：

```text
shared/agent-domain/
  identity/
  capabilities/
  permissions/
  modes/
  models/
  session-control/
  runtime-kinds/
  settings-contracts/
  connected-services/
  tools/
  voice/
  diagnostics/
  testkit/
```

`shared/agent-domain/` 负责 Happier 对 agent 的共享语义层。它定义 provider-agnostic agent concepts、capability models、permission semantics、session-control rules、runtime-kind semantics、model descriptor contracts 和 settings contract shapes。它不负责启动 agent、不负责安装 provider CLI、不负责 CLI command routing、不拥有 UI components，也不实现 provider-specific behavior。

- `identity/` 负责共享 agent identity 语义，例如 agent IDs、flavor aliases，以及从 session metadata 进行跨模块 agent ID 推断。它不是 provider registry；完整 provider registration、ordering、enablement 和 manifest aggregation 仍归 `provider/catalog/`。
- `capabilities/` 负责共享 agent capability models 和 evaluators，包括 resume、handoff、session storage、local control、session listing、fork、rollback 等 provider-agnostic capability surfaces。
- `permissions/` 负责 provider-agnostic permission intents 和 permission mode semantics，包括用户输入 alias 和 compatibility normalization。Provider-specific permission execution policy 仍归 provider-owned 目录。
- `modes/` 负责 provider-agnostic session mode 和 advanced mode semantics。
- `models/` 负责 model descriptor contracts、model-selection capability semantics、freeform model support、static-model contract shapes 和 model apply behavior。具体 provider model catalogs，例如 Claude、Codex、Gemini 的静态模型清单，应归 provider-owned model/catalog surface。
- `session-control/` 负责跨 provider 的 session-control semantics，例如 resume eligibility、handoff eligibility、existing-session automation eligibility、metadata override precedence、publish/normalization rules 和 monotonic update policies。Provider runtime descriptor fields 和 provider-specific metadata extras 仍归 provider-owned 目录。
- `runtime-kinds/` 负责 agent runtime kinds 的通用概念，以及 runtime kind 如何影响 capability surface。只对某个 provider 有意义的 runtime-kind 名称和 normalization rules 仍归该 provider。
- `settings-contracts/` 负责 provider settings definition shape、field metadata contract、registry contract 和共享 validation shape。具体 provider settings definitions、defaults、spawn extras 和 account-setting policy 仍归 `provider/<providerId>/settings/` 或拥有它们的 provider catalog surface。
- `connected-services/` 负责 connected-service compatibility、credential kind support 和 account/service compatibility checks 的共享语义。具体 provider-to-service support list 应由 provider manifests 或 catalog entries 提供。
- `tools/` 负责共享 agent tools capability semantics，例如 native MCP delivery、shell-bridge delivery、unsupported tools 和 support levels。MCP wire/schema contracts 仍归 `shared/protocol/mcp/` 或 `shared/protocol/tools/`；可执行 tool bridge runtime 归 `shared/agent-runtime/` 或 provider runtime。
- `voice/` 负责跨模块 voice-agent domain semantics，例如 transcript normalization 和共享 voice turn concepts。UI copy、prompt copy 和具体 voice runtime behavior 不放这里。
- `diagnostics/` 负责 agent-domain 层诊断，例如 invalid capability combinations、invalid mode/settings shapes、metadata parse failures 和 domain-level compatibility diagnostics。Provider CLI install/start failures 归 provider runtime 或 managed tools diagnostics。
- `testkit/` 负责 agent-domain fixtures、builders、fake capability surfaces 和 package-local test helpers。生产代码不能依赖它。

`shared/agent-domain/` 依赖规则：

```text
shared/agent-domain/ -> shared/protocol/
shared/agent-runtime/ -> shared/agent-domain/
provider/ -> shared/agent-domain/
app/ cli/ daemon/ service/ -> shared/agent-domain/
```

禁止依赖：

```text
shared/agent-domain/ -> app/
shared/agent-domain/ -> cli/
shared/agent-domain/ -> daemon/
shared/agent-domain/ -> service/
shared/agent-domain/ -> provider/
shared/agent-domain/ -> shared/agent-runtime/
shared/agent-domain/ -> tests/
shared/agent-domain/ -> ops/
```

当前代码来源分类建议：

- `packages/agents/src/types.ts`、`resolveAgentIdFromFlavor.ts`、`resolveAgentIdFromSessionMetadata.ts` 在拆出 provider catalog ownership 后，是 `identity/` 的候选来源。
- `tools.ts`、`localControl.ts`、`sessionControls/sessionCapabilities.ts` 是 `capabilities/` 的候选来源。
- `permissions/**` 是 `permissions/` 的候选来源。
- `sessionModes.ts` 和 `advancedModes.ts` 中 provider-agnostic 的部分，是 `modes/` 的候选来源。
- `models.ts` 中 model descriptor 和 model-selection contract 部分，是 `models/` 的候选来源；具体 provider model lists 和 provider-specific option enrichers 必须迁到 provider-owned model surfaces。
- `sessionControls/**` 下 provider-agnostic 的 vendor resume、handoff、automation eligibility、metadata precedence、publish 和 monotonic update policies，是 `session-control/` 的候选来源。
- `sessionControls/**` 下 provider-specific 的 Codex/OpenCode runtime descriptor extras、backend-mode resolution、runtime handles 和 provider-specific metadata parsing，不能进入 `agent-domain`，应迁到 provider-owned session/protocol areas。
- `runtimeKinds.ts` 中 generic runtime-kind types 和 capability override mechanics，是 `runtime-kinds/` 的候选来源；provider-specific runtime kind definitions 和 normalization 仍归 provider-owned 目录。
- `providerSettings/types.ts` 和 registry contract shape 是 `settings-contracts/` 的候选来源；`providerSettings/definitions/**` 下的具体 provider settings definitions 是 provider-owned。
- `providers/providerCliRuntime.ts` 和 `providers/providerCliInstallGuidance.ts` 不属于 `agent-domain`；它们属于 provider/managed-tools 或 provider/catalog 相关职责。
- `providers/**` 下的 provider-specific helpers，例如 Claude effort handling 或 Claude permission bridge request sources，必须保持 provider-owned，除非后续复核确认可以抽出真正 provider-agnostic primitive。

允许依赖：

- 其他 `shared/` 子模块，但依赖方向必须明确且无环。
- 外部库。

禁止依赖：

- 不依赖 `app/`、`service/`、`cli/`、`daemon/`、`provider/`、`agents/`、`tests/` 或 `ops/`。
- 当 provider hook 可以表达时，不在 generic shared code 中放 provider-specific executable policy。

`shared/mcp-domain/` 已确认的三级结构：

```text
shared/mcp-domain/
  server-catalog/
  bindings/
  session-selection/
  preview/
  detected-servers/
  auth/
  value-refs/
  tool-normalization/
  resource-contracts/
  diagnostics/
  testkit/
```

`shared/mcp-domain/` 负责 MCP 领域规则和稳定 MCP 语义。它定义 Happier-managed MCP server record 的含义，server bindings 和 session-level selections 如何解析，MCP preview 如何投影，provider-detected MCP server records 如何归一化成 provider-agnostic records，以及 MCP-specific tool/resource diagnostics 如何描述。它不启动 MCP servers、不拥有 stdio/HTTP bridge processes、不执行 provider-specific detection、不读取或解密 secret plaintext、不路由 CLI commands，也不直接注册 MCP SDK handlers。

- `server-catalog/` 负责 MCP server definition semantics，例如 server IDs、display names、transports、value-ref-bearing fields、enabled state 和 catalog-level validation。Protocol wire schemas 仍归 `shared/protocol/mcp/`；runtime materialized command records 仍归 `shared/agent-runtime/mcp-bridge/`、`daemon/` 或 `cli/`。
- `bindings/` 负责 all-machines、machine、workspace 等 scope 的 server binding precedence 和 override merge rules。它可以暴露纯 resolver functions，但 environment-specific path normalization 或 account-setting reads 必须作为输入注入。
- `session-selection/` 负责 per-session MCP enablement、forced include/exclude semantics、selected-server state、missing-server reason classification 和 managed-session selection rules。它不持久化 session metadata，也不直接访问 daemon/service storage。
- `preview/` 负责 provider-agnostic MCP preview projections，包括 managed、built-in、detected、unavailable 和 warning entries。CLI text output 和 app rendering 不归这里。
- `detected-servers/` 负责共享 detected-server record shape，以及通用 dedupe/filter/portability rules。Claude、Codex、OpenCode 或未来 provider 的具体 detection 仍归 provider-owned 目录。
- `auth/` 负责 MCP auth-mode classification、credential-required semantics 和 portability implications。它不解析或解密 credentials。
- `value-refs/` 负责 value reference classification、redaction-safe metadata 和 reference-shape helpers。Plaintext lookup、secret storage 和 decryption 仍归 runtime/security owner。
- `tool-normalization/` 负责 MCP tool name parsing、canonical MCP tool naming，以及 provider-agnostic MCP tool input/result display normalization。通用 agent tool capability semantics 仍归 `shared/agent-domain/tools/`。
- `resource-contracts/` 负责 Happier MCP resource identifiers、payload contracts 和 pure payload builders，例如 action-spec catalog resource contract。MCP SDK server registration 仍归 runtime/server owner。
- `diagnostics/` 负责 MCP-domain warning codes、reason codes、redaction-safe probe classifications、preview warnings 和 selection/materialization diagnostics。Provider-specific install/start failures 仍归 provider-owned。
- `testkit/` 负责 MCP-domain fixtures、catalog builders、binding builders、selection fixtures、preview builders 和 package-local test helpers。生产代码不能依赖它。

`shared/mcp-domain/` 依赖规则：

```text
shared/mcp-domain/ -> shared/protocol/
shared/agent-runtime/ -> shared/mcp-domain/
provider/ -> shared/mcp-domain/
app/ cli/ daemon/ service/ -> shared/mcp-domain/
```

禁止依赖：

```text
shared/mcp-domain/ -> app/
shared/mcp-domain/ -> cli/
shared/mcp-domain/ -> daemon/
shared/mcp-domain/ -> service/
shared/mcp-domain/ -> provider/
shared/mcp-domain/ -> shared/agent-runtime/
shared/mcp-domain/ -> agents/
shared/mcp-domain/ -> tests/
shared/mcp-domain/ -> ops/
```

当前代码来源分类建议：

- `packages/protocol/src/mcpServers/**` 下 versioned MCP wire schemas 仍是 `shared/protocol/mcp/` 的候选来源；pure binding、effective-server 和 managed-session selection resolvers 在拆分 schema ownership 后，可进入 `shared/mcp-domain/bindings/` 或 `shared/mcp-domain/session-selection/`。
- `apps/cli/src/mcp/preview/**` 只有在拆出 CLI output formatting 和 provider-specific detected-server mapping 后，才是 `shared/mcp-domain/preview/` 的候选来源。
- `apps/cli/src/mcp/providerDetection/**` 不迁入 `shared/mcp-domain/`；只有共享 detected-server record normalization 或 provider-agnostic dedupe rules 可以进入 `detected-servers/`。
- `apps/cli/src/mcp/resources/registerHappierMcpResources.ts` 应拆分：resource URI 和 pure payload contract 可以进入 `resource-contracts/`，MCP SDK registration 留在 runtime/server owner。
- 当前靠近 `apps/cli/src/agent/tools/normalization/families/mcp.ts` 的 MCP-specific tool name 和 result normalization，如果能脱离 CLI rendering，是 `tool-normalization/` 的候选来源。
- Runtime materialization、bridge launch、stdio/http server lifecycle、temporary runtime config files 和 plaintext value-ref resolution 不进入 `shared/mcp-domain/`。

物理迁移目录准入规则：

- 不一次性创建所有 `shared/mcp-domain/` 目录。只有当前源码或已批准移动内容有明确 owner 时，才创建目录。
- MCP 行为只有在 provider-agnostic、deterministic，且表达 domain rules 而不是 runtime side effects 时，才属于这里。
- 如果代码会启动进程、patch provider config、probe external server、解析 secret plaintext value，或命名具体 provider，应留在 `shared/mcp-domain/` 之外。
- 如果代码是被多个模块直接消费的 versioned wire/API schema，schema 留在 `shared/protocol/mcp/`，派生 domain policy 放在 `shared/mcp-domain/`。

`shared/agent-runtime/` 已确认的三级结构：

```text
shared/agent-runtime/
  core/
  session-lifecycle/
  turn-delivery/
  permission-flow/
  tool-delivery/
  mcp-bridge/
  local-control/
  execution-runs/
  voice-agent/
  process-io/
  state-sync/
  diagnostics/
  testkit/
```

`shared/agent-runtime/` 负责跨 provider 可复用的 agent 执行原语。它定义 agent runtime 如何被抽象成可启动、可接收 prompt、可取消、可同步状态、可接入工具的运行时对象。它不负责 provider 安装、provider CLI resolution、具体 provider protocol behavior、app UI、CLI command routing、daemon lifecycle internals 或 service persistence。

关键边界是：ACP runtime implementation 仍归 `provider/provider-families/acp-runtime/`。`shared/agent-runtime/` 只能承载可被 ACP、Claude SDK、OpenCode、voice agent、execution-run 或未来 runtime families 复用的 provider-agnostic primitives。因为共同 provider protocol family 产生的共享行为，仍然归 `provider/provider-families/`。

- `core/` 负责最小 runtime interfaces 和 types，例如 agent backend contracts、agent runtime handles、agent messages、session identifiers、tool call identifiers、backend factory contracts。它不包含 provider selection 或 provider-specific branching。
- `session-lifecycle/` 负责通用 start、load、resume、cancel、reset、dispose、cleanup 和 lifecycle state helpers。daemon report、terminal attachment persistence、service writes 等具体 startup side effects 留在各自 owner。
- `turn-delivery/` 负责 prompt delivery、in-flight steer、interrupt、cancellation、flush、response-completion waiting 和 abort-like error classification。它表达 turn mechanics，不硬编码 provider name。
- `permission-flow/` 负责 runtime permission request routing、permission queues、permission-mode runtime synchronization 和 pending permission cleanup。Permission semantics 仍在 `shared/agent-domain/permissions/`；ACP option mapping 仍在 `provider/provider-families/acp-runtime/permissions/`。
- `tool-delivery/` 负责 Happier tools 到 agent 的 runtime delivery，例如 native MCP delivery、shell-bridge delivery、unsupported delivery handling 和 runtime-side tool injection。Tool capability semantics 仍在 `shared/agent-domain/tools/`；provider-specific tool policy 仍由 provider 拥有。
- `mcp-bridge/` 负责 session-scoped MCP runtime primitives，例如 per-session MCP bridge/server lifecycle、session-agent HTTP/stdio bridge composition，以及 MCP servers 的 runtime materialization。MCP wire/schema/config contracts 仍在 `shared/protocol/mcp/` 或 `shared/mcp-domain/`。
- `local-control/` 负责 provider-agnostic local/remote control switching、provider-attach state publication、safe remote handoff state machines 和 local turn lifecycle snapshots。UI mounting、terminal rendering、daemon control 和具体 provider attach execution 留在各自 owner。
- `execution-runs/` 负责 bounded 和 long-lived execution runs 的 backend-agnostic substrate：run state、intent profile contracts、resume/send/stop/action flow 和 runtime manager primitives。CodeRabbit 等具体 engines 或 provider-specific execution-run factories 仍归 provider 或 feature owner。
- `voice-agent/` 负责 backend-agnostic voice-agent runtime orchestration，例如 voice agent manager state、chat/commit backend coordination、resume handles、idle reaping 和 stream-delta state。Voice semantics 归 `shared/agent-domain/voice/`；audio/native runtime modules 留在对应 shared native packages。
- `process-io/` 负责 provider-agnostic subprocess 和 stream primitives，例如 Node stream 到 Web Stream adapters、signal/termination helpers、process-output capture helpers、safe cross-platform IO utilities。Provider CLI resolution、installation、ACP spawn wrappers 仍在 provider-owned layers。
- `state-sync/` 负责 provider-agnostic 的 runtime synchronization helpers，例如 session metadata、agent state、runtime overrides、mode/model/config updates 和 startup metadata merge policy。具体 API writes、daemon reporting 和 storage implementation 不归这里。
- `diagnostics/` 负责 provider-agnostic runtime diagnostics，例如 runtime error classification、recoverability signals、redaction helpers、stderr summary helpers 和 debug artifact contracts。Provider-specific diagnostics 留在 provider。
- `testkit/` 负责 fake runtimes、fake backends、turn-delivery harnesses、MCP-bridge harnesses、lifecycle fixtures 和 package-local test helpers。生产代码不能依赖它。

允许依赖方向：

```text
shared/agent-runtime/ -> shared/protocol/
shared/agent-runtime/ -> shared/agent-domain/
shared/agent-runtime/ -> shared/mcp-domain/
provider/ -> shared/agent-runtime/
cli/ daemon/ app/ service/ -> shared/agent-runtime/
```

禁止依赖：

```text
shared/agent-runtime/ -> app/
shared/agent-runtime/ -> cli/
shared/agent-runtime/ -> daemon/
shared/agent-runtime/ -> service/
shared/agent-runtime/ -> provider/
shared/agent-runtime/ -> agents/
shared/agent-runtime/ -> tests/
shared/agent-runtime/ -> ops/
shared/agent-runtime/ -> provider/provider-families/acp-runtime/
shared/agent-runtime/ -> provider/managed-tools executable installers
```

当前代码来源分类建议：

- `apps/cli/src/agent/core/AgentBackend.ts` 及相关 backend/message factory types，在剥离 CLI-only 命名和 product-specific references 后，是 `core/` 的候选来源。
- 当前靠近 `apps/cli/src/agent/runtime/runStandardAcpProvider.ts`、`runPermissionModePromptLoop.ts`、`turnDelivery.ts` 的 provider-agnostic 部分，是 `session-lifecycle/`、`turn-delivery/`、`permission-flow/` 和 `state-sync/` 的候选来源。ACP-specific runner shape 和 Ink/CLI/API side effects 不能原样迁入。
- `apps/cli/src/agent/runtime/createHappierMcpBridge.ts`、`apps/cli/src/mcp/startHappyServer.ts`、`apps/cli/src/mcp/runtime/resolveRunnerMcpServers.ts` 只有在拆分 MCP domain contracts、credential materialization、packaged runtime resolution 和 CLI-specific launch behavior 后，才是 `mcp-bridge/` 的候选来源。
- `apps/cli/src/agent/localControl/**` 在拆出 provider attach execution、UI mounting、daemon control 和 API writes 后，是 `local-control/` 的候选来源。
- `apps/cli/src/agent/executionRuns/runtime/**` 是 `execution-runs/` 的候选来源；provider-specific execution-run factories、review engines 和 intent-specific product policies 不放入该模块。
- `apps/cli/src/agent/voice/agent/**` 在拆出 prompt content、voice-domain semantics 和 provider-specific backend creation 后，是 `voice-agent/` 的候选来源。
- `apps/cli/src/agent/acp/nodeToWebStreams.ts` 附近的 generic stream/process helpers 只有在 ACP 之外也复用时，才可能进入 `process-io/`。ACP spawn 和 ACP protocol lifecycle code 仍归 `provider/provider-families/acp-runtime/`。
- `packages/agents` 仍必须按 owner 拆分。Domain semantics 归 `shared/agent-domain/`；只有 provider-agnostic 且 executable 的 runtime primitives 才进入这里。

物理迁移目录准入规则：

- 不一次性创建所有 `shared/agent-runtime/` 目录。只有当前源码或已批准移动内容有明确 owner 时，才创建目录。
- 一个 primitive 只有在可被至少两个 runtime families 复用，或定义被多个一级模块消费的稳定 runtime contract 时，才属于 `shared/agent-runtime/`。
- 如果行为只是因为 providers 共享同一种 protocol family、launch model 或 transport profile，优先归 `provider/provider-families/`。
- 如果行为命名具体 provider、选择 provider defaults、解析 provider CLI、安装 provider tool、解释 provider-specific output，应留在 `provider/`。
- 如果行为控制 long-running daemon process 或持久化 daemon state，应留在 `daemon/`；必要时只向 `shared/agent-runtime/` 暴露窄 runtime contracts。

`shared/transfers/` 已确认的三级结构：

```text
shared/transfers/
  route-selection/
  availability/
  server-routed-policy/
  endpoint-fingerprints/
  route-viability-cache/
  diagnostics/
  testkit/
```

`shared/transfers/` 负责跨 app、CLI、daemon 和 service 复用的 provider-agnostic transfer routing 与 transfer policy decisions。它判断可以使用哪条 transfer route、当前是否可传输，以及 route 不可用的原因。它不负责真实 file IO、RPC handlers、HTTP 或 WebSocket stream implementations、service relay storage、direct-peer probing、UI copy、CLI commands、高层 session handoff orchestration 或 protocol wire schemas。

- `route-selection/` 负责纯 route negotiation，例如 `direct_peer`、`server_routed_stream` 和 `machine_rpc_direct` 之间如何选择。它基于 feature flags、调用方提供的 availability inputs 和 preferred strategy order 选择 route。它不探测 endpoint，也不执行真实传输。
- `availability/` 负责稳定的 transfer availability results 和机器可读 unavailable reason codes。UI 和 CLI 层可以把 reason codes 映射为产品文案，但 shared 层应避免拥有 user-facing output formatting。
- `server-routed-policy/` 负责 server-routed transfer feature policy、maximum byte limits 和 file-size limit checks。Process environment reads 应隔离在注入输入或窄 policy helper 后面，避免 core policy 变成非确定逻辑。
- `endpoint-fingerprints/` 负责安全的 transfer endpoint fingerprinting rules，包括在构造 cache fingerprints 之前移除不可信 URL userinfo、query 和 hash。
- `route-viability-cache/` 负责 in-memory route viability cache records、TTL policy、cache keys 和 invalidation rules。它不负责 network probing、persistent storage 或 connection lifecycle。
- `diagnostics/` 负责 transfer-domain reason codes、failure categories 和 redaction-safe diagnostic shapes。Service、daemon、CLI、app-specific error transport 不归这里。
- `testkit/` 负责 transfer fixtures、feature snapshots、endpoint candidates、route availability builders、cache records 和 package-local test helpers。生产代码不能依赖它。

`shared/transfers/` 依赖规则：

```text
shared/transfers/ -> shared/protocol/
app/ cli/ daemon/ service/ -> shared/transfers/
```

谨慎组合边界：

```text
app/ cli/ daemon/ service/ -> shared/connection-supervisor/
app/ cli/ daemon/ service/ -> shared/transfers/
```

`shared/transfers/` 和 `shared/connection-supervisor/` 通常应由消费它们的上层模块组合，而不是彼此直接依赖。Connection supervision 负责汇报 connection/endpoint state；transfer routing 基于传入状态判断 route policy。

禁止依赖：

```text
shared/transfers/ -> app/
shared/transfers/ -> cli/
shared/transfers/ -> daemon/
shared/transfers/ -> service/
shared/transfers/ -> provider/
shared/transfers/ -> shared/agent-runtime/
shared/transfers/ -> shared/connection-supervisor/
shared/transfers/ -> agents/
shared/transfers/ -> tests/
shared/transfers/ -> ops/
```

当前代码来源分类建议：

- `packages/transfers/src/route/resolveMachineTransferRoute.ts` 和 `packages/transfers/src/route/resolveAppSessionTransferRoute.ts` 是 `route-selection/` 的候选来源。
- `packages/transfers/src/route/resolveAppSessionTransferAvailability.ts` 是 `availability/` 的候选来源，但物理迁移时应复核其中 user-facing copy；如果属于展示文案而不是共享 diagnostic contract，应移动到 app/CLI 调用方。
- `packages/transfers/src/policy/serverRoutedTransferPolicy.ts` 是 `server-routed-policy/` 的候选来源。任何直接 `process.env` 使用都应保持为窄 helper，不能泄漏到 route-selection logic。
- `packages/transfers/src/cache/fingerprintTransferEndpoints.ts` 是 `endpoint-fingerprints/` 的候选来源。
- `packages/transfers/src/cache/createTransferRouteViabilityCache.ts` 和 `packages/transfers/src/cache/createMachineTransferRouteCache.ts` 是 `route-viability-cache/` 的候选来源。
- `packages/protocol` 中的 transfer stream envelopes、endpoint candidate schemas 和 feature payload schemas 仍归 `shared/protocol/`。
- `apps/ui/sources/sync/domains/transfers/**` 下的 UI runtime mapping，以及 CLI machine transfer wrappers 留在 app/CLI owner，作为 `shared/transfers/` 的消费者，不整体迁入。

物理迁移目录准入规则：

- 不一次性创建所有 `shared/transfers/` 目录。只有当前源码或已批准移动内容有明确 owner 时，才创建目录。
- 一个 primitive 只有在 deterministic、provider-agnostic，且能被多个一级模块复用时，才属于 `shared/transfers/`。
- 如果行为执行真实 IO、打开 streams、处理 RPC methods、probe direct peer、持久化 transfer state 或渲染 user-facing messages，应留在对应 app、CLI、daemon 或 service module。
- 如果 transfer shape 是 versioned wire/API schema，schema 留在 `shared/protocol/`，派生 transfer policy 放在 `shared/transfers/`。

`shared/connection-supervisor/` 已确认的三级结构：

```text
shared/connection-supervisor/
  state-model/
  transport-supervision/
  endpoint-supervision/
  readiness-contracts/
  retry-policy/
  request-supervision/
  diagnostics/
  testkit/
```

`shared/connection-supervisor/` 负责 provider-agnostic、platform-agnostic 的 connection supervision state machines。它监督 connection state、readiness probe outcomes、reconnect timing、status publication 和纯 request gating rules。它不负责 socket.io/WebSocket/fetch/axios implementations、CLI loopback probe implementations、UI token storage、React Native `AppState`、browser `window`/visibility listeners、endpoint supervisor pools、UI/CLI error classes、transfer route selection、daemon state persistence 或 service API endpoints。

- `state-model/` 负责 connection phases、reasons、state records、transport interfaces、readiness probe result types 和 supervisor public contracts。
- `transport-supervision/` 负责注入 transport 的 lifecycle supervision：connect、disconnect、error handling、listener cleanup、probe feedback 和 reconnect scheduling。具体 socket adapters 留在 app/CLI owner。
- `endpoint-supervision/` 负责 probe-only endpoint health supervision。它调用注入的 readiness probes 并发布 endpoint state，但不拥有 fetch/axios wrappers、token lookup、platform lifecycle listeners 或 endpoint supervisor pools。
- `readiness-contracts/` 负责 readiness result contracts，例如 ready、server-unreachable、auth-failed 和 retry-later。真实 readiness probe implementations 留在 CLI/UI/service owner。
- `retry-policy/` 负责默认 retry timing policy、fast retry behavior、exponential backoff、jitter normalization 和 retry delay calculation。
- `request-supervision/` 只负责纯 request gating 和 request outcome feedback helpers，前提是它们不绑定 concrete error classes 和 transport implementations。UI-specific `HappyError`、CLI HTTP error classes、`fetch`、`axios` 和 runtimeFetch wrappers 不放这里。
- `diagnostics/` 负责 event-to-reason derivation、connection failure categories、probe failure categories 和 redaction-safe diagnostic shapes。
- `testkit/` 负责 fake transports、fake supervisors、readiness probe fixtures、timer/backoff fixtures 和 package-local test helpers。生产代码不能依赖它。

`shared/connection-supervisor/` 依赖规则：

```text
app/ cli/ daemon/ service/ provider/ -> shared/connection-supervisor/
shared/agent-runtime/ -> shared/connection-supervisor/ only for generic runtime connection health
```

默认依赖立场：

```text
shared/connection-supervisor/ -> no shared/protocol dependency by default
```

当前 package 没有 production dependency 到 `shared/protocol/`，这是优点。除非未来复核确认需要引入稳定 wire/API schema，否则保持独立。

禁止依赖：

```text
shared/connection-supervisor/ -> app/
shared/connection-supervisor/ -> cli/
shared/connection-supervisor/ -> daemon/
shared/connection-supervisor/ -> service/
shared/connection-supervisor/ -> provider/
shared/connection-supervisor/ -> shared/transfers/
shared/connection-supervisor/ -> shared/agent-runtime/
shared/connection-supervisor/ -> agents/
shared/connection-supervisor/ -> tests/
shared/connection-supervisor/ -> ops/
```

与 `shared/transfers/` 的边界：

```text
connection-supervisor reports endpoint/connection state
transfers resolves transfer routes from supplied state and feature policy
app/cli/daemon performs the actual transfer
```

`shared/connection-supervisor/` 和 `shared/transfers/` 应由消费它们的上层模块组合，而不是彼此直接依赖。

主要影响范围：

- `packages/connection-supervisor` 是该模块的直接来源 package。
- `apps/cli/src/api/**` 是最大的消费面，包括 machine 和 session socket connection supervision、loopback readiness probes、supervised request handling 和 daemon connectivity coordination。
- `apps/ui/sources/sync/**` 是主要 app 消费面，包括 endpoint supervisor pools、endpoint readiness probes、sync socket transports、connectivity gating 和 connection-status display。
- `apps/stack`、`apps/cli` packaging、Dockerfile、CI 和 package artifact tests 会受到 internal workspace bundling path 影响。

当前代码来源分类建议：

- `packages/connection-supervisor/src/managedConnectionTypes.ts` 和 `managedEndpointSupervisorTypes.ts` 是 `state-model/` 和 `readiness-contracts/` 的候选来源。
- `packages/connection-supervisor/src/createManagedConnectionSupervisor.ts` 是 `transport-supervision/` 的候选来源。
- `packages/connection-supervisor/src/createManagedEndpointSupervisor.ts` 是 `endpoint-supervision/` 的候选来源。
- `packages/connection-supervisor/src/defaultManagedConnectionPolicy.ts` 和 `reconnectBackoff.ts` 是 `retry-policy/` 的候选来源。
- `packages/connection-supervisor/src/managedConnectionEvents.ts` 是 `diagnostics/` 的候选来源。
- `apps/cli/src/api/connection/requestSupervision/**` 下的 CLI request supervision helpers，以及 `apps/ui/sources/sync/runtime/connectivity/**` 下的 UI helpers，只有在剥离 concrete error types、fetch/axios/runtimeFetch、token lookup 和 UI/CLI-specific behavior 后，才可以进入 `request-supervision/`。
- `createLoopbackReadinessProbe`、`createEndpointReadinessProbe`、socket transport adapters、endpoint supervisor pools、app visibility listeners 和 token storage 留在各自 app/CLI owner。

物理迁移目录准入规则：

- 不一次性创建所有 `shared/connection-supervisor/` 目录。只有当前源码或已批准移动内容有明确 owner 时，才创建目录。
- 一个 primitive 只有在监督 connection 或 endpoint state，且不拥有具体 transport、platform lifecycle、request executor 或 product-specific error surface 时，才属于这里。
- 如果代码 import socket.io、React Native app state、browser globals、axios、runtimeFetch、token storage 或 CLI/UI error classes，应留在拥有它的 app/CLI module，或先拆出纯逻辑。
- 不从 `shared/connection-supervisor/` 依赖 `shared/transfers/`；由 consumer layer 把 connection state 传给 transfer policy。

`shared/release-runtime/` 已确认的三级结构：

```text
shared/release-runtime/
  release-rings/
  release-sources/
  asset-resolution/
  download-transport/
  integrity-verification/
  verified-downloads/
  extraction-plans/
  diagnostics/
  testkit/
```

`shared/release-runtime/` 负责读取 release metadata、选择 release assets、下载 release artifacts、校验完整性和规划 archive extraction 的共享 Node runtime 底座。它不是纯 release domain package：它包含 Node runtime behavior，例如 HTTP/HTTPS requests、向下载目录写文件、cryptographic signature verification、checksum validation 和 archive extraction command planning。

它不负责 release publishing、GitHub release creation、npm publishing、Docker builds、EAS/App Store publication、CLI self-update process replacement、Windows process quiesce、first-party component install layout、version promotion、rollback、shim synchronization、provider managed-tool install policy、relay/server business startup logic 或 shell command execution。

- `release-rings/` 负责 stable、preview、publicdev、internalpreview、internaldev release ring catalog entries、public labels、channel normalization，以及被 CLI、stack、daemon ownership 和 release pipelines 共享的 release-ring metadata。
- `release-sources/` 负责 release metadata source readers。当前主要来源是 GitHub release metadata by tag、latest release 和 first matching tag。它读取已发布 releases，不发布 releases。
- `asset-resolution/` 负责从 release metadata 中选择 release asset bundle，包括 product、version、OS、architecture、checksum、minisign 和 archive asset matching。
- `download-transport/` 负责 release download transport primitives，例如 Node HTTP/HTTPS GET、redirects、timeouts、data URL test support 和 cross-origin sensitive header stripping。它不能变成通用 network utility package。
- `integrity-verification/` 负责 checksum lookup、SHA-256 comparison support、minisign public key parsing、minisign signature parsing 和 Ed25519 signature verification。
- `verified-downloads/` 负责组合流程：下载 checksums、验证 minisign、下载 archive、验证 SHA-256，并把已验证 archive 写入 destination directory。它不负责 install 或 promote artifact。
- `extraction-plans/` 负责 archive extraction command planning，例如 `.tar.gz`/`.tar.xz` 使用 tar，Windows `.zip` 使用 PowerShell `Expand-Archive`。它只创建计划，不执行 shell commands。
- `diagnostics/` 负责 release-runtime stage names、error categories、status-aware error normalization 和 redaction-safe diagnostic shapes。CLI/UI/product presentation copy 留在调用方。
- `testkit/` 负责 fake release metadata、fake assets、checksum/minisign fixtures、HTTP fixtures、verified-download fixtures、extraction-plan fixtures 和 package-local test helpers。生产代码不能依赖它。

`shared/release-runtime/` 依赖规则：

```text
cli/ -> shared/release-runtime/
service/server-runner/ -> shared/release-runtime/
provider/managed-tools/ -> shared/release-runtime/
ops/ -> shared/release-runtime/
shared/first-party-runtime/ -> shared/release-runtime/
```

谨慎的前端/bootstrap 边界：

```text
app/ or bootstrap-like code -> shared/release-runtime/release-rings/ only
```

只有纯 release-ring metadata 可以被 app/bootstrap-like code 消费。`download-transport/`、`verified-downloads/` 和 `extraction-plans/` 是 Node runtime capabilities，不能进入 mobile/Web client runtime paths。

禁止依赖：

```text
shared/release-runtime/ -> app/
shared/release-runtime/ -> cli/
shared/release-runtime/ -> service/
shared/release-runtime/ -> provider/
shared/release-runtime/ -> ops/
shared/release-runtime/ -> shared/first-party-runtime/
shared/release-runtime/ -> tests/
```

主要影响范围：

- `packages/release-runtime` 是该模块的直接来源 package。
- `apps/cli` 是最大的消费面：CLI self-update、daemon/service ownership、relay runtime、doctor repair、provider dependency download 和 release channel handling。
- `packages/cli-common` 消费 release runtime，用于 first-party runtime payload preparation、install layout support、managed provider release asset extraction 和 component catalogs。
- `apps/stack` 消费 release runtime，用于 self-host runtime 和 companion CLI download/install flows。
- `packages/relay-server` 消费 release asset、checksum、minisign 和 extraction helpers，用于 server runner 相关 release assets。
- `scripts/pipeline` 消费 release ring/channel metadata 和 release helper logic，用于仓库运维。
- `apps/cli`、`apps/stack` 和 `packages/relay-server` packaging、Dockerfiles、CI 和 artifact tests 会受到 internal workspace bundling path 影响。

当前代码来源分类建议：

- `packages/release-runtime/src/releaseRings.ts` 是 `release-rings/` 的候选来源。
- `packages/release-runtime/src/github.ts` 是 `release-sources/` 的候选来源；GitHub publishing scripts 留在 `ops/`。
- `packages/release-runtime/src/assets.ts` 是 `asset-resolution/` 的候选来源。
- `packages/release-runtime/src/http.ts` 是 `download-transport/` 的候选来源，范围仅限 release downloads。
- `packages/release-runtime/src/checksums.ts` 和 `minisign.ts` 是 `integrity-verification/` 的候选来源。
- `packages/release-runtime/src/verifiedDownload.ts` 是 `verified-downloads/` 的候选来源。
- `packages/release-runtime/src/extractPlan.ts` 是 `extraction-plans/` 的候选来源。
- `packages/cli-common/src/firstPartyRuntime/**` 不能整体归入 `shared/release-runtime/`。它是 consumer，并且可能属于后续单独的 `shared/first-party-runtime/` 讨论，因为它负责 install layout、version promotion、rollback、shims 和 component lifecycle。
- CLI self-update 的文件替换和进程处理留在 `cli/`。
- Release publishing scripts 留在 `ops/`。

物理迁移目录准入规则：

- 不一次性创建所有 `shared/release-runtime/` 目录。只有当前源码或已批准移动内容有明确 owner 时，才创建目录。
- 一个 primitive 只有在读取 release metadata、解析 release assets、下载 release artifacts、校验完整性或规划 extraction，且不拥有 component installation 或 release publication 时，才属于这里。
- 如果行为发布 releases、安装产品组件、promote 或 rollback version、改写 shims、替换运行中的 CLI binary、注册 services、启动 relay/server business logic 或执行 extraction commands，应留在对应上层模块。
- 保持 Node runtime capabilities 不进入 mobile/Web runtime paths。只有 `release-rings/` 可以被视为 app/bootstrap-style consumers 安全消费的子面。

### `tests/`

负责验证资产。

预期范围：

- Unit、integration、end-to-end、provider、database contract、stress tests。
- 跨 package testkit 和 fixtures。
- 不属于通用仓库运维的 test runner scripts。

当前可能来源：

- `packages/tests`
- 当前嵌入各 workspace 的测试套件，视情况迁移。

允许依赖：

- 所有生产模块。

禁止依赖：

- 生产模块不能依赖 `tests/`。
- Test helpers 不能变成隐藏的运行时依赖。

### `ops/`

负责仓库运维、构建、发布、CI 和环境编排。

预期范围：

- CI workflows。
- Docker 和 Dagger assets。
- Release scripts。
- Build pipeline scripts。
- Repository-local orchestration scripts。
- Tooling validation scripts。

当前可能来源：

- `scripts`
- `.github`
- `docker`
- `dagger`
- `apps/stack` 的相关部分。
- 不属于产品配置的根级 operational configuration。

允许依赖：

- 可以通过命令或脚本调用和编排任意模块。

禁止依赖：

- 生产运行时模块不能 import `ops/`。
- `ops/` 不应拥有 application、provider 或 protocol 逻辑。

### `docs/`

负责人类可读文档和设计记录。

预期范围：

- 架构文档。
- 开发参考。
- 迁移计划。
- 面向人类读者的产品和 provider 文档。
- 本次重组的设计快照。

当前可能来源：

- `docs`
- `HAPPIER_DEVELOPMENT_REFERENCE.md`
- `HAPPIER_LOCAL_DEV_NOTES.md`
- 当前位于仓库根目录的 architecture/reference documents。

允许依赖：

- 文档可以引用任意模块。

禁止依赖：

- 没有生产运行时 import 依赖。
- 文档不能成为 machine-readable runtime config 的 source of truth。

## 依赖方向规则

预期依赖方向为：

```text
app/      cli/      daemon/      service/
  \        |          |            /
   \       |          |           /
    \      |          |          /
            provider/
                |
              shared/
```

支撑层：

```text
tests/  -> 可以依赖生产模块
ops/    -> 可以通过命令/脚本编排生产模块
docs/   -> 可以描述所有模块
agents/ -> 可以指导人/工具，但不是生产运行时输入
```

硬性规则：

- `shared/` 是最低层生产模块。
- `provider/` 依赖 `shared/`，不依赖 app/CLI/service 实现。
- `app/`、`cli/`、`daemon/`、`service/` 可以消费 `provider/` 和 `shared/`。
- `agents/` 与 `provider/` 不是同一概念。
- `tests/`、`ops/`、`docs/`、`agents/` 不允许成为生产运行时依赖根。

## 二级目录设计范围

下一步是定义每个一级模块下的二级目录。该步骤应当：

- 尽量保留当前 package ownership。
- 除非有明确所有权理由，否则避免把一个 package 拆到多个无关一级模块。
- 在物理移动前识别 import aliases 和 workspace package names。
- 只有在绝对必要时定义临时 migration aliases。
- 优先直接更新 imports，而不是留下长期 compatibility shims。
- 保持 provider-specific executable behavior 位于 provider-owned folders。
- 保持 AI-agent guidance 位于 `agents/`，而不是 provider runtime folders。

## 未来物理移动的安全规则

移动文件前：

- 盘点所有 workspace package names 和 import aliases。
- 基于当前 imports 构建依赖图。
- 决定 workspace package names 是否随路径改变，还是保持稳定。
- 一次只移动一个 module group。
- 每移动一组后，运行最小相关 typecheck/test lane。
- 全部移动后，运行 root workspace install validation、typecheck、unit tests，以及相关 provider/test lanes。
- 除非明确要求删除，否则保留现有 untracked local development files。

本文档只是已确认一级模块契约的快照，不是实施计划。
