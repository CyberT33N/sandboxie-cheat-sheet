# Boxed-Owned-Toolchain `node-gyp`

## Architectural status

This folder documents the **current boxed-owned-toolchain `node-gyp` runtime contract** that is already implemented in the shared bootstrap and project-owned install flows.

It intentionally documents only:

- the current bootstrap/runtime surfaces that now exist
- the current script-owned integration points
- the current ownership split between shared-governed toolchains and boxed projected Microsoft build infrastructure

It intentionally does **not** use this folder as a place to track open package-specific failures or unresolved follow-up work.

For repository-wide end-to-end `node-gyp` guidance, the documented comparison baseline still lives here:

- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\general.md`

Archived legacy installer-driven notes remain preserved here:

- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\box-owned-toolchain\general.md`

## Role of this file

This file is now the **TOC / entrypoint** for the boxed-owned-toolchain `node-gyp` source of truth.

The previous monolithic write-up has been split by concern so the current architecture can stay:

- current-state-only
- easier to re-reference from bootstrap and PNPM documents
- easier to evolve without mixing script inventory, ownership boundaries, and runtime contract details

## Current reference snapshot

The current boxed-owned-toolchain `node-gyp` reference truth is:

1. shared Python remains versioned under `dev\python\...` and is mirrored locally into the project box
2. governed boxed shell/helper artifacts now include:
   - boxed `cmd.exe`
   - boxed Windows PowerShell
   - boxed `reg.exe`
3. governed shared Microsoft build sources now include:
   - `C:\shared\sandbox-toolchains\dev\shells\vs-installer\3.1.7\vswhere.exe`
   - `C:\shared\sandbox-toolchains\dev\shells\visual-studio\2022\BuildTools\...`
   - `C:\shared\sandbox-toolchains\dev\shells\windows-kits\10\...`
   - `C:\shared\sandbox-toolchains\dev\shells\dotnet-framework\Framework\v4.0.30319`
   - `C:\shared\sandbox-toolchains\dev\shells\dotnet-framework\Framework64\v4.0.30319`
4. the project bootstrap exports explicit local helper lanes such as:
   - `BOXED_CMD_EXE`
   - `BOXED_POWERSHELL_EXE`
   - `BOXED_REG_EXE`
   - `BOXED_PYTHON_EXE`
   - `BOXED_SHARED_VSWHERE_EXE`
   - `BOXED_SHARED_VISUAL_STUDIO_ROOT`
   - `BOXED_SHARED_WINDOWS_SDK_ROOT`
   - `BOXED_SHARED_DOTNET_FRAMEWORK_ROOT`
   - `BOXED_SHARED_DOTNET_FRAMEWORK64_ROOT`
5. a dedicated `Initialize-NodeGypWindowsBuildEnvironment` helper now prepares the Windows native-build environment for direct `node-gyp` flows
6. that helper projects the governed Microsoft build sources into the expected Windows paths inside the box before `buildcheck` / `Add-Type` / direct `node-gyp` run
7. project-owned PNPM install and clean-reinstall scripts now invoke that helper before `pnpm install`
8. the common bootstrap layer no longer depends solely on `robocopy.exe`; it has a PowerShell tree-sync fallback for strict boxed child-process surfaces
9. the verified runtime result now includes successful boxed resolution of:
   - `vswhere.exe`
   - `cl.exe`
   - `MSBuild.exe`
   - `rc.exe`
   - `mt.exe`
   - `csc.exe`
   - `cvtres.exe`
10. the verified `.NET Framework` smoke surface now includes successful boxed PowerShell `Add-Type`
11. the boxed-owned-toolchain bootstrap now publishes a dedicated `node-gyp` wrapper surface instead of requiring edits inside downloaded dependencies
12. the currently validated direct proof surface is a green boxed `node-gyp rebuild --verbose` run through that wrapper

## Domain map

### Current state

- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\current-state.md`

Owns:

- the implemented boxed-owned-toolchain `node-gyp` state
- the current runtime/build-preparation posture
- the meaning of the current helper-driven direct `node-gyp` path

### Toolchain boundary

- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\toolchain-boundary.md`

Owns:

- the ownership split between shared-governed toolchains and host-provided Microsoft build infrastructure
- why Python / shell helper lanes are mirrored while Microsoft build surfaces are projected into their canonical runtime paths
- the current domain-driven / twelve-factor interpretation of that split

### Runtime contract

- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\runtime-contract.md`

Owns:

- the exact helper inputs and outputs
- the current `VsDevCmd.bat` import path through boxed `cmd.exe`
- the Python and Windows SDK environment contract
- the current boxed helper-lane assumptions for direct `node-gyp` use

### Microsoft build projection

- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\microsoft-build-projection.md`

Owns:

- the governed shared `vswhere.exe`, Visual Studio Build Tools, and Windows Kits source roots
- the canonical boxed Windows projection paths for those sources
- the verified `VsDevCmd` / compiler / MSBuild / Windows Kits runtime results

### .NET Framework projection

- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\dotnet-framework-projection.md`

Owns:

- the governed shared `.NET Framework` source roots
- the canonical boxed Windows projection paths for Framework / Framework64
- the verified boxed `csc.exe` / `cvtres.exe` / `Add-Type` results

### Bootstrap integration

- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\bootstrap-integration.md`

Owns:

- the exact live shared files that implement the current behavior
- the sanitized project-adapter contract
- the current install/reinstall script integration points
- the current common-bootstrap mirror fallback that keeps script execution resilient under strict boxed policy

### MSBuild file-tracking wrapper

- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\msbuild-file-tracking-wrapper.md`

Owns:

- the `TrackFileAccess` problem class
- the architectural decision against dependency patching
- the documented upstream Sandboxie options that could influence the failure class
- the reason those options are not preferred
- the complete sanitized wrapper code example

## Cross-domain references

- shared bootstrap script SSOT:
  `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\scripts.md`
- sanitized project-adapter boilerplate:
  `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\scripts.md`
- PNPM install-script contract:
  `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\install.md`
- PNPM clean-reinstall contract:
  `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\clean-reinstall.md`
- central shell-selection contract:
  `docs\cli\shell\general.md`

## Related

- `docs\applications\programming-languages\node\dependencies\node-gyp\general.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\general.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\box-owned-toolchain\general.md`
