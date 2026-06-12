# Current State

## Scope

This file documents only the **implemented current state** of `node-gyp` in the boxed-owned-toolchain architecture.

It intentionally focuses on:

- bootstrap/runtime contract
- toolchain preparation
- direct `node-gyp` environment readiness

It intentionally does **not** track open package-specific or follow-up work.

## Implemented state

The current boxed-owned-toolchain implementation now includes:

1. locally mirrored shared Node, Git, PNPM, Python, and Windows shell helper surfaces
2. explicit box-local helper lanes for:
   - boxed `cmd.exe`
   - boxed Windows PowerShell
   - boxed `reg.exe`
3. project bootstrap exports for those lanes and runtimes, including:
   - `BOXED_CMD_EXE`
   - `BOXED_POWERSHELL_EXE`
   - `BOXED_REG_EXE`
   - `BOXED_PYTHON_EXE`
4. a dedicated `Initialize-NodeGypWindowsBuildEnvironment` helper in the shared Node bootstrap stack
5. project-owned PNPM install and clean-reinstall scripts that opt into that helper before they run `pnpm install`
6. a common bootstrap mirror fallback that avoids hard dependence on `robocopy.exe` when strict boxed child-process policy blocks it

## What the current helper contract already does

The current helper-driven boxed-owned-toolchain `node-gyp` contract already:

- resolves a supported Visual Studio root from known host installation candidates
- resolves `VsDevCmd.bat`
- imports the Visual Studio developer environment through the **boxed local `cmd.exe` lane**
- sets:
  - `PYTHON`
  - `NODE_GYP_FORCE_PYTHON`
  - `npm_config_python`
- discovers the newest Windows SDK version under `C:\Program Files (x86)\Windows Kits\10\Lib`
- normalizes Windows SDK environment variables when they are missing or malformed
- prepends the Windows SDK `bin\<version>\x64` path into `PATH` when present
- publishes helper metadata back into the shell through:
  - `BOXED_VSROOT`
  - `BOXED_VSDEVCMD`
  - `BOXED_WINDOWS_SDK_ROOT`
  - `BOXED_WINDOWS_SDK_VERSION`

## What the current shell/helper lanes already provide

The current boxed-owned-toolchain `node-gyp` posture now assumes three explicit local helper lanes:

- boxed `cmd.exe`
- boxed Windows PowerShell
- boxed `reg.exe`

Architecturally:

- boxed `cmd.exe` owns the current developer-environment import path for `VsDevCmd.bat`
- boxed `reg.exe` exists as an explicit local helper lane for registry-facing diagnostics and scripted registry access
- boxed Windows PowerShell exists as an explicit local helper lane alongside the preferred interactive VS Code shell contract

This keeps child-process and helper-tool execution explicit instead of depending on accidental host defaults.

## Current direct `node-gyp` posture

The currently implemented direct boxed-owned-toolchain posture is:

1. start from the normal project bootstrap
2. call `Initialize-NodeGypWindowsBuildEnvironment`
3. enter the package directory that needs direct `node-gyp`
4. run the direct `node-gyp` command against the boxed local Python executable

That means the boxed-owned-toolchain method now has a **real current-state `node-gyp` preparation contract**, not only a shell-lane experiment.

## Why this matters architecturally

From a domain-driven and twelve-factor perspective, the important correction is:

- generic project authoring bootstrap stays lightweight
- native-build preparation is **opt-in** at the install/reinstall or direct-build surface
- Microsoft build environment concerns are injected as runtime configuration instead of being scattered through ad hoc terminal steps

So the boxed-owned-toolchain `node-gyp` state is now:

- explicit
- reviewable
- script-owned
- and reusable across direct precheck and install/reinstall surfaces

## Related

- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\runtime-contract.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\bootstrap-integration.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\toolchain-boundary.md`
