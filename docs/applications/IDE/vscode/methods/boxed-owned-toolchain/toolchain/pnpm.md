# pnpm

## Decision

`pnpm` is stored centrally in shared, but **not** as `@pnpm/exe`.

## Why not `@pnpm/exe`

`@pnpm/exe` embeds its own Node runtime.

That is incompatible with this method because the runtime truth must remain explicit in the shared versioned Node layer rather than being hidden inside pnpm itself.

## Selected form

- governed versioned `pnpm@<version>`
- unpacked as CLI content
- started through the selected shared Node runtime

## Canonical path

```text
C:\shared\sandbox-toolchains\dev\pnpm\<version>\package\bin\pnpm.cjs
```

## Runtime relationship

Bootstrap is responsible for ensuring that:

- the correct shared Node runtime hosts pnpm
- the project box gets a stable `pnpm` command surface across PowerShell, CMD, and Git Bash

This keeps the package-manager runtime explicit and reviewable.

## Why Git Bash needed an additional fix

The boxed-owned-toolchain method now uses Git Bash as the integrated VS Code shell.

That created one more shell-specific requirement:

- a generated `pnpm.cmd` wrapper is enough for PowerShell/CMD
- but Git Bash can still fail with `bash: pnpm: command not found` when only the `.cmd` wrapper exists

The current fix is therefore:

1. generate `pnpm.cmd` for Windows-shell compatibility
2. generate a shell-native `pnpm` wrapper for Git Bash
3. ensure the Bash RC files prepend `bootstrap-bin` to the shell `PATH`

Without that combination, the integrated Git Bash terminal can have a valid PNPM project contract and still fail to resolve the `pnpm` command by name.

## Sandboxie visibility note

The exact PNPM version belongs in the project contract, but the Sandboxie visibility rule should normally allow the broader PNPM subtree:

```ini
ReadFilePath=C:\shared\sandbox-toolchains\dev\pnpm\
```

instead of pinning the box rule to one exact version directory.

This avoids repeated box-config edits when PNPM must be updated for security reasons.

## Preferred install guidance

The PNPM-domain source of truth now lives in the PNPM architecture area and is split by concern.

Start here:

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\overview.md`

Then follow the domain-owned documents from there.

The current split covers:

- runtime contract and version selection
- lifecycle shell and command-surface behavior
- install and clean-reinstall scripts

The current real script-owned install / reinstall flows live here:

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\install.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\clean-reinstall.md`

## Related

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\runtime-contract.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\lifecycle-and-command-surface.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\versioning-and-provisioning.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\install.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\clean-reinstall.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\node.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\provisioning\shared-artifacts.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\execution-surfaces.md`
