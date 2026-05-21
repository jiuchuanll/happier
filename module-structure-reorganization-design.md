# Happier Module Structure Reorganization Design

Date: 2026-05-10
Status: first-level modules, CLI second-/third-level structure, provider
second-level structure, provider/catalog third-level structure,
provider/cross-provider third-level structure, provider/managed-tools third-level
structure, provider/provider-families third-level structure, provider-id
third-level principles, shared/protocol third-level structure,
shared/agent-domain third-level structure, shared/mcp-domain third-level
structure, shared/agent-runtime third-level structure, shared/transfers
third-level structure, shared/connection-supervisor third-level structure, and
shared/release-runtime third-level structure reviewed; physical moves are not
authorized yet

## Purpose

This document records the agreed first-level module structure for a future
physical directory reorganization of the Happier repository. It is intentionally
limited to design constraints and ownership rules. It does not authorize moving
files yet.

The goal is to make the repository easier to understand from a developer's
point of view while reducing the risk of broken imports, inverted dependencies,
or mixed ownership after the restructure.

## Current Structure Baseline

The current repository is mainly organized around:

- `apps/`: runnable applications and orchestration packages such as UI, CLI,
  server, stack scripts, docs site, website, and bootstrap.
- `packages/`: shared libraries such as protocol, agent runtime/domain logic,
  transfers, release runtime, connection supervision, native modules, relay
  server, and test suites.
- Root support directories: configuration, CI, Docker, Dagger, scripts,
  documentation, tool-specific folders, and agent guidance files.

The current layout is functional but mixes several different concepts at the
top level: product surfaces, shared runtime packages, provider adapters,
developer documentation, AI-agent guidance, and repository operations.

## Agreed First-Level Modules

The target first-level structure is:

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

These names describe ownership and dependency direction, not just file type.

## First-Level Module Subtask Documents

The detailed reorganization discussion is split by first-level module so the
approved decisions are not lost when conversation context is compacted:

- `app/`: `app-module-reorganization-notes.zh-CN.md`
- `service/`: `service-module-reorganization-notes.zh-CN.md`
- `cli/`: `cli-boundary-reorganization-design-notes.zh-CN.md`
- `provider/`: `provider-module-reorganization-notes.zh-CN.md`
- `shared/`: `shared-module-reorganization-notes.md`,
  `shared-module-reorganization-notes.zh-CN.md`
- Overall subtask index: `module-reorganization-subtask-index.zh-CN.md`

## Module Responsibilities

### `app/`

Owns user-facing product surfaces.

Expected scope:

- Mobile, desktop, and web application UI.
- Website and documentation site runtimes when they are delivered as apps.
- UI composition for provider capabilities, but not provider execution logic.
- Product-level interaction flows, screens, navigation, and presentation.

Current likely sources:

- `apps/ui`
- `apps/website`
- `apps/docs`

Agreed second-level directories:

```text
app/
  client/
  website/
  docs-site/
```

`app/client/` owns the main cross-platform product client. It corresponds to
the current `apps/ui` workspace and includes the mobile app, web app, and
Tauri desktop shell that are currently built from that workspace.

Agreed third-level directories for `app/client/`:

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

`entrypoints/` owns the thin client startup entrypoints, including Expo entry
loading and pre-router installation hooks.

`routes/` owns Expo Router pages, layouts, and route files. It corresponds to
the current `apps/ui/sources/app` router root. If the physical directory is
renamed, the Expo Router root in app-local configuration must be updated in the
same change.

`runtime/` owns application runtime orchestration: boot, provider wrappers,
notifications, tracking, connectivity, and sync runtime wiring.

`domains/` owns client-side business domains such as sessions, machines,
settings, messages, files, auth, voice, source control, artifacts, and
automations.

`ui/` owns reusable UI systems such as components, navigation, theme, modal,
text, hooks, and layout. Lightweight platform-specific sibling files may remain
co-located when they implement the same UI abstraction.

`provider-surfaces/` owns provider-facing UI surfaces: provider pickers, icons,
settings plugins, UI registries, and presentation behavior. It does not own
provider runtime execution, CLI backends, or provider protocol truth sources.

`platforms/` owns platform-specific capabilities that are heavier than
co-located component variants:

```text
platforms/
  mobile/
  web/
  desktop/
```

`native-modules/` owns app-local Expo/native extension modules.

`assets/` owns client runtime assets.

`devtools/` owns client-local development and test helpers, including UI
testkits, dev-only utilities, and debugging helpers.

`tooling/` owns client-local build, migration, postinstall, i18n, CodeMirror,
xterm, and Tauri helper scripts. General repository operations stay under
`ops/`.

`app/website/` owns the public website/marketing site. It corresponds to the
current `apps/website` workspace.

`app/website/` is the public static website for Happier. It owns the marketing
homepage, release/prerelease homepage variants, static website interactions,
public brand assets, and public installer entrypoints hosted by the website.

Current sources:

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

Agreed target structure:

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

`pages/` owns the static website entry pages and homepage variants. It maps to
the current `index.html`, `index.prerelease.html`, and `index.release.html`.

`interactions/` owns website-only browser behavior such as theme toggling,
mobile menu handling, copy-install-command buttons, toast notifications,
smooth scrolling, progressive reveal behavior, and feature tabs. It maps to
the current `src/main.js`.

`styles/` owns website-only styling such as Tailwind input CSS and custom
website CSS. It maps to the current `src/styles.css` plus app-local Tailwind
and PostCSS configuration.

`public/images/` owns public website images, logos, badges, favicons, and
website screenshots. These are not client runtime assets for `app/client/`.

The checked-in `public/install*` files and `public/happier-release.pub` are
website-published installer artifacts. They are hosted by the website, but
their generation and synchronization belong to release/operations tooling. Do
not move these files behind a different public URL unless the release and
installer compatibility plan explicitly preserves the existing endpoints such
as `/install`, `/install.sh`, and `/install.ps1`.

`tests/` owns website-local tests, currently including release homepage
contract checks.

`tooling/` is reserved for website-local tooling, such as future homepage
variant selection helpers. General release, deploy, and installer-generation
logic belongs in `ops/`, not in `app/website/`.

Allowed dependencies:

- Vite, Tailwind, PostCSS, and browser APIs.
- Public static assets.
- Release-published installer artifacts copied or synchronized by `ops/`.

Forbidden dependencies:

- No imports from `app/client/`.
- No imports from concrete `service/`, `cli/`, or `daemon/` implementation.
- No ownership of installer generation logic.
- No ownership of docs-site content/runtime.
- No provider runtime or provider protocol source of truth.

`app/docs-site/` owns the documentation site runtime. It corresponds to the
current `apps/docs` workspace. Human-authored documentation may still belong in
the top-level `docs/` module when it is not tied to the docs-site application
runtime.

`app/docs-site/` is the public Happier documentation site application. It owns
the Next/Fumadocs runtime that turns MDX documentation into the published docs
site, including search routes, LLM text routes, Open Graph image routes, health
checks, layout configuration, and docs-site-only UI components.

Current sources:

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

Agreed target structure:

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

`content/docs/` owns published documentation content. It maps to the current
`apps/docs/content/docs/**` tree and includes product, developer, provider,
protocol, deployment, security, release, and legal documentation that should be
served by the docs site.

Repository-internal design notes, temporary migration plans, and developer
working notes should remain under the top-level `docs/` module unless there is
an explicit decision to publish them through the docs site.

`src/app/` owns the Next App Router route tree. It maps to the current
`apps/docs/src/app/**`, including catch-all documentation pages, `/api/search`,
`/health`, `/llms.txt`, `/llms-full.txt`, `/llms.mdx`, and Open Graph image
routes. Because this is a Next App Router convention, the physical directory
should remain `src/app/` unless the framework configuration and build behavior
are changed and verified in the same migration.

`src/components/` owns docs-site-only React components, such as page actions
for copying or opening LLM-ready documentation content. These components may
use docs-site page context, but they are not the shared application component
system.

`src/lib/` owns docs-site support code such as the Fumadocs source loader,
shared layout options, URL helpers, and small utility functions.

`src/mdx-components.tsx` owns the MDX component registry for the docs site.

`generated/` is the conceptual home for generated Fumadocs output such as the
current `.source` tree. If Fumadocs requires the physical `.source/` path, keep
that path and document it as generated output rather than hand-maintained
source.

`tests/` is reserved for docs-site-local tests. Existing CI and release
contracts may still live in the broader `tests/` or `ops/` modules when they
validate deployment or release behavior rather than docs-site internals.

`tooling/` is reserved for docs-site-local tooling such as future link checks,
content-index generation, or MDX migration helpers. General deploy, release,
and pipeline orchestration belongs in `ops/`.

Allowed dependencies:

- Next, React, Fumadocs, MDX, Tailwind/PostCSS, and docs-site browser/server
  APIs.
- Published documentation content under `content/docs/`.
- Stable shared brand, asset, or type contracts from `shared/` when needed.
- Operations pipelines may build or deploy the docs site from `ops/`.

Forbidden dependencies:

- No ownership of generic repository design records unless they are explicitly
  published documentation.
- No imports from concrete `app/client/` implementation.
- No imports from concrete `service/`, `cli/`, or `daemon/` implementation.
- No provider runtime or provider protocol source of truth.
- No release/deploy orchestration logic; that belongs in `ops/`.

Explicit non-goal:

- `apps/bootstrap` is not part of `app/` by default. It exposes the `hsetup`
  command and contains system/bootstrap tasks, so it should be classified later
  under `cli/`, `daemon/`, or `ops/`.

Allowed dependencies:

- `shared/`
- provider capability and UI adapter surfaces from `provider/`
- service API contracts from `service/`

Forbidden dependencies:

- `app/` must not be required by `shared/`, `provider/`, `cli/`, or `service/`.
- `app/` must not own backend/provider execution policy.

### `service/`

Owns backend service behavior.

Expected scope:

- HTTP/WebSocket APIs.
- Database access and migrations.
- Authentication and authorization.
- Object storage, remote coordination, relay-style services, and server-side
  integrations.
- Server-side jobs and scheduled workflows.

Current likely sources:

- `apps/server`
- `packages/relay-server`

Agreed second-level directories:

```text
service/
  api-server/
  server-runner/
```

`service/api-server/` owns the backend service implementation. It corresponds
to the current `apps/server` workspace and should remain a coherent service
workspace during the physical reorganization.

Expected `api-server/` scope:

- Server entrypoints and startup orchestration.
- HTTP APIs and WebSocket/Socket.IO realtime APIs.
- Authentication, authorization, pairing, OAuth, and service-side identity
  behavior.
- Server-side business domains such as sessions, account state, sharing,
  presence, automations, artifacts, activity, feed, key-value state, social
  features, retention, and server-side feature flags.
- Database, blob/file storage, Redis, queues, locks, cache, sequencing, and
  private-file storage adapters.
- Server-side workers, scheduled jobs, retention processing, presence workers,
  metrics, observability, and diagnostics.
- Full and light server flavors, including SQLite/PGlite/local-file behavior.
- Prisma schema, provider-specific schema variants, migrations, and generated
  Prisma clients.
- Server-local testkit, Vitest configuration, integration/db-contract test
  configuration, and server-local development or migration scripts.
- Server-local deployment descriptors when they describe this service itself.
  Cross-module release/deploy orchestration still belongs in `ops/`.

Current `api-server/` sources:

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

Agreed third-level directories for `service/api-server/`:

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

`entrypoints/` owns process entrypoints such as full and light server mains. It
sets the outer runtime mode and delegates to `runtime/`; it does not own
business logic.

`runtime/` owns startup, shutdown, role selection, and service lifecycle
composition. It may wire HTTP, realtime, workers, storage, observability,
config, and flavors together, but it does not own domain rules.

`http/` owns Fastify HTTP routes, route registration, request/response
boundaries, HTTP validation, rate limits, and HTTP auth guards. Business rules
belong in `domains/` or `auth/`.

`realtime/` owns Socket.IO, rooms, event routing, socket auth policy, and
realtime transport behavior. Presence business state belongs in
`domains/presence`; realtime delivery belongs here.

`domains/` owns server-side business domains such as sessions, account state,
sharing, presence, automations, artifacts, activity, feed, key-value state,
social features, feature flags, retention, pets, and changes.

`auth/` owns authentication, authorization, pairing, terminal auth, OAuth,
keyless auth, account auth, identity policy, and other service-side identity
rules.

`storage/` owns runtime storage adapters such as DB client wrappers, blob/file
storage, Redis, queues, locks, cache, sequence, and private files. It must not
depend on HTTP route details.

`workers/` owns background workers, long-running loops, scheduled server work,
and `SERVER_ROLE=worker` process behavior. It owns scheduling and lifecycle;
domain-specific algorithms should remain in the relevant `domains/` area.

`integrations/` owns service-side external integrations such as GitHub
webhooks, Tailscale URL inference, ElevenLabs/voice service integration, and
other server-side external systems. Provider runtime does not belong here
unless the integration is explicitly a server-side provider hook.

`observability/` owns metrics, Sentry, diagnostics, and logging support.

`config/` owns server environment and configuration parsing, backend
selection, and resolved config objects consumed by `runtime/`. It should not
execute business behavior directly.

`flavors/` owns server runtime flavors such as full and light, including
SQLite, PGlite, and local-file defaults and flavor-specific setup behavior.

`database/` owns Prisma schema, database-provider schema variants, migrations,
and schema synchronization contracts. It is the server persistence contract,
not a replacement for `shared/protocol`.

`generated/` owns generated code such as Prisma generated clients. It is
generated output, not hand-maintained source.

`testkit/` owns server-local test helpers and harnesses. It must not become a
production runtime dependency.

`tooling/` owns server-local development, migration, schema sync, generated
client, runtime build, and validation scripts. General release and pipeline
orchestration belongs in `ops/`.

`deployment/` owns deployment descriptors that describe this service itself.
Cross-module deployment or release orchestration belongs in `ops/`.

Internal dependency direction for `api-server/`:

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

Hard internal rules:

- `storage/` must not depend on `http/` route details.
- `domains/` must not depend on HTTP or Socket.IO transport details.
- `testkit/` must not become a production dependency.
- `tooling/` and `deployment/` must not own release or pipeline orchestration.
- Provider-specific execution policy belongs in `provider/` unless it is a
  clearly server-side provider hook consumed through a provider-owned contract.

`service/server-runner/` owns the published server runner/distribution wrapper.
It corresponds to the current `packages/relay-server` workspace. It is not the
backend service implementation; it resolves, downloads, verifies, and starts
the appropriate Happier server binary for a target platform.

Expected `server-runner/` scope:

- Runner CLI entrypoints such as `happier-server` and `relay-server`.
- Target-platform and release-asset resolution.
- Checksum and minisign verification.
- Verified download and extraction orchestration through shared release
  runtime primitives.
- Runner assets, runner-local tests, and runner packaging helpers.

Current `server-runner/` sources:

- `packages/relay-server/bin/**`
- `packages/relay-server/src/**`
- `packages/relay-server/assets/**`
- `packages/relay-server/scripts/**`
- `packages/relay-server/package.json`

Role in the overall project:

`server-runner/` connects published release artifacts to a user's local runtime
environment. It is a lightweight launcher for Happier Server, not the backend
service itself.

At runtime it:

1. Parses runner invocation options.
2. Resolves the current operating system and CPU architecture.
3. Finds the matching Happier server release artifact.
4. Downloads and verifies the artifact with checksums and minisign signatures.
5. Extracts the artifact into a local cache.
6. Optionally downloads the UI web bundle.
7. Starts the real Happier server binary.

Relationship to other modules:

```text
ops/ builds and publishes server release artifacts
shared/release-runtime/ provides reusable download, verification, and extract primitives
service/server-runner/ consumes those primitives to obtain and start the server binary
service/api-server/ is the backend service that the runner starts
daemon/ or stack/self-host tooling may call the runner to manage a local service
app/client/ connects to the running api-server
```

Agreed third-level directories for `service/server-runner/`:

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

`entrypoints/` owns runner command entrypoints such as `happier-server` and
`relay-server`. It receives CLI invocation and delegates to `runtime/`.

`runtime/` owns the runner execution flow: parse invocation, resolve target,
resolve release assets, download, verify, extract, cache, and spawn the real
server binary. It is orchestration for launching the server, not server
business logic.

`invocation/` owns command-line option parsing and invocation configuration,
including channel, server tag, UI tag, optional UI bundle behavior, and
positional arguments passed through to the server binary.

`targets/` owns local target resolution: operating system, CPU architecture,
server executable name, and runner cache root. Windows, macOS, Linux, x64, and
arm64 compatibility rules belong here.

`release-assets/` owns release artifact selection from GitHub release metadata,
including the server binary artifact and optional UI web artifact. It consumes
release-runtime asset helpers rather than duplicating generic artifact
resolution behavior.

`verification/` owns the runner-facing verification boundary for checksums and
minisign signatures. Generic checksum, minisign, download, and extraction
implementation stays in `shared/release-runtime`.

`assets/` owns static runner assets shipped with the package, such as the
Happier release public key used for verification.

`tooling/` owns runner-local packaging and prepack helpers, such as bundling
workspace dependencies for the published runner package. General release,
deploy, and pipeline orchestration belongs in `ops/`.

Tests for this small runner may remain co-located with the module they cover
instead of moving into a separate `tests/` directory. If the runner grows a
shared testing surface later, introduce a `testkit/` directory then.

Internal dependency direction for `server-runner/`:

```text
entrypoints -> runtime
runtime -> invocation / targets / release-assets / verification / assets
release-assets -> shared/release-runtime
verification -> shared/release-runtime
tooling -> shared/release-runtime / workspace bundling helpers
```

Hard internal rules:

- `server-runner/` must not import `service/api-server/` source code.
- `server-runner/` must not own server business logic.
- Generic release download, checksum, minisign, and extraction behavior belongs
  in `shared/release-runtime`.
- `server-runner/` may consume `shared/release-runtime`, but `shared/` must not
  depend on `server-runner/`.
- `tooling/` must not own cross-module release or deploy orchestration.

Explicit non-goals:

- `packages/protocol` belongs under `shared/protocol`; `service/` implements
  protocol contracts but does not own them.
- `packages/transfers` belongs under `shared/transfers`.
- `packages/connection-supervisor` belongs under `shared/connection-supervisor`.
- `packages/release-runtime` belongs under `shared/release-runtime`; the server
  runner consumes it.
- Stack service lifecycle commands, self-host orchestration, and local service
  management scripts from `apps/stack` should be classified under `daemon/` or
  `ops/`, not under `service/`, unless a script is server-local implementation
  tooling.
- `apps/bootstrap` is not a service module. It should be classified later with
  setup/bootstrap command ownership.

Allowed dependencies:

- `shared/`
- provider contracts or server-side provider hooks when required.

Forbidden dependencies:

- No dependency on UI implementation.
- No dependency on CLI command implementation.
- No dependency on daemon lifecycle implementation.
- No dependency on `ops/` internals.
- No dependency on `tests/` as a production runtime input.
- No ownership of provider-specific executable behavior unless it is explicitly
  server-side and lives behind a provider-owned interface.

### `cli/`

Owns command-line user entrypoints and command-line runtime support. It turns
user commands into calls against `daemon/`, `provider/`, `service/`, and
`shared/` contracts, but should not permanently own those modules' core
implementations.

Expected scope:

- `happier`, `happier-dev`, `happier-mcp`, `hsetup`, and `hstack` command
  surfaces.
- Command parsing and command routing.
- CLI-local environment handling.
- User-facing local commands for sessions, providers, setup, diagnostics, and
  development workflows.
- CLI common utilities that are only meaningful for command execution.

Confirmed second-level structure:

```text
cli/
  happier-cli/
  setup-cli/
  stack-cli/
  command-runtime/
```

Second-level responsibilities:

- `happier-cli/`: the primary Happier CLI product surface. It owns the
  `happier`, `happier-dev`, and `happier-mcp` command entrypoints, command
  registry, help, argument parsing, JSON/TTY output, and command-level
  orchestration. It may expose commands such as daemon, provider, session,
  server, service, MCP, auth, and diagnostics commands, but the underlying
  daemon/provider/service implementations belong to their owning modules.
- `setup-cli/`: the setup/bootstrap CLI surface currently represented by
  `apps/bootstrap` and the `hsetup` entrypoint. It owns setup command flow and
  interactive execution for system tasks. Shared task schemas and reusable
  system-task primitives belong in `shared/`.
- `stack-cli/`: the `hstack` command facade and stack-local command UX
  currently represented by the user-facing parts of `apps/stack`. It owns stack
  command parsing, help, and delegation. Build, release, mobile, daemon
  lifecycle, service lifecycle, and self-host orchestration should be split to
  their owning modules when they are not purely command facade code.
- `command-runtime/`: CLI-only command execution support shared by the CLI
  surfaces. This includes command context, output rendering, TTY/JSON helpers,
  prompt helpers, CLI-only path/env helpers, bin wrapper helpers, command error
  formatting, network/proxy helpers that are only meaningful for CLI execution,
  and platform console hardening. This name is intentionally narrower than
  generic runtime and should not absorb first-party runtime, managed tools,
  daemon runtime, or provider runtime.

CLI overall review conclusion as of 2026-05-14:

- The confirmed `cli/` structure is sound. No fifth second-level CLI directory
  is needed based on the current repository shape.
- `cli/` is a connector layer, not an implementation owner for daemon,
  provider, service, app, shared runtime, or ops pipelines. Its long-term role is
  command entrypoints, command UX, command orchestration, and CLI-only command
  runtime support.
- `command-runtime/` must stay narrow. It should not become a generic shared
  utility package, first-party runtime owner, managed-tools owner, daemon
  runtime owner, or provider runtime owner.
- Dependency gates are part of the design, not an implementation detail:
  `happier-cli/module-adapters/`, `setup-cli/task-adapters/`, and
  `stack-cli/*-commands/` must call owner public contracts instead of importing
  daemon, service, provider, app, or ops internals directly.
- Current source families under `apps/cli/src` such as `backends/`, `daemon/`,
  `agent/`, `api/`, `scm/`, `workspaces/`, `transfers/`, `pets/`,
  `promptAssets/`, `promptRegistries/`, `capabilities/`, `cloud/`, and
  `installables/` must be inventoried and mapped to owner modules before any
  physical move. They must not be moved into `cli/` just because they currently
  live in the CLI package.
- Physical migration must separately preserve package boundaries: `bin`,
  `exports`, `files`, bundled workspace dependencies, prepack/bundling scripts,
  generated `dist` or `package-dist` artifacts, and the affected test/typecheck
  lanes.

Confirmed third-level structure for `cli/setup-cli/`:

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

`setup-cli/` is the `hsetup` command product surface. It owns setup user flows,
the system-task CLI execution entrypoint, interactive IO, hsetup task registry
composition, and setup-facing adapters. It must not become the owner of daemon,
service, provider, first-party runtime, release, or installer implementations.

`bin-entrypoints/` owns executable `hsetup` wrappers and startup fallback logic.
`command-router/` owns hsetup command parsing and dispatch, including
`hsetup system-tasks run [--spec-json <json>]`. `system-task-runner/` owns the
CLI shell around system task execution: stdin or flag input, `SystemTaskSpec`
parsing, task id generation, cancellation, registry execution, JSONL event/result
output, and exit-code mapping. `interactive-io/` owns prompt IO, abortable
readline, sanitized event output, and interactive answer parsing. `task-registry/`
owns hsetup task-kind registration and dependency composition. `setup-workflows/`
owns setup product journeys such as `setup.thisComputer.v1`. `task-adapters/`
owns thin calls into other modules such as auth, server selection, daemon service
install/start/status, and relay configuration through public command or module
contracts. `remote-bootstrap/` owns remote SSH bootstrap command-surface
orchestration and host-trust interaction. `secure-access/` owns setup-facing
secure-access flows such as Tailscale setup tasks. `packaging/` owns hsetup-local
binary build and dist wrapper generation. `help/` owns hsetup usage and help.
`testkit/` owns setup-cli-specific fake IO, registry fixtures, and task-runner
test helpers.

Allowed direction:

```text
cli/setup-cli/
  -> cli/command-runtime/
  -> shared/system-tasks
  -> shared/first-party-runtime
  -> daemon public service/auth/server contracts
  -> service/server-runner public relay/runtime contracts
  -> provider public install/runtime hooks
```

Forbidden direction:

```text
daemon/ service/ provider/ shared/
  -> cli/setup-cli/

cli/setup-cli/
  -> daemon/service/provider deep implementation
  -> ops/release or installer pipeline implementation
```

Current code migration targets:

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
  -> cli/setup-cli/packaging/ or ops/packaging when release-wide

packages/protocol/src/systemTasks/**
packages/cli-common/src/systemTasks/**
  -> shared/system-tasks

packages/cli-common/src/firstPartyRuntime/**
  -> shared/first-party-runtime
```

Confirmed third-level structure for `cli/stack-cli/`:

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

`stack-cli/` is the `hstack` command product surface. It owns the stack-local
command facade, command routing, stack context resolution, stack user workflows,
help, and command-facing delegation into owner modules. It must not become the
owner of app builds, mobile packaging, daemon lifecycle, service runtime,
provider install/runtime policy, release pipelines, remote execution primitives,
or self-host implementation.

Use `*-commands/` suffixes for command facades so these directories do not imply
ownership of the implementation they call. `stack-runtime-commands/` owns the
`start`, `dev`, `stop`, `logs`, and `tui` command surface, not the server,
daemon, Expo, or UI runtime. `worktree-commands/` owns `hstack wt ...` command
UX, not the source-control domain. `component-commands/` owns build, lint,
typecheck, test, mobile, dev-client, and EAS command facades, while the actual
app/mobile/build implementation belongs to `app/client/` or `ops/`.
`host-service-commands/` owns daemon, service, Tailscale, self, and self-host
command facades; the daemon, service, relay, and secure-access logic belongs to
their owner modules. `remote-setup-commands/` owns the remote setup command
surface; reusable SSH and remote install primitives belong in `shared/` or
`ops/`. `provider-commands/` owns the `hstack providers ...` surface while
provider resolution and install policy belong to `provider/`.
`maintainer-commands/` owns maintainer workflow command facades such as
setup-from-source, contrib, PR review/setup, monorepo, import, migrate, CI, and
pack; durable release, packaging, review sandbox, and migration implementations
belong in `ops/` or shared tooling. `happier-passthrough/` owns
`hstack happier <args...>` and repo-local Happier CLI wrapping.

Allowed direction:

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

Forbidden direction:

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

Current code migration targets:

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
  -> cli/stack-cli/worktree-commands/ or shared/source-control when reusable

apps/stack/scripts/build.mjs
apps/stack/scripts/lint.mjs
apps/stack/scripts/typecheck.mjs
apps/stack/scripts/test_cmd.mjs
apps/stack/scripts/mobile*.mjs
apps/stack/scripts/eas.mjs
  -> cli/stack-cli/component-commands/ for command facade
  -> app/client or ops/mobile for implementation

apps/stack/scripts/auth.mjs
apps/stack/scripts/utils/auth/**
  -> cli/stack-cli/auth-context/

apps/stack/scripts/daemon_cmd.mjs
apps/stack/scripts/service.mjs
apps/stack/scripts/tailscale.mjs
apps/stack/scripts/self_host.mjs
apps/stack/scripts/self.mjs
  -> cli/stack-cli/host-service-commands/ for command facade
  -> daemon/service/shared for implementation

apps/stack/scripts/remote_cmd.mjs
apps/stack/scripts/utils/remote/**
  -> cli/stack-cli/remote-setup-commands/ for command facade
  -> shared/remote-execution or shared/system-tasks for primitives

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
  -> cli/stack-cli/maintainer-commands/ for command facade
  -> ops/dev-workflows or ops/packaging for implementation

apps/stack/scripts/menubar.mjs
apps/stack/scripts/completion.mjs
apps/stack/extras/**
  -> cli/stack-cli/integrations/ for command facade
  -> owner module for reusable assets

apps/stack/scripts/happier.mjs
apps/stack/scripts/happier_main.mjs
apps/stack/scripts/repo_local.mjs
apps/stack/scripts/repo_cli_activate.mjs
  -> cli/stack-cli/happier-passthrough/

apps/stack/.claude/**
apps/stack/.cursor/**
apps/stack/.edison/**
apps/stack/.pal/**
  -> agents/, docs/, or ops workflow configuration; not cli/stack-cli/
```

Agreed third-level directories for `cli/command-runtime/`:

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

`context/` owns shared command execution context primitives, stdio/TTY metadata,
and command invocation metadata. Product-specific command registries remain in
the owning CLI product surface, such as `happier-cli/routing/`.

`argv/` owns shared argv parsing and normalization primitives such as flag
readers, special command parsing helpers, and generic argument shapes. Product
routing remains in `happier-cli/`, `setup-cli/`, or `stack-cli/`.

`output/` owns shared JSON envelopes, TTY/JSON rendering helpers, progress
output, table/list rendering, and stable command output primitives.
Command-domain-specific output models may stay inside the owning command domain,
such as `happier-cli/commands/sessions/output/`.

`errors/` owns command-level error normalization, exit-code mapping, user-facing
error wrapping, and debug-output policy. Domain errors should be defined by their
owning module and adapted here only for CLI display.

`prompts/` owns shared CLI prompts such as confirm, input, secret input, and
multiple-choice prompts.

`terminal/` owns command-line terminal support such as terminal runtime flags,
terminal metadata, attach planning, and headless/tmux command support. PTY and
session runtime behavior belong to daemon/provider/session owners, not here.

`environment/` owns CLI-only environment handling, including environment
sanitization, nested-session environment cleanup, and CLI release-channel/env
helpers. Daemon and service runtime configuration parsing belongs to their
owners.

`paths/` owns CLI-only path helpers such as home-path expansion/display, path
shape helpers, and CLI invocation path parsing. Cross-module path contracts
belong in `shared/`.

`process/` owns short-lived command execution helpers such as command existence
checks, streaming command runners, and Windows command invocation resolution.
Long-running supervision, daemon process management, and service supervisors do
not belong here.

`network/` owns CLI-only network helpers such as proxy resolution, no-proxy
matching, HTTP client proxy installation, and socket proxy helpers. If a network
primitive becomes stable across app/server/daemon, it should move to
`shared/network/`.

`bin-wrappers/` owns bin wrapper helpers, current-CLI spawning helpers, invoker
name resolution, and wrapper invocation compatibility logic. Package publishing,
release artifact construction, and workspace bundling belong to `ops/` or the
owning package tooling.

`console/` owns platform console hardening such as Windows UTF-8 code page
setup, stdout/stderr best-effort writing, and console write guards.

`testkit/` owns test helpers and fixtures for `command-runtime/` itself.

`command-runtime/` dependency direction:

```text
happier-cli/ setup-cli/ stack-cli/
  -> command-runtime/
  -> shared/
```

`command-runtime/` may depend on `shared/` and external libraries, but must not
depend on `app/`, `service/`, `daemon/`, `provider/`, `ops/`, or concrete
commands under a CLI product surface.

Do not place these areas under `command-runtime/`:

- Provider managed tools, provider installation, or provider resolution.
- First-party runtime installation, release payload, or managed component
  runtime.
- Daemon runtime, long-running process supervision, daemon state, or service
  supervisors.
- Relay host engine or server-runner behavior.
- System tasks and remote bootstrap task implementations.
- Release/package/workspace bundling and component artifact construction.
- MCP server/runtime/resource/provider-detection behavior.
- Generic utility code merely because CLI currently imports it; cross-module
  utilities belong in `shared/`, and owner-specific utilities belong with their
  owner.

Concrete CLI lifecycle behavior such as auto-update notices, runtime re-exec,
and self-update command wrapping belongs in `happier-cli/cli-lifecycle/`.
`command-runtime/` may only own reusable rendering/error primitives used by that
lifecycle surface.

Agreed third-level directories for `cli/happier-cli/`:

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

`entrypoints/` owns the user-runnable Happier CLI entrypoints, including
`happier`, `happier-dev`, and `happier-mcp`. It should stay thin: startup
normalization, startup hardening, and handoff into routing.

`routing/` owns top-level argument normalization, command dispatch, command
registry wiring, command-surface manifests, root `--help` and `--version`
handling, default provider command fallback, and provider command passthrough
decisions. It routes to owners; it does not implement daemon, provider, or
service behavior.

`commands/` owns the ordinary user-visible Happier command facade. This includes
command-local argument parsing, help, JSON/TTY output decisions, and
command-level orchestration for sessions, auth, machines, servers, daemon,
service, connections, profiles, diagnostics, installation, notifications, relay,
self, and tools. Underlying daemon/provider/service implementations remain in
their owning modules.

Agreed fourth-level directories for `cli/happier-cli/commands/`:

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

`sessions/` owns session-facing command facades such as `session`, `sessions`,
`attach`, `resume`, `send`, `wait`, `history`, `run`, `delegate`, `plan`,
`review`, and voice-agent session command entrypoints. Session runtime, PTY,
handoff runtime, and provider session execution remain outside commands.

`auth/` owns login, logout, pairing, approval, request, status, wait, and
server-scoped auth cleanup command facades. Token persistence, daemon auth
state, service auth APIs, and auth protocol schemas remain in their owners.

`machines/` owns machine command facades and display behavior. Machine
ownership, daemon-local machine state, registration, and sync behavior remain in
daemon/shared owners.

`servers/` owns server management command facades such as add, list, select,
reachable URL flows, server-selection UX, and self-heal prompts. It does not own
the API server implementation.

`daemon/` owns daemon command facades such as start, stop, status, takeover,
ownership-conflict display, and daemon service list/status entrypoints. Daemon
supervision and long-running process state remain in `daemon/`.

`service/` owns local service control command facades and the command entrypoint
for service repair. Local service lifecycle implementation belongs to
`daemon/`; service runtime belongs to `service/api-server/` or
`service/server-runner/`.

`connections/` owns connect command facades, connection target parsing, auth
intent resolution, and target service selection as CLI behavior. Shared
connection protocols and transport primitives remain outside commands.

`profiles/` owns profile command facades such as profile listing and current
profile display. Profile source of truth, provider profile schemas, and account
settings persistence remain in their owners.

`diagnostics/` owns doctor, bug-report, capabilities, repair report, health
check output, and diagnostic cleanup command facades. Owner-specific diagnostic
checks should be exposed through owner contracts and rendered here.

`installation/` owns `happier install ...` command facades and install result
display. Provider installation source of truth, managed runtime, and binary-safe
install plans remain in provider/runtime owners.

`notifications/` owns push notification command facades such as `notify`.

`relay/` owns relay command facades such as relay host, status, warnings, and
local server binary version display. Relay server implementation, runner
download/start behavior, and release asset resolution remain outside commands.

`self/` owns CLI self command facades such as self, self-update, self migrate,
and version-gated migration entrypoints. Low-level CLI lifecycle behavior belongs
in `cli-lifecycle/`; release and publishing pipelines belong in `ops/`.

`tools/` owns Happier tools command facades such as tools list/call and
session-bound tool call arguments. Tool catalogs, built-in tool runtime, and MCP
tool resolution remain in their owning modules.

Common command output helpers do not belong under `commands/`. Shared JSON
envelopes, TTY/JSON rendering, tables, and error formatting belong in
`cli/command-runtime/output/`. Command-domain-specific output models may stay
inside the relevant command domain, such as `commands/sessions/output/`.

`agent-commands/` owns the CLI command-hook layer for agent/provider subcommands.
It registers provider command surfaces and adapts their help/output behavior, but
provider-specific execution, runtime policy, install/detection, direct session,
fork, and resume behavior belong under `provider/<agent>/`.

Confirmed fourth-level structure for `cli/happier-cli/agent-commands/`:

```text
cli/happier-cli/agent-commands/
  registry/
  invocation/
  session-start/
  passthrough/
  help/
  testkit/
```

`agent-commands/` belongs to the Happier CLI product surface. It should expose
providers as user-facing subcommands such as `happier codex`, `happier claude`,
and `happier opencode`, but it must not own provider-specific behavior.
`registry/` maps provider catalog entries and `cliSubcommand` values into the
Happier command registry. `invocation/` lazy-loads and invokes provider command
hooks, adapting `CommandContext`, errors, and command-runtime output behavior.
`session-start/` owns the generic provider session-start CLI flow, including
shared parsing, account settings bootstrap, profile overlays, daemon autostart
coordination, and lock/error handling that is common to provider-backed session
commands. `passthrough/` owns provider CLI info passthrough decisions such as
native `--help` or `--version` requests. `help/` owns provider command group
help and provider list rendering. `testkit/` owns tests and fixtures for this
CLI hook layer only.

Provider-owned behavior remains under `provider/<agent>/`: provider-specific
command behavior, ACP/runtime/app-server/MCP clients, auth specs, detection,
capability contribution, daemon spawn hooks, direct sessions, attach, fork,
resume, install/runtime policy, prompts, permissions, tools, and provider
metadata shaping.

Dependency direction:

```text
cli/happier-cli/routing/
  -> cli/happier-cli/agent-commands/registry

cli/happier-cli/agent-commands/
  -> cli/command-runtime/
  -> cli/happier-cli/module-adapters/
  -> provider/catalog or shared/agent-domain
  -> provider/<agent> public hooks
```

Forbidden direction:

```text
provider/<agent>/
  -> cli/happier-cli/agent-commands/

provider/<agent>/
  -> cli/happier-cli/commands/*
```

The provider CLI hook contract should live in a neutral provider catalog or
shared agent-domain location, not inside `agent-commands/`, so providers do not
need to import the Happier CLI product module to expose their hooks.

Current code migration targets:

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
  -> provider/<agent>/ corresponding runtime capability folders
```

`mcp-surface/` owns the MCP operation surface that the Happier CLI exposes to
users and external MCP hosts. It covers `happier mcp ...`, `happier-mcp*.mjs`,
external MCP host startup, managed MCP server command facades, and generic
stdio bridge/launcher exposure. It is a CLI product surface, not the owner of
provider-specific MCP adapters, session-agent MCP runtime, or shared MCP
tool/resource contracts.

Confirmed fourth-level structure for `cli/happier-cli/mcp-surface/`:

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

`bin-entrypoints/` owns executable MCP entrypoint wrappers such as
`happier-mcp*.mjs`; these wrappers should only prepare the Node/runtime
entrypoint, apply startup hardening, and delegate to the owning surface. They
must not embed provider implementation behavior. `command-router/` owns the
`happier mcp` command routing and usage behavior. `external-server/` owns
`happier mcp serve`, including CLI argument handling, credential/machine context
preparation, account-settings bootstrap, stdio-safe startup, and handoff to the
external MCP server factory. `managed-servers/` owns the CLI facade for
configured MCP servers, including list/add/bind/unbind/detect/test command
flows. `stdio-launchers/` and `stdio-bridges/` own generic CLI-facing stdio MCP
launcher and bridge surfaces. `composition/` assembles CLI-surface dependencies;
it should remain a dependency composition layer, not a generic utility bucket.
`help/` owns MCP command help and grouping. `testkit/` owns MCP surface tests,
fixtures, and command harness helpers.

The `mcp-surface/` module must not absorb provider-specific MCP logic. Codex,
Claude, OpenCode, Gemini, and other provider MCP clients, detection, config
merge, spawn resolution, tool-name policy, and provider-specific bridges belong
under `provider/<agent>/mcp/` or the provider's equivalent public capability
folder. Session-scoped MCP runtime such as per-session MCP servers and
session-agent HTTP/stdio bridges belongs under `daemon/` or
`shared/agent-runtime/`, depending on final owner scope. Shared MCP schemas,
resource/tool registration contracts, server config records, and transport-
independent normalization belong under `shared/mcp-domain` or
`shared/agent-domain`.

Allowed direction:

```text
cli/happier-cli/routing/
  -> cli/happier-cli/mcp-surface/command-router

cli/happier-cli/mcp-surface/
  -> cli/command-runtime/
  -> cli/happier-cli/module-adapters/
  -> shared/mcp-domain or shared/agent-domain
  -> provider/catalog public MCP hooks
```

Forbidden direction:

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

Current code migration targets:

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
  -> daemon/ or shared/agent-runtime/ session MCP runtime

apps/cli/src/backends/<provider>/mcp/**
apps/cli/src/backends/codex/happyMcpStdioBridge.ts
apps/cli/src/backends/codex/codexMcpClient.ts
  -> provider/<agent>/mcp/

apps/cli/src/mcp/server/registerHappierMcpBuiltInTools.ts
apps/cli/src/mcp/resources/registerHappierMcpResources.ts
apps/cli/src/mcp/happierMcpToolCatalog.ts
  -> shared/mcp-domain or shared/agent-domain
```

`module-adapters/` owns thin adapters from the CLI product surface to other
modules, such as daemon control clients, service/server selection clients, and
provider command clients. These adapters may translate CLI arguments, call
module-owned contracts, map errors, and adapt output, but must not become the
implementation owner for the target module.

Confirmed fourth-level structure for `cli/happier-cli/module-adapters/`:

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

`module-adapters/` is the dependency gate between `commands/` and owner modules.
`commands/*` should depend on these adapters instead of importing deep daemon,
service, provider, session, API, terminal, or persistence internals directly.
Each adapter exposes a stable CLI-facing API for one owner capability boundary:
it normalizes command inputs, passes typed requests to the owner module, maps
owner errors into command-runtime error/output forms, and returns data shaped for
command rendering.

The adapter folders are grouped by owner-facing capability rather than by
technical helper type. `sessions/` adapts session query/create/send/stop/wait,
history, and execution-run operations. `daemon/` adapts daemon start/stop/status,
restart, takeover, ownership, and service-list workflows. `service/` adapts
background service install/uninstall/status/repair flows. `server/` adapts server
profiles, server selection, and local/remote server management. `connections/`
adapts connected-service flows such as OAuth and credential persistence.
`terminal/` adapts attach, tmux, Windows Terminal, and console-focus behavior
that is specific to CLI command execution. `capabilities/` adapts capability
queries while the actual capability registry remains with the owning module.
`shared/` is only for adapter-internal types and small composition helpers; it
must not become a generic utility bucket.

Allowed direction:

```text
cli/happier-cli/commands/*
  -> cli/happier-cli/module-adapters/*
  -> daemon/ service/ provider/ agents/ shared/

cli/happier-cli/module-adapters/*
  -> cli/command-runtime/*
  -> shared/*
```

Forbidden direction:

```text
daemon/ service/ provider/ shared/
  -> cli/happier-cli/module-adapters/

cli/happier-cli/module-adapters/*
  -> cli/happier-cli/commands/*
  -> cli/happier-cli/routing/*
```

`module-adapters/` must not absorb `agent-commands/` or `mcp-surface/`.
Provider command hooks and provider-specific command behavior belong to
`agent-commands/` plus `provider/<agent>/`. CLI-facing MCP launch and bridge
entrypoints belong to `mcp-surface/`; provider-specific MCP adapters belong
to `provider/`, and cross-module MCP contracts belong to `shared/`.

`help/` owns root and cross-command help material, command grouping, and
cross-command surface documentation. Command-local help can remain next to the
specific command when it is only meaningful there.

`cli-lifecycle/` owns CLI self-lifecycle behavior such as version display,
auto-update notices, runtime re-exec, and self-update command wrapping. Release
pipelines, cross-module packaging, and deploy orchestration still belong in
`ops/`.

`testkit/` owns test helpers and fixtures that are specific to the primary
Happier CLI product surface. Cross-package test utilities belong in `tests/` or
shared testkit locations, not in `happier-cli/`.

Current likely sources:

- `apps/cli`
- `packages/cli-common`
- `apps/bootstrap`
- user-facing command facade portions of `apps/stack`

Detailed CLI boundary analysis:

- `cli-boundary-reorganization-design-notes.zh-CN.md`

Allowed dependencies:

- `shared/`
- provider CLI backend surfaces from `provider/`
- daemon control interfaces when invoking local services.

Forbidden dependencies:

- No dependency on app UI implementation.
- No provider-specific behavior in generic CLI core when it can live in a
  provider-owned module.
- No daemon long-running process implementation in CLI command core.
- No service/api-server implementation in CLI command core.
- No release/deploy pipeline ownership in CLI command core.

### `daemon/`

Owns local long-running process behavior and local service lifecycle.

Expected scope:

- Local daemon lifecycle.
- Service install, uninstall, enable, disable, restart, status, and logs.
- Local state files, ports, process supervision, and runtime cleanup.
- Local wrapper scripts that start or stop the Happier local environment.

Current likely sources:

- relevant parts of `apps/stack/scripts`
- relevant daemon/runtime parts of `apps/cli`
- `scripts/local`

Allowed dependencies:

- `shared/`
- CLI-local command entrypoints where daemon commands are exposed through CLI.
- provider runtime only through provider-owned interfaces.

Forbidden dependencies:

- No provider policy embedded directly in daemon orchestration.
- No dependency on app UI implementation.

### `provider/`

Owns runtime integrations for external agent/provider systems.

Expected scope:

- Codex, Claude, OpenCode, Gemini, Kimi, Qwen, Copilot, and similar provider
  adapters.
- CLI backend implementations.
- Provider capability declarations.
- Provider-owned protocol adapters and projections. Shared wire/schema/API
  contracts remain under `shared/protocol`.
- Provider UI behavior adapters.
- Provider contract tests and provider-specific fixtures.

Current likely sources:

- `apps/cli/src/backends/*`
- `apps/ui/sources/agents/providers`
- `apps/ui/sources/agents/registry`
- `apps/ui/sources/agents/backendCatalog`
- `packages/cli-common/src/providers`
- provider-specific pieces of `packages/agents/src/providers`
- provider settings definitions and registries from
  `packages/agents/src/providerSettings`, split by catalog/domain/provider owner
- provider-specific parts of `packages/agents/src/sessionControls`
- provider-owned candidates inside `packages/protocol/src/providers`, after
  classifying shared protocol contracts versus executable provider policy
- provider-focused docs such as feature matrices where they describe provider
  behavior.

Source classification rule:

- `packages/protocol` remains the owner of shared protocol contracts. Provider
  wire/schema/API contracts under `packages/protocol/src/providers/**` should
  move with `shared/protocol`, not directly into `provider/`.
- Provider-specific executable behavior, install policy, manifest data, or
  family adapter behavior that currently sits near protocol/provider packages
  must be classified and moved to `provider/<providerId>/`,
  `provider/catalog/`, `provider/managed-tools/`, or
  `provider/provider-families/` according to its owner.
- `packages/agents` must not be moved as one block. Provider-agnostic agent
  domain/runtime primitives move under `shared/agent-domain` or
  `shared/agent-runtime`; provider-specific definitions move under `provider/`.

Confirmed second-level structure:

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

`catalog/` owns the provider registry and derived artifacts. It aggregates
provider entries, settings registries, UI/provider registry surfaces, and
provider id/schema catalogs. It must not own concrete provider runtime
implementation.

`cross-provider/` owns provider-domain primitives shared by multiple providers,
such as hook types, provider capability helpers, model capability normalization,
direct-session/fork hook contracts, and provider-agnostic adapter utilities.
It is the cross-provider layer inside `provider/`, not the repository-wide
`shared/` module.

`managed-tools/` owns provider CLI resolution and installation support,
including managed Node/PNPM runtime helpers, provider CLI path resolution,
vendor recipe planning, managed package installation, GitHub release download
and extraction, and binary-safe runtime guards.

`provider-families/` owns shared behavior for related provider families, such
as OpenCode-family providers or catalog-defined ACP providers. It prevents one
concrete provider from importing another concrete provider directly.

Concrete provider folders such as `codex/`, `claude/`, `opencode/`, and
`customAcp/` own the provider-specific runtime, CLI adapter, auth, cloud
connect, direct-session, MCP, settings, UI adapter, provider-owned protocol
adapter/projection, fixtures, and provider-local tests for that provider.
Shared protocol wire/schema/API contracts remain under `shared/protocol`. Keep `customAcp/` camelCase to
match the existing persisted `AgentId` and avoid an extra migration alias.

Confirmed third-level structure for `provider/catalog/`:

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

`provider/catalog/` is the thin entry layer that connects all providers through
public manifests and public hooks. It must not contain one provider's special
runtime integration.

- `entry-contracts/` defines catalog entries, manifest entries, hook types, and
  capability declaration shapes.
- `registry/` aggregates `provider/<providerId>/manifest/` entries and exposes
  provider lookup APIs.
- `identifiers/` owns provider ids, aliases, default provider facts, CLI
  subcommand mapping, and persisted-id compatibility rules.
- `capability-index/` derives provider capability matrices from manifests and
  hook presence.
- `hook-resolvers/` owns lazy hook lookup, caching, missing-hook errors, and
  generic fallback behavior.
- `settings-index/` aggregates provider settings definitions and validates
  settings-key ownership.
- `backend-targets/` owns generic backend target references, profile/flavor
  indexes, and lookup helpers.
- `installables/` declares provider installable keys and their provider
  ownership. Executable install logic belongs in `provider/managed-tools/`.
- `ui-projections/` builds pure data projections for UI consumers, not React
  screens or provider-specific UI behavior.
- `derived-artifacts/` contains generated or derived protocol, CLI, and UI
  artifacts from the registry.
- `validation/` checks catalog consistency, id uniqueness, hook/capability
  alignment, settings-key conflicts, and installable ownership.
- `testkit/` contains catalog fixtures and manifest builders for tests.

Allowed dependencies:

```text
provider/catalog/registry       -> provider/<providerId>/manifest/
provider/catalog/hook-resolvers -> provider/catalog/registry
provider/catalog/*              -> provider/catalog/entry-contracts
provider/catalog/*              -> provider/cross-provider/contracts
```

Forbidden dependencies:

```text
provider/catalog/ -> provider/<providerId>/runtime/
provider/catalog/ -> provider/<providerId>/cli-adapter/
provider/catalog/ -> provider/<providerId>/auth/
provider/catalog/ -> provider/<providerId>/sessions/
provider/catalog/ -> provider/<providerId>/ui-adapter/
provider/catalog/ -> provider/managed-tools executable installers
provider/catalog/ -> app/client screens
```

Confirmed third-level structure for `provider/cross-provider/`:

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

`provider/cross-provider/` is the provider-domain cross-provider layer. It contains
cross-provider primitives that are still specific to provider integrations. It
is not the repository-wide `shared/` module, not the provider catalog, and not a
place for concrete provider implementations.

- `contracts/` owns provider-shared type contracts, hook shapes, and base
  capability declaration primitives. It can describe the shape of provider
  hooks, but provider registration, lookup, and hook resolution remain in
  `provider/catalog/`.
- `direct-sessions/` owns cross-provider direct-session operation contracts,
  transcript/page/candidate/activity shapes, and provider-agnostic environment
  merge helpers. Concrete direct-session behavior remains in
  `provider/<providerId>/sessions/` or the provider-owned direct-session
  capability folder.
- `forking/` owns provider fork contracts such as ACP continuation handlers,
  provider-native fork points, and fork dispatch result shapes. It does not
  execute provider-specific fork behavior.
- `session-metadata/` owns generic provider session metadata update helpers,
  provider session id metadata handling, and metadata merge/update primitives.
- `session-capabilities/` owns provider-agnostic session capability
  normalization, such as resume, attach, follow, fork, or direct-session support
  results that can be consumed by CLI, UI, daemon, or catalog projections.
- `execution-runs/` owns provider-agnostic execution-run adapter primitives and
  simple backend factories. It must not become CLI command routing or a concrete
  provider runtime.
- `model-capabilities/` owns model capability normalization such as context
  window token parsing and stable model capability shapes.
- `cli-adapter-primitives/` owns small provider-adapter helpers needed by
  provider CLI adapters, such as generic terminal display factories, when those
  helpers are provider-adapter primitives rather than Happier command surfaces.
  Anything that depends deeply on CLI command routing belongs in `cli/`.
- `ui-adapter-contracts/` owns data-only UI adapter contracts for provider
  settings, local auth, install banners, and UI-facing provider behavior
  declarations. React components, screens, themes, and provider-specific UI
  behavior remain in `app/client` or `provider/<providerId>/ui-adapter/`.
- `permissions/` owns provider-shared permission request sources, permission
  capability contracts, and permission normalization helpers. Concrete provider
  policy remains provider-owned.
- `diagnostics/` owns normalized provider diagnostics, warnings, capability
  mismatch reports, and runtime diagnostic result shapes. It should output
  structured diagnostics, not UI rendering.
- `testkit/` owns provider-shared fixtures, typed builders, fake provider hooks,
  and contract test helpers. It must not become a production runtime dependency.

Allowed dependencies:

```text
provider/catalog/*           -> provider/cross-provider/contracts
provider/<providerId>/*      -> provider/cross-provider/*
provider/provider-families/* -> provider/cross-provider/*
provider/managed-tools/*     -> provider/cross-provider/contracts or model-capabilities
provider/cross-provider/*            -> shared/ public contracts and primitives
```

Forbidden dependencies:

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

Current source mapping guidance:

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
  -> provider/cross-provider/cli-adapter-primitives/ only if it remains a thin
     provider-adapter primitive; otherwise keep it under cli/command-runtime/

apps/ui/sources/agents/providers/shared/* data contracts
  -> provider/cross-provider/ui-adapter-contracts/

apps/ui/sources/agents/providers/shared/* React/theme/rendering code
  -> app/client provider UI integration or provider/<providerId>/ui-adapter/

packages/agents/src/providers/providerCliRuntime.ts provider-specific data
  -> provider/catalog/installables/ or provider/managed-tools/, not
     provider/cross-provider/
```

Confirmed third-level structure for `provider/managed-tools/`:

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

`provider/managed-tools/` owns provider-related external tool resolution,
installation, managed runtime support, status, update checks, and binary-safe
launch guarantees. It is not provider runtime, not the catalog, and not the CLI
command surface. Its job is to turn catalog/provider manifest installable
declarations into tools that can be resolved, installed, launched, diagnosed,
and updated safely.

- `contracts/` owns managed-tool type contracts such as install modes,
  resolution sources, install results, tool status, and managed runtime
  availability. It must stay scoped to managed-tools and must not replace
  `provider/cross-provider/contracts/`.
- `installable-resolvers/` converts `provider/catalog/installables/`, provider
  manifests, and provider CLI runtime declarations into managed-tool
  definitions. It is not a second installable registry.
- `provider-cli-resolution/` owns provider CLI path resolution: explicit
  overrides, system PATH lookup, known candidates, managed install paths, and
  source preference. It answers which CLI path should be used.
- `provider-cli-launch/` turns a resolved CLI path into a spawn-safe launch
  spec, including JavaScript runtime wrapping, shebang checks, `.js`/`.mjs`
  entrypoint handling, and command/argument composition. It answers how to
  launch the resolved tool.
- `install-plans/` owns dry-run and confirmation-safe install planning:
  platform selection, install mode, admin requirements, commands, and structured
  plan output. It must not execute installers.
- `installers/` owns installer execution. Its fourth-level split should be
  `vendor-recipes/`, `managed-packages/`, and `github-release-binaries/`.
  Vendor recipes must remain confirmation-gated by default; managed packages
  must use managed runtime/tooling; GitHub release binaries must download,
  verify, extract, and stage atomically.
- `managed-runtimes/` owns Happier-managed runtime prerequisites such as the
  managed JavaScript runtime and managed pnpm. This is the binary-safe boundary
  that keeps first-party runtime paths working without system `node`, `npm`,
  `npx`, `pnpm`, `yarn`, or `bunx`.
- `release-assets/` owns provider/tool-specific release asset selection,
  required digest handling, and managed-tool placement adaptation. Generic
  archive extraction, checksum, and release download primitives should stay in
  `shared/release-runtime` when they are reusable outside provider tools.
- `tool-store/` owns `$HAPPIER_HOME/tools/**` layout, `current/next` staged
  installs, scratch directories, locks, install-state files, and install log
  paths.
- `tool-status/` owns managed tool status: installed status, resolved path,
  source kind, installed version, last install log, and last background check
  timestamp. The explicit name prevents this area from absorbing provider,
  daemon, or session status.
- `updates/` owns latest-version checks, version comparison, background
  auto-update eligibility, throttling, and update result recording. It stays
  separate from `tool-status/` so querying state does not imply an update side
  effect.
- `diagnostics/` owns structured managed-tool diagnostics such as invalid
  overrides, missing runtime prerequisites, unavailable CLIs, failed checksum
  verification, install failures, and update failures. UI and CLI rendering
  remains in the consuming product surface.
- `testkit/` owns fake releases, fake provider CLIs, fake tool stores, install
  plan fixtures, and managed-runtime test helpers. It must not become a
  production dependency.

Allowed dependencies:

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

Forbidden dependencies:

```text
provider/catalog/ -> provider/managed-tools executable installers
provider/managed-tools/ -> app/client screens
provider/managed-tools/ -> cli/happier-cli command routing
provider/managed-tools/ -> provider/<providerId>/runtime implementation
provider/managed-tools/ -> service concrete storage/API implementation
provider/managed-tools/ -> ops release pipeline
```

Current source mapping guidance:

```text
packages/cli-common/src/providers/resolution.ts
apps/cli/src/runtime/managedTools/providerCliResolution.ts
  -> provider/managed-tools/provider-cli-resolution/

apps/cli/src/runtime/managedTools/requireProviderCliLaunchSpec.ts
  -> provider/managed-tools/provider-cli-launch/

packages/cli-common/src/providers/install.ts
apps/cli/src/runtime/managedTools/invokeProviderCliInstall.ts
  -> provider/managed-tools/install-plans/ and provider/managed-tools/installers/

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
  -> provider/managed-tools/installers/, tool-status/, and updates/

packages/agents/src/providers/providerCliRuntime.ts
  -> provider/catalog/installables/, provider/<providerId>/manifest/, and
     provider/managed-tools/contracts/
```

Tool-specific adapters, such as Codex release asset selection, GitHub CLI
installation, or Codex ACP installation, may live in `provider/managed-tools/`
when they are installable/tool adapters. They must not import concrete provider
runtime implementation or become provider business behavior.

Confirmed third-level structure for `provider/provider-families/`:

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

`provider/provider-families/` owns shared behavior for provider families: code
that is reused by multiple provider ids or dynamic provider mechanisms because
they share a protocol, launch model, transport profile, or runtime pattern. It
is not a second catalog, not a replacement for `provider/cross-provider/`, and not a
place to hide one provider's concrete implementation.

Decision rule: shared contracts, normalization, and lightweight primitives used
across providers belong in `provider/cross-provider/`; shared implementation
caused by a common protocol family, runtime family, transport profile, or
launch model belongs in `provider/provider-families/`.

- `contracts/` owns family-layer contracts such as factory parameters, family
  capability overrides, transport profile hooks, and family diagnostic result
  shapes. It must stay scoped to family internals and must not declare the
  provider catalog.
- `acp-runtime/` owns the shared ACP protocol/runtime family: ACP backend
  creation, ACP runtime construction, bridge/update/history handling,
  permission mapping, spawn handling, stdout/stderr stream handling, and
  transport extension points. Provider-specific transport policy remains in the
  provider-owned folder and plugs in through family contracts.
- `built-in-acp/` owns builders for built-in providers whose implementation
  shape is generic ACP, such as catalog-defined ACP entry/backend construction.
  Provider identity and public manifest output still belong to
  `provider/<providerId>/manifest/`.
- `configured-acp/` owns account-settings/user-configured ACP backend
  materialization. It handles dynamic backend config, launch environment
  materialization, configured runtime creation, and configured ACP CLI command
  delegation. Product-facing `customAcp/` remains a provider facade, not the
  family implementation itself.
- `opencode-compatible/` owns OpenCode-compatible family behavior currently
  shared by OpenCode and Kilo, such as native permission env/ruleset mapping.
  OpenCode server runtime, Kilo transport, provider-specific stderr parsing,
  auth, session metadata, and UI behavior remain provider-owned.
- `diagnostics/` owns family-level diagnostics such as ACP handshake shape,
  transport profile resolution, factory inputs, and family runtime lifecycle
  checks. Concrete provider auth/model/install diagnostics remain in the
  concrete provider or `provider/managed-tools/`.
- `testkit/` owns family-level fixtures and harnesses such as fake ACP servers,
  subprocess harnesses, family manifest fixtures, and transport-profile test
  doubles. It must not become a production dependency.

Allowed dependencies:

```text
provider/<providerId>/acp -> provider/provider-families/acp-runtime
provider/<providerId>/manifest -> provider/provider-families/built-in-acp builders
provider/customAcp/settings -> provider/provider-families/configured-acp contracts
provider/opencode|kilo -> provider/provider-families/opencode-compatible
provider/provider-families/* -> provider/cross-provider/contracts
provider/provider-families/* -> provider/managed-tools public launch/resolution APIs
provider/catalog -> provider/<providerId>/manifest outputs
```

Forbidden dependencies:

```text
provider/provider-families/* -> provider/<providerId> concrete implementation
provider/provider-families/* -> provider/catalog registry internals
provider/provider-families/* -> cli/happier-cli command routing
provider/provider-families/* -> app/client screens
provider/cross-provider/ -> provider/provider-families
provider/<providerId> -> provider/<otherProviderId>
```

Current source mapping guidance:

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
  -> shared/protocol/providers/kiro/ if it remains a wire/schema/API contract
  -> provider/provider-families/built-in-acp/ only for family builder or
     materialization behavior after classification

apps/cli/src/agent/acp/catalog/configured/**
  -> provider/provider-families/configured-acp/

apps/cli/src/backends/openCodeFamily/**
  -> provider/provider-families/opencode-compatible/
```

Design note: `provider/provider-families/` should expose factories and hooks
that concrete providers consume. It must not import concrete provider
implementations. The current catalog-defined ACP transport resolver should be
split during physical migration so the family layer defines the transport
profile contract while provider-owned manifests or hooks provide concrete
transport implementations such as Kiro.

Confirmed third-level principle for `provider/<providerId>/`:

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

This is a capability menu, not a mandatory template. Only `manifest/` is
expected for every provider. Other directories should exist only when the
provider actually owns that capability. Complex providers such as Codex,
Claude, and OpenCode may have deeper provider-owned structure, while lightweight
ACP/CLI providers should stay small and must not grow empty compatibility
folders for symmetry. Per-provider fine-grained design is intentionally
deferred until the overall physical skeleton is established.

Allowed dependencies:

- `shared/`
- internal provider support such as `provider/cross-provider/`, `provider/catalog/`,
  `provider/managed-tools/`, and `provider/provider-families/`

Allowed consumers:

- `app/`
- `cli/`
- `daemon/`
- `service/`
- `tests/`

Forbidden dependencies:

- Provider modules must not depend on app screens, CLI command routing, service
  concrete storage, or test packages.
- Provider modules must not directly depend on each other. Shared provider
  family behavior should live in `provider/provider-families/` or
  `provider/cross-provider/`, not in one provider importing another provider.

### `agents/`

Owns guidance for AI agents and developer tools. This is a first-level module
and is intentionally separate from runtime provider code.

Expected scope:

- Codex guidance documents.
- Claude guidance documents.
- Cursor guidance documents.
- Shared agent policies, prompts, operating procedures, and tool-use guidance.
- Product configuration notes intended for agent/tool consumption when they are
  not runtime code.

Current likely sources:

- `AGENTS.md`
- `CLAUDE.md`
- `.claude`
- `.cursor`
- `skills`
- developer guidance documents that are written for agent/tool behavior rather
  than application runtime behavior.

Allowed dependencies:

- Conceptual references to all modules through documentation.
- No production import dependency.

Forbidden dependencies:

- Production runtime code must not import from `agents/`.
- `agents/` must not become the home for provider runtime adapters. Provider
  runtime belongs in `provider/`.

### `shared/`

Owns reusable product/runtime primitives and cross-module contracts.

Expected scope:

- Protocol contracts.
- Agent runtime/domain primitives that are not tool guidance.
- Transfer protocols.
- Release runtime helpers.
- Connection supervision.
- Shared native modules.
- Shared schemas, catalogs, and provider-agnostic domain logic.

Current likely sources:

- `packages/protocol`, including shared provider protocol wire/schema/API
  contracts under `packages/protocol/src/providers/**`.
- provider-agnostic portions of `packages/agents`, renamed in the target
  structure to avoid conflict with the new first-level `agents/` guidance
  module.
- `packages/transfers`
- `packages/release-runtime`
- `packages/connection-supervisor`
- `packages/audio-stream-native`
- `packages/sherpa-native`

Agreed naming direction for current `packages/agents`:

- Move under `shared/agent-runtime` or `shared/agent-domain`.
- Prefer `shared/agent-runtime` if the package primarily exposes executable
  agent runtime behavior.
- Prefer `shared/agent-domain` if the package primarily exposes provider-
  agnostic domain models, settings, permissions, and session-control concepts.
- Do not move the entire package as a single block if it mixes provider-
  agnostic domain primitives with provider-specific definitions. Split by
  owner first, then choose the target path.

Agreed third-level structure for `shared/protocol/`:

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

`shared/protocol/` owns the lowest shared wire/schema/API contracts that are
consumed across app, service, CLI, daemon, and provider modules. It corresponds
primarily to the current `packages/protocol` workspace. The target structure is
organized by protocol domain rather than by current file layout, so callers can
find the owner by the product contract they are consuming.

- `core/` owns base protocol primitives such as identifiers, timestamps,
  result envelopes, pagination, generic error shapes, and small shared
  contract helpers. It must not become a fallback bucket for unclear business
  domains.
- `rpc/` owns transport-neutral RPC request/response contracts, socket RPC
  contracts, daemon/server RPC shapes, and shared RPC error contracts.
- `sessions/` owns session, direct-session, session-control, session-message,
  session-metadata, session-authoring, fork, rollback, replay, execution-run,
  and related session protocol contracts.
- `accounts/` owns account, auth-facing account state, pairing, profile,
  social/account relationship, and user-scoped protocol contracts that are not
  concrete service auth implementation.
- `security/` owns protocol-level auth, approval, crypto/encryption envelopes,
  permission request shapes, and security policy contracts. Runtime crypto,
  secret storage, and service/provider enforcement remain with their owning
  modules.
- `features/` owns feature catalogs, feature decisions, capability payloads,
  compatibility gates, and shared feature policy contracts.
- `workspaces/` owns workspace, machine, host, ownership, transfer, and
  workspace location contracts. Source-control-specific contracts stay in
  `source-control/`.
- `source-control/` owns SCM repository, branch, worktree, stash, pull request,
  path-scope, policy, capability, and error-code contracts.
- `mcp/` owns shared MCP server, resource/tool registration, auth mode,
  selection, settings, and transport-independent MCP normalization contracts.
  Provider-specific MCP client/spawn/config behavior stays in `provider/`.
- `actions/` owns shared action, automation, voice action, sent-from, checklist,
  and user-intent contracts when they are protocol-level records rather than UI
  or command implementation.
- `tools/` owns shared tool schema, tool naming, tool metadata, alias,
  sub-agent-family, and tool-v2 protocol contracts.
- `prompts/` owns prompt, prompt-library, prompt-asset, structured-message, and
  LLM-task protocol contracts when they are shared contracts rather than
  provider prompt behavior.
- `providers/` owns provider wire/schema/API contracts, provider IDs, and
  provider-facing protocol records that must be shared across modules. It does
  not own provider runtime, install policy, managed-tool behavior, or
  provider-specific executable defaults.
- `system-tasks/` owns system task schemas, task state, task execution protocol
  records, and bootstrap/setup task contracts. Concrete setup CLI flows remain
  under `cli/setup-cli/`.
- `diagnostics/` owns bug report, server diagnostics, capability diagnostics,
  health/status payloads, and protocol-level diagnostic records.
- `generated/` owns generated protocol artifacts when the generation output
  must be checked in or consumed as a package artifact. Hand-maintained source
  must not be placed here.
- `testkit/` owns protocol fixtures, contract builders, compatibility fixtures,
  and package-local test helpers. Production modules must not depend on it.

`shared/protocol/` dependency rules:

```text
shared/protocol/<domain> -> shared/protocol/core
shared/protocol/<domain> -> external schema/validation libraries owned by the package
app/ cli/ daemon/ service/ provider/ -> shared/protocol public contracts
```

Forbidden dependencies:

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

Directory admission rules for physical migration:

- Do not create every target directory as an empty physical folder up front.
  Create a directory when current protocol source or an approved contract move
  has a clear owner there.
- A contract belongs in `shared/protocol/` only when it is consumed by multiple
  top-level modules or is a stable published protocol surface.
- Provider material under `packages/protocol/src/providers/**` must be
  classified before movement. Shared provider wire/schema/API contracts stay in
  `shared/protocol/providers/`; executable provider policy, runtime defaults,
  installables, manifests, and family adapter behavior move to `provider/`.
- `core/` may contain only narrow primitives used across protocol domains. A
  business concept that has a named protocol domain must live in that domain.
- `security/` may contain protocol-level security contracts only. Runtime
  enforcement, storage, or cryptographic implementation belongs to the owning
  service, app, daemon, or provider module.
- `generated/` and `testkit/` must stay isolated from production contract
  ownership.

Agreed third-level structure for `shared/agent-domain/`:

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
an agent. It defines provider-agnostic agent concepts, capability models,
permission semantics, session-control rules, runtime-kind semantics, model
descriptor contracts, and settings contract shapes. It does not start agents,
install provider CLIs, route CLI commands, own UI components, or implement
provider-specific behavior.

- `identity/` owns shared agent identity semantics such as agent IDs, flavor
  aliases, and cross-module agent ID inference from session metadata. It is not
  the provider registry; full provider registration, ordering, enablement, and
  manifest aggregation remain under `provider/catalog/`.
- `capabilities/` owns shared agent capability models and evaluators, including
  resume, handoff, session storage, local control, session listing, fork,
  rollback, and other provider-agnostic capability surfaces.
- `permissions/` owns provider-agnostic permission intents and permission mode
  semantics, including user-facing aliases and compatibility normalization.
  Provider-specific permission execution policy remains provider-owned.
- `modes/` owns provider-agnostic session mode and advanced mode semantics.
- `models/` owns model descriptor contracts, model-selection capability
  semantics, freeform model support, static-model contract shapes, and model
  apply behavior. Concrete provider model catalogs, such as static Claude,
  Codex, or Gemini model lists, belong under the provider-owned model/catalog
  surface.
- `session-control/` owns cross-provider session-control semantics, such as
  resume eligibility, handoff eligibility, existing-session automation
  eligibility, metadata override precedence, publish/normalization rules, and
  monotonic update policies. Provider runtime descriptor fields and
  provider-specific metadata extras remain provider-owned.
- `runtime-kinds/` owns the generic concept of agent runtime kinds and how a
  runtime kind modifies a capability surface. Concrete runtime-kind names and
  normalization rules that are meaningful only for one provider remain in that
  provider.
- `settings-contracts/` owns the provider settings definition shape, field
  metadata contract, registry contract, and shared validation shape. Concrete
  provider settings definitions, defaults, spawn extras, and account-setting
  policy remain under `provider/<providerId>/settings/` or the owning provider
  catalog surface.
- `connected-services/` owns shared semantics for connected-service
  compatibility, credential kind support, and account/service compatibility
  checks. Concrete provider-to-service support lists should come from provider
  manifests or catalog entries.
- `tools/` owns shared agent tools capability semantics such as native MCP
  delivery, shell-bridge delivery, unsupported tools, and support levels. MCP
  wire/schema contracts remain in `shared/protocol/mcp/` or
  `shared/protocol/tools/`; executable tool bridge runtime belongs in
  `shared/agent-runtime/` or provider runtime.
- `voice/` owns cross-module voice-agent domain semantics such as transcript
  normalization and shared voice turn concepts. UI copy, prompt copy, and
  concrete voice runtime behavior do not belong here.
- `diagnostics/` owns agent-domain diagnostics such as invalid capability
  combinations, invalid mode/settings shapes, metadata parse failures, and
  domain-level compatibility diagnostics. Provider CLI install/start failures
  belong in provider runtime or managed tools diagnostics.
- `testkit/` owns agent-domain fixtures, builders, fake capability surfaces,
  and package-local test helpers. Production code must not depend on it.

`shared/agent-domain/` dependency rules:

```text
shared/agent-domain/ -> shared/protocol/
shared/agent-runtime/ -> shared/agent-domain/
provider/ -> shared/agent-domain/
app/ cli/ daemon/ service/ -> shared/agent-domain/
```

Forbidden dependencies:

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

Current source classification guidance:

- `packages/agents/src/types.ts`, `resolveAgentIdFromFlavor.ts`, and
  `resolveAgentIdFromSessionMetadata.ts` are candidate sources for
  `identity/` after any provider catalog ownership is separated.
- `tools.ts`, `localControl.ts`, and
  `sessionControls/sessionCapabilities.ts` are candidate sources for
  `capabilities/`.
- `permissions/**` is a candidate source for `permissions/`.
- `sessionModes.ts` and provider-agnostic parts of `advancedModes.ts` are
  candidate sources for `modes/`.
- The model descriptor and model-selection contract portions of `models.ts`
  are candidates for `models/`; concrete provider model lists and
  provider-specific option enrichers must move to provider-owned model
  surfaces.
- Provider-agnostic files under `sessionControls/**`, such as vendor resume,
  handoff, automation eligibility, metadata precedence, publish, and monotonic
  update policies, are candidates for `session-control/`.
- Provider-specific files under `sessionControls/**`, such as Codex/OpenCode
  runtime descriptor extras, backend-mode resolution, runtime handles, and
  provider-specific metadata parsing, must stay out of `agent-domain` and move
  to provider-owned session/protocol areas.
- Generic runtime-kind types and capability override mechanics in
  `runtimeKinds.ts` are candidates for `runtime-kinds/`; provider-specific
  runtime kind definitions and normalization remain provider-owned.
- `providerSettings/types.ts` and the registry contract shape are candidates
  for `settings-contracts/`; concrete provider settings definitions under
  `providerSettings/definitions/**` are provider-owned.
- `providers/providerCliRuntime.ts` and
  `providers/providerCliInstallGuidance.ts` do not belong in `agent-domain`;
  they are provider/managed-tools or provider/catalog concerns.
- `providers/**` provider-specific helpers, such as Claude effort handling or
  Claude permission bridge request sources, must remain provider-owned unless a
  later review extracts a genuinely provider-agnostic primitive.

Allowed dependencies:

- Other `shared/` submodules when the direction is explicit and acyclic.
- External libraries.

Forbidden dependencies:

- No dependency on `app/`, `service/`, `cli/`, `daemon/`, `provider/`,
  `agents/`, `tests/`, or `ops/`.
- No provider-specific executable policy in generic shared code when a provider
  hook can express it.

Agreed third-level structure for `shared/mcp-domain/`:

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
records, and how MCP-specific tool/resource diagnostics are described. It does
not start MCP servers, own stdio/HTTP bridge processes, perform provider-specific
detection, read or decrypt secret plaintext, route CLI commands, or register MCP
SDK handlers directly.

- `server-catalog/` owns MCP server definition semantics such as server IDs,
  display names, transports, value-ref-bearing fields, enabled state, and
  catalog-level validation. Protocol wire schemas remain in
  `shared/protocol/mcp/`; runtime materialized command records remain in
  `shared/agent-runtime/mcp-bridge/`, `daemon/`, or `cli/`.
- `bindings/` owns server binding precedence and override merge rules for
  all-machines, machine, workspace, and similar scopes. It may expose pure
  resolver functions but must accept environment-specific path normalization or
  account-setting reads as injected inputs.
- `session-selection/` owns per-session MCP enablement, forced include/exclude
  semantics, selected-server state, missing-server reason classification, and
  managed-session selection rules. It does not persist session metadata or talk
  to daemon/service storage directly.
- `preview/` owns provider-agnostic MCP preview projections, including managed,
  built-in, detected, unavailable, and warning entries. CLI text output and app
  rendering remain outside this module.
- `detected-servers/` owns the shared detected-server record shape and generic
  dedupe/filter/portability rules. Concrete detection for Claude, Codex,
  OpenCode, or future providers remains provider-owned.
- `auth/` owns MCP auth-mode classification, credential-required semantics, and
  portability implications. It does not resolve or decrypt credentials.
- `value-refs/` owns value reference classification, redaction-safe metadata, and
  reference-shape helpers. Plaintext lookup, secret storage, and decryption
  remain runtime/security responsibilities.
- `tool-normalization/` owns MCP tool name parsing, canonical MCP tool naming,
  and provider-agnostic MCP tool input/result display normalization. Generic
  agent tool capability semantics remain in `shared/agent-domain/tools/`.
- `resource-contracts/` owns Happier MCP resource identifiers, payload contracts,
  and pure payload builders such as action-spec catalog resource contracts. MCP
  SDK server registration remains in the runtime/server owner.
- `diagnostics/` owns MCP-domain warning codes, reason codes, redaction-safe
  probe classifications, preview warnings, and selection/materialization
  diagnostics. Provider-specific install/start failures stay provider-owned.
- `testkit/` owns MCP-domain fixtures, catalog builders, binding builders,
  selection fixtures, preview builders, and package-local test helpers.
  Production modules must not depend on it.

`shared/mcp-domain/` dependency rules:

```text
shared/mcp-domain/ -> shared/protocol/
shared/agent-runtime/ -> shared/mcp-domain/
provider/ -> shared/mcp-domain/
app/ cli/ daemon/ service/ -> shared/mcp-domain/
```

Forbidden dependencies:

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

Current source classification guidance:

- Versioned MCP wire schemas in `packages/protocol/src/mcpServers/**` remain
  candidates for `shared/protocol/mcp/`; pure binding, effective-server, and
  managed-session selection resolvers can move to `shared/mcp-domain/bindings/`
  or `shared/mcp-domain/session-selection/` once schema ownership is separated.
- `apps/cli/src/mcp/preview/**` is a candidate source for
  `shared/mcp-domain/preview/` only after CLI output formatting and provider-
  specific detected-server mapping are separated.
- `apps/cli/src/mcp/providerDetection/**` does not move into
  `shared/mcp-domain/`; only shared detected-server record normalization or
  provider-agnostic dedupe rules may move to `detected-servers/`.
- `apps/cli/src/mcp/resources/registerHappierMcpResources.ts` should be split:
  resource URI and pure payload contract can move to `resource-contracts/`, while
  MCP SDK registration remains with the runtime/server owner.
- MCP-specific tool name and result normalization currently near
  `apps/cli/src/agent/tools/normalization/families/mcp.ts` is a candidate for
  `tool-normalization/` if it is independent from CLI rendering.
- Runtime materialization, bridge launch, stdio/http server lifecycle, temporary
  runtime config files, and plaintext value-ref resolution remain outside
  `shared/mcp-domain/`.

Directory admission rules for physical migration:

- Do not create every `shared/mcp-domain/` directory up front. Create a
  directory only when current source or an approved move has a clear owner there.
- MCP behavior belongs here only when it is provider-agnostic, deterministic,
  and expresses domain rules rather than runtime side effects.
- If code starts a process, patches provider config, probes an external server,
  resolves a secret plaintext value, or names a concrete provider, keep it out
  of `shared/mcp-domain/`.
- If code is a versioned wire/API schema consumed directly across modules, keep
  the schema in `shared/protocol/mcp/` and place derived domain policy in
  `shared/mcp-domain/`.

Agreed third-level structure for `shared/agent-runtime/`:

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
tool-aware runtime object. It must not own provider installation, provider CLI
resolution, concrete provider protocol behavior, app UI, CLI command routing,
daemon lifecycle internals, or service persistence.

The key boundary is that ACP runtime implementation remains under
`provider/provider-families/acp-runtime/`. `shared/agent-runtime/` may own only
provider-agnostic primitives that are reused across ACP, Claude SDK, OpenCode,
voice agent, execution-run, or future runtime families. Shared behavior caused
by a common provider protocol family still belongs in `provider/provider-families/`.

- `core/` owns minimal runtime interfaces and types such as agent backend
  contracts, agent runtime handles, agent messages, session identifiers, tool
  call identifiers, and backend factory contracts. It does not contain provider
  selection or provider-specific branching.
- `session-lifecycle/` owns generic start, load, resume, cancel, reset, dispose,
  cleanup, and lifecycle state helpers. Concrete startup side effects such as
  daemon reporting, terminal attachment persistence, and service writes remain
  in the owning module.
- `turn-delivery/` owns prompt delivery, in-flight steer, interrupt,
  cancellation, flush, response-completion waiting, and abort-like error
  classification. It should express turn mechanics without hard-coding provider
  names.
- `permission-flow/` owns runtime permission request routing, permission queues,
  permission-mode runtime synchronization, and pending permission cleanup.
  Permission semantics remain in `shared/agent-domain/permissions/`; ACP option
  mapping remains in `provider/provider-families/acp-runtime/permissions/`.
- `tool-delivery/` owns runtime delivery of Happier tools to an agent, such as
  native MCP delivery, shell-bridge delivery, unsupported delivery handling, and
  runtime-side tool injection. Tool capability semantics remain in
  `shared/agent-domain/tools/`; provider-specific tool policy remains provider
  owned.
- `mcp-bridge/` owns session-scoped MCP runtime primitives such as per-session
  MCP bridge/server lifecycle, session-agent HTTP/stdio bridge composition, and
  runtime materialization of MCP servers. MCP wire/schema/config contracts remain
  in `shared/protocol/mcp/` or `shared/mcp-domain/`.
- `local-control/` owns provider-agnostic local/remote control switching,
  provider-attach state publication, safe remote handoff state machines, and
  local turn lifecycle snapshots. UI mounting, terminal rendering, daemon
  control, and concrete provider attach execution remain in their owners.
- `execution-runs/` owns the backend-agnostic substrate for bounded and
  long-lived execution runs: run state, intent profile contracts, resume/send/
  stop/action flow, and runtime manager primitives. Concrete engines such as
  CodeRabbit or provider-specific execution-run factories remain provider or
  feature owned.
- `voice-agent/` owns backend-agnostic voice-agent runtime orchestration such as
  voice agent manager state, chat/commit backend coordination, resume handles,
  idle reaping, and stream-delta state. Voice semantics belong in
  `shared/agent-domain/voice/`; audio/native runtime modules remain in their
  shared native packages.
- `process-io/` owns provider-agnostic subprocess and stream primitives such as
  Node stream to Web Stream adapters, signal/termination helpers, process-output
  capture helpers, and safe cross-platform IO utilities. Provider CLI resolution,
  installation, and ACP spawn wrappers remain in provider-owned layers.
- `state-sync/` owns runtime synchronization helpers for session metadata, agent
  state, runtime overrides, mode/model/config updates, and startup metadata
  merge policy when they are provider-agnostic. Concrete API writes, daemon
  reporting, and storage implementation remain outside this module.
- `diagnostics/` owns provider-agnostic runtime diagnostics such as runtime error
  classification, recoverability signals, redaction helpers, stderr summary
  helpers, and debug artifact contracts. Provider-specific diagnostics remain
  provider owned.
- `testkit/` owns fake runtimes, fake backends, turn-delivery harnesses,
  MCP-bridge harnesses, lifecycle fixtures, and package-local test helpers.
  Production modules must not depend on it.

Allowed dependencies:

```text
shared/agent-runtime/ -> shared/protocol/
shared/agent-runtime/ -> shared/agent-domain/
shared/agent-runtime/ -> shared/mcp-domain/
provider/ -> shared/agent-runtime/
cli/ daemon/ app/ service/ -> shared/agent-runtime/
```

Forbidden dependencies:

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

Current source classification guidance:

- `apps/cli/src/agent/core/AgentBackend.ts` and related backend/message factory
  types are candidate extraction sources for `core/` after CLI-only naming and
  product-specific references are removed.
- Provider-agnostic pieces currently near
  `apps/cli/src/agent/runtime/runStandardAcpProvider.ts`,
  `runPermissionModePromptLoop.ts`, and `turnDelivery.ts` are candidate sources
  for `session-lifecycle/`, `turn-delivery/`, `permission-flow/`, and
  `state-sync/`. The ACP-specific runner shape and Ink/CLI/API side effects must
  not move as-is.
- `apps/cli/src/agent/runtime/createHappierMcpBridge.ts`,
  `apps/cli/src/mcp/startHappyServer.ts`, and
  `apps/cli/src/mcp/runtime/resolveRunnerMcpServers.ts` are candidate sources
  for `mcp-bridge/` only after MCP domain contracts, credential materialization,
  packaged runtime resolution, and CLI-specific launch behavior are separated.
- `apps/cli/src/agent/localControl/**` is a candidate source for
  `local-control/` when provider attach execution, UI mounting, daemon control,
  and API writes are split from the provider-agnostic control state machines.
- `apps/cli/src/agent/executionRuns/runtime/**` is a candidate source for
  `execution-runs/`; provider-specific execution-run factories, review engines,
  and intent-specific product policies remain outside this module.
- `apps/cli/src/agent/voice/agent/**` is a candidate source for `voice-agent/`
  when prompt content, voice-domain semantics, and provider-specific backend
  creation are separated.
- Generic stream/process helpers near `apps/cli/src/agent/acp/nodeToWebStreams.ts`
  may be candidates for `process-io/` only if they are reused outside ACP. ACP
  spawn and ACP protocol lifecycle code remain in
  `provider/provider-families/acp-runtime/`.
- `packages/agents` must still be split by owner. Domain semantics belong in
  `shared/agent-domain/`; runtime execution primitives belong here only when
  they are provider-agnostic and executable.

Directory admission rules for physical migration:

- Do not create every `shared/agent-runtime/` directory up front. Create a
  directory only when a current source or approved move has a clear owner there.
- A primitive belongs in `shared/agent-runtime/` only when it is reusable across
  at least two runtime families or when it defines a stable runtime contract
  consumed by multiple top-level modules.
- If behavior is shared only because providers use the same protocol family,
  launch model, or transport profile, prefer `provider/provider-families/`.
- If behavior names a concrete provider, chooses provider defaults, resolves a
  provider CLI, installs a provider tool, or interprets provider-specific output,
  keep it in `provider/`.
- If behavior controls a long-running daemon process or persists daemon state,
  keep it in `daemon/`; expose only narrow runtime contracts to
  `shared/agent-runtime/` when needed.

Agreed third-level structure for `shared/transfers/`:

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
route is unavailable. It must not own actual file IO, RPC handlers, HTTP or
WebSocket stream implementations, service relay storage, direct-peer probing,
UI copy, CLI commands, high-level session handoff orchestration, or protocol
wire schemas.

- `route-selection/` owns pure route negotiation for routes such as
  `direct_peer`, `server_routed_stream`, and `machine_rpc_direct`. It chooses a
  route from feature flags, caller-provided availability inputs, and preferred
  strategy order. It does not probe endpoints or perform the transfer.
- `availability/` owns stable transfer availability results and machine-readable
  unavailable reason codes. UI and CLI layers may map those reason codes to
  product copy, but the shared layer should avoid owning user-facing output
  formatting.
- `server-routed-policy/` owns server-routed transfer feature policy, maximum
  byte limits, and file-size limit checks. Process environment reads should be
  isolated behind injected inputs or narrow policy helpers so the core policy
  remains deterministic.
- `endpoint-fingerprints/` owns safe transfer endpoint fingerprinting rules,
  including stripping untrusted URL userinfo, query, and hash components before
  cache fingerprints are built.
- `route-viability-cache/` owns in-memory route viability cache records, TTL
  policy, cache keys, and invalidation rules. It does not own network probing,
  persistent storage, or connection lifecycle.
- `diagnostics/` owns transfer-domain reason codes, failure categories, and
  redaction-safe diagnostic shapes. Service, daemon, CLI, and app-specific error
  transport remains outside this module.
- `testkit/` owns transfer fixtures, feature snapshots, endpoint candidates,
  route availability builders, cache records, and package-local test helpers.
  Production modules must not depend on it.

`shared/transfers/` dependency rules:

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

Forbidden dependencies:

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

Current source classification guidance:

- `packages/transfers/src/route/resolveMachineTransferRoute.ts` and
  `packages/transfers/src/route/resolveAppSessionTransferRoute.ts` are candidate
  sources for `route-selection/`.
- `packages/transfers/src/route/resolveAppSessionTransferAvailability.ts` is a
  candidate source for `availability/`, but user-facing copy should be reviewed
  during physical migration and may move to app/CLI callers if it is presentation
  text rather than a shared diagnostic contract.
- `packages/transfers/src/policy/serverRoutedTransferPolicy.ts` is a candidate
  source for `server-routed-policy/`. Any direct `process.env` use should remain
  a narrow helper rather than leaking into route-selection logic.
- `packages/transfers/src/cache/fingerprintTransferEndpoints.ts` is a candidate
  source for `endpoint-fingerprints/`.
- `packages/transfers/src/cache/createTransferRouteViabilityCache.ts` and
  `packages/transfers/src/cache/createMachineTransferRouteCache.ts` are candidate
  sources for `route-viability-cache/`.
- Transfer stream envelopes, endpoint candidate schemas, and feature payload
  schemas in `packages/protocol` remain under `shared/protocol/`.
- UI runtime mapping in `apps/ui/sources/sync/domains/transfers/**` and CLI
  machine transfer wrappers remain with their app/CLI owners, consuming
  `shared/transfers/` rather than moving wholesale into it.

Directory admission rules for physical migration:

- Do not create every `shared/transfers/` directory up front. Create a directory
  only when current source or an approved move has a clear owner there.
- A primitive belongs in `shared/transfers/` only when it is deterministic,
  provider-agnostic, and reusable by more than one top-level module.
- If behavior performs real IO, opens streams, handles RPC methods, probes a
  direct peer, persists transfer state, or renders user-facing messages, keep it
  in the owning app, CLI, daemon, or service module.
- If a transfer shape is a versioned wire/API schema, keep the schema in
  `shared/protocol/` and place derived transfer policy in `shared/transfers/`.

Agreed third-level structure for `shared/connection-supervisor/`:

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
rules. It must not own socket.io/WebSocket/fetch/axios implementations, CLI
loopback probe implementations, UI token storage, React Native `AppState`,
browser `window`/visibility listeners, endpoint supervisor pools, UI/CLI error
classes, transfer route selection, daemon state persistence, or service API
endpoints.

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

`shared/connection-supervisor/` dependency rules:

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

Forbidden dependencies:

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

Boundary with `shared/transfers/`:

```text
connection-supervisor reports endpoint/connection state
transfers resolves transfer routes from supplied state and feature policy
app/cli/daemon performs the actual transfer
```

`shared/connection-supervisor/` and `shared/transfers/` should be composed by
their consumers rather than depending on each other directly.

Primary affected areas:

- `packages/connection-supervisor` is the direct source package for this module.
- `apps/cli/src/api/**` is the largest consumer surface, including machine and
  session socket connection supervision, loopback readiness probes, supervised
  request handling, and daemon connectivity coordination.
- `apps/ui/sources/sync/**` is the main app consumer surface, including endpoint
  supervisor pools, endpoint readiness probes, sync socket transports,
  connectivity gating, and connection-status display.
- `apps/stack`, `apps/cli` packaging, Dockerfile, CI, and package artifact tests
  are affected by the internal workspace bundling path.

Current source classification guidance:

- `packages/connection-supervisor/src/managedConnectionTypes.ts` and
  `managedEndpointSupervisorTypes.ts` are candidate sources for `state-model/`
  and `readiness-contracts/`.
- `packages/connection-supervisor/src/createManagedConnectionSupervisor.ts` is
  the candidate source for `transport-supervision/`.
- `packages/connection-supervisor/src/createManagedEndpointSupervisor.ts` is the
  candidate source for `endpoint-supervision/`.
- `packages/connection-supervisor/src/defaultManagedConnectionPolicy.ts` and
  `reconnectBackoff.ts` are candidate sources for `retry-policy/`.
- `packages/connection-supervisor/src/managedConnectionEvents.ts` is a candidate
  source for `diagnostics/`.
- CLI request supervision helpers under
  `apps/cli/src/api/connection/requestSupervision/**` and UI helpers under
  `apps/ui/sources/sync/runtime/connectivity/**` may only move into
  `request-supervision/` after concrete error types, fetch/axios/runtimeFetch,
  token lookup, and UI/CLI-specific behavior are separated.
- `createLoopbackReadinessProbe`, `createEndpointReadinessProbe`, socket
  transport adapters, endpoint supervisor pools, app visibility listeners, and
  token storage remain in their app/CLI owners.

Directory admission rules for physical migration:

- Do not create every `shared/connection-supervisor/` directory up front. Create
  a directory only when current source or an approved move has a clear owner
  there.
- A primitive belongs here only when it supervises connection or endpoint state
  without owning a concrete transport, platform lifecycle, request executor, or
  product-specific error surface.
- If code imports socket.io, React Native app state, browser globals, axios,
  runtimeFetch, token storage, or CLI/UI error classes, keep it in the owning
  app/CLI module or split out the pure logic first.
- Do not introduce a dependency from `shared/connection-supervisor/` to
  `shared/transfers/`; pass connection state into transfer policy from the
  consumer layer.

Agreed third-level structure for `shared/release-runtime/`:

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

`shared/release-runtime/` dependency rules:

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

Forbidden dependencies:

```text
shared/release-runtime/ -> app/
shared/release-runtime/ -> cli/
shared/release-runtime/ -> service/
shared/release-runtime/ -> provider/
shared/release-runtime/ -> ops/
shared/release-runtime/ -> shared/first-party-runtime/
shared/release-runtime/ -> tests/
```

Primary affected areas:

- `packages/release-runtime` is the direct source package.
- `apps/cli` is the largest consumer surface: CLI self-update, daemon/service
  ownership, relay runtime, doctor repair, provider dependency download, and
  release channel handling.
- `packages/cli-common` consumes release runtime for first-party runtime payload
  preparation, install layout support, managed provider release asset extraction,
  and component catalogs.
- `apps/stack` consumes release runtime for self-host runtime and companion CLI
  download/install flows.
- `packages/relay-server` consumes release asset, checksum, minisign, and
  extraction helpers for server runner related release assets.
- `scripts/pipeline` consumes release ring/channel metadata and release helper
  logic from repo operations.
- `apps/cli`, `apps/stack`, and `packages/relay-server` packaging, Dockerfiles,
  CI, and artifact tests are affected by the internal workspace bundling path.

Current source classification guidance:

- `packages/release-runtime/src/releaseRings.ts` is the candidate source for
  `release-rings/`.
- `packages/release-runtime/src/github.ts` is the candidate source for
  `release-sources/`; GitHub publishing scripts remain in `ops/`.
- `packages/release-runtime/src/assets.ts` is the candidate source for
  `asset-resolution/`.
- `packages/release-runtime/src/http.ts` is the candidate source for
  `download-transport/`, scoped only to release downloads.
- `packages/release-runtime/src/checksums.ts` and `minisign.ts` are candidate
  sources for `integrity-verification/`.
- `packages/release-runtime/src/verifiedDownload.ts` is the candidate source for
  `verified-downloads/`.
- `packages/release-runtime/src/extractPlan.ts` is the candidate source for
  `extraction-plans/`.
- `packages/cli-common/src/firstPartyRuntime/**` is not part of
  `shared/release-runtime/` as a whole. It is a consumer and likely belongs to a
  separate `shared/first-party-runtime/` discussion because it owns install
  layout, version promotion, rollback, shims, and component lifecycle.
- CLI self-update file replacement and process handling remain under `cli/`.
- Release publishing scripts remain under `ops/`.

Directory admission rules for physical migration:

- Do not create every `shared/release-runtime/` directory up front. Create a
  directory only when current source or an approved move has a clear owner there.
- A primitive belongs here only when it reads release metadata, resolves release
  assets, downloads release artifacts, verifies integrity, or plans extraction
  without owning component installation or release publication.
- If behavior publishes releases, installs a product component, promotes or
  rolls back a version, rewrites shims, replaces a running CLI binary, registers
  services, starts relay/server business logic, or executes extraction commands,
  keep it in the owning higher-level module.
- Keep Node runtime capabilities out of mobile/Web runtime paths. Only
  `release-rings/` should be considered safe for app/bootstrap-style consumers.

### `tests/`

Owns verification assets.

Expected scope:

- Unit, integration, end-to-end, provider, database contract, and stress tests.
- Cross-package testkit and fixtures.
- Test runner scripts that are not general repository operations.

Current likely sources:

- `packages/tests`
- test suites currently embedded inside each workspace, where appropriate.

Allowed dependencies:

- All production modules.

Forbidden dependencies:

- Production modules must not depend on `tests/`.
- Test helpers must not become hidden runtime dependencies.

### `ops/`

Owns repository operation, build, release, CI, and environment orchestration.

Expected scope:

- CI workflows.
- Docker and Dagger assets.
- Release scripts.
- Build pipeline scripts.
- Repository-local orchestration scripts.
- Tooling validation scripts.

Current likely sources:

- `scripts`
- `.github`
- `docker`
- `dagger`
- relevant parts of `apps/stack`
- root-level operational configuration that is not product configuration.

Allowed dependencies:

- May call or orchestrate any module through commands or scripts.

Forbidden dependencies:

- Production runtime modules must not import from `ops/`.
- `ops/` should not own application, provider, or protocol logic.

### `docs/`

Owns human-facing documentation and design records.

Expected scope:

- Architecture documents.
- Development references.
- Migration plans.
- Product and provider documentation intended for human readers.
- Design snapshots for this reorganization.

Current likely sources:

- `docs`
- `HAPPIER_DEVELOPMENT_REFERENCE.md`
- `HAPPIER_LOCAL_DEV_NOTES.md`
- architecture/reference documents currently located at the repository root.

Allowed dependencies:

- Documentation may reference any module.

Forbidden dependencies:

- No production runtime import dependency.
- Documentation must not become the source of truth for machine-readable runtime
  config.

## Dependency Direction Rules

The intended dependency direction is:

```text
app/      cli/      daemon/      service/
  \        |          |            /
   \       |          |           /
    \      |          |          /
            provider/
                |
              shared/
```

Support layers:

```text
tests/  -> may depend on production modules
ops/    -> may orchestrate production modules through commands/scripts
docs/   -> may describe all modules
agents/ -> may guide people/tools, but is not production runtime input
```

Hard rules:

- `shared/` is the lowest production layer.
- `provider/` depends on `shared/`, not on app/CLI/service implementations.
- `app/`, `cli/`, `daemon/`, and `service/` may consume `provider/` and
  `shared/`.
- `agents/` is not the same thing as `provider/`.
- `tests/`, `ops/`, `docs/`, and `agents/` are not allowed to become production
  runtime dependency roots.

## Second-Level Directory Design Scope

The next design step is to define second-level directories under each first-
level module. That step should:

- Preserve current package ownership as much as possible.
- Avoid splitting one package across unrelated top-level modules unless there is
  a clear ownership reason.
- Identify import aliases and workspace package names before physical movement.
- Define temporary migration aliases only if absolutely necessary.
- Prefer direct import updates over long-lived compatibility shims.
- Keep provider-specific executable behavior inside provider-owned folders.
- Keep AI-agent guidance under `agents/`, not under provider runtime folders.

## Safety Rules For The Future Physical Move

Before moving files:

- Inventory all workspace package names and import aliases.
- Build a dependency graph from current imports.
- Decide whether workspace package names change with paths or remain stable.
- Move one module group at a time.
- After each group, run the smallest relevant typecheck/test lane.
- After all groups, run root workspace install validation, typecheck, unit tests,
  and the relevant provider/test lanes.
- Preserve existing untracked local development files unless explicitly asked to
  remove them.

This design document is only a snapshot of the agreed first-level module
contract. It is not an implementation plan.
