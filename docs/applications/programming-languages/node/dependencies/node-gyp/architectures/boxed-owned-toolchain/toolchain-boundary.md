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
- governed `vswhere.exe` source under `dev\shells\vs-installer\...`
- governed Visual Studio Build Tools source under `dev\shells\visual-studio\...`
- governed Windows Kits source under `dev\shells\windows-kits\...`
- shared Microsoft .NET Framework compiler snapshot under `dev\shells\dotnet-framework\...`
- optional boxed `Clink`
- optional boxed Starship

These are treated as governed artifacts because:

- their location is explicit
- their versioning is reviewable
- the bootstrap can mirror them deterministically into the box
- they behave like toolchain/runtime inventory rather than installer-managed platform state

## Boxed projected Microsoft build infrastructure

The following surfaces are now treated as **shared-governed sources with boxed projection into canonical Windows paths**:

- `C:\Program Files (x86)\Microsoft Visual Studio\Installer\vswhere.exe`
- `C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\...`
- `C:\Program Files (x86)\Windows Kits\10\...`
- `C:\Windows\Microsoft.NET\Framework\v4.0.30319\...`
- `C:\Windows\Microsoft.NET\Framework64\v4.0.30319\...`

The source of truth for those projected paths now lives under the shared `dev\shells\...` subtree, and bootstrap materializes them into the exact Windows paths that `vswhere.exe`, `VsDevCmd.bat`, `buildcheck`, and PowerShell `Add-Type` expect.

The currently verified Windows Kits projection includes:

- `bin\<version>\x64`
- `Include\<version>`
- `Lib\<version>`
- `DesignTime\CommonConfiguration\Neutral`
- `SDKManifest.xml`

That `DesignTime\CommonConfiguration\Neutral` subtree is part of the validated boxed-owned-toolchain runtime contract because MSBuild's Windows SDK validation requires:

- `uCRT.props`
- `UAP\<version>\UAP.props`

## Why this split is correct

This boundary is correct because it keeps each concern in the right domain:

1. **shared-governed developer toolchains and Microsoft build sources**
   - versioned
   - mirrored or projected by bootstrap
   - box-local at runtime

That means the boxed-owned-toolchain method no longer depends on direct host execution of Microsoft build-tool paths inside the project box.

## Role of the `node-gyp` wrapper

The current `node-gyp` wrapper belongs to the same bootstrap/control-plane boundary.

It is not:

- a dependency patch
- a vendor-source customization
- or a Sandboxie-isolation exception

Instead it is:

- a bootstrap-owned command-surface adapter
- generated into `bootstrap-bin`
- responsible only for the Windows build-phase tracking override

That ownership split is important because the problem belongs to the **runtime execution boundary**, not to the dependency authoring boundary.

## Twelve-factor reading

The current boundary is twelve-factor aligned:

- shared-governed binaries are explicit dependencies
- shared-governed Microsoft build sources are projected into canonical runtime paths through environment initialization
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

`VsDevCmd.bat` remains the current bridge between the projected Microsoft build world and the boxed-owned-toolchain shell.

The boxed-owned-toolchain contract therefore treats:

- boxed `cmd.exe`
- boxed Python
- boxed `reg.exe`

as the explicit local helper lanes, while the following are provided as shared-governed source trees and projected into canonical Windows paths inside the box:

- `vswhere.exe`
- `VsDevCmd.bat`
- Visual Studio Build Tools
- Windows SDK trees
- .NET Framework compiler trees

This keeps the helper logic compatible with tools that hard-code Windows paths, while still preserving the shared-governed / boxed-projected architecture.

## Related

- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\runtime-contract.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\msbuild-file-tracking-wrapper.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\general.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\visual-studio-build-tools.md`
