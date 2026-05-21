# 模块重组子任务索引

日期：2026-05-11
状态：用于跟踪一级模块拆分讨论进度和对应设计文档

## 目的

本文档按一级模块拆分重组讨论子任务，记录每个一级模块的当前状态和对应补充文档。它不替代总设计文档；总设计文档仍是整体结构和依赖规则的主入口。

## 子任务状态

| 一级模块 | 当前状态 | 对应文档 | 说明 |
| --- | --- | --- | --- |
| `app/` | 已确认二级、三级结构 | `app-module-reorganization-notes.zh-CN.md` | 覆盖 `client/`、`website/`、`docs-site/`。 |
| `service/` | 已确认二级、三级结构 | `service-module-reorganization-notes.zh-CN.md` | 覆盖 `api-server/`、`server-runner/`。 |
| `cli/` | 已确认二级、三级、关键四级边界，并完成总体复核 | `cli-boundary-reorganization-design-notes.zh-CN.md` | 覆盖 `happier-cli/`、`setup-cli/`、`stack-cli/`、`command-runtime/`；后续进入物理迁移前 source inventory、package boundary、执行计划。 |
| `daemon/` | 待讨论 | 暂无 | 需要结合当前 CLI daemon、stack lifecycle、local service/session runtime 代码确认边界。 |
| `provider/` | 已确认二级结构、`provider/catalog/`、`provider/cross-provider/`、`provider/managed-tools/`、`provider/provider-families/` 三级结构和 `provider/<providerId>/` 三级原则 | `provider-module-reorganization-notes.zh-CN.md` | 二级结构为 `catalog/`、`cross-provider/`、`managed-tools/`、`provider-families/` 和具体 provider 目录；`provider/catalog/` 已限定为薄总入口和派生索引层；`provider/cross-provider/` 是跨 provider 公共能力层；逐个 provider 的深层细化暂缓到整体物理骨架搭建后。 |
| `agents/` | 待讨论 | 暂无 | 已确定作为一级模块；需要进一步界定 agent 指导文档、Claude/Codex/Cursor 等 agent 专属目录和 runtime/domain 的边界。 |
| `shared/` | 部分已确认 | `shared-module-reorganization-notes.md` / `shared-module-reorganization-notes.zh-CN.md` | 已确认 `protocol/`、`agent-domain/`、`mcp-domain/`、`agent-runtime/`、`transfers/`、`connection-supervisor/`、`release-runtime/`；shared native modules 等仍待继续讨论。 |
| `tests/` | 待讨论 | 暂无 | 需要保持当前 test lanes、testkit、e2e/provider/stress suites 的边界。 |
| `ops/` | 待讨论 | 暂无 | 需要收拢 release、build、packaging、deployment、local orchestration 等跨模块操作能力。 |
| `docs/` | 待讨论 | 暂无 | 需要区分公开 docs site 内容、仓库内部设计文档、agent 指导文档和产品配置文档。 |

## 维护规则

- 每完成一个一级模块的职责界定和目录方案，新增或更新对应补充文档。
- 每次新增补充文档后，同步在 `module-structure-reorganization-design.md` 和 `module-structure-reorganization-design.zh-CN.md` 中加入链接。
- 补充文档只记录设计方案，不授权物理移动目录。
- 物理迁移前仍需单独制定执行计划、依赖改写策略、测试矩阵和回滚方案。
