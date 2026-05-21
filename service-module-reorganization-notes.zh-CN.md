# service/ 模块重组归纳说明

日期：2026-05-11
状态：`service/` 一级模块下二级、三级目录设计已确认，作为未来物理目录迁移前的设计依据

## 目的

本文档按一级模块维度归纳已经讨论确认的 `service/` 目录职责、目标结构、原项目来源、内部依赖方向和跨模块边界。

它是 `module-structure-reorganization-design.md` 和 `module-structure-reorganization-design.zh-CN.md` 的补充材料。主设计文档记录整体结构；本文档保留 `service/` 子任务的细节，避免后续上下文压缩后丢失已经确认的设计。

## 一级职责边界

`service/` 负责服务器侧可运行服务和服务分发运行器。它不等同于当前所有 server-adjacent 代码，也不应收容所有协议、传输、release runtime 或本地 daemon 管理逻辑。

`service/` 的核心边界是：

- `api-server/`：真正的 Happier 后端服务实现。
- `server-runner/`：面向发布/安装后的轻量服务启动器，用于获得、校验、解压并启动已发布的 server binary。

协议类型、传输工具、release 下载/校验通用原语、本地 daemon 生命周期、自托管编排和发布流水线应分别归入 `shared/`、`daemon/` 或 `ops/`，不能因为它们和 server 相关就全部放入 `service/`。

## 已确认的二级结构

```text
service/
  api-server/
  server-runner/
```

- `api-server/`：后端服务实现，主要来自当前 `apps/server`。
- `server-runner/`：服务发布物运行器，主要来自当前 `packages/relay-server`。

## `service/api-server/`

### 当前项目来源

`service/api-server/` 对应当前 `apps/server`。当前来源包括：

- `apps/server/sources/**`。
- `apps/server/prisma/**`。
- `apps/server/generated/**`。
- `apps/server/scripts/**`。
- `apps/server/deploy/**`。
- `apps/server/.env.example`、`.env.dev`。
- `apps/server` 内 package、TypeScript、Vitest、README 等配置。

它是 backend service implementation，而不是 CLI、daemon、provider 或 release pipeline。

### 目标三级结构

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

### 职责说明

| 目录 | 职责 | 说明 |
| --- | --- | --- |
| `entrypoints/` | 服务进程入口 | full/light server main、不同启动入口。 |
| `runtime/` | 服务生命周期编排 | 启动、关闭、角色选择、运行时组合，不承载具体业务。 |
| `http/` | HTTP API 层 | Fastify routes、registration、request/response、validation、rate limit、auth guards。 |
| `realtime/` | 实时通信层 | Socket.IO、rooms、event routing、socket auth、realtime transport。presence 状态本身归 domain。 |
| `domains/` | 服务端业务领域 | sessions、account、sharing、presence、automations、artifacts、activity、feed、kv、social、feature flags、retention、pets、changes 等。 |
| `auth/` | 服务端认证授权 | pairing、terminal auth、OAuth、keyless、account auth、identity policy。 |
| `storage/` | 数据访问和持久化基础设施 | DB wrappers、blob/files、Redis、queues、locks、cache、sequence、private files。不得依赖 HTTP route detail。 |
| `workers/` | 后台任务和 worker 角色 | long-running loops、scheduled work、`SERVER_ROLE=worker` 行为。算法和业务规则应在 domains。 |
| `integrations/` | 服务端外部集成 | GitHub webhooks、Tailscale inference、ElevenLabs/voice 等 server-side integrations。 |
| `observability/` | 可观测性 | metrics、Sentry、diagnostics、logging。 |
| `config/` | 服务配置解析 | env/config parsing、backend selection、resolved config。 |
| `flavors/` | 服务运行变体 | full/light runtime flavors、SQLite/PGlite/local file defaults。 |
| `database/` | 数据库 schema 和迁移 | Prisma schema、provider variants、migrations、schema sync。 |
| `generated/` | 生成产物 | Prisma generated clients 等生成输出。 |
| `testkit/` | server-local 测试辅助 | 只服务测试，不作为生产依赖。 |
| `tooling/` | server-local 工具脚本 | dev、migration、schema sync、generated client、runtime build、validation scripts。 |
| `deployment/` | 服务专属部署描述 | service-specific deploy descriptors，不承载跨模块 release pipeline。 |

### 内部依赖方向

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

### 边界规则

- `storage/` 不依赖 HTTP route detail。
- `domains/` 不依赖 HTTP 或 Socket transport detail。
- `testkit/` 不进入 production dependency。
- `tooling/` 和 `deployment/` 不承载跨模块 release/deploy pipeline。
- provider-specific policy 应归 `provider/`，除非是明确的 server-side provider hook。

## `service/server-runner/`

### 当前项目来源

`service/server-runner/` 对应当前 `packages/relay-server`。它是发布给用户使用的轻量 server runner/distribution wrapper，不是后端服务本体。

它的职责是把已发布的服务制品和用户本地运行环境连接起来：

```text
parse invocation
-> resolve OS/arch target
-> find release artifact
-> download
-> checksum/minisign verification
-> extract/cache
-> optional UI web bundle handling
-> start real Happier server binary
```

### 目标三级结构

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

### 职责说明

| 目录 | 职责 | 说明 |
| --- | --- | --- |
| `entrypoints/` | runner 命令入口 | `happier-server`、历史 `relay-server` 等入口。 |
| `runtime/` | runner 主流程 | 解析调用、解析 target、定位 release asset、下载、校验、解压、缓存、spawn server binary。 |
| `invocation/` | 命令调用模型 | channel、server tag、UI tag、UI bundle、pass-through args 等。 |
| `targets/` | 平台目标解析 | OS/CPU/exe name/cache root，Windows/macOS/Linux/x64/arm64 规则。 |
| `release-assets/` | 发布物选择 | 从 GitHub release metadata 选择 server binary 和可选 UI web artifact。 |
| `verification/` | runner-facing 校验边界 | checksum/minisign 使用边界；通用实现应在 `shared/release-runtime`。 |
| `assets/` | runner 打包资产 | 例如 `happier-release.pub`。 |
| `tooling/` | runner-local 打包工具 | prepack、package helper 等 runner 本地工具。 |

测试可以继续随实现共置；当前不强制单独设置 `tests/` 目录。

### 内部依赖方向

```text
entrypoints -> runtime
runtime -> invocation / targets / release-assets / verification / assets
release-assets -> shared/release-runtime
verification -> shared/release-runtime
tooling -> shared/release-runtime / workspace bundling helpers
```

### 边界规则

- `service/server-runner` 不直接 import `service/api-server` 源码。
- runner 不承载 server business logic。
- download、checksum、minisign、extract 等通用能力归 `shared/release-runtime`，runner 只消费。
- `shared/release-runtime` 不反向依赖 runner。
- runner-local tooling 不等于跨模块 release/deploy pipeline；后者归 `ops/`。

## 明确不归入 `service/` 的内容

| 当前内容或能力 | 推荐目标归属 | 原因 |
| --- | --- | --- |
| `packages/protocol` | `shared/protocol` | 共享协议和 schema，不是 server 私有实现。 |
| `packages/transfers` | `shared/transfers` | 跨模块传输能力，应保持 provider/service/client 可共享。 |
| `packages/connection-supervisor` | `shared/connection-supervisor` | 连接监督原语不是 api-server 私有能力。 |
| `packages/release-runtime` | `shared/release-runtime` | release 下载、校验、解压原语被 runner、CLI、ops 等消费。 |
| `apps/stack` 的服务生命周期、自托管、本地服务管理 | `daemon/` 或 `ops/` | 命令 facade 之外的生命周期和编排不属于 api-server 实现。 |
| `apps/bootstrap` | `cli/setup-cli` | setup/bootstrap 是命令入口，不是服务模块。 |

## 跨模块关系

```text
ops/ builds and publishes server release artifacts
shared/release-runtime/ provides reusable download, verification, and extract primitives
service/server-runner/ consumes release-runtime and starts the server binary
service/api-server/ is the backend service started by the runner
daemon/ or stack/self-host tooling may call the runner to manage a local service
app/client/ connects to the running api-server
```

## 后续迁移风险

- `apps/server` 内 schema、generated client、runtime、routes、workers、deployment 跨目录关联强，未来移动必须先建立 import alias 和 package boundary 方案。
- `packages/relay-server` 当前可能依赖 release-runtime 和 workspace bundling 逻辑，迁移时必须保留 npm package 发布行为。
- `service/api-server` 与 `service/server-runner` 名字相近但职责完全不同，不能把 runner 当成 server implementation。
- `apps/stack` 中 service-adjacent 能力需要单独拆分，不能因为它能启动 server 就并入 `service/`。
