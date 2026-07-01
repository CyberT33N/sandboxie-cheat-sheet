# Nx Architecture Rationale

## Scope

This document explains **why** the current boxed-owned-toolchain Nx design is architecturally correct.

It is intentionally not the bootstrap implementation document and not the shell-troubleshooting log.

It owns:

- the enterprise-grade architecture reading
- the domain-driven interpretation
- the twelve-factor interpretation
- the trust-boundary reasoning for the current Nx posture

## Bounded-context view

In the current repository architecture, Nx belongs to its own bounded context:

- **Nx bounded context**
  - workspace graph
  - task orchestration
  - daemon/socket/plugin topology
  - native cache posture

It is related to, but not identical with:

- the **VS Code method context**
  - editor startup
  - integrated terminal defaults
  - extension/catalog/runtime orchestration

- the **PNPM context**
  - dependency materialization
  - lifecycle shell behavior
  - package-manager command surface

- the **Sandboxie context**
  - process mediation
  - token/integrity model
  - file/registry/IPC boundaries

- the **Bootstrap context**
  - environment construction
  - local mirror initialization
  - command-surface publication

This split matters because the current problem is not “VS Code is broken” and not “PNPM is broken”.

It is specifically an **Nx execution-surface problem inside a boxed Windows shell model**.

So the Nx domain must own the explanation instead of leaving that behavior fragmented across VS Code, PNPM, and Sandboxie notes.

## Bootstrap as anti-corruption layer

From a domain-driven perspective, bootstrap is the anti-corruption layer between:

- canonical shared toolchain assets
- the local boxed runtime
- the project's command surface

For Nx, bootstrap currently does all of the following:

- selects the local mirrored Node runtime that will host Nx
- publishes the standard workspace-local Nx command surface through `pnpm exec nx ...`
- optionally publishes a legacy plain-`nx` alias only when a team explicitly enables it
- sets the Nx runtime environment variables
- keeps cache and socket locations box-local
- owns the daemon reset/start preflight for installed Nx workspaces

That is exactly what an anti-corruption layer should do:

- keep outside infrastructure explicit
- normalize runtime inputs before the application/tooling boundary
- prevent the project shell from depending on incidental host state

## Enterprise-grade interpretation

The current Nx contract is enterprise-correct because it prefers:

- explicit background-process ownership
- explicit daemon lifecycle
- fewer hidden mutable runtime surfaces
- more explicit startup-time configuration
- clearer box-local ownership of execution state

That is why the current configuration does **not** treat:

- long deep socket paths
- isolated plugin workers
- host-shared native cache

as unquestioned defaults.

In the current validated repository state, the daemon itself is no longer treated as something to disable by default.

Why:

- the real watch/serve path needs the daemon
- the earlier `Daemon is not running` failure proved that the boxed runtime must support that contract explicitly
- the later `Io error. Look inside err_kind for more details.` failure class showed that daemon state also needs explicit hygiene

So the current boxed-owned-toolchain answer is:

- enable the daemon by default
- keep the socket path short and box-local
- keep plugin isolation conservative
- and let bootstrap reset and start the daemon explicitly for installed Nx workspaces

That still treats daemon/runtime topology as something that must be governed deliberately instead of left to ambient host state.

Additional runtime topology means:

- more processes
- more sockets
- more coordination
- more timeout surfaces
- more mediation paths through Sandboxie

Reducing or governing those surfaces is a valid enterprise choice, not a regression in architectural maturity.

## Twelve-factor interpretation

The current Nx design maps cleanly to several twelve-factor principles.

### Config in the environment

The core Nx runtime contract is environment-driven:

- `NX_DAEMON`
- `NX_SOCKET_DIR`
- `NX_ISOLATE_PLUGINS`
- `NX_NATIVE_FILE_CACHE_DIRECTORY`

These are injected by bootstrap at process start, not hidden in editor UI settings and not left to ad hoc per-terminal mutation.

That is the correct twelve-factor reading:

- config belongs in the environment
- config is explicit at startup
- config is separate from code

### Dependencies are explicit

The current Nx command surface is explicit:

- `node <resolved nxCli>`
- `pnpm exec nx`
- optional legacy plain-`nx` wrapper surface when a team explicitly enables it

The method does not depend on:

- a global host Nx install
- a mutable host shim
- an undocumented ambient alias

### Build/release/run separation

The current architecture keeps the layers distinct:

- shared provisioning chooses which Node and pnpm artifacts exist
- project bootstrap selects which of those artifacts become active at runtime
- Nx then executes inside that already-governed runtime

Nx runtime behavior is therefore not silently fused with package provisioning or editor settings.

### Processes should be disposable and stateless where possible

This does not mean Nx has no cache or no socket.
It means those runtime artifacts should not be promoted to canonical shared truth.

The current design therefore keeps:

- socket surfaces local
- native cache local
- runtime wrappers local
- daemon lifecycle bootstrap-owned

so they remain disposable execution artifacts, not shared stateful infrastructure.

## Why the current answer is not “just refactor the project”

For the current architecture discussion, the accepted solution space is:

- the sandbox/bootstrap layer must adapt to the project's legitimate command surface

The answer is **not**:

- refactor Nx `run-commands`
- rewrite the project command graph
- or bypass the project orchestration model simply because the strict box blocks one Windows shell path

Why this matters:

- the project's orchestration surface is part of the application's real operating contract
- other developers must be able to run the same workspace commands
- the sandbox architecture should prove compatibility with that contract rather than define success by narrowing the contract away

So the first-class architectural task is:

- make the boxed environment compatible with the project's legitimate Nx invocation patterns

## Why the current first focus is `ComSpec`, not token-loosening

The currently strongest validated evidence is:

- direct guard scripts succeed
- direct Electron smoke succeeds
- direct `nx` command surface works
- `node -> spawn(cmd.exe)` fails with `spawn EPERM`
- `node -> spawn(bash)` works
- manually setting `ComSpec=bash` makes `nx run backend:port-guard` and `nx run frontend:port-guard` succeed

That means the current highest-probability runtime lever is:

- shell selection for Windows shell-based command execution

not:

- disabling security isolation
- dropping child-process tokens as a baseline
- or refactoring project targets away from Nx

## What remains outside the preferred solution space

The following remain technically possible but architecturally weaker:

- Green-box / compartment-mode compatibility settings
- token-bypass style settings
- broad box-loosening rules

Those options exist, but they weaken the fortress-style isolation model.

So they are fallback compatibility levers, not the preferred first answer.

## Related

- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\cache-boundary.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\execution-surfaces.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\runtime-contract.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\bootstrap-integration.md`
