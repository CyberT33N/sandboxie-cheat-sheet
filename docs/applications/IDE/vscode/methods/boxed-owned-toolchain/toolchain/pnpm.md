# pnpm

## Decision

`pnpm` is stored centrally in shared, but **not** as `@pnpm/exe`.

## Why not `@pnpm/exe`

`@pnpm/exe` embeds its own Node runtime.

That is incompatible with this method because the runtime truth must remain explicit in the shared versioned Node layer rather than being hidden inside pnpm itself.

## Selected form

- `pnpm@11.2.2`
- unpacked as CLI content
- started through the selected shared Node runtime

## Canonical path

```text
C:\shared\sandbox-toolchains\dev\pnpm\11.2.2\package\bin\pnpm.cjs
```

## Runtime relationship

Bootstrap is responsible for ensuring that:

- the correct shared Node runtime hosts pnpm
- the project box gets a stable `pnpm` command surface

This keeps the package-manager runtime explicit and reviewable.

## Related

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\node.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\provisioning\shared-artifacts.md`
