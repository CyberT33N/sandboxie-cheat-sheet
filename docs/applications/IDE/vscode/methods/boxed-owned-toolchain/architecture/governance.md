# Governance

## Scope

This document explains the author/consumer model and governance rules of the boxed-owned-toolchain method.

## Single-writer model

The method uses a strict single-writer governance model for shared IDE assets.

### Shared canonical surfaces

The following are treated as canonical shared assets:

- VS Code runtime
- canonical `settings.json`
- canonical `keybindings.json`
- canonical snippets
- canonical extension store
- canonical seed material for path-hardcoded extension state
- versioned shared Git / Node / pnpm runtimes
- versioned shared Starship runtime

### Author

The Maintenance Box is the only global author for:

- catalog content
- extensions
- seed material
- maintenance `user-data`

### Consumers

Project boxes consume those shared artifacts and materialize local runtime copies as needed.

They do not directly mutate the shared canonical state.

## Why this matters

This model exists to preserve:

- clear authorship
- auditability
- reproducibility
- predictable promotion of global changes
- blast-radius isolation between projects

Without this split, project boxes would be able to overwrite globally shared IDE assets and create cross-project drift.

## Shared versus local state

### Shared

Shared holds:

- canonical source artifacts
- runtime binaries
- seeds
- bootstrap logic
- runner/export helper surfaces

### Local to each project box

Each project box owns:

- `user-data`
- local mirrored extension runtime copy
- `globalStorage`
- `.roo`
- `workspaceStorage`
- logs
- caches
- sessions

## Promotion model

Changes that should become canonical must be promoted through the Maintenance Box, not authored ad hoc from project boxes.

Examples:

- install a new extension into the shared extension store
- update the canonical `settings.json`
- update seed-backed state such as `globalStorage` payloads
- update seed-backed `.roo` content

## Extension store governance

The project box does **not** use the shared extension directory as its live writable runtime `--extensions-dir`.

Instead:

- Maintenance Box writes to the canonical shared extension store
- bootstrap mirrors that store into a box-local project extension directory
- the project VS Code instance uses the local mirrored directory

This avoids turning the canonical extension store into a multi-writer surface.

## Seed governance

For path-hardcoded or difficult-to-relocate extension data:

- shared keeps the canonical seed
- the box receives a local copy
- promotion remains explicit and controlled

This applies to cases such as:

- `.roo`
- extension-managed `globalStorage` state

## Bootstrap governance

Bootstrap is the architecture center for runtime selection and local-state materialization.

It is the governed place where the method:

- selects shared runtimes
- mirrors shared artifacts
- copies canonical catalog files
- initializes seeds
- injects toolchain paths
- launches `Code.exe` or `code.cmd`

This is why the method does **not** treat:

- `$PROFILE`
- workspace-local absolute toolchain settings
- mutable host shims

as the source of truth.

## Host governance

Final host target state:

- no host VS Code
- no host Git
- no host Node
- no host pnpm
- no host nvm
- no host Starship binary

## Related

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\architecture\target-state.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boxes\maintenance-box.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boxes\project-box.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\state\extensions-seeds-and-local-state.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\overview.md`
