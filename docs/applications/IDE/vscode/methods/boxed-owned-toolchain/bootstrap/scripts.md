# Shared Bootstrap Scripts

## Scope

This document captures the **current shared bootstrap script contract** for the boxed-owned-toolchain VS Code method.

It documents:

- the exact live shared script paths
- the responsibility split between the script layers
- the current runtime contracts that matter architecturally
- selected current code excerpts that define the behavior

The live shared files under `C:\shared\sandbox-toolchains\...` remain the operational implementation.

## Current live script inventory

### Core

- `C:\shared\sandbox-toolchains\dev\bootstrap\core\Bootstrap.Common.psm1`

### VS Code platform

- `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode\Bootstrap.VSCode.psm1`
- `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode\Start-VSCodeMaintenance.ps1`
- `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode\Start-VSCodeProjectBase.ps1`
- `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode\Publish-VSCodeMaintenance.ps1`

### Toolchain stacks

- `C:\shared\sandbox-toolchains\dev\bootstrap\stacks\node\Bootstrap.Node.psm1`
- `C:\shared\sandbox-toolchains\dev\bootstrap\stacks\shells\Bootstrap.WindowsShells.psm1`
- `C:\shared\sandbox-toolchains\dev\bootstrap\stacks\python\Bootstrap.Python.psm1`
- `C:\shared\sandbox-toolchains\dev\bootstrap\stacks\starship\Bootstrap.Starship.psm1`

### Project adapter example

- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Project.Config.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoVSCode.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoTerminal.ps1`

## Why this document changed

Older repository text described:

- `%APPDATA%\VSCodeBoxes\...`
- shared live maintenance `user-data`
- direct runtime use from shared without the current local mirror/publish shape

That is no longer the current implementation.

The current architecture is:

- shared canonical source artifacts
- local mirrored execution
- local maintenance authoring state
- explicit promotion back into shared canonical surfaces

## `Bootstrap.Common.psm1`

This is the filesystem/bootstrap helper kernel.

Current helper families include:

- path assertions
- directory creation
- file copy helpers
- tree mirroring
- PowerShell fallback tree mirroring when `robocopy.exe` is denied on the current child-process surface
- tree-copy initialization with exclusion support
- ASCII file generation
- `PATH` prefix composition

Representative current helpers:

```powershell
function Copy-TreeContents {
  param(
    [string]$Source,
    [string]$Destination,
    [string[]]$ExcludeRelativePaths = @()
  )

  # copies a source tree into a destination while allowing excluded relative paths
}

function Initialize-TreeCopyIfMissing {
  param(
    [string]$Source,
    [string]$Destination,
    [string[]]$ExcludeRelativePaths = @()
  )

  # initializes a destination tree once from a source tree copy
}

function Sync-TreeMirrorWithPowerShell {
  param(
    [string]$Source,
    [string]$Destination
  )

  # mirrors a tree through PowerShell-native file operations when robocopy is not usable
}
```

These helpers are what make local runtime mirroring practical without forcing every higher-level script to reimplement filesystem logic.

## `Bootstrap.Node.psm1`

This is the governed Node/Git/PNPM runtime adapter.

Current responsibilities:

- validate the shared Git / Node / PNPM surfaces
- mirror them locally into the box execution tree
- prepare the boxed Windows native-build environment for direct `node-gyp` usage
- generate wrapper commands such as `pnpm.cmd`
- generate wrapper commands such as `nx.cmd`
- generate wrapper commands such as `pnpm.ps1`
- generate wrapper commands such as `nx.ps1`
- generate shell-native wrappers such as `pnpm`
- generate shell-native wrappers such as `nx`
- generate additional node aliases such as `node20.cmd`
- generate additional node aliases such as `node20.ps1`
- generate shell-native additional aliases such as `node20`
- prepend the correct local runtime paths into `PATH`
- publish a boxed `ComSpec` / `COMSPEC` shell contract for Windows shell-based child-process execution
- keep that productive child-process contract aligned to the preferred boxed `cmd.exe` lane

Representative current contract:

```powershell
function Initialize-NodeToolchainMirror {
  param(
    [string]$GitRoot,
    [string]$NodeRoot,
    [string]$PnpmCli,
    [string]$LocalToolchainRoot,
    [hashtable]$AdditionalNodeCommands = @{}
  )

  # mirrors Git, Node, PNPM, and optional additional Node runtimes locally
}

function Initialize-NodeToolchainRuntime {
  param(
    [string]$GitRoot,
    [string]$NodeRoot,
    [string]$PnpmCli,
    [string]$BootstrapBin,
    [string]$LocalToolchainRoot,
    [hashtable]$AdditionalNodeCommands = @{}
  )

  # generates wrapper commands and exposes the local mirrored toolchain
}

function Initialize-NodeGypWindowsBuildEnvironment {
  param(
    [string]$CmdExe,
    [string]$RegExe,
    [string]$PythonExe,
    [string]$WindowsSdkRoot = 'C:\Program Files (x86)\Windows Kits\10'
  )

  # imports VsDevCmd.bat through boxed cmd.exe, binds the boxed Python helper,
  # normalizes Windows SDK env values, and publishes boxed node-gyp helper metadata
}
```

Representative wrapper publication now includes all three relevant shell families:

```powershell
Write-AsciiFile -Path (Join-Path $BootstrapBin 'pnpm.cmd') -Content $pnpmCmdContent
Write-AsciiFile -Path (Join-Path $BootstrapBin 'pnpm.ps1') -Content $pnpmPs1Content
Write-AsciiFile -Path (Join-Path $BootstrapBin 'pnpm') -Content $pnpmShellContent

Write-AsciiFile -Path (Join-Path $BootstrapBin 'nx.cmd') -Content $nxCmdContent
Write-AsciiFile -Path (Join-Path $BootstrapBin 'nx.ps1') -Content $nxPs1Content
Write-AsciiFile -Path (Join-Path $BootstrapBin 'nx') -Content $nxShellContent

Write-AsciiFile -Path (Join-Path $BootstrapBin 'node20.cmd') -Content $nodeCmdContent
Write-AsciiFile -Path (Join-Path $BootstrapBin 'node20.ps1') -Content $nodePs1Content
Write-AsciiFile -Path (Join-Path $BootstrapBin 'node20') -Content $nodeShellContent
```

This matters because the current contract must support:

- PowerShell command resolution
- CMD command resolution
- Git Bash command resolution
- direct native-build environment preparation for `node-gyp`-bearing install/reinstall flows

at the same time.

## `Bootstrap.Python.psm1`

This is the version-pointer-based Python runtime adapter.

Current responsibilities:

- read `C:\shared\sandbox-toolchains\dev\python\current.txt`
- validate the selected shared Python runtime
- mirror that version locally into the box execution tree
- prepend the local Python root into `PATH`

Representative current contract:

```powershell
function Get-SharedPythonCurrentVersion {
  param([string]$PythonRoot)
}

function Initialize-PythonToolchainRuntime {
  param(
    [string]$PythonRoot,
    [string]$LocalToolchainRoot
  )

  # resolves the selected version and mirrors it locally
}
```

## `Bootstrap.Starship.psm1`

This is the Starship prompt/runtime adapter.

Current responsibilities:

- validate whether shared Starship exists
- mirror Starship locally
- prepend the local Starship directory into `PATH`
- generate `bash.minimal.rc`
- generate `bash.starship.rc`

The current Bash RC contract is important because Git Bash terminals rely on it to expose bootstrap-generated wrappers such as:

- `pnpm`
- `node20`

inside the shell `PATH`.

Representative current contract:

```powershell
function Initialize-StarshipRuntime {
  param(
    [string]$StarshipRoot,
    [string]$LocalToolchainRoot,
    [string]$BootstrapBin,
    [string]$StarshipConfigPath
  )

  # mirrors Starship locally and writes the Bash RC files
}
```

## `Bootstrap.WindowsShells.psm1`

This is the Windows shell runtime adapter.

Current responsibilities:

- mirror governed shared `cmd.exe` locally
- mirror governed shared Windows PowerShell locally
- mirror governed shared `reg.exe` locally
- mirror governed shared `Clink` locally when provisioned
- prepend the local mirrored `Clink` root into `PATH`
- prepend the local mirrored `reg.exe` root into `PATH` when provisioned
- generate:
  - `cmd.minimal.init.cmd`
  - `cmd.starship.init.cmd`
  - `powershell.minimal.init.ps1`
  - `powershell.starship.init.ps1`
- keep mutable Clink state box-local under:
  - `state\shells\cmd\clink\profile\`

Important current nuance:

- `cmd.exe`, Windows PowerShell, `reg.exe`, and `Clink` are all governed shared shell/helper artifacts under `dev\shells\...`
- bootstrap mirrors them locally into the box execution tree
- mutable `Clink` state still remains box-local

Representative current contract:

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

  # mirrors the governed shared Windows shell/helper artifacts and prepares CMD/PowerShell init files
}
```

## `Bootstrap.VSCode.psm1`

This is the VS Code platform adapter.

Current responsibilities:

- compute the local box-root-aligned state and execution paths
- mirror the VS Code runtime locally
- mirror shared extensions into local runtime state
- initialize local maintenance authoring state
- publish/promote approved maintenance state back into shared surfaces

Current path model:

```powershell
function New-VSCodeProjectPaths {
  param([string]$ProjectName)

  $boxRoot = Join-Path $env:ProgramFiles "SandboxToolchains\VSCodeBoxes\$ProjectName"
  $stateRoot = Join-Path $boxRoot 'state'
  $localExecutionRoot = Join-Path $boxRoot 'execution'
}

function New-VSCodeMaintenancePaths {
  $boxRoot = Join-Path $env:ProgramFiles 'SandboxToolchains\VSCodeBoxes\maintenance'
}
```

Current mirror/publish contract:

```powershell
function Initialize-VSCodeRuntimeMirror {
  param(
    [string]$CodeExe,
    [string]$CodeCli,
    [string]$LocalRuntimeRoot
  )

  # mirrors the shared VS Code runtime locally and excludes blocked components
}

function Initialize-VSCodeMaintenanceAuthoringState {
  # initializes local maintenance catalog/extensions/seeds
}

function Publish-VSCodeMaintenanceAuthoringState {
  # promotes local maintenance extensions/catalog/seeds back into shared surfaces
}
```

## `Start-VSCodeMaintenance.ps1`

This is the maintenance control-plane entry point.

Current behavior:

- mirrors the VS Code runtime locally
- initializes local maintenance authoring state
- initializes local Node/Python/Starship toolchain layers
- sets the Nx environment contract
- sets local temp/cache paths
- sets the productive boxed-CMD child-process contract
- exposes the promotion script path

Representative current initialization:

```powershell
$maintenancePaths = New-VSCodeMaintenancePaths
$localRuntimeRoot = Join-Path $maintenancePaths.LocalRuntimeRoot "vscode\$VSCodeVersion"
$localNxCacheRoot = Join-Path $maintenancePaths.LocalCacheRoot 'nx-native'
$promotionScript = Join-Path $PSScriptRoot 'Publish-VSCodeMaintenance.ps1'

$localRuntime = Initialize-VSCodeRuntimeMirror `
  -CodeExe $codeExe `
  -CodeCli $codeCli `
  -LocalRuntimeRoot $localRuntimeRoot

Initialize-VSCodeMaintenanceAuthoringState `
  -SharedCatalogUserRoot $catalogUserRoot `
  -SharedExtensionsRoot $sharedExtensionsRoot `
  -SharedSeedGlobalStorageRoot $sharedSeedGlobalStorageRoot `
  -SharedSeedRooRoot $sharedSeedRooRoot `
  -MaintenancePaths $maintenancePaths
```

Current env contract highlights:

```powershell
$boxedComSpec = $windowsShellRuntime.CmdExe
if ([string]::IsNullOrWhiteSpace($boxedComSpec) -or -not (Test-Path -LiteralPath $boxedComSpec)) {
  throw 'Local boxed CMD executable not found for ComSpec override.'
}

$env:NX_DAEMON = 'false'
$env:NX_SOCKET_DIR = 'C:\nxs'
$env:NX_ISOLATE_PLUGINS = 'false'
$env:ComSpec = $boxedComSpec
$env:COMSPEC = $boxedComSpec
$env:BOXED_COMSPEC = $boxedComSpec
$env:BOXED_NX_LAUNCHER = $nodeRuntime.NxLauncher
$env:BOXED_PROMOTION_SCRIPT = $promotionScript
$env:BOXED_CODE_CLI = $localRuntime.CodeCli
$env:BOXED_LOCAL_EXTENSIONS = $maintenancePaths.ExtensionsDir
```

## `Start-VSCodeProjectBase.ps1`

This is the reusable project-box entry point.

Current behavior:

- validates the repo path
- mirrors the VS Code runtime locally
- mirrors the shared extension store locally
- initializes seeds
- initializes Node, Windows-shell, optional Python, and Starship layers
- publishes explicit boxed helper lanes including `BOXED_REG_EXE`
- sets local temp/Nx environment state
- sets the productive boxed-CMD child-process contract

## Why this changed

The current integrated Git Bash path exposed one more shell-specific problem:

- the bootstrap already generated `pnpm.cmd`
- PowerShell/CMD could use that command surface
- but Git Bash still reported `bash: pnpm: command not found`

The reason was:

1. a `.cmd` wrapper alone is not a sufficient command surface for bare `pnpm` resolution in Git Bash
2. the Bash RC startup path also needs the local `bootstrap-bin` directory on `PATH`

So the current boxed-owned-toolchain contract is now:

- `Bootstrap.Node.psm1` generates both Windows wrapper commands and shell-native wrappers
- `Bootstrap.Starship.psm1` writes Bash RC files that prepend `bootstrap-bin` to `PATH`
- the integrated Git Bash terminal resolves `pnpm` through the shell-native wrapper rather than depending on `.cmd` lookup behavior

The current Windows-shell contract adds another lane:

- `Bootstrap.WindowsShells.psm1` mirrors local `cmd.exe` and local Windows PowerShell into the box execution tree
- it provisions a box-local Clink profile directory for the `CMD + Starship` lane
- it keeps CMD plain usable without requiring Clink
- it lets bootstrap use boxed `cmd.exe` as the productive `ComSpec` / `COMSPEC` lane while boxed PowerShell stays the preferred interactive default

Representative current initialization:

```powershell
$projectPaths = New-VSCodeProjectPaths -ProjectName $ProjectName
$localRuntimeRoot = Join-Path $projectPaths.LocalRuntimeRoot "vscode\$vsCodeVersion"
$localRuntime = Initialize-VSCodeRuntimeMirror `
  -CodeExe $CodeExe `
  -CodeCli $CodeCli `
  -LocalRuntimeRoot $localRuntimeRoot

Sync-VSCodeExtensionsMirror -SharedExtensionsRoot $SharedExtensionsRoot -LocalExtensionsDir $projectPaths.ExtensionsDir
$starshipRuntime = Initialize-StarshipRuntime `
  -StarshipRoot $StarshipRoot `
  -LocalToolchainRoot $projectPaths.LocalToolchainRoot `
  -BootstrapBin $projectPaths.BootstrapBin `
  -StarshipConfigPath $StarshipConfigPath
```

Current env contract highlights:

```powershell
$boxedComSpec = $windowsShellRuntime.CmdExe
if ([string]::IsNullOrWhiteSpace($boxedComSpec) -or -not (Test-Path -LiteralPath $boxedComSpec)) {
  throw 'Local boxed CMD executable not found for ComSpec override.'
}

$env:NX_DAEMON = 'false'
$env:NX_SOCKET_DIR = 'C:\nxs'
$env:NX_ISOLATE_PLUGINS = 'false'
$env:ComSpec = $boxedComSpec
$env:COMSPEC = $boxedComSpec
$env:BOXED_COMSPEC = $boxedComSpec
$env:BOXED_NX_SOCKET_DIR = $localNxSocketRoot
$env:BOXED_CODE_EXE = $localRuntime.CodeExe
$env:BOXED_CODE_CLI = $localRuntime.CodeCli
$env:BOXED_LOCAL_TOOLCHAIN_ROOT = $projectPaths.LocalToolchainRoot
```

## `Publish-VSCodeMaintenance.ps1`

This is the current publish/promotion entry point.

It promotes:

- extensions
- catalog user content
- seed content

into the canonical shared surfaces below `C:\shared\sandbox-toolchains\ide\vscode\...`.

Representative contract:

```powershell
Publish-VSCodeMaintenanceAuthoringState `
  -MaintenancePaths $maintenancePaths `
  -SharedCatalogUserRoot $sharedCatalogUserRoot `
  -SharedExtensionsRoot $sharedExtensionsRoot `
  -SharedSeedGlobalStorageRoot $sharedSeedGlobalStorageRoot `
  -SharedSeedRooRoot $sharedSeedRooRoot
```

## Project adapter example

This section uses the sanitized project adapter example `test-mono`.

It is a documentation-safe boilerplate example, not a claim that the live shared project adapter currently uses that literal project name.

Representative contract:

```powershell
return @{
  ProjectName = 'test-mono'
  BoxName = 'VS_CODE_TEST_MONO'
  Toolchain = @{
    GitRoot = Join-Path $devRoot 'git\2.54.0'
    NodeRoot = Join-Path $devRoot 'node\26.2.0\node-v26.2.0-win-x64'
    PnpmCli = Join-Path $devRoot 'pnpm\11.5.0\package\bin\pnpm.cjs'
    PythonRoot = Join-Path $devRoot 'python'
    AdditionalNodeCommands = [ordered]@{
      node20 = Join-Path $devRoot 'node\20.9.0\node-v20.9.0-win-x64\node.exe'
    }
  }
  Shells = @{
    CmdRoot = Join-Path $devRoot 'shells\cmd\10.0.26100.8457'
    PowerShellRoot = Join-Path $devRoot 'shells\powershell\10.0.26100.8457'
    RegRoot = Join-Path $devRoot 'shells\reg\10.0.26100.8457'
    ClinkRoot = Join-Path $devRoot 'shells\clink\1.9.26'
    StarshipRoot = Join-Path $devRoot 'starship\1.25.1'
    StarshipConfigPath = Join-Path $env:USERPROFILE '.config\starship.toml'
  }
}
```

## Operational conclusion

The current bootstrap implementation is no longer:

- APPDATA-centered
- shared-live-maintenance-user-data-centered
- or direct-shared-runtime execution as the live model

It is now:

- local-mirror execution
- local maintenance authoring
- explicit publish/promotion
- governed multi-runtime bootstrap wiring

## Related

- `docs\cli\shell\general.md`
- `docs\cli\shell\clink.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\shared-layout.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\scripts.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\bootstrap-integration.md`
