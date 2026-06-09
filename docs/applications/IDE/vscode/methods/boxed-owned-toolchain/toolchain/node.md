# Node

## Decision

Node is provided as versioned shared runtimes.

## Selected versions

The current validated example architecture uses:

- `26.2.0` as the primary control-plane / tooling runtime
- `20.9.0` as an additional secondary runtime domain

## Why multiple versions exist

This supports architectures where:

- the outer monorepo control plane uses a newer Node runtime
- one inner application/runtime domain still targets an older fixed runtime

Bootstrap exposes this deliberately and explicitly rather than relying on mutable version switching.

## Canonical paths

Primary Node:

```text
C:\shared\sandbox-toolchains\dev\node\26.2.0\node-v26.2.0-win-x64\
```

Secondary Node:

```text
C:\shared\sandbox-toolchains\dev\node\20.9.0\node-v20.9.0-win-x64\
```

## Multi-runtime monorepo model

For sanitized monorepo examples such as `test-mono`, the method allows:

- primary tooling and package-manager shell on `Node 26.2.0`
- a secondary project-visible command such as `node20` bound to `Node 20.9.0`

This keeps the runtime contract explicit and reviewable.

## Bootstrap command-surface note

The current bootstrap contract now has to support multiple shell families:

- PowerShell / CMD
- integrated Git Bash

That means the Node stack does not only expose Windows `.cmd` wrappers.

It also needs shell-native wrappers for Bash-oriented command resolution, for example:

- `pnpm`
- `node20`
- `nx`

This is why the boxed-owned-toolchain bootstrap generates more than just `pnpm.cmd` and `node20.cmd`.

It also publishes:

- `nx.cmd`
- `nx`
- `nx-cli.cjs`

so the plain `nx` command surface is available inside the boxed bootstrap environment without requiring developers to resolve the local Nx entrypoint manually every time.

## `nvm` is not part of the final contract

`nvm` is not part of the final architecture contract.

Why:

- `nvm use` mutates shims and runtime selection state
- it creates implicit runtime switching
- it conflicts with the explicit shared-runtime model

The method uses fixed versioned binaries selected by bootstrap.

## Related

- `docs\cli\shell\general.md`
- `docs\applications\programming-languages\node\runtime\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\pnpm.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\provisioning\shared-artifacts.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\bootstrap-integration.md`
