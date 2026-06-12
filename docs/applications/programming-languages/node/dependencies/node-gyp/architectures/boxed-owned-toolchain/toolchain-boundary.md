# Toolchain Boundary

## Purpose

This file owns the **toolchain ownership split** for the current boxed-owned-toolchain `node-gyp` implementation.

That split is the key architectural boundary which keeps the method enterprise-grade and twelve-factor aligned.

## Shared-governed and mirrored into the box

The following surfaces are current **shared-governed** toolchains which the bootstrap mirrors into the local project execution tree:

- Node runtime
- PNPM CLI
- Git runtime
- shared Python build helper selected through `dev\python\current.txt`
- boxed `cmd.exe`
- boxed Windows PowerShell
- boxed `reg.exe`
- optional boxed `Clink`
- optional boxed Starship

These are treated as governed artifacts because:

- their location is explicit
- their versioning is reviewable
- the bootstrap can mirror them deterministically into the box
- they behave like toolchain/runtime inventory rather than installer-managed platform state

## Host-provided Microsoft build infrastructure

The following surfaces remain **host-provided** in the current boxed-owned-toolchain `node-gyp` contract:

- Visual Studio Build Tools installation root
- `VsDevCmd.bat`
- Windows SDK include/lib/bin trees
- Microsoft .NET compiler chain that PowerShell `Add-Type` may invoke indirectly

These remain host-provided because they behave like heavier Windows platform infrastructure rather than like simple portable copied binaries.

## Why this split is correct

This boundary is correct because it keeps each concern in the right domain:

1. **shared-governed developer toolchains**
   - versioned
   - mirrored
   - box-local at runtime

2. **host-provided Microsoft platform toolchains**
   - installer-managed
   - OS-anchored
   - imported through runtime environment bootstrap

That means the boxed-owned-toolchain method does **not** need to pretend that all Microsoft build infrastructure is already a portable shared artifact just because Node, PNPM, Python, and shell helper binaries are.

## Twelve-factor reading

The current boundary is twelve-factor aligned:

- shared-governed binaries are explicit dependencies
- host-provided Microsoft build state is imported through environment initialization
- the runtime shell is prepared on demand for native builds
- direct terminal defaults are not overloaded with native-build-only setup

## Role of `reg.exe`

The boxed `reg.exe` lane is part of the current helper surface, but it does **not** redefine the overall Microsoft toolchain boundary.

Its role is:

- give the box an explicit local registry helper binary
- avoid relying on incidental host `PATH` resolution
- keep registry-facing scripted helper flows explicit and bootstrap-owned

It is **not** a signal that the whole Microsoft build chain has now become a shared mirrored toolchain.

## Role of `VsDevCmd.bat`

`VsDevCmd.bat` remains the current bridge between the host-provided Microsoft build world and the boxed-owned-toolchain shell.

The boxed-owned-toolchain contract therefore treats:

- boxed `cmd.exe`
- boxed Python
- boxed `reg.exe`

as the explicit local helper lanes, while:

- `VsDevCmd.bat`
- Visual Studio Build Tools
- Windows SDK trees

remain host-provided infrastructure that the helper imports into the active environment.

## Related

- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\runtime-contract.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\general.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\visual-studio-build-tools.md`
