# CLI 边界重组设计说明

日期：2026-05-11
状态：CLI 一级职责、二级目录、三级目录和关键四级边界已复核确认；尚未授权物理迁移

## 目的

本文档记录当前 Happier 仓库中 CLI 相关代码的职责边界混杂点、已确认的 `cli/` 二级目录结构，以及未来物理目录重组时应如何收拢到 `cli/`、`daemon/`、`provider/`、`service/`、`shared/`、`ops/` 等模块。

它是 `module-structure-reorganization-design.md` 和 `module-structure-reorganization-design.zh-CN.md` 的补充材料。主设计文档记录已经确认的整体模块结构；本文档记录 `cli/` 的细化设计，以及为什么 CLI 不能按当前 `apps/cli` 和 `packages/cli-common` 原样搬迁。

## 一级职责边界

`cli/` 的一级职责是用户命令入口层和命令行运行支撑层。它负责把用户输入的命令转换为对 `daemon/`、`provider/`、`service/`、`shared/` 的调用，但不长期拥有这些模块的核心实现。

`cli/` 应关注：

- 用户能直接运行的 bin entrypoints。
- command parser、command router、command registry。
- 用户可见命令的 help、参数、输出、交互和错误格式化。
- CLI 作为 client 调用 daemon、service、provider、shared contract 的 orchestration。
- 多个 CLI 产品面共享、且只对命令行执行有意义的 command runtime。

`cli/` 不应成为 provider runtime、daemon 长驻进程、service/api-server、release pipeline 或 app/client 构建逻辑的长期容器。

## 当前观察

当前 CLI 相关代码主要来自：

- `apps/cli`
- `packages/cli-common`
- `apps/bootstrap`
- `apps/stack` 中面向用户的命令入口部分

其中 `apps/cli` 同时包含：

- `happier`、`happier-dev`、`happier-mcp` 命令入口。
- 命令解析、命令路由、输出格式、CLI runtime。
- session、auth、settings、machines、diagnostics、doctor、installables 等命令面。
- `src/backends/**` provider 后端执行实现。
- `src/daemon/**` 本地 daemon 长驻进程和生命周期实现。
- `src/mcp/**` MCP bridge、launcher 和远程/本地 MCP 连接能力。
- `src/server/**` 服务器可达性和 server 相关 CLI 辅助。
- `src/runtime/**`、`src/subprocess/**`、`src/terminal/**` 等本地运行支撑。
- `scripts/**` build、prepack、package-dist、tool tracing、postinstall、release-it 相关脚本。

`packages/cli-common` 同时包含：

- CLI/stack 共享输出、链接、路径、进程、workspace helper。
- first-party runtime、service helper、relay host、tailscale、system tasks。
- provider 相关 helper。
- component artifacts、update、workspaces、release/runtime-adjacent helper。

这说明当前目录结构是按历史实现演进组织的，不完全等同于目标模块所有权。未来重组必须拆出真实职责，不能把当前 `apps/cli` 整体等价为目标 `cli/`。

## 已确认的 `cli/` 二级模块

已确认把 `cli/` 一级模块划分为：

```text
cli/
  happier-cli/
  setup-cli/
  stack-cli/
  command-runtime/
```

`happier-cli/` 是 Happier 主 CLI 产品面，承载 `happier`、`happier-dev`、`happier-mcp` 这些用户入口。它包含 command registry、参数解析、help、JSON/TTY 输出、auth/session/server/service/mcp/daemon/provider 相关命令入口，但只保留命令层编排。

`setup-cli/` 承载当前 `apps/bootstrap` 的 `hsetup` 能力。它是 setup/bootstrap CLI 产品面，适合放 system task 执行入口、remote SSH bootstrap、setup-this-computer、安装/配对流程的命令面。通用 system task schema 仍应在 `shared/protocol` 或 `shared/system-tasks`，setup-cli 只负责 CLI 执行入口和交互流程。

`stack-cli/` 承载当前 `apps/stack` 中真正面向用户的 `hstack` 命令 facade，例如 stack new/list/start/stop、stack auth、worktree、service passthrough、local dev 命令入口。`apps/stack` 中的 build、release、mobile、self-host、service lifecycle、daemon control、local orchestration 后续要分别归 `ops/`、`app/client`、`daemon/` 或 `service/`；`stack-cli/` 只保留命令 facade 和 stack-local 用户体验。

`command-runtime/` 是三个 CLI 产品面共享的命令行运行支撑。它放命令解析模型、CommandContext、输出格式、TTY/JSON 渲染、prompt helpers、CLI-only path/env helper、bin wrapper helper、错误格式化、只对 CLI 命令执行有意义的 network/proxy helper、Windows console hardening 等。该命名刻意指向 command execution，避免和 `shared/first-party-runtime`、managed runtime、daemon runtime 混淆。具体 auto-update notice、runtime re-exec、self-update command wrapping 归 `happier-cli/cli-lifecycle/`，不归 `command-runtime/`。

## CLI 总体复核结论

复核日期：2026-05-14。

复核对象包括当前已写入文档的 `happier-cli/`、`setup-cli/`、`stack-cli/`、`command-runtime/` 结构，以及现有 `apps/cli`、`apps/bootstrap`、`apps/stack`、`packages/cli-common` 的真实源码形态。

结论：当前 `cli/` 目录结构可以收口，不需要新增第五个二级目录。四个二级目录分别覆盖主 CLI 产品面、setup/bootstrap 产品面、stack-local 产品面和 CLI-only command runtime。这个划分能表达 CLI 在项目中的连接作用，同时不会把 provider、daemon、service、app、ops 或 shared runtime 的实现所有权吞进 CLI。

复核后确认的硬性规则：

- `cli/` 是用户命令入口层和命令行运行支撑层，不是 provider runtime、daemon 长驻进程、service/api-server、release pipeline、app/client 构建逻辑或 shared runtime primitive 的长期容器。
- `happier-cli/module-adapters/` 是主 CLI 普通命令到 owner module 的依赖闸口。
- `setup-cli/task-adapters/` 是 `hsetup` 到 daemon、service、provider 和一级 `shared/` 能力的依赖闸口。
- `stack-cli/*-commands/` 只表达 `hstack` 命令 facade，不表达底层实现 owner。
- `command-runtime/` 只放 CLI-only command execution support，不能扩张成 generic shared utils、first-party runtime、managed tools、daemon runtime 或 provider runtime。
- `daemon/`、`service/`、`app/`、`provider/`、`ops/`、`shared/` 不应反向依赖具体 CLI 产品面。

当前仍需在物理迁移前补齐的是 source inventory 和 package boundary 计划，而不是继续扩展 `cli/` 目标目录。特别是当前 `apps/cli/src` 中还有大量不应直接归入目标 `cli/` 的源码族：

| 当前源码族 | 迁移前判断 | 目标倾向 |
| --- | --- | --- |
| `apps/cli/src/backends/**` | provider-specific executable behavior | `provider/` |
| `apps/cli/src/daemon/**` | 本地 daemon 长驻进程和生命周期实现 | `daemon/` |
| `apps/cli/src/agent/**` | agent domain/runtime 能力，需要区分 CLI surface 和共享 domain | `shared/agent-runtime`、`shared/agent-domain`、`provider/` 或 `cli/happier-cli` |
| `apps/cli/src/api/**` | client API、server API helper 或协议适配可能混杂 | `shared/protocol`、`service/`、`daemon/` 或 CLI adapter |
| `apps/cli/src/scm/**` | source-control/worktree/hosting provider 能力 | `shared/source-control`、`ops/` 或 owner module |
| `apps/cli/src/workspaces/**` | workspace replication、关系、传输和状态能力 | `shared/workspace-domain`、`daemon/` 或 `ops/` |
| `apps/cli/src/transfers/**` | transfer domain、download、policy、targets | `shared/transfers` 或 owner module |
| `apps/cli/src/pets/**` | product/domain 能力，不应因历史位置默认归 CLI | 需按功能归 `shared/`、`daemon/` 或产品 owner |
| `apps/cli/src/promptAssets/**`、`apps/cli/src/promptRegistries/**` | agent 指导文档、prompt asset/registry 适配 | `agents/`、`provider/`、`shared/agent-domain` 或 CLI command facade |
| `apps/cli/src/capabilities/**` | capability registry/probe/system task 混合 | `shared/capabilities`、`provider/`、`daemon/` 或 CLI diagnostics facade |
| `apps/cli/src/cloud/**`、`apps/cli/src/installables/**` | connected services、install source-of-truth 可能与 provider/runtime 混杂 | `provider/`、`shared/first-party-runtime`、`service/` 或 CLI facade |

真实物理迁移前还必须单独处理 package 边界。`apps/cli` 和 `apps/stack` 不是普通源码目录，它们同时是 published package 或可执行 package，包含 `bin`、`exports`、`files`、bundled workspace dependencies、prepack/bundling scripts、`dist`、`package-dist` 等发布契约。迁移源码时必须先设计兼容路径或一次性更新所有引用，避免破坏 npm 包入口、MCP bridge exports、bundled workspace dependency closure 和二进制安全运行时约束。

## `setup-cli/` 已确认的三级目录

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

三级目录职责：

- `bin-entrypoints/`：负责 `hsetup` 可执行入口包装和启动 fallback。
- `command-router/`：负责 hsetup 命令解析和分发，包括 `hsetup system-tasks run [--spec-json <json>]`。
- `system-task-runner/`：负责 system task 执行的 CLI 壳：stdin 或 flag 输入、`SystemTaskSpec` 解析、task id 生成、取消信号、registry 执行、JSONL event/result 输出和 exit code 映射。
- `interactive-io/`：负责 prompt IO、abortable readline、敏感字段 redaction 后的 event 输出和交互答案解析。
- `task-registry/`：负责 hsetup task-kind 注册和依赖组装；可以组合 owner 暴露的 handler，但不能把 daemon、relay、provider、shared runtime 的实现吞进来。
- `setup-workflows/`：负责 setup 产品旅程，例如 `setup.thisComputer.v1`。
- `task-adapters/`：负责通过 public command 或模块 contract 薄适配 auth、server selection、daemon service install/start/status、relay configuration 等能力。
- `remote-bootstrap/`：负责 remote SSH bootstrap 的命令面编排和 host-trust 交互；可复用 SSH/path/archive/task primitive 应归 shared。
- `secure-access/`：负责 setup-facing secure access 流程，例如 Tailscale setup task；通用 Tailscale command runner、状态解析、安装策略如被多方复用，应归 shared 或 secure-access owner。
- `packaging/`：负责 hsetup-local binary build 和 dist wrapper 生成；跨产品 release、installer、publish pipeline 归 `ops/packaging` 或 `ops/release`。
- `help/`：负责 hsetup usage/help。
- `testkit/`：负责 setup-cli 专属 fake IO、registry fixture 和 task-runner 测试 helper。

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

## `stack-cli/` 已确认的三级目录

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

`*-commands/` 目录只表达命令 facade，不表达实现 owner。`stack-runtime-commands/` 只承载 `start/dev/stop/logs/tui` 命令面；`worktree-commands/` 只承载 `hstack wt ...`；`component-commands/` 只承载 build/lint/typecheck/test/mobile/eas 命令面；`host-service-commands/` 只承载 daemon/service/tailscale/self/self-host 命令面；`remote-setup-commands/` 只承载 remote setup 命令面；`provider-commands/` 只承载 `hstack providers ...`；`maintainer-commands/` 只承载 setup-from-source、contrib、PR review/setup、monorepo、import、migrate、CI、pack 等维护者命令面。底层实现分别归 `app/client/`、`daemon/`、`service/`、`provider/`、`ops/` 或 `shared/`。

依赖方向：

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

禁止方向：

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

这四个目录描述的是命令入口和命令运行支撑，不代表当前源码可以不拆分地整体搬入。

## `command-runtime/` 已确认的三级目录

`command-runtime/` 的职责是“多个 CLI 产品面共享的命令执行支撑”，不是 CLI utils 垃圾桶。它被 `happier-cli/`、`setup-cli/`、`stack-cli/` 消费，但不能反向依赖这些产品面的具体 commands。

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

`context/` 负责共享 command execution context、stdio/TTY metadata、command invocation metadata。具体产品 command registry 仍归产品面，例如 `happier-cli/routing/`。

`argv/` 负责共享 argv parsing 和 normalization 原语，例如 flag readers、special command parsing helpers、generic argument shapes。产品级 routing 仍归 `happier-cli/`、`setup-cli/` 或 `stack-cli/`。

`output/` 负责共享 JSON envelope、TTY/JSON rendering helper、progress output、table/list rendering、稳定命令输出原语。某个命令领域专属输出模型可以放在该命令领域内部，例如 `happier-cli/commands/sessions/output/`。

`errors/` 负责 command-level error normalization、exit-code mapping、用户可读错误包装、debug-output policy。领域错误由 owner module 定义；这里只做 CLI 展示适配。

`prompts/` 负责共享 CLI prompt，例如 confirm、input、secret input、multiple-choice prompt。`setup-cli/`、`stack-cli/`、`happier-cli/` 都可以复用。

`terminal/` 负责 terminal runtime flags、terminal metadata、terminal attach planning、headless/tmux command support 等命令行支撑。PTY/session runtime 不属于这里，归 `daemon/` 或 provider/session owner。

`environment/` 负责 CLI-only env handling，例如 env sanitization、nested session detection env cleanup、CLI release channel/env helpers。Daemon/service runtime config parsing 不归这里。

`paths/` 负责 CLI-only path helper，例如 home path expansion/display、path shape、CLI invocation path parsing。跨 app/server/daemon 复用的 path contract 应归 `shared/`。

`process/` 负责短生命周期 command execution helper，例如 command exists、streaming command runner、Windows command invocation resolution。`subprocess/supervision` 这类长驻监督能力不默认归这里；daemon process manager、service supervisor 归对应 owner。

`network/` 负责 CLI-only network helper，例如 proxy resolution、no-proxy matching、HTTP client proxy installation、socket proxy helper。若某个 network primitive 稳定被 app/server/daemon 共用，应迁到 `shared/network/`。

`bin-wrappers/` 负责 bin wrapper helper、spawn current CLI、resolve invoker name、wrapper invocation compatibility logic。Package publishing、release artifact construction、workspace bundling 不归这里，归 `ops/` 或 package-local tooling owner。

`console/` 负责 Windows UTF-8 code page、console write guard、stdout/stderr best-effort writing 等平台 console hardening。它是命令行执行支撑，不是 UI rendering。

`testkit/` 负责 `command-runtime/` 自身测试 harness 和 fixture。产品命令测试仍在对应 CLI 产品面内。

依赖方向：

```text
happier-cli/ setup-cli/ stack-cli/
  -> command-runtime/
  -> shared/
```

`command-runtime/` 可以依赖 `shared/` 和 external libraries，但不能依赖 `app/`、`service/`、`daemon/`、`provider/`、`ops/`，也不能依赖具体 `happier-cli/commands/*`、`setup-cli/commands/*` 或 `stack-cli/commands/*`。

以下能力明确不归 `command-runtime/`：

- `apps/cli/src/runtime/managedTools/**`、`packages/cli-common/src/providers/**`：provider managed tools、provider install/resolution 应归 `provider/` 或 provider runtime owner。
- `packages/cli-common/src/firstPartyRuntime/**`：应归 `shared/first-party-runtime`。
- `packages/cli-common/src/service/**`、长驻 service manager、daemon lifecycle：应归 `daemon/` 或 service owner。
- `packages/cli-common/src/relayHost/**`：应归 `daemon/`、`service/server-runner/` 或 relay owner，不归 command runtime。
- `packages/cli-common/src/systemTasks/**`：setup 入口归 `setup-cli/`，可复用 task schema/执行原语归 `shared/system-tasks`。
- `packages/cli-common/src/workspaces/**`、`componentArtifacts/**`：更偏 `ops/packaging` 或发布构建 owner。
- `apps/cli/src/mcp/**` 中的 MCP server/runtime/resource/provider-detection：CLI-facing 操作面归 `happier-cli/mcp-surface/`，provider-specific MCP adapter 归 `provider/`，跨模块 MCP contract 归 `shared/`，session-scoped MCP runtime 归 `daemon/` 或 `shared/agent-runtime/`，不归 command runtime。
- `safeJson`、`deterministicJson`、`MessageQueue`、`lru` 等 generic utility 不能因为 CLI 当前使用就自动进入 command runtime。跨模块通用则归 `shared/`，owner-specific 则贴近 owner。

## `happier-cli/` 已确认的三级目录

`happier-cli/` 是 Happier 主 CLI 产品面，因此它的三级目录按产品面职责拆分，而不是按当前 `apps/cli/src/cli` 原样搬迁。

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

`entrypoints/` 负责用户可执行的 Happier CLI 入口，包括 `happier`、`happier-dev` 和 `happier-mcp`。当前对应 `apps/cli/bin/*.mjs` 和 `apps/cli/src/index.ts` 的入口启动逻辑。该目录只做启动期 argv 规范化、启动期加固和进入 routing 的交接，不承载具体命令实现。

`routing/` 负责顶层参数规范化、命令分发、command registry wiring、command surface manifest、根级 `--help`/`--version`、默认 provider 命令兜底，以及 provider 命令透传判断。当前对应 `apps/cli/src/cli/dispatch.ts`、`commandRegistry.ts`、`parseArgs.ts`、`commandSurfaceManifest.ts`、`providerCliPassthrough.ts` 等能力。它只决定用户输入进入哪个命令或哪个 provider hook，不实现 daemon、provider 或 service 行为。

`commands/` 负责普通用户可见 Happier 命令 facade。当前对应 `apps/cli/src/cli/commands/**` 中的命令面，例如 session、auth、machine、server、daemon、service、connect、profiles、diagnostics、install、notify、relay、self、tools。该目录保留命令局部参数解析、help、JSON/TTY 输出选择和命令层编排；底层 daemon/provider/service 实现仍归对应 owner。

`agent-commands/` 负责 agent/provider 子命令的 CLI command-hook 层。它注册 provider command surface，并适配 help/output 行为；每个 provider 的执行实现、运行时策略、安装/探测、direct session、fork、resume 等 provider-specific executable behavior 归 `provider/<agent>/`。当前 `apps/cli/src/backends/catalog.ts` 暴露 `getCliCommandHandler` 的方向可以保留为接口形态，但物理归属应迁出 CLI core。

`agent-commands/` 已确认的四级目录：

```text
cli/happier-cli/agent-commands/
  registry/
  invocation/
  session-start/
  passthrough/
  help/
  testkit/
```

`registry/` 负责把 provider catalog entry 和 `cliSubcommand` 注册成 `happier codex`、`happier claude`、`happier opencode` 等用户命令。当前 `apps/cli/src/cli/commandRegistry.ts` 中的 `buildAgentCommandRegistry` 属于这个方向。

`invocation/` 负责统一 lazy-load provider command hook、调用 hook、适配 `CommandContext`、处理错误映射和 command-runtime 输出行为。它不写 provider-specific 分支。

`session-start/` 负责 provider session 启动的通用 CLI 流程。当前 `apps/cli/src/cli/runBackendSessionCliCommand.ts` 所承载的共享参数解析、account settings bootstrap、profile overlay、daemon autostart coordination、runner lock 和通用错误处理应迁入这里。具体 provider run 函数仍归 `provider/<agent>/`。

`passthrough/` 负责 provider CLI info passthrough，例如原生 `--help`、`--version` 请求判断。当前 `apps/cli/src/cli/providerCliPassthrough.ts` 属于这个方向。

`help/` 负责 provider command group help、provider 列表和 provider 命令挂载提示，不写 provider 私有长文档。`testkit/` 只放 agent command registry/invocation/session-start 的 CLI 测试辅助。

不属于 `agent-commands/` 的内容：

```text
provider/<agent>/
  cli-surface/
  detection/
  auth/
  capabilities/
  runtime/
  acp/
  app-server/
  mcp/
  daemon-hooks/
  direct-sessions/
  attach/
  fork/
  resume/
  install/
```

这些目录承载 provider-specific command behavior、runtime policy、ACP/app-server/MCP client、auth spec、detect、capability contribution、daemon spawn hook、direct session、attach、fork、resume、install/runtime policy、prompt、permission、tool 和 metadata shaping。`agent-commands/` 可以调用这些 provider public hooks，但不能把实现搬进 CLI 产品模块。

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

`mcp-surface/` 负责 Happier CLI 暴露给用户和外部 MCP Host 的 MCP 操作面。它覆盖 `happier mcp ...`、`happier-mcp*.mjs`、外部 MCP Host 启动、managed MCP servers 命令 facade，以及通用 stdio bridge/launcher 暴露层。它是 CLI 产品面，不是 provider-specific MCP adapter、session-agent MCP runtime、共享 MCP tool/resource contract 的 owner。

`mcp-surface/` 已确认的四级目录：

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

四级目录职责：

- `bin-entrypoints/`：负责 `happier-mcp*.mjs` 这类 MCP 可执行入口包装；只做 Node/runtime entrypoint 准备、启动期加固和委派。
- `command-router/`：负责 `happier mcp` 命令路由、usage 和子命令分发。
- `external-server/`：负责 `happier mcp serve`，包括 CLI 参数、credential/machine context、account-settings bootstrap、stdio-safe startup，以及交给 external MCP server factory。
- `managed-servers/`：负责用户配置的 MCP servers 的 CLI facade，包括 list/add/bind/unbind/detect/test。
- `stdio-launchers/`：负责通用 CLI-facing stdio MCP launcher surface。
- `stdio-bridges/`：负责通用 CLI-facing stdio MCP bridge surface。
- `composition/`：负责 MCP surface 的依赖组装，不能演变成通用 utils。
- `help/`：负责 MCP command help、命令分组和 help 渲染。
- `testkit/`：负责 MCP surface 测试 fixture、命令 harness 和 helper。

不属于 `mcp-surface/` 的内容：

```text
provider/<agent>/mcp/**
  provider-specific MCP client/detect/config merge/spawn/tool-name policy/bridge

daemon/ or shared/agent-runtime/
  session-scoped MCP runtime, per-session MCP server, session-agent bridge

shared/mcp-domain or shared/agent-domain
  shared MCP schema, resource/tool registration contract, server config record,
  transport-independent normalization
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

`module-adapters/` 负责从 CLI 产品面到其他模块的薄适配层，例如 daemon control client、service/server selection client、provider command client。它可以转换 CLI 参数、调用模块 owner 暴露的 contract、映射错误、适配输出，但不能成为目标模块实现 owner。该目录用于防止 `commands/` 直接 import 过深的 daemon/service/provider 内部实现。

`module-adapters/` 已确认的四级目录：

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

该目录是 `commands/` 到 owner module 的依赖闸口。当前 `apps/cli/src/cli/commands/**` 中存在直接 import `daemon/`、`session/`、`backends/`、`api/`、`terminal/`、`rpc/`、`persistence/` 等深层实现的情况；目标结构中，普通命令应先调用 `module-adapters/*`，再由 adapter 调用 owner module 暴露的 public contract。

每个 adapter 的职责是薄适配，而不是实现 owner 业务：规范化 CLI argv、TTY/JSON mode、server/profile/credentials、交互式确认和输出偏好；把这些转换为 owner module 的 typed request；把 owner error 映射成 command-runtime error/output；把 owner 返回结果整理为 command rendering 所需的数据。

四级目录职责：

- `sessions/`：适配 session 查询、创建、发送、停止、等待、历史、execution-run 操作。
- `daemon/`：适配 daemon start/stop/status/restart/takeover/ownership/service-list。
- `service/`：适配 background service install/uninstall/status/repair。
- `server/`：适配 server profile、server selection、本地和远端 server 管理。
- `auth/`：适配登录、登出、配对、token/status 等认证命令所需 owner contract。
- `machines/`：适配 machine 注册、绑定、状态和远端 machine 相关能力。
- `connections/`：适配 OAuth、connected service、credential persistence。
- `profiles/`：适配 profile 读取、列表、选择和命令上下文应用。
- `relay/`：适配 relay host/status/install/control 能力；relay runtime owner 不在 CLI command 中实现。
- `diagnostics/`：适配 doctor、repair、bug report、health check 等诊断入口。
- `installation/`：适配 installable 查询、安装引导和 provider/tool install 状态；真实安装策略归 owner。
- `notifications/`：适配 notify command 和 activity notification dispatch。
- `capabilities/`：适配 capability query；真实 capability registry 仍归 owner。
- `terminal/`：适配 attach、tmux、Windows Terminal、console focus 等 CLI terminal 行为。
- `shared/`：仅放 adapter 内部共享类型和小型组合 helper，不能变成新的通用 utils。

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

它与相邻目录的边界：`module-adapters/` 不吸收 `agent-commands/`，provider 命令 hook 和 provider-specific 命令行为应归 `agent-commands/` 与 `provider/<agent>/`；也不吸收 `mcp-surface/`，CLI-facing MCP launch/bridge 入口归 `mcp-surface/`，provider-specific MCP adapter 归 `provider/`，跨模块 MCP contract 归 `shared/`。

`help/` 负责 root help、跨命令 help material、命令分组和跨命令 surface 文档。只对某个命令有意义的局部 help 可以继续贴近该命令目录，避免为了集中 help 而破坏命令内聚性。

`cli-lifecycle/` 负责 CLI 自身生命周期能力，例如版本展示、auto-update notice、runtime re-exec、self-update 命令包装。当前 `apps/cli/src/cli/runtime/update/**` 和部分 `self` 命令包装能力可参考迁入这里。release pipeline、跨模块 packaging、deploy orchestration 仍归 `ops/`。

`testkit/` 负责 Happier 主 CLI 产品面专属的测试 helper、fixture 和命令执行 harness。跨 package 测试工具应归 `tests/` 或 shared testkit 位置，不应塞进 `happier-cli/`。

## `commands/` 已确认的四级目录

`commands/` 的四级目录按用户命令领域收拢，而不是按当前平铺文件名照搬。它们都只拥有命令 facade：参数、help、输出、命令层编排，以及对 owner module contract 的调用。

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

`sessions/` 负责会话类用户命令：`session`、`sessions`、`attach`、`resume`、`send`、`wait`、`history`、`run`、`delegate`、`plan`、`review`、`voiceAgent` 等命令面。当前 `attach.ts`、`resume.ts` 和 `commands/session/**` 应在目标结构中统一收拢到这里。长驻 session runtime、PTY、provider session execution、handoff runtime 不应沉淀在这里，分别归 `daemon/`、`provider/`、`shared/agent-runtime` 等 owner。

`auth/` 负责登录、登出、pairing、approve、request、status、wait、server-scoped auth 清理等命令面。CLI 交互和输出归这里；token 持久化策略、daemon auth state、server auth API、auth protocol schema 不归这里。

`machines/` 负责 machine 相关命令面，例如 machine list/status/identity/metadata 展示和用户操作入口。Machine ownership、daemon-local machine state、同步/注册核心逻辑不归这里；这里只是 CLI 入口和展示。

`servers/` 负责 server 管理命令面，例如 server add/list/select、reachable URL flow、server selection、server self-heal 提示。API server 实现不归这里；server 配置契约和服务可达性 primitive 如果跨模块复用，应归 `shared/` 或 `service/` owner。

`daemon/` 负责 daemon 进程命令面，例如 start/stop/status/takeover、ownership conflict 展示、daemon service list/status 的入口。Daemon 长驻进程、监督、状态文件、session runtime 不归这里；这些归 `daemon/`。

`service/` 负责 `happier service ...` 这类本地服务控制命令面，以及 service repair 的命令入口。本地服务生命周期实现大概率归 `daemon/`；服务本体归 `service/api-server` 或 `service/server-runner`；这里只是触发和展示。当前 `serviceRepair/` 不作为独立四级目录，目标上应由 `service/repair` 或 `diagnostics/repair` 复用同一 repair command contract。

`connections/` 负责 `connect`、远程连接目标解析、连接时 auth intent、目标 service id 解析等命令面。连接协议、传输、service discovery 的通用契约不归这里；如果是跨 UI/daemon/CLI 复用能力，应进入 `shared/` 或对应 owner。

`profiles/` 负责 profile 命令面，例如 `profile`/`profiles list`、当前 profile 展示、命令帮助。Profile 的 source of truth、provider profile schema、账号设置持久化不归这里。

`diagnostics/` 负责 `doctor`、`bug-report`、`capabilities`、repair report、健康检查输出、诊断型清理入口。实际检查逻辑如果属于 daemon/service/provider，应由对应模块提供诊断 contract；这里负责聚合、渲染、交互式修复入口。

`installation/` 负责 `happier install ...` 命令面，例如 install doctor、install provider、dry-run/force 参数和结果输出。命名使用 `installation/` 而不是 `installables/`，避免把 provider install catalog、managed runtime source-of-truth 放进 commands。Provider 安装 source of truth、managed runtime、binary-safe install plan 不归这里。

`notifications/` 负责通知类命令面，例如当前 `notify`。它不应塞进 `tools/`、`diagnostics/` 或 `sessions/`。

`relay/` 负责 relay 相关 CLI facade，例如 relay host、relay status、relay host warnings、local server binary version 展示。Relay server 实现、server runner 下载/启动、release asset resolution 不归这里；这里只负责用户命令和输出。

`self/` 负责 CLI 自身用户命令面，例如 self、self-update、self migrate、版本门控迁移入口。真正 release/publish pipeline 不归这里；runtime reexec、auto-update notice 这类底层生命周期能力更适合放在 `cli-lifecycle/`，`commands/self/` 只暴露命令。

`tools/` 负责 Happier tools 的 CLI 命令面，例如 tools list/call、session-bound tool 调用参数和 JSON envelope。Tool catalog、built-in tool runtime、custom MCP tool resolution 不应长期归 commands；这里只是 CLI-facing surface。

`commands/` 不设置通用 `output/` 四级目录。通用 `jsonEnvelope`、TTY/JSON 渲染、表格/列表输出、错误输出格式应归 `cli/command-runtime/output/`。只服务某个命令领域的输出模型可以放在该领域内部，例如 `commands/sessions/output/`。

依赖规则：`commands/*` 不应直接依赖 daemon/service/provider 的深层实现，优先通过 `module-adapters/` 或 owner 暴露的 public contract 调用。这个规则用于防止未来物理重组后 `commands/` 重新变成 daemon、service、provider 的实现容器。

## Agent 智能体接入边界

agent 智能体接入不单独设置 `agent-cli/` 二级目录。更清晰的边界是：

```text
cli/happier-cli/
  agent-commands/

provider/
  codex/
  claude/
  opencode/
  ...
```

`cli/happier-cli/agent-commands/` 只放 agent 命令注册、命令解析、help/output adapter 和 CLI command hook。每个 agent/provider 的真实执行实现、运行时策略、安装/探测、direct session 行为、fork/resume 等 provider-specific executable behavior 应归 `provider/<agent>/`。

当前 `apps/cli/src/backends/catalog.ts` 的 catalog-driven hook 方向是正确的，但未来物理位置应从 `apps/cli/src/backends/**` 迁到 `provider/**`。CLI 只消费 provider 暴露的 `getCliCommandHandler` 或等价 hook，不在 CLI core 中硬编码 provider 行为。

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

## 边界模糊点与目标归属

| 当前位置或能力 | 当前问题 | 推荐目标归属 | 说明 |
| --- | --- | --- | --- |
| `apps/cli/src/backends/**` | 位于 CLI 包内，但实际承载 Codex、Claude、OpenCode、Gemini、Kimi、Qwen、Copilot 等 provider 执行逻辑 | `provider/` | CLI 可以调用 provider CLI surface，但 provider-specific executable behavior 不应留在 generic CLI core。 |
| `apps/cli/src/backends/shared/**` | 名字像 shared，但共享范围是 provider backend family 内部 | `provider/cross-provider/` 或 provider-owned feature submodule | 不应放入一级 `shared/`，除非它真正 provider-agnostic 且被多个生产模块稳定复用。 |
| `apps/cli/src/daemon/**` | 位于 CLI 包内，但负责本地 daemon 生命周期、进程监督、session runtime、terminal pty、machine ownership 等长驻行为 | `daemon/` | CLI 可以暴露 daemon 命令，但不应拥有 daemon 长驻进程实现。 |
| `apps/cli/src/server/**` | CLI 内部存在 server 相关辅助，容易和 `service/api-server` 混淆 | 视内容归 `cli/happier-cli`、`daemon/` 或 `shared/service-contract` | 如果只是 CLI 调用 server 的 client-side reachability helper，可留在 CLI；如果是服务运行实现，归 `service/`；如果是跨模块服务契约，归 `shared/`。 |
| `apps/cli/src/mcp/**` | 既有 CLI 暴露的 MCP bridge/launcher，也可能有 provider-specific bridge 行为 | `cli/happier-cli` 或 `provider/` | 通用 CLI command entry 和 bridge launcher 可归 CLI；provider-specific MCP 适配应归 provider。 |
| `apps/cli/src/session/**` | 混合命令面、handoff、local runtime、provider/session coordination | `cli/happier-cli`、`daemon/`、`provider/`、`shared/agent-runtime` | 命令解析和用户操作归 CLI；长驻 session runtime 归 daemon；provider session 行为归 provider；跨模块 domain primitives 归 shared。 |
| `apps/cli/src/auth/**` | 可能包含 CLI 登录命令、server auth client、daemon auth state | `cli/happier-cli`、`daemon/`、`shared/protocol` | CLI 交互留在 CLI；daemon 持久状态和本地 token 生命周期归 daemon；协议 schema 归 shared/protocol。 |
| `apps/cli/src/installables/**` | 可能混合 UI/daemon/provider/runtime install source-of-truth | `provider/`、`daemon/`、`shared/first-party-runtime` 或 `cli/` | provider install/detection/catalog 不应硬编码在 CLI；CLI 只负责展示和触发。 |
| `apps/cli/src/runtime/**` | 名称过宽，可能包含 command runtime、managed runtime、process runtime、binary-safe runtime | 视内容拆到 `cli/command-runtime`、`daemon/`、`shared/first-party-runtime` | 任何 first-party runtime path 必须符合 binary-safe runtime contract。 |
| `apps/cli/scripts/**` | 混合 build、package-dist、prepack、postinstall、release-it、tool trace、dev setup | `cli/*/tooling` 或 `ops/` | CLI-local build/package helper 可归 CLI tooling；跨模块 release/publish pipeline 归 ops。 |
| `apps/cli/package-dist/**`、`dist/**` | 构建产物 | generated/build output | 应标记为生成物或发布产物，不是手写源码目录。 |
| `apps/bootstrap` | 暴露 `hsetup`，是 setup/bootstrap CLI 入口 | `cli/setup-cli` | setup 命令面归 CLI；可复用安装原语不要全部塞入 setup-cli。 |
| `apps/stack` 命令入口 | 同时包含 stack 命令、daemon service lifecycle、自托管编排、本地 dev/mobile/web/server orchestration | `cli/stack-cli`、`daemon/`、`ops/` 分拆 | 命令 facade 可归 CLI；daemon lifecycle 归 daemon；release/build/deploy/local orchestration 归 ops。 |
| `packages/cli-common/src/providers/**` | 以 cli-common 命名，但涉及 provider install/detection/runtime helper | `provider/` 或 `shared/agent-runtime` | 如果是 provider-specific 或 provider catalog hook，应归 provider；如果是 provider-agnostic domain primitive，可归 shared。 |
| `packages/cli-common/src/firstPartyRuntime/**` | 被 server/stack/CLI 等多方需要时不应是 CLI-owned | `shared/first-party-runtime` | 这类能力不应形成 `service -> cli` 依赖。 |
| `packages/cli-common/src/service/**` | 名称显示服务生命周期/服务 helper | `daemon/` 或 `shared/service-contract` | 如果是本地服务生命周期实现，归 daemon；如果只是跨模块契约，归 shared。 |
| `packages/cli-common/src/tailscale/**` | 可能被 service 或 stack 使用 | `shared/tailscale` 或 `ops/` | 若是纯计算/解析原语，归 shared；若是运维任务编排，归 ops。 |
| `packages/cli-common/src/workspaces/**` | workspace bundling 和 package helper | `ops/packaging` 或 `shared/workspace-tooling` | 如果只服务发布打包，偏 ops；如果是多个 published package 运行时 bundling helper，可考虑 shared/tooling。 |

## CLI 应保留的能力

`cli/` 应该保留的是命令行用户入口和命令运行时支撑：

- bin entrypoints。
- command parser 和 command router。
- 用户可见命令的 help、参数、输出、交互。
- CLI-local config、profiles、server selection 的命令侧处理。
- CLI 作为 client 调用 daemon、service、provider 的 orchestration。
- CLI-only output、prompt、terminal interaction、command error formatting。
- CLI package 的本地构建和发布前 helper，但只限 CLI package 自身。

## CLI 不应保留的能力

`cli/` 不应长期保留：

- provider-specific executable behavior。
- daemon 长驻进程和生命周期核心实现。
- service/api-server 后端实现。
- release/deploy pipeline。
- app/client 构建或 UI 实现。
- provider catalog/source-of-truth 的硬编码分支。
- 跨模块 first-party runtime 原语。
- 生产模块要依赖的通用 shared primitives。

## 物理迁移前检查清单

CLI 目标结构已确认，但真实移动文件前必须完成以下检查：

1. 建立完整 source inventory：逐个确认 `apps/cli/src/**`、`apps/bootstrap/**`、`apps/stack/**`、`packages/cli-common/src/**` 的 owner，不允许按历史 package 整体搬迁。
2. 建立 import alias 和 public contract 计划：先定义 owner module 的 public exports，再让 CLI adapter 调用 public contracts。
3. 保护 package 边界：核对 `bin`、`exports`、`files`、bundled workspace dependencies、prepack/bundling scripts、generated `dist` 和 `package-dist`。
4. 先迁移纯命令面和 `command-runtime/`，再迁移 provider-owned backends、daemon-owned runtime、shared/ops 提取内容。
5. 对 `apps/cli/src/backends/**`、`apps/cli/src/daemon/**`、`apps/cli/src/session/**`、`apps/cli/src/mcp/**` 这类高风险区域建立分阶段迁移计划。
6. 每一步后运行对应 workspace 的 typecheck 和最小测试 lane；涉及 package bundling 时补跑 bundling/script 相关测试。

## 风险提示

CLI 区域是当前目录重组中风险最高的区域之一，因为它同时连接用户命令、provider 执行、本地 daemon、server runner、stack/self-host tooling、MCP bridge、package bundling 和 first-party runtime。

真实物理迁移时不应一次移动整个 `apps/cli`。更稳妥的顺序是：

1. 先建立目标目录和 import alias 计划。
2. 先迁移纯命令面和 command-runtime。
3. 再迁移 provider-owned backends。
4. 再迁移 daemon-owned runtime。
5. 再处理 cli-common 的 shared 提取。
6. 每一步后运行对应 workspace 的 typecheck 和最小测试 lane。
