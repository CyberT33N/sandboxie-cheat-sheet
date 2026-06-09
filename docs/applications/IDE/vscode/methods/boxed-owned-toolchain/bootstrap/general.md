# Bootstrap

## Scope

This document explains why bootstrap is the architecture center of the boxed-owned-toolchain method.

## Why bootstrap is mandatory

Bootstrap is the place where the method:

- selects the correct shared artifacts
- initializes the local runtime state
- mirrors the selected runtime binaries into the local execution tree
- copies canonical catalog files
- initializes seed-backed paths
- mirrors the shared extension store locally
- prepares local maintenance authoring state
- applies the current Nx runtime contract
- wires the toolchain environment
- launches `Code.exe` or `code.cmd`

## Why not `$PROFILE`

`$PROFILE` is not the architecture center because it is:

- implicit
- user-specific
- drift-prone
- hard to govern

## Why not workspace settings for toolchain selection

Workspace settings are not the canonical place for:

- machine-local absolute binary paths
- shared runtime paths
- bootstrap-selected command surfaces

Those belong in the bootstrap layer.

## Why no box-specific shell copies

The final method does not rely on project-specific `cmd.exe` / `powershell.exe` copies as an architecture principle.

The final design prefers:

- a locally mirrored shell runtime selected by bootstrap
- canonical VS Code terminal settings
- explicit bootstrap-provided environment

## Bootstrap layer split

The current bootstrap split is:

- `core\`
- `platforms\vscode\`
- `stacks\node\`
- `stacks\python\`
- `stacks\starship\`
- `projects\test-mono\bootstrap\`

This separates:

- generic primitives
- VS Code platform orchestration
- Node stack wiring
- project adapter logic

## Related

- `docs\cli\shell\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\shared-layout.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\scripts.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\scripts.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\bootstrap-integration.md`
