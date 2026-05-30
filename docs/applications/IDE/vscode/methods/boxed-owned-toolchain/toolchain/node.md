# Node

## Decision

Node is provided as versioned shared runtimes.

## Selected versions

The current validated example architecture uses:

- `26.2.0` as the primary control-plane / tooling runtime
- `20.19.6` as an additional secondary runtime domain

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
C:\shared\sandbox-toolchains\dev\node\20.19.6\node-v20.19.6-win-x64\
```

## Multi-runtime monorepo model

For sanitized monorepo examples such as `test-mono`, the method allows:

- primary tooling and package-manager shell on `Node 26.2.0`
- a secondary project-visible command such as `node20` bound to `Node 20.19.6`

This keeps the runtime contract explicit and reviewable.

## `nvm` is not part of the final contract

`nvm` is not part of the final architecture contract.

Why:

- `nvm use` mutates shims and runtime selection state
- it creates implicit runtime switching
- it conflicts with the explicit shared-runtime model

The method uses fixed versioned binaries selected by bootstrap.

## Related

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\pnpm.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\provisioning\shared-artifacts.md`
