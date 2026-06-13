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
  -PythonExe $env:BOXED_PYTHON_EXE `
  -VsWhereExe $env:BOXED_SHARED_VSWHERE_EXE `
  -VisualStudioRoot $env:BOXED_SHARED_VISUAL_STUDIO_ROOT `
  -WindowsSdkRoot $env:BOXED_SHARED_WINDOWS_SDK_ROOT `
  -DotNetFrameworkRoot $env:BOXED_SHARED_DOTNET_FRAMEWORK_ROOT `
  -DotNetFramework64Root $env:BOXED_SHARED_DOTNET_FRAMEWORK64_ROOT
```

Current behavior:

- `CmdExe` is the explicit local helper lane used to import `VsDevCmd.bat`
- `RegExe` is the explicit local registry helper lane available to the build environment
- `PythonExe` is the boxed shared Python helper mirrored into the project execution tree
- `VsWhereExe` is the governed shared `vswhere.exe` source path that bootstrap projects into the canonical Visual Studio installer path inside the box
- `VisualStudioRoot` is the governed shared Visual Studio Build Tools source root that bootstrap projects into `C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\...`
- `WindowsSdkRoot` is the governed shared Windows Kits source root that bootstrap projects into `C:\Program Files (x86)\Windows Kits\10\...`
- `DotNetFrameworkRoot` / `DotNetFramework64Root` are the governed shared compiler-source paths which the helper projects into `C:\Windows\Microsoft.NET\...` inside the box

## Visual Studio discovery contract

The helper now starts from governed shared Microsoft build sources and projects them into the canonical Windows paths inside the box:

```text
C:\Program Files (x86)\Microsoft Visual Studio\Installer\vswhere.exe
C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\...
C:\Program Files (x86)\Windows Kits\10\...
```

That means:

- tools that hard-code `vswhere.exe` or `VsDevCmd.bat` still see the expected Windows path
- the boxed helper lane executes those projected paths instead of direct host paths
- `VSRoot` and `VsDevCmd` resolve from the projected Visual Studio tree

## Developer-environment import contract

When `CmdExe` and `VsDevCmd.bat` are present, the helper imports the Visual Studio developer environment through the boxed local CMD lane.

The current implementation uses a generated temporary CMD bridge so the helper can:

- call `VsDevCmd.bat`
- capture the resulting environment explicitly
- rehydrate the selected variables back into the PowerShell session

Representative current behavior:

```powershell
$vsEnvVariables = @(
  'VCINSTALLDIR',
  'VCToolsInstallDir',
  'VSINSTALLDIR',
  'WindowsSdkDir',
  'WindowsSDKVersion',
  'WindowsSDKLibVersion',
  'UCRTVersion',
  'UniversalCRTSdkDir',
  'Path',
  'INCLUDE',
  'LIB',
  'LIBPATH'
)

$vsEnvImportOutput = & $CmdExe /d /c "`"$vsEnvImportScript`" `"$vsDevCmd`"" 2>$null

foreach ($line in $vsEnvImportOutput) {
  if ($line -match '^BOXED_ENV_([^=]+)=(.*)$') {
    Set-Item -Path "Env:$($matches[1])" -Value $matches[2]
  }
}
```

This is the current boxed-owned-toolchain bridge between:

- local mirrored shell/runtime surfaces
- and the projected canonical Microsoft build-tool paths inside the box

## Python environment contract

When the boxed shared Python helper is present, the helper sets:

```powershell
$env:PYTHON = $PythonExe
$env:NODE_GYP_FORCE_PYTHON = $PythonExe
$env:npm_config_python = $PythonExe
```

That keeps Python selection explicit for direct `node-gyp` usage and for install/reinstall flows that may auto-trigger native builds.

## Microsoft .NET Framework projection contract

When the shared `.NET Framework` compiler roots are present, the helper projects them into the hard-coded runtime paths that PowerShell `Add-Type` / `buildcheck` expect:

```text
C:\Windows\Microsoft.NET\Framework\v4.0.30319
C:\Windows\Microsoft.NET\Framework64\v4.0.30319
```

Current behavior:

- source of truth stays in `C:\shared\sandbox-toolchains\dev\shells\dotnet-framework\...`
- bootstrap copies the full tree into the boxed `C:\Windows\Microsoft.NET\...` surface
- the projected framework directories are prepended into `PATH`
- helper metadata is published through:
  - `BOXED_DOTNET_FRAMEWORK_ROOT`
  - `BOXED_DOTNET_FRAMEWORK64_ROOT`
  - `BOXED_DOTNET_FRAMEWORK_CSC_EXE`
  - `BOXED_DOTNET_FRAMEWORK64_CSC_EXE`

## Windows SDK environment contract

The helper resolves the newest SDK version from:

```text
C:\Program Files (x86)\Windows Kits\10\Lib
```

The current boxed projection also includes the Windows SDK DesignTime metadata path required by MSBuild:

```text
C:\Program Files (x86)\Windows Kits\10\DesignTime\CommonConfiguration\Neutral
```

Verified required files there include:

- `uCRT.props`
- `UAP\<version>\UAP.props`

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

- `BOXED_VSWHERE_EXE`
- `BOXED_VSROOT`
- `BOXED_VSDEVCMD`
- `BOXED_WINDOWS_SDK_ROOT`
- `BOXED_WINDOWS_SDK_VERSION`
- `BOXED_DOTNET_FRAMEWORK_ROOT`
- `BOXED_DOTNET_FRAMEWORK64_ROOT`
- `BOXED_DOTNET_FRAMEWORK_CSC_EXE`
- `BOXED_DOTNET_FRAMEWORK64_CSC_EXE`

And the broader boxed bootstrap already publishes:

- `BOXED_CMD_EXE`
- `BOXED_POWERSHELL_EXE`
- `BOXED_REG_EXE`
- `BOXED_PYTHON_EXE`
- `BOXED_SHARED_VSWHERE_EXE`
- `BOXED_SHARED_VISUAL_STUDIO_ROOT`
- `BOXED_SHARED_WINDOWS_SDK_ROOT`
- `BOXED_SHARED_DOTNET_FRAMEWORK_ROOT`
- `BOXED_SHARED_DOTNET_FRAMEWORK64_ROOT`

Together those variables form the current boxed-owned-toolchain `node-gyp` runtime contract.

The current bootstrap also publishes the wrapper-specific command-surface metadata:

- `BOXED_NODE_GYP_JS`
- `BOXED_NODE_GYP_REAL_JS`
- `npm_config_node_gyp`

That means the boxed project shell now has both:

- a direct `node-gyp` command on `PATH`
- and explicit environment metadata describing the wrapper-vs-real-entrypoint split

## Verified helper result

The following helper result is already runtime-verified in the boxed project shell:

- `VCINSTALLDIR` is populated
- `VCToolsInstallDir` is populated
- `VSINSTALLDIR` is populated
- `WindowsSdkDir` is populated
- `WindowsSDKVersion` is populated
- `Get-Command cl.exe` succeeds
- `Get-Command msbuild.exe` succeeds
- `Get-Command rc.exe` succeeds
- `Get-Command mt.exe` succeeds
- `Get-Command csc.exe` succeeds
- `Get-Command cvtres.exe` succeeds

## Current direct-use posture

The current direct-use posture is:

1. enter the normal project bootstrap shell
2. call `Initialize-NodeGypWindowsBuildEnvironment`
3. enter the package directory
4. run direct `node-gyp` commands against the boxed Python executable
5. let the bootstrap-published wrapper own the Windows MSBuild file-tracking override for direct boxed build flows

Representative current direct-use proof surface:

```powershell
Set-Location (Join-Path $env:BOXED_LOCAL_TEMP_ROOT 'msgpackr-extract-node-gyp-test\package')
node-gyp rebuild --verbose
```

## Wrapper-owned Windows build-phase behavior

The current runtime contract now assumes that direct boxed `node-gyp` commands resolve to the bootstrap-owned wrapper surface rather than to the raw dependency entrypoint.

The important current behavior is:

- `clean` and `configure` remain dependency-owned
- the Windows `build` phase is adapted at the wrapper layer
- the wrapper injects:
  - `/nologo`
  - `/nodeReuse:false`
  - `/p:TrackFileAccess=false`

at the MSBuild invocation layer

That keeps the fix:

- outside dependency source
- outside package-local vendor patches
- and outside Sandboxie isolation weakening

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
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\msbuild-file-tracking-wrapper.md`
