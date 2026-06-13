# Bootstrap Integration

## Scope

This file owns the **bootstrap integration** for the current boxed-owned-toolchain `node-gyp` contract.

It documents:

- the exact live shared files that now implement the behavior
- the sanitized project-adapter surfaces that must pass the contract through
- the current install/reinstall integration points
- the common bootstrap helper changes that matter to this contract

## Current live shared implementation files

The current live shared implementation surfaces are:

- `C:\shared\sandbox-toolchains\dev\bootstrap\core\Bootstrap.Common.psm1`
- `C:\shared\sandbox-toolchains\dev\bootstrap\stacks\microsoft-build\Bootstrap.MicrosoftBuild.psm1`
- `C:\shared\sandbox-toolchains\dev\bootstrap\stacks\dotnet-framework\Bootstrap.DotNetFramework.psm1`
- `C:\shared\sandbox-toolchains\dev\bootstrap\stacks\node\Bootstrap.Node.psm1`
- `C:\shared\sandbox-toolchains\dev\bootstrap\stacks\shells\Bootstrap.WindowsShells.psm1`
- `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode\Start-VSCodeProjectBase.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Project.Config.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoVSCode.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoPnpmInstall.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoPnpmUninstall.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoPnpmCleanReinstall.ps1`

## Sanitized project adapter contract

The sanitized project adapter now includes both:

- explicit local helper lanes
- explicit shared Microsoft build-source roots

```powershell
MicrosoftBuild = @{
  VsWhereExe = Join-Path $devRoot 'shells\vs-installer\3.1.7\vswhere.exe'
  VisualStudioRoot = Join-Path $devRoot 'shells\visual-studio\2022\BuildTools'
  WindowsSdkRoot = Join-Path $devRoot 'shells\windows-kits\10'
  DotNetFrameworkRoot = Join-Path $devRoot 'shells\dotnet-framework\Framework\v4.0.30319'
  DotNetFramework64Root = Join-Path $devRoot 'shells\dotnet-framework\Framework64\v4.0.30319'
}
```

It also includes an explicit local `reg.exe` lane:

```powershell
Shells = @{
  CmdRoot = Join-Path $devRoot 'shells\cmd\10.0.26100.8457'
  PowerShellRoot = Join-Path $devRoot 'shells\powershell\10.0.26100.8457'
  RegRoot = Join-Path $devRoot 'shells\reg\10.0.26100.8457'
  ClinkRoot = Join-Path $devRoot 'shells\clink\1.9.26'
  StarshipRoot = Join-Path $devRoot 'starship\1.25.1'
  StarshipConfigPath = Join-Path $env:USERPROFILE '.config\starship.toml'
}
```

That `RegRoot` value is passed through the sanitized project launcher into the reusable base script:

```powershell
$parameters = @{
  Action = $Action
  ProjectName = $config.ProjectName
  RepoPath = $resolvedRepoPath
  CodeExe = $config.VSCode.CodeExe
  CodeCli = $config.VSCode.CodeCli
  CatalogUserRoot = $config.VSCode.CatalogUserRoot
  SharedExtensionsRoot = $config.VSCode.SharedExtensionsRoot
  SeedGlobalStorageRoot = $config.VSCode.SeedGlobalStorageRoot
  SeedRooRoot = $config.VSCode.SeedRooRoot
  GitRoot = $config.Toolchain.GitRoot
  NodeRoot = $config.Toolchain.NodeRoot
  PnpmCli = $config.Toolchain.PnpmCli
  PythonRoot = $config.Toolchain.PythonRoot
  VsWhereExe = $config.MicrosoftBuild.VsWhereExe
  VisualStudioRoot = $config.MicrosoftBuild.VisualStudioRoot
  WindowsSdkRoot = $config.MicrosoftBuild.WindowsSdkRoot
  DotNetFrameworkRoot = $config.MicrosoftBuild.DotNetFrameworkRoot
  DotNetFramework64Root = $config.MicrosoftBuild.DotNetFramework64Root
  CmdRoot = $config.Shells.CmdRoot
  PowerShellRoot = $config.Shells.PowerShellRoot
  RegRoot = $config.Shells.RegRoot
  StarshipRoot = $config.Shells.StarshipRoot
  ClinkRoot = $config.Shells.ClinkRoot
  StarshipConfigPath = $config.Shells.StarshipConfigPath
  AdditionalNodeCommands = $config.Toolchain.AdditionalNodeCommands
}
```

## Windows shell stack integration

The current shared Windows-shell bootstrap now mirrors and exports `reg.exe` as a first-class boxed helper surface.

Representative contract:

```powershell
function Initialize-WindowsShellRuntime {
  param(
    [string]$CmdRoot,
    [string]$PowerShellRoot,
    [string]$RegRoot,
    [string]$ClinkRoot,
    [string]$LocalToolchainRoot,
    [string]$BootstrapBin,
    [string]$StateRoot,
    [string]$StarshipExe,
    [string]$StarshipConfigPath
  )
}
```

And the current environment publication includes:

```powershell
$env:BOXED_REG_ROOT = $localRegRoot
$env:BOXED_REG_EXE = $localRegExe
$env:BOXED_REG_AVAILABLE = if ($regAvailable) { 'true' } else { 'false' }
```

That makes `reg.exe` available to any higher-level helper or project-owned script without falling back to accidental host `PATH` resolution.

## Node stack integration

The current shared Node bootstrap now owns the native-build preparation helper:

```powershell
function Initialize-NodeGypWindowsBuildEnvironment {
  param(
    [string]$CmdExe,
    [string]$RegExe,
    [string]$PythonExe,
    [string]$VsWhereExe = $env:BOXED_SHARED_VSWHERE_EXE,
    [string]$VisualStudioRoot = $env:BOXED_SHARED_VISUAL_STUDIO_ROOT,
    [string]$WindowsSdkRoot = $env:BOXED_SHARED_WINDOWS_SDK_ROOT,
    [string]$DotNetFrameworkRoot = $env:BOXED_SHARED_DOTNET_FRAMEWORK_ROOT,
    [string]$DotNetFramework64Root = $env:BOXED_SHARED_DOTNET_FRAMEWORK64_ROOT
  )
}
```

This helper is responsible for:

- projecting governed `vswhere.exe`, Visual Studio Build Tools, and Windows Kits sources into their canonical Windows paths inside the box
- importing `VsDevCmd.bat` through boxed `cmd.exe`
- binding the boxed Python helper into the active shell
- projecting the shared `.NET Framework` compiler tree into `C:\Windows\Microsoft.NET\...`
- normalizing Windows SDK environment values
- ensuring the projected Windows Kits surface includes `DesignTime\CommonConfiguration\Neutral`
- prepending the Windows SDK `bin\<version>\x64` path
- publishing helper metadata back into the shell

The same shared Node bootstrap now also owns the boxed `node-gyp` wrapper publication surface.

Representative current publication:

```powershell
$nodeGypWrapperPath = Join-Path $BootstrapBin 'node-gyp-wrapper.cjs'

Write-AsciiFile -Path $nodeGypWrapperPath -Content $nodeGypWrapperContent
Write-AsciiFile -Path (Join-Path $BootstrapBin 'node-gyp.cmd') -Content $nodeGypCmdContent
Write-AsciiFile -Path (Join-Path $BootstrapBin 'node-gyp.ps1') -Content $nodeGypPs1Content
Write-AsciiFile -Path (Join-Path $BootstrapBin 'node-gyp') -Content $nodeGypShellContent

$env:BOXED_NODE_GYP_JS = $nodeGypWrapperPath
$env:BOXED_NODE_GYP_REAL_JS = $nodeGypJs
$env:npm_config_node_gyp = $nodeGypWrapperPath
```

That means the boxed project shell now exposes a bootstrap-owned `node-gyp` command surface without requiring any edits inside downloaded dependency sources.

## Project-owned install integration

The current project-owned PNPM install script now explicitly opts into the `node-gyp` helper before it runs `pnpm install`:

```powershell
& $launcher -Action OpenTerminal -RepoPath $RepoPath

if ([string]::IsNullOrWhiteSpace($env:BOXED_GIT_ROOT)) {
  throw 'BOXED_GIT_ROOT was not initialized by project bootstrap.'
}

$bashExe = Join-Path $env:BOXED_GIT_ROOT 'bin\bash.exe'
if (-not (Test-Path -LiteralPath $bashExe)) {
  $bashExe = Join-Path $env:BOXED_GIT_ROOT 'usr\bin\bash.exe'
}

if (-not (Test-Path -LiteralPath $bashExe)) {
  throw 'Local boxed Git Bash executable not found.'
}

if ([string]::IsNullOrWhiteSpace($env:BOXED_CMD_EXE)) {
  throw 'BOXED_CMD_EXE was not initialized by project bootstrap.'
}

if (-not (Test-Path -LiteralPath $env:BOXED_CMD_EXE)) {
  throw 'Local boxed CMD executable not found.'
}

$nativeBuildRuntime = Initialize-NodeGypWindowsBuildEnvironment `
  -CmdExe $env:BOXED_CMD_EXE `
  -RegExe $env:BOXED_REG_EXE `
  -PythonExe $env:BOXED_PYTHON_EXE

Write-Host "ProjectedVsWhereExe: $($nativeBuildRuntime.VsWhereExe)"
Write-Host "ProjectedVsRoot: $($nativeBuildRuntime.VSRoot)"
Write-Host "ProjectedWindowsSdkRoot: $($nativeBuildRuntime.WindowsSdkRoot)"
Write-Host "ProjectedDotNetFramework64Csc: $($nativeBuildRuntime.DotNetFramework64CscExe)"

Write-Host "LifecycleShell: $bashExe"
pnpm config set --location=project scriptShell "$bashExe"
pnpm install
```

This is the current bootstrap-owned answer for package installs that may auto-trigger `node-gyp`.

Architecturally important nuance:

- the project script does **not** patch `node-gyp` in `node_modules`
- the project script enters the normal boxed bootstrap
- the bootstrap-published `node-gyp` command surface and `npm_config_node_gyp` metadata then carry the wrapper behavior into native build flows
- the native-build helper still uses boxed `cmd.exe` for `VsDevCmd.bat` import
- but the PNPM lifecycle itself returns to Git Bash as the preferred install shell

## Project-owned clean-reinstall integration

The current clean-reinstall script also opts into the same helper before the fresh install:

```powershell
$uninstallScript = Join-Path $PSScriptRoot 'Start-TestMonoPnpmUninstall.ps1'
& $uninstallScript -RepoPath $resolvedRepoPath -SkipBootstrap

if ([string]::IsNullOrWhiteSpace($env:BOXED_GIT_ROOT)) {
  throw 'BOXED_GIT_ROOT was not initialized by project bootstrap.'
}

$bashExe = Join-Path $env:BOXED_GIT_ROOT 'bin\bash.exe'
if (-not (Test-Path -LiteralPath $bashExe)) {
  $bashExe = Join-Path $env:BOXED_GIT_ROOT 'usr\bin\bash.exe'
}

if (-not (Test-Path -LiteralPath $bashExe)) {
  throw 'Local boxed Git Bash executable not found.'
}

if ([string]::IsNullOrWhiteSpace($env:BOXED_CMD_EXE)) {
  throw 'BOXED_CMD_EXE was not initialized by project bootstrap.'
}

if (-not (Test-Path -LiteralPath $env:BOXED_CMD_EXE)) {
  throw 'Local boxed CMD executable not found.'
}

$nativeBuildRuntime = Initialize-NodeGypWindowsBuildEnvironment `
  -CmdExe $env:BOXED_CMD_EXE `
  -RegExe $env:BOXED_REG_EXE `
  -PythonExe $env:BOXED_PYTHON_EXE

Write-Host "VsRoot: $($nativeBuildRuntime.VSRoot)"
Write-Host "VsDevCmd: $($nativeBuildRuntime.VsDevCmd)"
Write-Host "WindowsSdkRoot: $($nativeBuildRuntime.WindowsSdkRoot)"
Write-Host "WindowsSdkVersion: $($nativeBuildRuntime.WindowsSdkVersion)"
Write-Host "ProjectedDotNetFramework64Csc: $($nativeBuildRuntime.DotNetFramework64CscExe)"
Write-Host "LifecycleShell: $bashExe"
pnpm config set --location=project scriptShell "$bashExe"
pnpm install
```

This keeps the native-build preparation explicit in the project-owned reinstall surface instead of mutating the generic project bootstrap path.

## Project-owned uninstall integration

The current uninstall surface now prefers PowerShell-native tree deletion and only falls back to the explicit boxed `cmd.exe` lane if needed.

Representative current behavior:

```powershell
function Remove-BoxedPathTree {
  param([string]$Path)

  try {
    Remove-Item -LiteralPath $Path -Recurse -Force -ErrorAction Stop
  }
  catch {
    $cmdExe = $env:BOXED_CMD_EXE
    if (-not [string]::IsNullOrWhiteSpace($cmdExe) -and (Test-Path -LiteralPath $cmdExe)) {
      & $cmdExe '/d' '/c' ('rmdir /s /q "{0}"' -f $Path)
    }

    if (Test-Path -LiteralPath $Path) {
      Remove-Item -LiteralPath $Path -Recurse -Force -ErrorAction Stop
    }
  }
}
```

That keeps the cleanup path bootstrap-owned and box-lane-aware instead of depending on a bare `cmd /c rmdir` call.

## Verified runtime evidence

The following integration points are already verified from the boxed project shell:

- `vswhere.exe` resolves from the projected boxed installer path
- `VsDevCmd.bat` resolves from the projected boxed Visual Studio tree
- `VCINSTALLDIR`, `VCToolsInstallDir`, and `VSINSTALLDIR` are imported into PowerShell
- `WindowsSdkDir` and `WindowsSDKVersion` are imported into PowerShell
- `cl.exe`, `MSBuild.exe`, `rc.exe`, and `mt.exe` resolve after helper execution
- `csc.exe` and `cvtres.exe` resolve after `.NET Framework` projection
- boxed PowerShell `Add-Type` succeeds against the projected `.NET Framework` compiler chain
- a direct boxed `node-gyp rebuild --verbose` succeeds through the bootstrap-published wrapper surface

## Common bootstrap fallback that now matters here

The current common bootstrap no longer assumes `robocopy.exe` is always a safe child-process surface.

`Invoke-RobocopySync` now falls back to a PowerShell-owned tree operation when `robocopy.exe` cannot be executed under the strict box:

```powershell
try {
  & robocopy @arguments | Out-Null
}
catch {
  if ($Mirror) {
    Sync-TreeMirrorWithPowerShell -Source $Source -Destination $Destination
  }
  else {
    Copy-TreeContents -Source $Source -Destination $Destination
  }
}
```

That change matters for `node-gyp` because the toolchain/bootstrap contract now depends on local mirroring being robust even when canonical Windows helper processes are denied on a specific child-process surface.

## Related

- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\runtime-contract.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\msbuild-file-tracking-wrapper.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\scripts.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\scripts.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\install.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\clean-reinstall.md`
