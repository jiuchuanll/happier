# provider/ 模块重组说明

日期：2026-05-14
状态：已确认二级结构、`provider/catalog/`、`provider/cross-provider/`、`provider/managed-tools/`、`provider/provider-families/` 三级结构和 `provider/<providerId>/` 三级原则；尚未授权物理迁移

## 职责定位

`provider/` 负责外部 agent/provider 系统的生产运行时集成。它收拢 Codex、Claude、OpenCode、Gemini、Auggie、Qwen、Kimi、Kilo、Kiro、Pi、Copilot、customAcp 等 provider 的 runtime adapter、CLI backend、provider capability、provider settings、provider-owned protocol adapter/projection、UI behavior adapter、fixtures 和 provider-local tests。共享 protocol wire/schema/API contracts 仍归 `shared/protocol`。

`provider/` 不是 CLI command layer，不负责 `happier`、`hsetup`、`hstack` 的命令路由；也不是 app screen owner、service storage owner、daemon process owner 或 agent 指导文档目录。

## 已确认二级结构

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

## 二级目录职责

`catalog/` 负责 provider registry 和派生产物。它聚合 provider entries、settings registries、UI/provider registry surfaces、provider id/schema catalogs。它是目录和派生产物 owner，不拥有具体 provider runtime implementation。

`cross-provider/` 负责多个 provider 共同需要的 provider-domain 原语，例如 hook types、provider capability helpers、model capability normalization、direct-session/fork hook contracts、provider-agnostic adapter utilities。它是 `provider/` 内部的跨 provider 公共能力层，不是仓库一级 `shared/`。

`managed-tools/` 负责 provider CLI resolution 和 installation support，包括 managed Node/PNPM runtime helpers、provider CLI path resolution、vendor recipe planning、managed package installation、GitHub release download/extraction，以及 binary-safe runtime guards。

`provider-families/` 负责相关 provider family 的共享行为，例如 ACP runtime family、catalog-defined ACP providers、configured ACP providers 和 OpenCode-compatible providers。它用于避免一个具体 provider 直接 import 另一个具体 provider，也避免把只适用于某个协议族的行为塞进 `provider/cross-provider/`。

具体 provider 目录，例如 `codex/`、`claude/`、`opencode/`、`customAcp/`，负责该 provider 的 runtime、CLI adapter、auth、cloud connect、direct-session、MCP、settings、UI adapter、provider-owned protocol adapter/projection、fixtures 和 provider-local tests。共享 wire/schema/API contracts 仍归 `shared/protocol`。`customAcp/` 先保留 camelCase，以匹配当前持久化 `AgentId`，避免额外 migration alias。

## `provider/catalog/` 三级结构

`provider/catalog/` 是连接各 provider 的总入口，但它必须保持为薄 catalog layer。它只认识 provider 的公开声明和公开 hook，不包含单个 agent/provider 的特殊 runtime 接入方式。

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

三级目录职责：

- `entry-contracts/` 定义 catalog entry、manifest entry、hook 类型、capability 声明结构。它只定义 provider 如何向 catalog 声明能力，不写 provider 行为。
- `registry/` 是总入口，聚合 `provider/<providerId>/manifest/` 暴露的 entry，提供 `requireCatalogEntry`、`listProviderEntries`、`resolveProviderEntry` 等通用查询。它不能直接 import `provider/<providerId>/runtime/`、`cli-adapter/`、`auth/` 等实现目录。
- `identifiers/` 管理 provider id、默认 provider、alias、CLI subcommand 映射、持久化 id 兼容规则。这里可以枚举 provider 名称，但只能作为身份和路由事实，不能写 provider-specific 行为分支。
- `capability-index/` 聚合 provider 能力矩阵，例如 direct session、native fork、ACP、cloud connect、models probe、vendor resume、runtime installables 等。能力来源必须来自 manifest 声明或 hook presence，不允许在这里写 provider 特判。
- `hook-resolvers/` 提供统一 lazy hook 访问层，例如 direct-session ops、provider-native fork handler、preflight session controls probe adapter。它负责缓存、缺失 hook 错误和通用 fallback；具体 hook 实现仍在 `provider/<providerId>/` 内。
- `settings-index/` 聚合 provider settings registry。具体 provider 的设置定义放在 `provider/<providerId>/manifest/` 或 provider 自己的 `settings/`，这里只负责构建统一 registry、校验字段冲突、输出给 UI/CLI。
- `backend-targets/` 管理 backend target 的通用解析、target ref schema、flavor/profile 索引。provider-specific profile 数据由 provider manifest 声明，catalog 只做统一索引和查询。
- `installables/` 声明 provider runtime installable key、安装能力索引、provider 与 installable 的关系。真正的下载、安装、Node/PNPM、GitHub release、binary-safe 逻辑属于 `provider/managed-tools/`。
- `ui-projections/` 输出 UI 可消费的纯数据投影，例如 agent picker、provider details、resume UI flags、connected-services config、local-control config。它不放 React screen，也不写 provider 特殊 UI 逻辑；provider 特殊 UI 行为归 `provider/<providerId>/ui-adapter/`。
- `derived-artifacts/` 放由 registry 派生出的稳定产物，例如协议导出、CLI/UI projection builder、兼容快照。原则是从 manifest/catalog 生成，不维护第二套手写 registry。
- `validation/` 放 catalog 一致性校验，包括 id 唯一、alias 不冲突、hook 与 capability 声明一致、settings key 不冲突、installable key 有 owner。
- `testkit/` 放 catalog 测试 fixtures、manifest builder、假 provider entry。它服务 catalog 自身测试，不作为生产 runtime 依赖。

`provider/catalog/` 允许依赖：

```text
provider/catalog/registry       -> provider/<providerId>/manifest/
provider/catalog/hook-resolvers -> provider/catalog/registry
provider/catalog/*              -> provider/catalog/entry-contracts
provider/catalog/*              -> provider/cross-provider/contracts
```

允许消费者：

```text
app/client -> provider/catalog/ui-projections
cli/*      -> provider/catalog/registry or provider/catalog/hook-resolvers
daemon/    -> provider/catalog/hook-resolvers
service/   -> provider/catalog/identifiers or provider/catalog/entry-contracts only
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

设计结论：`hook-resolvers/` 留在 `provider/catalog/`，不放入 `provider/cross-provider/`，因为它依赖 registry 并构成 provider 总入口能力。`settings-index/`、`ui-projections/`、`backend-targets/` 保持拆分，避免 UI、settings 和 target 解析重新混在一个 projection 目录中。

## `provider/cross-provider/` 三级结构

`provider/cross-provider/` 是 provider 域内部的跨 provider 公共能力层。它服务于 `provider/catalog/`、`provider/<providerId>/`、`provider/provider-families/` 和 `provider/managed-tools/`，但自身不能变成 catalog，不能包含具体 provider 实现，也不能吸收 CLI/UI 的产品界面逻辑。

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

三级目录职责：

- `contracts/` 定义 provider 共享类型契约、hook shape 和基础 capability 声明原语。它只定义 provider 共享能力的形状，不负责 provider 注册、查询或 hook 解析。
- `direct-sessions/` 负责跨 provider direct-session 操作契约、transcript/page/candidate/activity 结构，以及 provider-agnostic environment merge helper。具体 provider 的 direct-session 实现仍在 `provider/<providerId>/` 内。
- `forking/` 负责 provider fork 契约，例如 ACP continuation handler、provider-native fork point 和 fork dispatch result shape。它描述 fork 能力，不执行具体 provider 的 fork 流程。
- `session-metadata/` 负责 provider session metadata 的通用更新、merge、provider session id metadata helper。
- `session-capabilities/` 负责 provider-agnostic session capability 归一化，例如 resume、attach、follow、fork、direct-session 支持结果。UI 展示选项和 CLI 命令体验仍归各自消费层。
- `execution-runs/` 负责 provider-agnostic execution-run adapter 原语和简单 backend factory。它不能变成 CLI command routing，也不能放具体 provider runtime。
- `model-capabilities/` 负责模型能力归一化，例如 context window token parsing 和稳定 model capability shape。
- `cli-adapter-primitives/` 负责 provider CLI adapter 可复用的小型基础件，例如通用 terminal display factory。深度依赖 CLI command routing 的内容不放这里，应归 `cli/`。
- `ui-adapter-contracts/` 只放 data-only 的 UI adapter 契约，例如 provider settings、local auth、install banner 和 UI-facing provider behavior 声明。React component、screen、theme 和 provider-specific UI 行为仍归 `app/client` 或 `provider/<providerId>/ui-adapter/`。
- `permissions/` 负责 provider 共享 permission request source、permission capability contract 和权限归一化 helper。具体 provider policy 仍在 provider-owned 目录内。
- `diagnostics/` 负责标准化 provider diagnostics、warnings、capability mismatch report 和 runtime diagnostic result shape。这里输出结构化诊断结果，不负责 UI 渲染。
- `testkit/` 负责 provider-shared fixtures、typed builders、fake provider hooks 和 contract test helpers。它不能成为 production runtime dependency。

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

当前代码来源映射：

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

设计结论：`provider/cross-provider/` 只承接两个以上 provider 共同需要的 provider-domain 原语。`hook-resolvers/` 留在 `provider/catalog/`，具体 runtime 留在 `provider/<providerId>/`，provider CLI 安装/解析执行逻辑留在 `provider/managed-tools/`。这可以避免重新形成 `shared/utils` 式兜底目录。

## `provider/managed-tools/` 三级结构

`provider/managed-tools/` 负责 provider 相关外部工具的解析、安装、托管运行时支撑、状态、更新检查和 binary-safe 启动保障。它不是 provider runtime，不是 catalog，也不是 CLI command surface。它的职责是把 catalog/provider manifest 中声明的 installable 转换为可解析、可安装、可启动、可诊断、可更新的真实工具。

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

三级目录职责：

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

当前代码来源映射：

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

设计结论：`provider/managed-tools/` 使用生命周期为主、技术栈为辅的划分方式，避免把 install declaration、path resolution、launch wrapping、install execution、managed runtime、tool status 和 background update 混在一起。`tool-status/` 用于避免泛化为所有状态目录，`updates/` 用于隔离版本检查和后台更新副作用。

## `provider/provider-families/` 三级结构

`provider/provider-families/` 是 provider family 共享行为层。它承接多个 provider id 或动态 provider 机制因为共享协议、启动模式、transport profile 或 runtime pattern 而复用的代码。它不是第二套 catalog，不替代 `provider/cross-provider/`，也不能成为具体 provider 实现的隐藏目录。

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

三级目录职责：

- `contracts/` 负责 family 层内部契约，例如 family factory 参数、family capability override、transport profile hook 和 family diagnostic result shape。它必须限制在 family 内部，不能声明 provider catalog。
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

当前代码来源映射：

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

设计结论：`provider/provider-families/` 应暴露供具体 provider 消费的 factory 和 hook，不能 import 具体 provider implementation。当前 catalog-defined ACP transport resolver 在物理迁移时应拆分：family 层定义 transport profile contract，Kiro 等 provider-owned manifest 或 hook 提供具体 transport implementation。只有两个以上 provider 或动态 provider 机制共享的接入模式/协议族实现，才进入 `provider/provider-families/`。

## `provider/<providerId>/` 三级原则

具体 provider 目录采用“最小共同骨架 + 可选能力目录 + provider 自有扩展目录”的方式。它不是所有 provider 必须长得一样的强制模板。

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

阶段性规则：

- `manifest/` 是唯一建议每个 provider 都具备的目录。它是 `provider/catalog/` 聚合 provider 能力和 hook 的入口。
- 其他目录只有在 provider 实际拥有对应能力时才创建，不创建空目录来追求外观一致。
- Codex、Claude、OpenCode 这类复杂 provider 可以保留更深的 provider-owned 子结构，不能为了抽象统一而压平或混合其 runtime 模式。
- Gemini、Pi 这类中等能力 provider 应保留其实际存在的 cloud、models、preflight、rpc 等差异能力，不强行套复杂 provider 的完整结构。
- Auggie、Qwen、Kimi、Kilo、Kiro、Copilot 这类轻量 ACP/CLI provider 应保持小结构，主要保留实际存在的 `manifest/`、`acp/`、`cli-adapter/`、`execution-runs/`、`ui-adapter/` 等能力目录。
- `customAcp/` 是配置型 provider，主要围绕 catalog-defined ACP 配置、settings 和 UI adapter，不应伪装成完整 runtime provider。
- 不使用 `utils/` 作为兜底目录。迁移时应按真实职责拆到 `runtime/`、`sessions/`、`auth/`、`mcp/`、`diagnostics/` 或 provider 自有 feature 目录。
- 两个以上 provider 共享的逻辑才允许上提到 `provider/provider-families/` 或 `provider/cross-provider/`；不要让具体 provider 之间互相 import。
- 跨 provider 的通用契约、归一化、轻量原语进入 `provider/cross-provider/`；因为同一协议族、runtime family、transport profile 或启动模型而共享的实现进入 `provider/provider-families/`。不要仅凭“两个以上 provider 使用”就默认放进 `cross-provider/`。

逐个 provider 的细粒度目录，例如 Codex 的 `app-server`/`acp`、Claude 的 `local-cli`/`remote`/`sdk`、OpenCode 的 `server`/`acp`，暂缓到整体物理项目骨架搭建后再逐项细化和实现。

## 当前代码来源映射

```text
apps/cli/src/backends/catalog.ts
apps/cli/src/backends/types.ts
apps/ui/sources/agents/registry/**
apps/ui/sources/agents/backendCatalog/**
apps/ui/sources/agents/providers/registry/**
packages/agents/src/providerSettings/** registry/index portions
packages/protocol/src/providers/agentProviderIdsV1.ts
cross-provider provider schema/catalog files
  -> shared/protocol/providers/ for wire/schema/identifier contracts
  -> provider/catalog/identifiers, settings-index, or derived-artifacts only
     for catalog-owned projections after classification

packages/agents/src/providerSettings/**
  -> provider/catalog/settings-index/ for registry aggregation
  -> provider/<providerId>/settings/ or manifest/ for provider-specific definitions
  -> shared/agent-domain/ for provider-agnostic setting/domain types

packages/cli-common/src/providers/**
packages/protocol/src/providers/github/** installable/tool declaration portions
  -> provider/managed-tools/

apps/cli/src/backends/directSessions/**
apps/cli/src/backends/forking/**
apps/cli/src/backends/modelCapabilities/**
shared provider helper files under apps/cli/src/backends/shared/**
  -> provider/cross-provider/

apps/cli/src/backends/openCodeFamily/**
apps/cli/src/agent/acp/catalog/**
catalog-defined ACP provider helper code
  -> provider/provider-families/

apps/cli/src/backends/<providerId>/**
apps/ui/sources/agents/providers/<providerId>/**
packages/agents/src/providers/<providerId>/**
  -> provider/<providerId>/

packages/protocol/src/providers/<providerId>/**
  -> shared/protocol/providers/<providerId>/ for wire/schema/API contracts
  -> provider/<providerId>/, provider/catalog/, provider/managed-tools/, or
     provider/provider-families/ only after classifying executable policy,
     manifest/installable data, or family adapter behavior
```

来源拆分规则：

- `packages/protocol` 不能按目录名整体搬入 `provider/`。provider-specific protocol schema、wire type、public API contract 仍属于 `shared/protocol`；只有 provider runtime policy、manifest/installable 声明、family adapter 等非纯协议契约内容，才按 owner 进入 `provider/`。
- `packages/agents` 也不能整体搬入 `provider/` 或 `shared/agent-domain`。provider-agnostic domain/runtime primitives 进入 `shared/agent-domain` 或 `shared/agent-runtime`；provider-specific settings、permission source、runtime/install metadata、provider manifest 候选内容进入 `provider/` 对应子目录。
- provider 目录落地时执行“有真实能力才建目录”。除 `manifest/` 外，不为轻量 provider 创建空的 `runtime/`、`mcp/`、`auth/`、`sessions/` 等目录来保持外观一致。

## 依赖边界

允许方向：

```text
app/client -> provider/catalog or provider/<providerId>/ui-adapter
cli/*      -> provider/catalog or provider/<providerId>/cli-adapter
daemon/    -> provider/catalog runtime hooks
service/   -> provider public contracts only
provider/  -> shared/
provider/<providerId>/ -> provider/cross-provider, provider/managed-tools, provider/provider-families, provider/catalog, shared/
provider/provider-families/ -> provider/cross-provider, provider/managed-tools public APIs, shared/
```

禁止方向：

```text
provider/ -> app/client screens
provider/ -> cli command routing
provider/ -> daemon implementation internals
provider/ -> service concrete storage/API implementation
provider/<providerId>/ -> provider/<otherProviderId>/
shared/ -> provider/
agents/ -> provider production runtime
```

Provider-specific executable behavior 必须留在 provider-owned folders。共享 provider family 行为应进入 `provider/provider-families/` 或 `provider/cross-provider/`，不要通过具体 provider 之间互相 import 复用。

## 后续讨论重点

`provider/catalog/`、`provider/cross-provider/`、`provider/managed-tools/`、`provider/provider-families/` 和 `provider/<providerId>/` 已确认三级结构或三级原则；逐个 provider 的更深层结构暂缓到整体物理项目骨架搭建后再讨论。

后续更适合进入 provider 模块整体复核、真实物理骨架落位方案，或转向下一个一级模块继续讨论。各具体 provider 的细粒度实现边界应等骨架稳定后再展开，避免过度抽象。
