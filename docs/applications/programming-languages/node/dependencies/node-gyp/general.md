# node-gyp

## Single source of truth

This document is the primary architectural reference for `node-gyp` inside Sandboxie in this repository.

It defines:

- the documented architecture track for `node-gyp` in governed Node monorepos
- the current recommended host-sync overlay for host-sync VS Code / Cursor workflows
- the split between Python provisioning, boxed-owned projected Microsoft build sources, install-box config, and build commands

## Documentation map

### Architecture tracks

- recommended host-sync overlay:
  `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\general.md`
- current boxed-owned-toolchain runtime/bootstrap contract:
  `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\overview.md`
- boxed-owned-toolchain Windows build-tracking wrapper:
  `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\msbuild-file-tracking-wrapper.md`

### Host-sync modules

- `docs\applications\programming-languages\python\architectures\host-sync\dev-python-build-helper.md`
- `docs\applications\operating-systems\windows\build-toolchain\microsoft\architectures\host-sync\visual-studio-build-tools.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\install-box-config.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\clean-reinstall.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\commands.md`

### Cross-domain Microsoft build infrastructure

- `docs\applications\operating-systems\windows\build-toolchain\microsoft\general.md`
- `docs\applications\operating-systems\windows\build-toolchain\microsoft\architectures\host-sync\overview.md`
- `docs\applications\operating-systems\windows\build-toolchain\microsoft\architectures\boxed-owned-toolchain\overview.md`

### Generic baselines that remain the source of truth

- `docs\applications\IDE\vscode\methods\host-sync\dependencies-installed-in-box.md`
- `docs\applications\IDE\vscode\methods\host-sync\templates\node-monorepo-materialized-dependencies.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\host-sync\overview.md`
- `docs\applications\programming-languages\node\nvm\general.md`

## Architectural status

For the current recommended Windows Sandboxie model in this repository, `node-gyp` is **not** a standalone architecture. It is an overlay on top of the validated:

- install-box dependency workflow
- host-visible monorepo materialization workflow
- run-box daily execution workflow

The currently validated `node-gyp` posture for Node monorepos is:

- keep dependency installation in the install box
- keep daily execution in the run box
- keep the host IDE on the host
- provide a central shared Python build helper binary under `C:\shared\sandbox-toolchains\dev\python\...`
- provide governed shared Microsoft build sources under `C:\shared\sandbox-toolchains\dev\shells\...`
- project those shared Microsoft build sources into the expected Windows paths inside the box during bootstrap
- bootstrap the install-box session with both the Python path and the projected Visual Studio developer environment before running `node-gyp`
- keep the Windows build-phase tracking workaround in a bootstrap-owned wrapper outside dependency source

The repository-wide fully validated end-to-end `node-gyp` baseline remains the documented host-sync method.

At the same time, the boxed-owned-toolchain architecture now has a documented **current-state runtime/bootstrap contract** for direct boxed `node-gyp` environment preparation. That current-state contract lives in the boxed-owned-toolchain folder and should be read as:

- implemented current bootstrap/runtime behavior
- not a replacement for the host-sync comparison baseline
- and not a catch-all place for open package-specific failures
- including the current governed Microsoft source trees under:
  - `C:\shared\sandbox-toolchains\dev\shells\vs-installer\...`
  - `C:\shared\sandbox-toolchains\dev\shells\visual-studio\...`
  - `C:\shared\sandbox-toolchains\dev\shells\windows-kits\...`
  - `C:\shared\sandbox-toolchains\dev\shells\dotnet-framework\...`
- plus bootstrap-owned projection into the expected Windows runtime paths inside the box for:
  - `vswhere.exe`
  - `VsDevCmd.bat`
  - Windows SDK binaries/includes/libs
  - PowerShell `Add-Type` / `buildcheck`

## Why `node-gyp` needs its own overlay

Compared to a generic `pnpm install` / `pnpm rebuild` flow, `node-gyp` adds several extra moving parts:

- a Python runtime that is spawned by `node.exe`
- generated GYP and MSBuild project files that must materialize to the real host-visible repo path
- Microsoft compiler and linker tools that still need canonical Windows path semantics at runtime
- Visual Studio environment bootstrapping through projected `VsDevCmd.bat`
- ABI-sensitive rebuilds against the exact Node runtime that will later load the native addon

That is why the generic monorepo boilerplate is necessary but not sufficient on its own.

## Related documents

- `docs\applications\programming-languages\node\dependencies\frameworks\electron\general.md`
- `docs\applications\programming-languages\node\dependencies\esbuild\general.md`
- `docs\applications\programming-languages\python\architectures\host-sync\dev-python-build-helper.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\clean-reinstall.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\msbuild-file-tracking-wrapper.md`
