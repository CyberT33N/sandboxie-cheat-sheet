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

## Optional prompt/runtime addition

When a boxed shell needs to execute a prompt binary locally, the same shared toolchain root may additionally contain:

- `starship\`

This is not because `Starship` becomes project dependency governance.

It remains shell/prompt infrastructure.

However, it can still be modeled as a versioned shared runtime when:

- a boxed shell must avoid direct execution of host `Program Files` prompt binaries
- the prompt binary should be mirrored locally into the box execution tree
- shell startup should remain explicit and bootstrap-driven rather than profile-driven

## Current refinement

The current method therefore distinguishes between:

- project toolchain governance such as `Git`, `Node`, and `pnpm`
- shell/prompt runtime support such as `Starship`

Both can live under `C:\shared\sandbox-toolchains\dev\` when local boxed execution requires a mirrored runtime surface.

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
- Starship: optional shared prompt runtime under `dev\starship\1.25.1\`

## Related

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\git.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\node.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\pnpm.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\starship.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\host-state.md`
