# shared/ 模块重组归纳说明

日期：2026-05-18
状态：`shared/` 一级模块讨论进行中。`protocol/`、`agent-domain/`、`mcp-domain/`、`agent-runtime/`、`transfers/`、`connection-supervisor/` 和 `release-runtime/` 的三级结构已确认；尚未授权物理迁移。

## 目的

本文档记录 `shared/` 模块已经确认的设计方案，是 `module-structure-reorganization-design.md` 和 `module-structure-reorganization-design.zh-CN.md` 的补充材料。

主设计文档仍是整体架构入口。本文档把 `shared/` 子任务决策集中保存，避免后续上下文压缩后丢失，也便于在真实物理目录迁移前单独复核。

## 一级职责边界

`shared/` 负责可复用的产品/运行时原语和跨模块 contracts。它是目标架构中最低层的生产模块。App、CLI、daemon、service 和 provider 可以依赖 `shared/`，但 `shared/` 不能反向依赖这些上层模块。

`shared/` 应包含稳定 contracts、provider-agnostic domain rules、可复用 runtime primitives、transfer routing policy、connection supervision、release runtime helpers 和 shared native packages。它不能变成“多个地方 import 过就放进来”的兜底目录。

## 已确认的二级模块

```text
shared/
  protocol/
  agent-domain/
  mcp-domain/
  agent-runtime/
  transfers/
  connection-supervisor/
  release-runtime/
```

## 待继续讨论的 shared 模块

以下当前来源已经识别为可能属于 `shared/`，但还需要按同样标准继续讨论边界和三级结构：

- `packages/audio-stream-native` -> 可能归 shared native runtime module
- `packages/sherpa-native` -> 可能归 shared native runtime module
- 前面 CLI 和 service 讨论中提到的 reusable system task 或 first-party runtime primitives

## `shared/protocol/`

已确认三级结构：

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

`shared/protocol/` 负责 app、service、CLI、daemon 和 provider 共同消费的最低层 wire/schema/API contracts。它主要对应当前 `packages/protocol` workspace。它不负责 runtime enforcement、provider executable policy、service persistence 或 UI behavior。

依赖规则：

```text
shared/protocol/<domain> -> shared/protocol/core
app/ cli/ daemon/ service/ provider/ -> shared/protocol/
```

禁止方向：

```text
shared/protocol/ -> app/ cli/ daemon/ service/ provider/ agents/ tests/ ops/
```

## `shared/agent-domain/`

已确认三级结构：

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

`shared/agent-domain/` 负责 Happier 对 agent 的共享语义层。它定义 provider-agnostic agent identity、capability models、permission semantics、session-control rules、runtime-kind semantics、model descriptor contracts 和 settings contract shapes。

它不启动 agent、不安装 provider CLI、不路由 CLI commands、不拥有 UI components，也不实现 provider-specific behavior。具体 provider settings、provider model lists、executable provider policy 和 provider-specific metadata parsing 仍归 provider-owned 目录。

依赖规则：

```text
shared/agent-domain/ -> shared/protocol/
shared/agent-runtime/ -> shared/agent-domain/
provider/ -> shared/agent-domain/
app/ cli/ daemon/ service/ -> shared/agent-domain/
```

禁止方向：

```text
shared/agent-domain/ -> app/ cli/ daemon/ service/ provider/
shared/agent-domain/ -> shared/agent-runtime/
shared/agent-domain/ -> agents/ tests/ ops/
```

## `shared/mcp-domain/`

已确认三级结构：

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

`shared/mcp-domain/` 负责 MCP 领域规则和稳定 MCP 语义。它定义 Happier-managed MCP server record 的含义，server bindings 和 session-level selections 如何解析，MCP preview 如何投影，provider-detected MCP server records 如何归一化成 provider-agnostic records，以及 MCP-specific tool/resource diagnostics 如何描述。

它不启动 MCP servers、不拥有 stdio/HTTP bridge processes、不执行 provider-specific detection、不读取或解密 secret plaintext、不路由 CLI commands，也不直接注册 MCP SDK handlers。

依赖规则：

```text
shared/mcp-domain/ -> shared/protocol/
shared/agent-runtime/ -> shared/mcp-domain/
provider/ -> shared/mcp-domain/
app/ cli/ daemon/ service/ -> shared/mcp-domain/
```

禁止方向：

```text
shared/mcp-domain/ -> app/ cli/ daemon/ service/ provider/
shared/mcp-domain/ -> shared/agent-runtime/
shared/mcp-domain/ -> agents/ tests/ ops/
```

## `shared/agent-runtime/`

已确认三级结构：

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

`shared/agent-runtime/` 负责跨 provider 可复用的 agent 执行原语。它定义 agent runtime 如何被抽象成可启动、可接收 prompt、可取消、可同步状态、可接入工具的运行时对象。

它不负责 provider 安装、provider CLI resolution、具体 provider protocol behavior、app UI、CLI command routing、daemon lifecycle internals 或 service persistence。ACP runtime implementation 仍归 `provider/provider-families/acp-runtime/`；如果共享行为只是因为多个 providers 使用同一种 provider protocol family，也仍归 `provider/provider-families/`。

依赖规则：

```text
shared/agent-runtime/ -> shared/protocol/
shared/agent-runtime/ -> shared/agent-domain/
shared/agent-runtime/ -> shared/mcp-domain/
provider/ -> shared/agent-runtime/
cli/ daemon/ app/ service/ -> shared/agent-runtime/
```

禁止方向：

```text
shared/agent-runtime/ -> app/ cli/ daemon/ service/ provider/
shared/agent-runtime/ -> agents/ tests/ ops/
shared/agent-runtime/ -> provider/provider-families/acp-runtime/
shared/agent-runtime/ -> provider/managed-tools executable installers
```

## `shared/transfers/`

已确认三级结构：

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

`shared/transfers/` 负责跨 app、CLI、daemon 和 service 复用的 provider-agnostic transfer routing 与 transfer policy decisions。它判断可以使用哪条 transfer route、当前是否可传输，以及 route 不可用的原因。

它不负责真实 file IO、RPC handlers、HTTP 或 WebSocket stream implementations、service relay storage、direct-peer probing、UI copy、CLI commands、高层 session handoff orchestration 或 protocol wire schemas。

依赖规则：

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

禁止方向：

```text
shared/transfers/ -> app/ cli/ daemon/ service/ provider/
shared/transfers/ -> shared/agent-runtime/
shared/transfers/ -> shared/connection-supervisor/
shared/transfers/ -> agents/ tests/ ops/
```

## `shared/connection-supervisor/`

已确认三级结构：

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

`shared/connection-supervisor/` 负责 provider-agnostic、platform-agnostic 的 connection supervision state machines。它监督 connection state、readiness probe outcomes、reconnect timing、status publication 和纯 request gating rules。

它不负责 socket.io/WebSocket/fetch/axios implementations、CLI loopback probe implementations、UI token storage、React Native `AppState`、browser `window`/visibility listeners、endpoint supervisor pools、UI/CLI error classes、transfer route selection、daemon state persistence 或 service API endpoints。

- `state-model/` 负责 connection phases、reasons、state records、transport interfaces、readiness probe result types 和 supervisor public contracts。
- `transport-supervision/` 负责注入 transport 的 lifecycle supervision：connect、disconnect、error handling、listener cleanup、probe feedback 和 reconnect scheduling。具体 socket adapters 留在 app/CLI owner。
- `endpoint-supervision/` 负责 probe-only endpoint health supervision。它调用注入的 readiness probes 并发布 endpoint state，但不拥有 fetch/axios wrappers、token lookup、platform lifecycle listeners 或 endpoint supervisor pools。
- `readiness-contracts/` 负责 readiness result contracts，例如 ready、server-unreachable、auth-failed 和 retry-later。真实 readiness probe implementations 留在 CLI/UI/service owner。
- `retry-policy/` 负责默认 retry timing policy、fast retry behavior、exponential backoff、jitter normalization 和 retry delay calculation。
- `request-supervision/` 只负责纯 request gating 和 request outcome feedback helpers，前提是它们不绑定 concrete error classes 和 transport implementations。UI-specific `HappyError`、CLI HTTP error classes、`fetch`、`axios` 和 runtimeFetch wrappers 不放这里。
- `diagnostics/` 负责 event-to-reason derivation、connection failure categories、probe failure categories 和 redaction-safe diagnostic shapes。
- `testkit/` 负责 fake transports、fake supervisors、readiness probe fixtures、timer/backoff fixtures 和 package-local test helpers。生产代码不能依赖它。

依赖规则：

```text
app/ cli/ daemon/ service/ provider/ -> shared/connection-supervisor/
shared/agent-runtime/ -> shared/connection-supervisor/ only for generic runtime connection health
```

默认依赖立场：

```text
shared/connection-supervisor/ -> no shared/protocol dependency by default
```

当前 package 没有 production dependency 到 `shared/protocol/`，这是优点。除非未来复核确认需要引入稳定 wire/API schema，否则保持独立。

禁止方向：

```text
shared/connection-supervisor/ -> app/ cli/ daemon/ service/ provider/
shared/connection-supervisor/ -> shared/transfers/
shared/connection-supervisor/ -> shared/agent-runtime/
shared/connection-supervisor/ -> agents/ tests/ ops/
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
- `apps/cli/src/api/**` 是最大的消费面，包括 machine/session socket connection supervision、loopback readiness probes、supervised request handling 和 daemon connectivity coordination。
- `apps/ui/sources/sync/**` 是主要 app 消费面，包括 endpoint supervisor pools、endpoint readiness probes、sync socket transports、connectivity gating 和 connection-status display。
- `apps/stack`、`apps/cli` packaging、Dockerfile、CI 和 package artifact tests 会受到 internal workspace bundling path 影响。

## `shared/release-runtime/`

已确认三级结构：

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

依赖规则：

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

禁止方向：

```text
shared/release-runtime/ -> app/ cli/ service/ provider/ ops/
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

## 当前来源映射

| 当前来源 | 目标方向 |
| --- | --- |
| `packages/protocol` | `shared/protocol/` |
| `packages/agents` 中 provider-agnostic 部分 | 在 `shared/agent-domain/` 和 `shared/agent-runtime/` 之间拆分 |
| 当前靠近 `packages/protocol/src/mcpServers/**` 和 `apps/cli/src/mcp/**` 的 MCP domain rules | 在 `shared/protocol/mcp/`、`shared/mcp-domain/` 和 runtime owners 之间拆分 |
| `packages/transfers` | `shared/transfers/` |
| `packages/connection-supervisor` | `shared/connection-supervisor/` |
| `packages/release-runtime` | `shared/release-runtime/` |
| `packages/audio-stream-native` | 待讨论 shared native module |
| `packages/sherpa-native` | 待讨论 shared native module |

## 全局依赖规则

允许：

```text
app/ cli/ daemon/ service/ provider/ -> shared/
shared/<higher reusable runtime> -> shared/<lower contract/domain layer>
```

禁止：

```text
shared/ -> app/
shared/ -> cli/
shared/ -> daemon/
shared/ -> service/
shared/ -> provider/
shared/ -> agents/
shared/ -> tests/
shared/ -> ops/
```

在 `shared/` 内部，依赖方向必须明确且无环。Protocol contracts 应保持最低层。Domain policy 可以依赖 protocol contracts。Reusable runtime primitives 可以依赖 protocol 和 domain modules。任何 shared module 都不能 import 上层 owner 的实现细节。

## 物理迁移规则

- 不一次性创建所有目标目录。只有当前源码或已批准移动内容有明确 owner 时，才创建目录。
- 如果当前 package 混合了 domain、runtime、provider-specific、protocol 和 presentation concerns，不能整包直接移动，必须先按 owner 拆分。
- 如果代码命名具体 provider、选择 provider defaults、解析或安装 provider CLIs、patch provider config，或解释 provider-specific output，应保留在 `provider/`。
- 如果代码启动 long-running daemon processes 或持久化 daemon state，应保留在 `daemon/`，必要时只向 `shared/` 暴露窄 reusable contracts。
- 如果代码是跨模块消费的 versioned wire/API schema，应保留在 `shared/protocol/`，派生 policy 放入对应 domain module。
- 生产模块不能依赖 `testkit/` 目录。

## 后续讨论顺序

建议后续 `shared/` 讨论顺序：

1. shared native runtime modules
2. reusable system task / first-party runtime primitives
