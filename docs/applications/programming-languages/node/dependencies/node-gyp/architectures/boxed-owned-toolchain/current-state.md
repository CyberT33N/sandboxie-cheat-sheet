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
3. governed shared Microsoft build sources under:
   - `C:\shared\sandbox-toolchains\dev\shells\vs-installer\3.1.7\vswhere.exe`
   - `C:\shared\sandbox-toolchains\dev\shells\visual-studio\2022\BuildTools\...`
   - `C:\shared\sandbox-toolchains\dev\shells\windows-kits\10\...`
   - `C:\shared\sandbox-toolchains\dev\shells\dotnet-framework\Framework\v4.0.30319`
   - `C:\shared\sandbox-toolchains\dev\shells\dotnet-framework\Framework64\v4.0.30319`
4. project bootstrap exports for those lanes and runtimes, including:
   - `BOXED_CMD_EXE`
   - `BOXED_POWERSHELL_EXE`
   - `BOXED_REG_EXE`
   - `BOXED_PYTHON_EXE`
   - `BOXED_SHARED_VSWHERE_EXE`
   - `BOXED_SHARED_VISUAL_STUDIO_ROOT`
   - `BOXED_SHARED_WINDOWS_SDK_ROOT`
   - `BOXED_SHARED_DOTNET_FRAMEWORK_ROOT`
   - `BOXED_SHARED_DOTNET_FRAMEWORK64_ROOT`
5. a dedicated `Initialize-NodeGypWindowsBuildEnvironment` helper in the shared Node bootstrap stack
6. project-owned PNPM install and clean-reinstall scripts that opt into that helper before they run `pnpm install`
7. a common bootstrap mirror fallback that avoids hard dependence on `robocopy.exe` when strict boxed child-process policy blocks it
8. verified boxed build-environment results for:
   - `VCINSTALLDIR`
   - `VCToolsInstallDir`
   - `VSINSTALLDIR`
   - `WindowsSdkDir`
   - `WindowsSDKVersion`
9. verified boxed tool resolution for:
   - `cl.exe`
   - `MSBuild.exe`
   - `rc.exe`
   - `mt.exe`
   - `csc.exe`
   - `cvtres.exe`
10. a verified boxed PowerShell `Add-Type` smoke-test result through the projected `.NET Framework` compiler path
11. bootstrap-published `node-gyp` wrapper surfaces:
   - `node-gyp-wrapper.cjs`
   - `node-gyp.cmd`
   - `node-gyp.ps1`
   - `node-gyp`
12. wrapper metadata exported back into the boxed shell through:
   - `BOXED_NODE_GYP_JS`
   - `BOXED_NODE_GYP_REAL_JS`
   - `npm_config_node_gyp`
13. a validated direct boxed `node-gyp rebuild --verbose` proof path that reaches the Windows MSBuild phase through the wrapper and succeeds with `TrackFileAccess=false`

## What the current helper contract already does

The current helper-driven boxed-owned-toolchain `node-gyp` contract already:

- projects the governed `vswhere.exe` source into:
  - `C:\Program Files (x86)\Microsoft Visual Studio\Installer\vswhere.exe`
- projects the governed Visual Studio Build Tools source into:
  - `C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\...`
- projects the governed Windows Kits source into:
  - `C:\Program Files (x86)\Windows Kits\10\...`
- projects the governed Windows Kits DesignTime metadata into:
  - `C:\Program Files (x86)\Windows Kits\10\DesignTime\CommonConfiguration\Neutral\...`
- resolves `VsDevCmd.bat` from the projected boxed Visual Studio tree
- imports the Visual Studio developer environment through the **boxed local `cmd.exe` lane**
- sets:
  - `PYTHON`
  - `NODE_GYP_FORCE_PYTHON`
  - `npm_config_python`
- projects the shared Microsoft .NET Framework compiler tree into:
  - `C:\Windows\Microsoft.NET\Framework\v4.0.30319`
  - `C:\Windows\Microsoft.NET\Framework64\v4.0.30319`
- publishes the projected compiler paths back through:
  - `BOXED_DOTNET_FRAMEWORK_ROOT`
  - `BOXED_DOTNET_FRAMEWORK64_ROOT`
  - `BOXED_DOTNET_FRAMEWORK64_CSC_EXE`
- discovers the newest Windows SDK version under `C:\Program Files (x86)\Windows Kits\10\Lib`
- normalizes Windows SDK environment variables when they are missing or malformed
- prepends the Windows SDK `bin\<version>\x64` path into `PATH` when present
- publishes helper metadata back into the shell through:
  - `BOXED_VSROOT`
  - `BOXED_VSDEVCMD`
  - `BOXED_VSWHERE_EXE`
  - `BOXED_WINDOWS_SDK_ROOT`
  - `BOXED_WINDOWS_SDK_VERSION`
- publishes wrapper metadata through:
  - `BOXED_NODE_GYP_JS`
  - `BOXED_NODE_GYP_REAL_JS`
  - `npm_config_node_gyp`

## Current wrapper contract

The current boxed-owned-toolchain posture now includes a bootstrap-owned wrapper around the delivered `node-gyp` code.

This wrapper:

- leaves the dependency source under `node_modules` untouched
- intercepts the Windows `build` phase only
- injects `/p:TrackFileAccess=false` at the MSBuild layer
- keeps the fix in the bootstrap/control-plane layer rather than in vendor source

The wrapper is not a replacement implementation of `node-gyp`.

It is a **boxed-owned command-surface adapter** around the delivered `node-gyp` entrypoint.

## Verified boxed outcomes

The following outcomes are already runtime-verified in the boxed project shell:

- `vswhere.exe` resolves from the projected boxed Visual Studio installer path
- `VCINSTALLDIR` resolves after helper execution
- `VCToolsInstallDir` resolves after helper execution
- `VSINSTALLDIR` resolves after helper execution
- `WindowsSdkDir` resolves after helper execution
- `WindowsSDKVersion` resolves after helper execution
- `Get-Command cl.exe` succeeds
- `Get-Command msbuild.exe` succeeds
- `Get-Command rc.exe` succeeds
- `Get-Command mt.exe` succeeds
- `Get-Command csc.exe` succeeds
- `Get-Command cvtres.exe` succeeds
- boxed PowerShell `Add-Type` succeeds through the projected `.NET Framework` compiler chain
- a direct boxed `node-gyp rebuild --verbose` succeeds through the bootstrap-published wrapper surface

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
4. run the direct `node-gyp` command through the bootstrap-published wrapper surface

Representative current proof surface:

```powershell
Set-Location (Join-Path $env:BOXED_LOCAL_TEMP_ROOT 'msgpackr-extract-node-gyp-test\package')
node-gyp rebuild --verbose
```

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
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\msbuild-file-tracking-wrapper.md`
