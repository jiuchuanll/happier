# shared/ Module Reorganization Notes

Date: 2026-05-18
Status: `shared/` first-level module is in progress. `protocol/`,
`agent-domain/`, `mcp-domain/`, `agent-runtime/`, `transfers/`, and
`connection-supervisor/`, and `release-runtime/` have agreed third-level
structures. Physical moves are not authorized yet.

## Purpose

This document records the confirmed `shared/` module design as a supplement to
`module-structure-reorganization-design.md` and
`module-structure-reorganization-design.zh-CN.md`.

The main design document remains the overall architecture entry point. This
document keeps the `shared/` subtask decisions together so they are not lost
during context compaction and can be reviewed independently before physical
directory migration.

## First-Level Responsibility Boundary

`shared/` owns reusable product/runtime primitives and cross-module contracts.
It is the lowest production layer in the target architecture. App, CLI, daemon,
service, and provider modules may depend on `shared/`, but `shared/` must not
depend on those higher-level modules.

`shared/` should contain stable contracts, provider-agnostic domain rules,
reusable runtime primitives, transfer routing policy, connection supervision,
release runtime helpers, and shared native packages. It must not become a
catch-all bucket for code that merely happens to be imported by more than one
place.

## Confirmed Second-Level Modules

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

## Pending Shared Modules

The following current sources are already identified as likely `shared/`
material but still need the same boundary discussion before their target
third-level structures are confirmed:

- `packages/audio-stream-native` -> likely a shared native runtime module
- `packages/sherpa-native` -> likely a shared native runtime module
- reusable system task or first-party runtime primitives referenced from CLI and
  service discussions

## `shared/protocol/`

Confirmed third-level structure:

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

`shared/protocol/` owns the lowest shared wire/schema/API contracts consumed
across app, service, CLI, daemon, and provider modules. It primarily corresponds
to the current `packages/protocol` workspace. It does not own runtime
enforcement, provider executable policy, service persistence, or UI behavior.

Dependency rule:

```text
shared/protocol/<domain> -> shared/protocol/core
app/ cli/ daemon/ service/ provider/ -> shared/protocol/
```

Forbidden direction:

```text
shared/protocol/ -> app/ cli/ daemon/ service/ provider/ agents/ tests/ ops/
```

## `shared/agent-domain/`

Confirmed third-level structure:

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

`shared/agent-domain/` owns the shared semantic layer for what Happier means by
an agent. It defines provider-agnostic agent identity, capability models,
permission semantics, session-control rules, runtime-kind semantics, model
descriptor contracts, and settings contract shapes.

It does not start agents, install provider CLIs, route CLI commands, own UI
components, or implement provider-specific behavior. Concrete provider settings,
provider model lists, executable provider policy, and provider-specific metadata
parsing stay provider-owned.

Dependency rule:

```text
shared/agent-domain/ -> shared/protocol/
shared/agent-runtime/ -> shared/agent-domain/
provider/ -> shared/agent-domain/
app/ cli/ daemon/ service/ -> shared/agent-domain/
```

Forbidden direction:

```text
shared/agent-domain/ -> app/ cli/ daemon/ service/ provider/
shared/agent-domain/ -> shared/agent-runtime/
shared/agent-domain/ -> agents/ tests/ ops/
```

## `shared/mcp-domain/`

Confirmed third-level structure:

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

`shared/mcp-domain/` owns MCP domain rules and stable MCP semantics. It defines
what a Happier-managed MCP server record means, how server bindings and
session-level selections are resolved, how MCP previews are projected, how
provider-detected MCP server records are normalized into provider-agnostic
records, and how MCP-specific tool/resource diagnostics are described.

It does not start MCP servers, own stdio/HTTP bridge processes, perform
provider-specific detection, read or decrypt secret plaintext, route CLI
commands, or register MCP SDK handlers directly.

Dependency rule:

```text
shared/mcp-domain/ -> shared/protocol/
shared/agent-runtime/ -> shared/mcp-domain/
provider/ -> shared/mcp-domain/
app/ cli/ daemon/ service/ -> shared/mcp-domain/
```

Forbidden direction:

```text
shared/mcp-domain/ -> app/ cli/ daemon/ service/ provider/
shared/mcp-domain/ -> shared/agent-runtime/
shared/mcp-domain/ -> agents/ tests/ ops/
```

## `shared/agent-runtime/`

Confirmed third-level structure:

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

`shared/agent-runtime/` owns cross-provider reusable execution primitives for
running and controlling agent sessions. It defines how an agent runtime is
abstracted as a startable, prompt-receiving, cancellable, state-synchronizing,
tool-aware runtime object.

It must not own provider installation, provider CLI resolution, concrete
provider protocol behavior, app UI, CLI command routing, daemon lifecycle
internals, or service persistence. ACP runtime implementation remains under
`provider/provider-families/acp-runtime/`; shared behavior caused only by a
common provider protocol family still belongs in `provider/provider-families/`.

Dependency rule:

```text
shared/agent-runtime/ -> shared/protocol/
shared/agent-runtime/ -> shared/agent-domain/
shared/agent-runtime/ -> shared/mcp-domain/
provider/ -> shared/agent-runtime/
cli/ daemon/ app/ service/ -> shared/agent-runtime/
```

Forbidden direction:

```text
shared/agent-runtime/ -> app/ cli/ daemon/ service/ provider/
shared/agent-runtime/ -> agents/ tests/ ops/
shared/agent-runtime/ -> provider/provider-families/acp-runtime/
shared/agent-runtime/ -> provider/managed-tools executable installers
```

## `shared/transfers/`

Confirmed third-level structure:

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

`shared/transfers/` owns provider-agnostic transfer routing and transfer policy
decisions reused across app, CLI, daemon, and service modules. It decides which
transfer route can be used, whether transfer is currently available, and why a
route is unavailable.

It must not own actual file IO, RPC handlers, HTTP or WebSocket stream
implementations, service relay storage, direct-peer probing, UI copy, CLI
commands, high-level session handoff orchestration, or protocol wire schemas.

Dependency rule:

```text
shared/transfers/ -> shared/protocol/
app/ cli/ daemon/ service/ -> shared/transfers/
```

Cautious composition boundary:

```text
app/ cli/ daemon/ service/ -> shared/connection-supervisor/
app/ cli/ daemon/ service/ -> shared/transfers/
```

`shared/transfers/` and `shared/connection-supervisor/` should usually be
composed by their consumers rather than depending on each other directly.
Connection supervision reports connection/endpoint state; transfer routing
decides route policy from supplied inputs.

Forbidden direction:

```text
shared/transfers/ -> app/ cli/ daemon/ service/ provider/
shared/transfers/ -> shared/agent-runtime/
shared/transfers/ -> shared/connection-supervisor/
shared/transfers/ -> agents/ tests/ ops/
```

## `shared/connection-supervisor/`

Confirmed third-level structure:

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

`shared/connection-supervisor/` owns provider-agnostic and platform-agnostic
connection supervision state machines. It supervises connection state, readiness
probe outcomes, reconnect timing, status publication, and pure request gating
rules.

It must not own socket.io/WebSocket/fetch/axios implementations, CLI loopback
probe implementations, UI token storage, React Native `AppState`, browser
`window`/visibility listeners, endpoint supervisor pools, UI/CLI error classes,
transfer route selection, daemon state persistence, or service API endpoints.

- `state-model/` owns connection phases, reasons, state records, transport
  interfaces, readiness probe result types, and supervisor public contracts.
- `transport-supervision/` owns lifecycle supervision for an injected transport:
  connect, disconnect, error handling, listener cleanup, probe feedback, and
  reconnect scheduling. Concrete socket adapters stay in app/CLI owners.
- `endpoint-supervision/` owns probe-only endpoint health supervision. It calls
  injected readiness probes and publishes endpoint state, but does not own
  fetch/axios wrappers, token lookup, platform lifecycle listeners, or endpoint
  supervisor pools.
- `readiness-contracts/` owns readiness result contracts such as ready,
  server-unreachable, auth-failed, and retry-later. Actual readiness probe
  implementations stay with CLI/UI/service owners.
- `retry-policy/` owns default retry timing policy, fast retry behavior,
  exponential backoff, jitter normalization, and retry delay calculation.
- `request-supervision/` owns pure request gating and request outcome feedback
  helpers when they are independent from concrete error classes and transport
  implementations. UI-specific `HappyError`, CLI HTTP error classes, `fetch`,
  `axios`, and runtimeFetch wrappers stay outside this module.
- `diagnostics/` owns event-to-reason derivation, connection failure categories,
  probe failure categories, and redaction-safe diagnostic shapes.
- `testkit/` owns fake transports, fake supervisors, readiness probe fixtures,
  timer/backoff fixtures, and package-local test helpers. Production modules
  must not depend on it.

Dependency rule:

```text
app/ cli/ daemon/ service/ provider/ -> shared/connection-supervisor/
shared/agent-runtime/ -> shared/connection-supervisor/ only for generic runtime connection health
```

Default dependency stance:

```text
shared/connection-supervisor/ -> no shared/protocol dependency by default
```

The current package has no production dependency on `shared/protocol/`; keep that
independence unless a future reviewed change introduces a stable wire/API schema.

Forbidden direction:

```text
shared/connection-supervisor/ -> app/ cli/ daemon/ service/ provider/
shared/connection-supervisor/ -> shared/transfers/
shared/connection-supervisor/ -> shared/agent-runtime/
shared/connection-supervisor/ -> agents/ tests/ ops/
```

Boundary with `shared/transfers/`:

```text
connection-supervisor reports endpoint/connection state
transfers resolves transfer routes from supplied state and feature policy
app/cli/daemon performs the actual transfer
```

`shared/connection-supervisor/` and `shared/transfers/` should be composed by
their consumers rather than depending on each other directly.

Primary affected areas:

- `packages/connection-supervisor` is the direct source package.
- `apps/cli/src/api/**` is the largest consumer surface: machine/session socket
  connection supervision, loopback readiness probes, supervised request handling,
  and daemon connectivity coordination.
- `apps/ui/sources/sync/**` is the main app consumer surface: endpoint
  supervisor pools, endpoint readiness probes, sync socket transports,
  connectivity gating, and connection-status display.
- `apps/stack`, `apps/cli` packaging, Dockerfile, CI, and package artifact tests
  are affected by the internal workspace bundling path.

## `shared/release-runtime/`

Confirmed third-level structure:

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

`shared/release-runtime/` owns the shared Node runtime substrate for reading
release metadata, selecting release assets, downloading release artifacts,
verifying integrity, and planning archive extraction. It is not just a release
domain package: it includes Node runtime behavior such as HTTP/HTTPS requests,
filesystem writes to download directories, cryptographic signature verification,
checksum validation, and archive extraction command planning.

It does not own release publishing, GitHub release creation, npm publishing,
Docker builds, EAS/App Store publication, CLI self-update process replacement,
Windows process quiesce, first-party component install layout, version
promotion, rollback, shim synchronization, provider managed-tool install policy,
relay/server business startup logic, or shell command execution.

- `release-rings/` owns stable, preview, publicdev, internalpreview, and
  internaldev release ring catalog entries, public labels, channel
  normalization, and release-ring metadata shared by CLI, stack, daemon
  ownership, and release pipelines.
- `release-sources/` owns release metadata source readers. The current source is
  GitHub release metadata by tag, latest release, and first matching tag. It
  reads already-published releases; it does not publish releases.
- `asset-resolution/` owns release asset bundle selection from release metadata,
  including product, version, OS, architecture, checksum, minisign, and archive
  asset matching.
- `download-transport/` owns release download transport primitives such as
  Node HTTP/HTTPS GET, redirects, timeouts, data URL test support, and
  cross-origin sensitive header stripping. It must not become a general network
  utility package.
- `integrity-verification/` owns checksum lookup, SHA-256 comparison support,
  minisign public key parsing, minisign signature parsing, and Ed25519
  signature verification.
- `verified-downloads/` owns the composed flow that downloads checksums,
  verifies minisign, downloads the archive, verifies SHA-256, and writes the
  verified archive into a destination directory. It does not install or promote
  the artifact.
- `extraction-plans/` owns archive extraction command planning such as tar for
  `.tar.gz`/`.tar.xz` and PowerShell `Expand-Archive` for Windows `.zip`
  archives. It creates plans; it does not execute shell commands.
- `diagnostics/` owns release-runtime stage names, error categories, status-aware
  error normalization, and redaction-safe diagnostic shapes. CLI/UI/product
  presentation copy stays with callers.
- `testkit/` owns fake release metadata, fake assets, checksum/minisign fixtures,
  HTTP fixtures, verified-download fixtures, extraction-plan fixtures, and
  package-local test helpers. Production modules must not depend on it.

Dependency rule:

```text
cli/ -> shared/release-runtime/
service/server-runner/ -> shared/release-runtime/
provider/managed-tools/ -> shared/release-runtime/
ops/ -> shared/release-runtime/
shared/first-party-runtime/ -> shared/release-runtime/
```

Cautious frontend/bootstrap boundary:

```text
app/ or bootstrap-like code -> shared/release-runtime/release-rings/ only
```

Only pure release-ring metadata should be consumed by app/bootstrap-like code.
`download-transport/`, `verified-downloads/`, and `extraction-plans/` are Node
runtime capabilities and must not enter mobile/Web client runtime paths.

Forbidden direction:

```text
shared/release-runtime/ -> app/ cli/ service/ provider/ ops/
shared/release-runtime/ -> shared/first-party-runtime/
shared/release-runtime/ -> tests/
```

Primary affected areas:

- `packages/release-runtime` is the direct source package.
- `apps/cli` is the largest consumer surface: CLI self-update, daemon/service
  ownership, relay runtime, doctor repair, provider dependency download, and
  release channel handling.
- `packages/cli-common` consumes release runtime for first-party runtime payload
  preparation, install layout support, managed provider release asset
  extraction, and component catalogs.
- `apps/stack` consumes release runtime for self-host runtime and companion CLI
  download/install flows.
- `packages/relay-server` consumes release asset, checksum, minisign, and
  extraction helpers for server runner related release assets.
- `scripts/pipeline` consumes release ring/channel metadata and release helper
  logic from repo operations.
- `apps/cli`, `apps/stack`, and `packages/relay-server` packaging, Dockerfiles,
  CI, and artifact tests are affected by the internal workspace bundling path.

## Current Source Mapping

| Current source | Target direction |
| --- | --- |
| `packages/protocol` | `shared/protocol/` |
| provider-agnostic parts of `packages/agents` | split between `shared/agent-domain/` and `shared/agent-runtime/` |
| MCP domain rules currently near `packages/protocol/src/mcpServers/**` and `apps/cli/src/mcp/**` | split between `shared/protocol/mcp/`, `shared/mcp-domain/`, and runtime owners |
| `packages/transfers` | `shared/transfers/` |
| `packages/connection-supervisor` | `shared/connection-supervisor/` |
| `packages/release-runtime` | `shared/release-runtime/` |
| `packages/audio-stream-native` | pending shared native module discussion |
| `packages/sherpa-native` | pending shared native module discussion |

## Global Dependency Rules

Allowed:

```text
app/ cli/ daemon/ service/ provider/ -> shared/
shared/<higher reusable runtime> -> shared/<lower contract/domain layer>
```

Forbidden:

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

Within `shared/`, dependency direction must be explicit and acyclic. Protocol
contracts should stay lowest. Domain policy can depend on protocol contracts.
Reusable runtime primitives can depend on protocol and domain modules. A shared
module must not import implementation details from a higher-level owner.

## Physical Migration Rules

- Do not create every target directory as an empty physical folder up front.
  Create a directory only when current source or an approved move has a clear
  owner there.
- Do not move a whole current package as one block if it mixes domain,
  runtime, provider-specific, protocol, and presentation concerns. Split by
  owner first.
- If code names a concrete provider, chooses provider defaults, resolves or
  installs provider CLIs, patches provider config, or interprets
  provider-specific output, keep it in `provider/`.
- If code starts long-running daemon processes or persists daemon state, keep it
  in `daemon/` and expose only narrow reusable contracts to `shared/` when
  needed.
- If code is a versioned wire/API schema consumed across modules, keep it in
  `shared/protocol/` and place derived policy in the appropriate domain module.
- Production modules must not depend on `testkit/` directories.

## Remaining Discussion Order

Recommended next `shared/` discussions:

1. shared native runtime modules
2. reusable system task / first-party runtime primitives
