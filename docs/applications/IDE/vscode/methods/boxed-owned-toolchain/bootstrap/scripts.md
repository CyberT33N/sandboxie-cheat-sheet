# Shared Bootstrap Scripts

## Scope

This document contains the current shared PowerShell bootstrap scripts for the boxed-owned-toolchain VS Code method.

These are the reusable shared scripts that live under:

```text
C:\shared\sandbox-toolchains\dev\bootstrap\
```

Project-specific example scripts are documented separately here:

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\scripts.md`

## Why this document exists

The cheat-sheet should not only describe the bootstrap tree conceptually. It should also contain the complete script bodies so the implementation remains reviewable even when the live shared files are stored outside the repository.

## `Bootstrap.Common.psm1`

```powershell
Set-StrictMode -Version Latest

function Assert-PathExists {
  [CmdletBinding()]
  param(
    [Parameter(Mandatory = $true)]
    [string]$Path,

    [Parameter(Mandatory = $true)]
    [string]$Label
  )

  if (-not (Test-Path -LiteralPath $Path)) {
    throw "$Label not found: $Path"
  }
}

function Ensure-Directory {
  [CmdletBinding()]
  param(
    [Parameter(Mandatory = $true)]
    [string]$Path
  )

  if (-not (Test-Path -LiteralPath $Path)) {
    New-Item -ItemType Directory -Force -Path $Path | Out-Null
  }
}

function Copy-FileIfExists {
  [CmdletBinding()]
  param(
    [Parameter(Mandatory = $true)]
    [string]$Source,

    [Parameter(Mandatory = $true)]
    [string]$Destination
  )

  if (-not (Test-Path -LiteralPath $Source)) {
    return
  }

  $parent = Split-Path -Parent $Destination
  if ($parent) {
    Ensure-Directory -Path $parent
  }

  Copy-Item -LiteralPath $Source -Destination $Destination -Force
}

function Test-DirectoryHasContent {
  [CmdletBinding()]
  param(
    [Parameter(Mandatory = $true)]
    [string]$Path
  )

  if (-not (Test-Path -LiteralPath $Path)) {
    return $false
  }

  return @(
    Get-ChildItem -LiteralPath $Path -Force -ErrorAction SilentlyContinue
  ).Count -gt 0
}

function Invoke-RobocopySync {
  [CmdletBinding()]
  param(
    [Parameter(Mandatory = $true)]
    [string]$Source,

    [Parameter(Mandatory = $true)]
    [string]$Destination,

    [switch]$Mirror
  )

  Assert-PathExists -Path $Source -Label 'Sync source'
  Ensure-Directory -Path $Destination

  $arguments = @(
    $Source
    $Destination
  )

  if ($Mirror) {
    $arguments += '/MIR'
  }
  else {
    $arguments += '/E'
  }

  $arguments += @(
    '/R:1'
    '/W:1'
    '/NFL'
    '/NDL'
    '/NJH'
    '/NJS'
    '/NP'
  )

  & robocopy @arguments | Out-Null

  $exitCode = $LASTEXITCODE
  if ($exitCode -ge 8) {
    throw "robocopy failed: $Source -> $Destination (exit $exitCode)"
  }

  $global:LASTEXITCODE = 0
}

function Sync-TreeMirror {
  [CmdletBinding()]
  param(
    [Parameter(Mandatory = $true)]
    [string]$Source,

    [Parameter(Mandatory = $true)]
    [string]$Destination
  )

  Invoke-RobocopySync -Source $Source -Destination $Destination -Mirror
}

function Initialize-TreeIfMissing {
  [CmdletBinding()]
  param(
    [Parameter(Mandatory = $true)]
    [string]$Source,

    [Parameter(Mandatory = $true)]
    [string]$Destination
  )

  if (-not (Test-Path -LiteralPath $Source)) {
    return
  }

  if ((Test-Path -LiteralPath $Destination) -and (Test-DirectoryHasContent -Path $Destination)) {
    return
  }

  Invoke-RobocopySync -Source $Source -Destination $Destination
}

function Write-AsciiFile {
  [CmdletBinding()]
  param(
    [Parameter(Mandatory = $true)]
    [string]$Path,

    [Parameter(Mandatory = $true)]
    [string]$Content
  )

  $parent = Split-Path -Parent $Path
  if ($parent) {
    Ensure-Directory -Path $parent
  }

  [System.IO.File]::WriteAllText($Path, $Content, [System.Text.Encoding]::ASCII)
}

function Prepend-PathEntries {
  [CmdletBinding()]
  param(
    [Parameter(Mandatory = $true)]
    [string[]]$Entries
  )

  $allEntries = New-Object 'System.Collections.Generic.List[string]'
  $seen = New-Object 'System.Collections.Generic.HashSet[string]' ([System.StringComparer]::OrdinalIgnoreCase)

  foreach ($entry in $Entries + ($env:PATH -split ';')) {
    if ([string]::IsNullOrWhiteSpace($entry)) {
      continue
    }

    if ($seen.Add($entry)) {
      $allEntries.Add($entry)
    }
  }

  $env:PATH = ($allEntries -join ';')
  return $env:PATH
}

Export-ModuleMember -Function @(
  'Assert-PathExists',
  'Ensure-Directory',
  'Copy-FileIfExists',
  'Test-DirectoryHasContent',
  'Sync-TreeMirror',
  'Initialize-TreeIfMissing',
  'Write-AsciiFile',
  'Prepend-PathEntries'
)
```

## `Bootstrap.Node.psm1`

```powershell
Set-StrictMode -Version Latest

Import-Module (Join-Path $PSScriptRoot '..\..\core\Bootstrap.Common.psm1') -Force -DisableNameChecking

function Assert-NodeToolchainLayout {
  [CmdletBinding()]
  param(
    [Parameter(Mandatory = $true)]
    [string]$GitRoot,

    [Parameter(Mandatory = $true)]
    [string]$NodeRoot,

    [Parameter(Mandatory = $true)]
    [string]$PnpmCli,

    [hashtable]$AdditionalNodeCommands = @{}
  )

  Assert-PathExists -Path $GitRoot -Label 'Shared Git root'
  Assert-PathExists -Path (Join-Path $GitRoot 'cmd\git.exe') -Label 'Git executable'
  Assert-PathExists -Path $NodeRoot -Label 'Primary Node root'
  Assert-PathExists -Path (Join-Path $NodeRoot 'node.exe') -Label 'Primary Node executable'
  Assert-PathExists -Path $PnpmCli -Label 'pnpm CLI entrypoint'

  foreach ($name in $AdditionalNodeCommands.Keys) {
    Assert-PathExists -Path $AdditionalNodeCommands[$name] -Label "Additional Node executable '$name'"
  }
}

function Initialize-NodeToolchainRuntime {
  [CmdletBinding()]
  param(
    [Parameter(Mandatory = $true)]
    [string]$GitRoot,

    [Parameter(Mandatory = $true)]
    [string]$NodeRoot,

    [Parameter(Mandatory = $true)]
    [string]$PnpmCli,

    [Parameter(Mandatory = $true)]
    [string]$BootstrapBin,

    [hashtable]$AdditionalNodeCommands = @{}
  )

  Assert-NodeToolchainLayout `
    -GitRoot $GitRoot `
    -NodeRoot $NodeRoot `
    -PnpmCli $PnpmCli `
    -AdditionalNodeCommands $AdditionalNodeCommands

  Ensure-Directory -Path $BootstrapBin

  $primaryNodeExe = Join-Path $NodeRoot 'node.exe'

  Write-AsciiFile -Path (Join-Path $BootstrapBin 'pnpm.cmd') -Content @"
@echo off
set "NODE_EXE=$primaryNodeExe"
set "PNPM_CLI=$PnpmCli"
"%NODE_EXE%" "%PNPM_CLI%" %*
"@

  foreach ($name in $AdditionalNodeCommands.Keys) {
    $nodeExe = $AdditionalNodeCommands[$name]

    Write-AsciiFile -Path (Join-Path $BootstrapBin "$name.cmd") -Content @"
@echo off
"$nodeExe" %*
"@
  }

  Prepend-PathEntries -Entries @(
    $BootstrapBin
    (Join-Path $GitRoot 'cmd')
    $NodeRoot
  ) | Out-Null

  $env:BOXED_GIT_ROOT = $GitRoot
  $env:BOXED_NODE_ROOT = $NodeRoot
  $env:BOXED_PNPM_CLI = $PnpmCli
  $env:BOXED_BOOTSTRAP_BIN = $BootstrapBin

  return [pscustomobject]@{
    GitRoot = $GitRoot
    NodeRoot = $NodeRoot
    PnpmCli = $PnpmCli
    BootstrapBin = $BootstrapBin
    AdditionalNodeCommands = $AdditionalNodeCommands
  }
}

Export-ModuleMember -Function @(
  'Assert-NodeToolchainLayout',
  'Initialize-NodeToolchainRuntime'
)
```

## `Bootstrap.VSCode.psm1`

```powershell
Set-StrictMode -Version Latest

Import-Module (Join-Path $PSScriptRoot '..\..\core\Bootstrap.Common.psm1') -Force -DisableNameChecking

function Assert-VSCodeLayout {
  [CmdletBinding()]
  param(
    [Parameter(Mandatory = $true)]
    [string]$CodeExe,

    [Parameter(Mandatory = $true)]
    [string]$CodeCli,

    [Parameter(Mandatory = $true)]
    [string]$CatalogUserRoot,

    [Parameter(Mandatory = $true)]
    [string]$SharedExtensionsRoot
  )

  Assert-PathExists -Path $CodeExe -Label 'VS Code GUI'
  Assert-PathExists -Path $CodeCli -Label 'VS Code CLI'
  Ensure-Directory -Path $CatalogUserRoot
  Ensure-Directory -Path $SharedExtensionsRoot
}

function New-VSCodeProjectPaths {
  [CmdletBinding()]
  param(
    [Parameter(Mandatory = $true)]
    [string]$ProjectName
  )

  $boxRoot = Join-Path $env:APPDATA "VSCodeBoxes\$ProjectName"

  return [pscustomobject]@{
    BoxRoot = $boxRoot
    UserData = Join-Path $boxRoot 'user-data'
    ExtensionsDir = Join-Path $boxRoot 'extensions'
    BootstrapBin = Join-Path $boxRoot 'bootstrap-bin'
    LocalGlobalStorage = Join-Path $boxRoot 'user-data\User\globalStorage'
    RooPath = Join-Path $env:USERPROFILE '.roo'
  }
}

function Sync-VSCodeUserCatalog {
  [CmdletBinding()]
  param(
    [Parameter(Mandatory = $true)]
    [string]$CatalogUserRoot,

    [Parameter(Mandatory = $true)]
    [string]$UserDataDir
  )

  $userRoot = Join-Path $UserDataDir 'User'
  $settingsPath = Join-Path $CatalogUserRoot 'settings.json'
  $keybindingsPath = Join-Path $CatalogUserRoot 'keybindings.json'
  $snippetsSource = Join-Path $CatalogUserRoot 'snippets'
  $snippetsTarget = Join-Path $userRoot 'snippets'

  Ensure-Directory -Path $userRoot

  Copy-FileIfExists -Source $settingsPath -Destination (Join-Path $userRoot 'settings.json')
  Copy-FileIfExists -Source $keybindingsPath -Destination (Join-Path $userRoot 'keybindings.json')

  if (Test-Path -LiteralPath $snippetsSource) {
    Sync-TreeMirror -Source $snippetsSource -Destination $snippetsTarget
  }
}

function Initialize-VSCodeSeedRuntime {
  [CmdletBinding()]
  param(
    [string]$SeedGlobalStorageRoot,
    [string]$LocalGlobalStorageDir,
    [string]$SeedRooRoot,
    [string]$LocalRooDir
  )

  if (-not [string]::IsNullOrWhiteSpace($SeedGlobalStorageRoot) -and -not [string]::IsNullOrWhiteSpace($LocalGlobalStorageDir)) {
    Initialize-TreeIfMissing -Source $SeedGlobalStorageRoot -Destination $LocalGlobalStorageDir
  }

  if (-not [string]::IsNullOrWhiteSpace($SeedRooRoot) -and -not [string]::IsNullOrWhiteSpace($LocalRooDir)) {
    Initialize-TreeIfMissing -Source $SeedRooRoot -Destination $LocalRooDir
  }
}

function Sync-VSCodeExtensionsMirror {
  [CmdletBinding()]
  param(
    [Parameter(Mandatory = $true)]
    [string]$SharedExtensionsRoot,

    [Parameter(Mandatory = $true)]
    [string]$LocalExtensionsDir
  )

  Ensure-Directory -Path $SharedExtensionsRoot
  Sync-TreeMirror -Source $SharedExtensionsRoot -Destination $LocalExtensionsDir
}

function Invoke-VSCodeCliInstallExtension {
  [CmdletBinding()]
  param(
    [Parameter(Mandatory = $true)]
    [string]$CodeCli,

    [Parameter(Mandatory = $true)]
    [string]$UserDataDir,

    [Parameter(Mandatory = $true)]
    [string]$ExtensionsDir,

    [Parameter(Mandatory = $true)]
    [string]$ExtensionId
  )

  & $CodeCli `
    --user-data-dir $UserDataDir `
    --extensions-dir $ExtensionsDir `
    --install-extension $ExtensionId

  return $LASTEXITCODE
}

function Invoke-VSCodeCliListExtensions {
  [CmdletBinding()]
  param(
    [Parameter(Mandatory = $true)]
    [string]$CodeCli,

    [Parameter(Mandatory = $true)]
    [string]$UserDataDir,

    [Parameter(Mandatory = $true)]
    [string]$ExtensionsDir
  )

  & $CodeCli `
    --user-data-dir $UserDataDir `
    --extensions-dir $ExtensionsDir `
    --list-extensions

  return $LASTEXITCODE
}

function Invoke-VSCodeGui {
  [CmdletBinding()]
  param(
    [Parameter(Mandatory = $true)]
    [string]$CodeExe,

    [Parameter(Mandatory = $true)]
    [string]$UserDataDir,

    [Parameter(Mandatory = $true)]
    [string]$ExtensionsDir,

    [string]$WorkspacePath
  )

  $arguments = @(
    '--user-data-dir'
    $UserDataDir
    '--extensions-dir'
    $ExtensionsDir
  )

  if (-not [string]::IsNullOrWhiteSpace($WorkspacePath)) {
    $arguments += $WorkspacePath
  }

  & $CodeExe @arguments
  return $LASTEXITCODE
}

Export-ModuleMember -Function @(
  'Assert-VSCodeLayout',
  'New-VSCodeProjectPaths',
  'Sync-VSCodeUserCatalog',
  'Initialize-VSCodeSeedRuntime',
  'Sync-VSCodeExtensionsMirror',
  'Invoke-VSCodeCliInstallExtension',
  'Invoke-VSCodeCliListExtensions',
  'Invoke-VSCodeGui'
)
```

## `Start-VSCodeMaintenance.ps1`

```powershell
param(
  [ValidateSet('InstallExtension', 'ListExtensions', 'OpenTerminal', 'LaunchVSCode')]
  [string]$Action = 'OpenTerminal',

  [string]$ExtensionId,

  [string]$SharedRoot = 'C:\shared\sandbox-toolchains',

  [string]$VSCodeVersion = '1.121.0'
)

$ErrorActionPreference = 'Stop'
Set-StrictMode -Version Latest

Import-Module (Join-Path $PSScriptRoot 'Bootstrap.VSCode.psm1') -Force -DisableNameChecking -Global

$vsCodeRoot = Join-Path $SharedRoot 'ide\vscode'
$codeExe = Join-Path $vsCodeRoot "runtime\$VSCodeVersion\Code.exe"
$codeCli = Join-Path $vsCodeRoot "runtime\$VSCodeVersion\bin\code.cmd"
$catalogUserRoot = Join-Path $vsCodeRoot 'catalog\vscode-user'
$sharedExtensionsRoot = Join-Path $vsCodeRoot 'extensions'
$maintenanceUserData = Join-Path $vsCodeRoot 'maintenance\user-data'

Assert-VSCodeLayout `
  -CodeExe $codeExe `
  -CodeCli $codeCli `
  -CatalogUserRoot $catalogUserRoot `
  -SharedExtensionsRoot $sharedExtensionsRoot

if (-not (Test-Path -LiteralPath $maintenanceUserData)) {
  New-Item -ItemType Directory -Force -Path $maintenanceUserData | Out-Null
}
Sync-VSCodeUserCatalog -CatalogUserRoot $catalogUserRoot -UserDataDir $maintenanceUserData

$env:BOXED_VSCODE_MODE = 'Maintenance'
$env:BOXED_VSCODE_USERDATA = $maintenanceUserData
$env:BOXED_SHARED_EXTENSIONS = $sharedExtensionsRoot

switch ($Action) {
  'InstallExtension' {
    if ([string]::IsNullOrWhiteSpace($ExtensionId)) {
      throw 'ExtensionId is required for -Action InstallExtension.'
    }

    $exitCode = Invoke-VSCodeCliInstallExtension `
      -CodeCli $codeCli `
      -UserDataDir $maintenanceUserData `
      -ExtensionsDir $sharedExtensionsRoot `
      -ExtensionId $ExtensionId

    exit $exitCode
  }

  'ListExtensions' {
    $exitCode = Invoke-VSCodeCliListExtensions `
      -CodeCli $codeCli `
      -UserDataDir $maintenanceUserData `
      -ExtensionsDir $sharedExtensionsRoot

    exit $exitCode
  }

  'LaunchVSCode' {
    $exitCode = Invoke-VSCodeGui `
      -CodeExe $codeExe `
      -UserDataDir $maintenanceUserData `
      -ExtensionsDir $sharedExtensionsRoot

    exit $exitCode
  }

  'OpenTerminal' {
    Write-Host 'Maintenance terminal ready.'
    Write-Host "UserData: $maintenanceUserData"
    Write-Host "SharedExtensions: $sharedExtensionsRoot"
    return
  }

  default {
    throw "Unsupported action: $Action"
  }
}
```

## `Start-VSCodeProjectBase.ps1`

```powershell
param(
  [ValidateSet('LaunchVSCode', 'OpenTerminal')]
  [string]$Action = 'LaunchVSCode',

  [Parameter(Mandatory = $true)]
  [string]$ProjectName,

  [Parameter(Mandatory = $true)]
  [string]$RepoPath,

  [Parameter(Mandatory = $true)]
  [string]$CodeExe,

  [Parameter(Mandatory = $true)]
  [string]$CodeCli,

  [Parameter(Mandatory = $true)]
  [string]$CatalogUserRoot,

  [Parameter(Mandatory = $true)]
  [string]$SharedExtensionsRoot,

  [string]$SeedGlobalStorageRoot,

  [string]$SeedRooRoot,

  [Parameter(Mandatory = $true)]
  [string]$GitRoot,

  [Parameter(Mandatory = $true)]
  [string]$NodeRoot,

  [Parameter(Mandatory = $true)]
  [string]$PnpmCli,

  [hashtable]$AdditionalNodeCommands = @{}
)

$ErrorActionPreference = 'Stop'
Set-StrictMode -Version Latest

Import-Module (Join-Path $PSScriptRoot 'Bootstrap.VSCode.psm1') -Force -DisableNameChecking -Global
Import-Module (Join-Path $PSScriptRoot '..\..\stacks\node\Bootstrap.Node.psm1') -Force -DisableNameChecking -Global

if (-not (Test-Path -LiteralPath $RepoPath)) {
  throw "Project repo not found: $RepoPath"
}

Assert-VSCodeLayout `
  -CodeExe $CodeExe `
  -CodeCli $CodeCli `
  -CatalogUserRoot $CatalogUserRoot `
  -SharedExtensionsRoot $SharedExtensionsRoot

$projectPaths = New-VSCodeProjectPaths -ProjectName $ProjectName

if (-not (Test-Path -LiteralPath $projectPaths.UserData)) {
  New-Item -ItemType Directory -Force -Path $projectPaths.UserData | Out-Null
}

if (-not (Test-Path -LiteralPath $projectPaths.ExtensionsDir)) {
  New-Item -ItemType Directory -Force -Path $projectPaths.ExtensionsDir | Out-Null
}

if (-not (Test-Path -LiteralPath $projectPaths.BootstrapBin)) {
  New-Item -ItemType Directory -Force -Path $projectPaths.BootstrapBin | Out-Null
}

Sync-VSCodeUserCatalog -CatalogUserRoot $CatalogUserRoot -UserDataDir $projectPaths.UserData
Sync-VSCodeExtensionsMirror -SharedExtensionsRoot $SharedExtensionsRoot -LocalExtensionsDir $projectPaths.ExtensionsDir

Initialize-VSCodeSeedRuntime `
  -SeedGlobalStorageRoot $SeedGlobalStorageRoot `
  -LocalGlobalStorageDir $projectPaths.LocalGlobalStorage `
  -SeedRooRoot $SeedRooRoot `
  -LocalRooDir $projectPaths.RooPath

$nodeRuntime = Initialize-NodeToolchainRuntime `
  -GitRoot $GitRoot `
  -NodeRoot $NodeRoot `
  -PnpmCli $PnpmCli `
  -BootstrapBin $projectPaths.BootstrapBin `
  -AdditionalNodeCommands $AdditionalNodeCommands

$env:BOXED_VSCODE_MODE = 'Project'
$env:BOXED_PROJECT_NAME = $ProjectName
$env:BOXED_REPO_PATH = $RepoPath
$env:BOXED_VSCODE_USERDATA = $projectPaths.UserData
$env:BOXED_LOCAL_EXTENSIONS = $projectPaths.ExtensionsDir
$env:BOXED_SHARED_EXTENSIONS = $SharedExtensionsRoot
$env:BOXED_NODE_ROOT = $nodeRuntime.NodeRoot
$env:BOXED_PNPM_CLI = $nodeRuntime.PnpmCli
$env:BOXED_NODE_EXTRA_COMMANDS = (($AdditionalNodeCommands.Keys | Sort-Object) -join ',')

Set-Location $RepoPath

switch ($Action) {
  'LaunchVSCode' {
    $null = Invoke-VSCodeGui `
      -CodeExe $CodeExe `
      -UserDataDir $projectPaths.UserData `
      -ExtensionsDir $projectPaths.ExtensionsDir `
      -WorkspacePath $RepoPath

    return
  }

  'OpenTerminal' {
    Write-Host 'Project terminal ready.'
    Write-Host "ProjectName: $ProjectName"
    Write-Host "RepoPath: $RepoPath"
    Write-Host "UserData: $($projectPaths.UserData)"
    Write-Host "LocalExtensions: $($projectPaths.ExtensionsDir)"
    Write-Host 'Commands on PATH: git, node, pnpm'

    if ($AdditionalNodeCommands.Count -gt 0) {
      Write-Host ("Additional commands: " + (($AdditionalNodeCommands.Keys | Sort-Object) -join ', '))
    }

    return
  }

  default {
    throw "Unsupported action: $Action"
  }
}
```

## Related

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\shared-layout.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\scripts.md`
