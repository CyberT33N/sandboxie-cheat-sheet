# Toolchain

## Scope

This area documents the shared toolchain model used by the boxed-owned-toolchain method.

## Core model

The method uses:

- canonical shared toolchain runtimes
- fixed versioned binaries
- bootstrap-selected runtime wiring

It does **not** rely on mutable host shims or implicit runtime switching.

## Shared toolchain surfaces

The canonical shared toolchain root is:

```text
C:\shared\sandbox-toolchains\dev\
```

This currently contains:

- `git\`
- `node\`
- `pnpm\`
- `bootstrap\`

## Runtime selection

Bootstrap selects the relevant shared binaries and wires them into the process environment.

This is why the method does not treat:

- host-installed toolchains
- `$PROFILE`
- workspace-local absolute path configuration
- `nvm use`

as the architecture center.

## Current toolchain split

- Git: `PortableGit` under `dev\git\2.54.0\`
- Node: versioned runtimes under `dev\node\...`
- pnpm: unpacked CLI content under `dev\pnpm\11.2.2\package\bin\pnpm.cjs`

## Related

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\git.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\node.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\pnpm.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\host-state.md`
