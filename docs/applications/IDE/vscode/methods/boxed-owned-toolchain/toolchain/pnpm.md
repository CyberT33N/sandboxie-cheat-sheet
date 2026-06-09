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

## Sandboxie visibility note

The exact PNPM version belongs in the project contract, but the Sandboxie visibility rule should normally allow the broader PNPM subtree:

```ini
ReadFilePath=C:\shared\sandbox-toolchains\dev\pnpm\
```

instead of pinning the box rule to one exact version directory.

This avoids repeated box-config edits when PNPM must be updated for security reasons.

## Preferred install guidance

For the current preferred PNPM workflow in this repository, read:

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\overview.md`

That document covers:

- the preferred boxed-owned-toolchain PNPM posture
- why a shared host PNPM store is not the baseline
- how lifecycle shell spawning was fixed with a box-local Bash executable

## Related

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\node.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\provisioning\shared-artifacts.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\overview.md`
