# Boxed-Owned-Toolchain `node-gyp`

## Architectural status

This folder documents the **current boxed-owned-toolchain `node-gyp` runtime contract** that is already implemented in the shared bootstrap and project-owned install flows.

It intentionally documents only:

- the current bootstrap/runtime surfaces that now exist
- the current script-owned integration points
- the current ownership split between shared-governed toolchains and host-provided Microsoft build infrastructure

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
3. the project bootstrap exports explicit local helper lanes such as:
   - `BOXED_CMD_EXE`
   - `BOXED_POWERSHELL_EXE`
   - `BOXED_REG_EXE`
   - `BOXED_PYTHON_EXE`
4. a dedicated `Initialize-NodeGypWindowsBuildEnvironment` helper now prepares the Windows native-build environment for direct `node-gyp` flows
5. project-owned PNPM install and clean-reinstall scripts now invoke that helper before `pnpm install`
6. the common bootstrap layer no longer depends solely on `robocopy.exe`; it has a PowerShell tree-sync fallback for strict boxed child-process surfaces

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
- why Python / shell helper lanes are mirrored while Visual Studio / Windows SDK remain host-provided
- the current domain-driven / twelve-factor interpretation of that split

### Runtime contract

- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\runtime-contract.md`

Owns:

- the exact helper inputs and outputs
- the current `VsDevCmd.bat` import path through boxed `cmd.exe`
- the Python and Windows SDK environment contract
- the current boxed helper-lane assumptions for direct `node-gyp` use

### Bootstrap integration

- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\bootstrap-integration.md`

Owns:

- the exact live shared files that implement the current behavior
- the sanitized project-adapter contract
- the current install/reinstall script integration points
- the current common-bootstrap mirror fallback that keeps script execution resilient under strict boxed policy

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
