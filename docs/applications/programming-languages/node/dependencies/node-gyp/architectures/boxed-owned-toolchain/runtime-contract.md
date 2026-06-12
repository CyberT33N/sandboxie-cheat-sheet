# Runtime Contract

## Scope

This file owns the **runtime contract** for the current boxed-owned-toolchain `node-gyp` helper path.

It documents:

- the helper inputs
- the helper outputs
- the environment it prepares
- the current assumptions for direct boxed `node-gyp` usage

## Helper entry point

The current shared helper entry point is:

```powershell
Initialize-NodeGypWindowsBuildEnvironment
```

It lives in:

- `C:\shared\sandbox-toolchains\dev\bootstrap\stacks\node\Bootstrap.Node.psm1`

## Current helper inputs

The current helper accepts these key inputs:

```powershell
$null = Initialize-NodeGypWindowsBuildEnvironment `
  -CmdExe $env:BOXED_CMD_EXE `
  -RegExe $env:BOXED_REG_EXE `
  -PythonExe $env:BOXED_PYTHON_EXE
```

Current behavior:

- `CmdExe` is the explicit local helper lane used to import `VsDevCmd.bat`
- `RegExe` is the explicit local registry helper lane available to the build environment
- `PythonExe` is the boxed shared Python helper mirrored into the project execution tree

## Visual Studio discovery contract

The helper currently searches known Visual Studio 2022 roots directly:

```powershell
[string[]]$VisualStudioRootCandidates = @(
  'C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools',
  'C:\Program Files (x86)\Microsoft Visual Studio\2022\Community',
  'C:\Program Files (x86)\Microsoft Visual Studio\2022\Professional',
  'C:\Program Files (x86)\Microsoft Visual Studio\2022\Enterprise'
)
```

It then resolves:

- `VSRoot`
- `VsDevCmd`

from the first candidate that contains:

- `Common7\Tools\VsDevCmd.bat`

## Developer-environment import contract

When `CmdExe` and `VsDevCmd.bat` are present, the helper imports the Visual Studio developer environment through the boxed local CMD lane:

```powershell
& $CmdExe /d /c "`"$vsDevCmd`" -arch=x64 -host_arch=x64 && set" 2>$null |
  ForEach-Object {
    if ($_ -match '^(.*?)=(.*)$') {
      Set-Item -Path "Env:$($matches[1])" -Value $matches[2]
    }
  }
```

This is the current boxed-owned-toolchain bridge between:

- local mirrored shell/runtime surfaces
- and host-provided Microsoft build infrastructure

## Python environment contract

When the boxed shared Python helper is present, the helper sets:

```powershell
$env:PYTHON = $PythonExe
$env:NODE_GYP_FORCE_PYTHON = $PythonExe
$env:npm_config_python = $PythonExe
```

That keeps Python selection explicit for direct `node-gyp` usage and for install/reinstall flows that may auto-trigger native builds.

## Windows SDK environment contract

The helper resolves the newest SDK version from:

```text
C:\Program Files (x86)\Windows Kits\10\Lib
```

Then it normalizes these environment variables when missing or malformed:

- `WindowsSDKVersion`
- `WindowsSDKLibVersion`
- `UCRTVersion`
- `WindowsSdkDir`
- `UniversalCRTSdkDir`

Representative current behavior:

```powershell
if ([string]::IsNullOrWhiteSpace($env:WindowsSDKVersion) -or $env:WindowsSDKVersion -eq '\') {
  $env:WindowsSDKVersion = "$windowsSdkVersion\"
}
if ([string]::IsNullOrWhiteSpace($env:WindowsSDKLibVersion) -or $env:WindowsSDKLibVersion -eq '\') {
  $env:WindowsSDKLibVersion = "$windowsSdkVersion\"
}
if ([string]::IsNullOrWhiteSpace($env:UCRTVersion)) {
  $env:UCRTVersion = $windowsSdkVersion
}
if ([string]::IsNullOrWhiteSpace($env:WindowsSdkDir)) {
  $env:WindowsSdkDir = "$WindowsSdkRoot\"
}
if ([string]::IsNullOrWhiteSpace($env:UniversalCRTSdkDir)) {
  $env:UniversalCRTSdkDir = "$WindowsSdkRoot\"
}
```

The helper also prepends:

```text
C:\Program Files (x86)\Windows Kits\10\bin\<version>\x64
```

into `PATH` when present, so tools such as `rc.exe` and `mt.exe` resolve in the active shell.

## Helper outputs

The helper publishes these metadata values:

- `BOXED_VSROOT`
- `BOXED_VSDEVCMD`
- `BOXED_WINDOWS_SDK_ROOT`
- `BOXED_WINDOWS_SDK_VERSION`

And the broader boxed bootstrap already publishes:

- `BOXED_CMD_EXE`
- `BOXED_POWERSHELL_EXE`
- `BOXED_REG_EXE`
- `BOXED_PYTHON_EXE`

Together those variables form the current boxed-owned-toolchain `node-gyp` runtime contract.

## Current direct-use posture

The current direct-use posture is:

1. enter the normal project bootstrap shell
2. call `Initialize-NodeGypWindowsBuildEnvironment`
3. enter the package directory
4. run direct `node-gyp` commands against the boxed Python executable

This keeps the native-build setup explicit and reusable without forcing every normal project-terminal startup to load the Microsoft build environment preemptively.

## Current non-goal of the helper

The helper currently does **not** automatically write HKLM Windows SDK compatibility keys.

That behavior was intentionally removed from the default helper so the baseline runtime contract stays focused on:

- environment bootstrap
- tool resolution
- direct native-build readiness

instead of registry mutation in the normal helper path.

## Related

- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\current-state.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\bootstrap-integration.md`
