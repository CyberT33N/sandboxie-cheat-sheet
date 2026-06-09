# Boxed-Owned-Toolchain NX

## Architectural status

This is the **preferred Nx architecture path** in this repository.

Use this area when the architecture is:

- fully boxed authoring
- box-local runtime mirroring
- box-local mutable working state
- explicit promotion instead of live shared authoring

The legacy host-sync write-up remains only as a secondary reference:

- `docs\applications\version-control\monorepo\nx\architectures\host-sync\overview.md`

## Role of this file

This file is now the **TOC / entrypoint** for the boxed-owned-toolchain Nx source of truth.

The previous dense overview has been split by concern so the Nx domain can stay:

- single-source-of-truth oriented
- easier to scale by bounded context
- easier to re-reference from VS Code and PNPM documents without duplicating Nx detail
- easier to keep current-state-only as the bootstrap and shell surfaces evolve

No previously established boxed-owned-toolchain Nx truth is intentionally dropped by this split. The content now lives in the documents below.

## Current reference snapshot

The current boxed-owned-toolchain Nx reference truth is:

1. box-local Nx native cache and socket surfaces remain the baseline
2. the Nx runtime contract is bootstrap-owned and environment-driven
3. plain `nx` is now provided through bootstrap-generated wrappers:
   - `nx`
   - `nx.cmd`
   - `nx-cli.cjs`
4. the plain `nx` command surface is validated in the boxed project terminal
5. `pnpm exec nx ...` remains a separate failure surface
6. `nx:run-commands` / `command` targets on Windows still sit on a shell-selection boundary
7. the bootstrap now sets `ComSpec` / `COMSPEC` to the box-local Git Bash executable as the prioritized no-loosening shell-selection solution
8. the currently validated boxed target set now succeeds without manual per-command `ComSpec=bash ...` overrides

## Current architectural boundary

The current accepted solution-space boundary is:

- the box/bootstrap layer must adapt to the project command surface
- the first-class answer is **not** to refactor the project away from Nx `run-commands` / `command` usage just to fit the sandbox

That boundary is part of the current architecture discussion and is therefore recorded here explicitly.

## Domain map

### Architecture rationale

- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\architecture-rationale.md`

Owns:

- the enterprise-grade / domain-driven interpretation
- the twelve-factor reading of the current Nx model
- why Nx belongs to its own bounded context instead of being absorbed into the VS Code method docs

### Cache boundary

- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\cache-boundary.md`

Owns:

- box-local cache posture
- socket and native-cache boundary rationale
- why host-shared native-cache reuse is not the baseline

### Execution surfaces

- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\execution-surfaces.md`

Owns:

- the validated failure chain
- direct `nx` versus `pnpm exec nx`
- `nx:run-commands` shell behavior on Windows
- the current `ComSpec`-related findings

### Runtime contract

- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\runtime-contract.md`

Owns:

- `NX_DAEMON`
- `NX_SOCKET_DIR`
- `NX_ISOLATE_PLUGINS`
- `NX_NATIVE_FILE_CACHE_DIRECTORY`
- the current boxed-owned-toolchain runtime posture

### Bootstrap integration

- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\bootstrap-integration.md`

Owns:

- the exact live shared bootstrap files
- the current Nx wrapper implementation in shared bootstrap
- how the project and maintenance entrypoints call into that implementation
- how the current command surface is validated

## Cross-domain references

- PNPM lifecycle and command-surface behavior:
  `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\lifecycle-and-command-surface.md`
- Sandboxie shell-spawn troubleshooting:
  `docs\troubleshooting\sandboxie\process-spawning\cmd-based-shells.md`
- VS Code boxed-owned-toolchain bootstrap:
  `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\scripts.md`

## Related

- `docs\cli\shell\general.md`
- `docs\applications\version-control\monorepo\nx\general.md`
- `docs\applications\version-control\monorepo\nx\architectures\host-sync\overview.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\overview.md`
- `docs\troubleshooting\sandboxie\process-spawning\cmd-based-shells.md`
- `docs\troubleshooting\sandboxie\privacy-mode\host-user-space-vs-box-root.md`
