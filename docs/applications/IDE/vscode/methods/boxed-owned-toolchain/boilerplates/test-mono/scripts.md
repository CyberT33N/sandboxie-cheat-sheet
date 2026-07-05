# `test-mono` Boilerplate Scripts

## Scope

This document contains the complete project-adapter PowerShell boilerplate for a sanitized example project named `test-mono`.

Replace:

- `test-mono`
- `VS_CODE_TEST_MONO`
- `CURSOR_TEST_MONO`
- `C:\Users\yourusername\source\test-mono`

with your real project slug, editor-specific box names, and repo path.

## Why this document lives here

This is not a generic CLI document and not a shared bootstrap-kernel document.

It is a sanitized project adapter example, so the correct documentation location is:

```text
docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\
```

That keeps:

- generic CLI semantics in `docs\cli\...`
- shared bootstrap scripts in `bootstrap\...`
- project-adapter examples in `boilerplates\...`

## `Project.Config.ps1`

Important ownership split:

- the `MicrosoftBuild` block shown below is only the project-adapter pass-through
- the actual Microsoft build-toolchain source of truth now lives in:
  - `docs\applications\operating-systems\windows\build-toolchain\microsoft\general.md`
  - `docs\applications\operating-systems\windows\build-toolchain\microsoft\architectures\boxed-owned-toolchain\overview.md`

```powershell
$sharedRoot = 'C:\shared\sandbox-toolchains'
$vscodeFamilySharedRoot = Join-Path $sharedRoot 'ide\vscode'
$cursorRuntimeRoot = Join-Path $sharedRoot 'ide\cursor'
$devRoot = Join-Path $sharedRoot 'dev'

$sharedEditorState = @{
  CatalogUserRoot = Join-Path $vscodeFamilySharedRoot 'catalog\vscode-user'
  SharedExtensionsRoot = Join-Path $vscodeFamilySharedRoot 'extensions'
  SeedGlobalStorageRoot = Join-Path $vscodeFamilySharedRoot 'catalog\seed\globalStorage'
  SeedRooRoot = Join-Path $vscodeFamilySharedRoot 'catalog\seed\roo'
}

return @{
  SharedRoot = $sharedRoot
  ProjectName = 'test-mono'
  DefaultRepoPath = 'C:\Users\yourusername\source\test-mono'
  VSCode = @{
    BoxName = 'VS_CODE_TEST_MONO'
    RuntimeNamespace = 'vscode'
    CodeExe = Join-Path $vscodeFamilySharedRoot 'runtime\1.121.0\Code.exe'
    CodeCli = Join-Path $vscodeFamilySharedRoot 'runtime\1.121.0\bin\code.cmd'
    CatalogUserRoot = $sharedEditorState.CatalogUserRoot
    SharedExtensionsRoot = $sharedEditorState.SharedExtensionsRoot
    SeedGlobalStorageRoot = $sharedEditorState.SeedGlobalStorageRoot
    SeedRooRoot = $sharedEditorState.SeedRooRoot
  }
  Cursor = @{
    BoxName = 'CURSOR_TEST_MONO'
    RuntimeNamespace = 'cursor'
    CodeExe = Join-Path $cursorRuntimeRoot 'runtime\3.9.16\Cursor.exe'
    CodeCli = Join-Path $cursorRuntimeRoot 'runtime\3.9.16\resources\app\codeBin\code.cmd'
    CatalogUserRoot = $sharedEditorState.CatalogUserRoot
    SharedExtensionsRoot = $sharedEditorState.SharedExtensionsRoot
    SeedGlobalStorageRoot = $sharedEditorState.SeedGlobalStorageRoot
    SeedRooRoot = $sharedEditorState.SeedRooRoot
  }
  Toolchain = @{
    GitRoot = Join-Path $devRoot 'git\2.54.0'
    NodeRoot = Join-Path $devRoot 'node\26.2.0\node-v26.2.0-win-x64'
    PnpmCli = Join-Path $devRoot 'pnpm\11.7.0\package\bin\pnpm.cjs'
    PythonRoot = Join-Path $devRoot 'python'
    AdditionalNodeCommands = [ordered]@{
      node20 = Join-Path $devRoot 'node\20.9.0\node-v20.9.0-win-x64\node.exe'
    }
  }
  MicrosoftBuild = @{
    VsWhereExe = Join-Path $devRoot 'shells\vs-installer\3.1.7\vswhere.exe'
    VisualStudioRoot = Join-Path $devRoot 'shells\visual-studio\2022\BuildTools'
    WindowsSdkRoot = Join-Path $devRoot 'shells\windows-kits\10'
    DotNetFrameworkRoot = Join-Path $devRoot 'shells\dotnet-framework\Framework\v4.0.30319'
    DotNetFramework64Root = Join-Path $devRoot 'shells\dotnet-framework\Framework64\v4.0.30319'
  }
  Nx = @{
    DaemonBootstrapMode = 'ResetAndStart'
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

## Current wrapper split

The current project no longer keeps one editor-specific bootstrap body as the only source of truth.

It now uses:

- one shared project editor core
- thin VS Code and Cursor wrappers
- shared dependency and repair scripts that accept `-Editor`

The current wrapper inventory is:

- `Start-TestMonoEditor.ps1`
- `Start-TestMonoVSCode.ps1`
- `Start-TestMonoCursor.ps1`
- `Start-TestMonoTerminal.ps1`
- `Start-TestMonoElectronTerminal.ps1`
- `Start-TestMonoCursorTerminal.ps1`
- `Start-TestMonoCursorElectronTerminal.ps1`

Representative thin wrappers:

```powershell
# Start-TestMonoVSCode.ps1
param(
  [ValidateSet('LaunchVSCode', 'OpenTerminal')]
  [string]$Action = 'LaunchVSCode',
  [string]$RepoPath,
  [ValidateSet('General', 'ElectronServe')]
  [string]$OpenTerminalIntent = 'General',
  [switch]$EnableComSpecTraceProxy
)

$editorAction = switch ($Action) {
  'LaunchVSCode' { 'LaunchEditor' }
  'OpenTerminal' { 'OpenTerminal' }
}

& (Join-Path $PSScriptRoot 'Start-TestMonoEditor.ps1') `
  -Editor VSCode `
  -Action $editorAction `
  -RepoPath $RepoPath `
  -OpenTerminalIntent $OpenTerminalIntent `
  -EnableComSpecTraceProxy:$EnableComSpecTraceProxy
```

```powershell
# Start-TestMonoCursor.ps1
param(
  [ValidateSet('LaunchCursor', 'OpenTerminal')]
  [string]$Action = 'LaunchCursor',
  [string]$RepoPath,
  [ValidateSet('General', 'ElectronServe')]
  [string]$OpenTerminalIntent = 'General',
  [switch]$EnableComSpecTraceProxy
)

$editorAction = switch ($Action) {
  'LaunchCursor' { 'LaunchEditor' }
  'OpenTerminal' { 'OpenTerminal' }
}

& (Join-Path $PSScriptRoot 'Start-TestMonoEditor.ps1') `
  -Editor Cursor `
  -Action $editorAction `
  -RepoPath $RepoPath `
  -OpenTerminalIntent $OpenTerminalIntent `
  -EnableComSpecTraceProxy:$EnableComSpecTraceProxy
```

```powershell
# Start-TestMonoCursorTerminal.ps1
param([string]$RepoPath)
& (Join-Path $PSScriptRoot 'Start-TestMonoTerminal.ps1') -Editor Cursor -RepoPath $RepoPath
```

```powershell
# Start-TestMonoCursorElectronTerminal.ps1
param(
  [string]$RepoPath,
  [switch]$EnableComSpecTraceProxy
)

& (Join-Path $PSScriptRoot 'Start-TestMonoElectronTerminal.ps1') `
  -Editor Cursor `
  -RepoPath $RepoPath `
  -EnableComSpecTraceProxy:$EnableComSpecTraceProxy
```

## `Start-TestMonoVSCode.ps1`

This is the full sanitized project-adapter example for the current Electron-serve-aware bootstrap contract.

In the current multi-editor architecture, the same inner pattern is owned by `Start-TestMonoEditor.ps1`, while `Start-TestMonoVSCode.ps1` and `Start-TestMonoCursor.ps1` stay thin wrappers over that shared project core.

```powershell
param(
  [ValidateSet('LaunchVSCode', 'OpenTerminal')]
  [string]$Action = 'LaunchVSCode',

  [string]$RepoPath,

  [ValidateSet('General', 'ElectronServe')]
  [string]$OpenTerminalIntent = 'General',

  [switch]$EnableComSpecTraceProxy
)

$ErrorActionPreference = 'Stop'
Set-StrictMode -Version Latest

function Write-AsciiText {
  param(
    [Parameter(Mandatory = $true)]
    [string]$Path,

    [Parameter(Mandatory = $true)]
    [string]$Content
  )

  $parent = Split-Path -Parent $Path
  if (-not [string]::IsNullOrWhiteSpace($parent) -and -not (Test-Path -LiteralPath $parent)) {
    New-Item -ItemType Directory -Force -Path $parent | Out-Null
  }

  [System.IO.File]::WriteAllText($Path, $Content, [System.Text.Encoding]::ASCII)
}

function Append-AsciiText {
  param(
    [Parameter(Mandatory = $true)]
    [string]$Path,

    [Parameter(Mandatory = $true)]
    [string]$Content
  )

  $parent = Split-Path -Parent $Path
  if (-not [string]::IsNullOrWhiteSpace($parent) -and -not (Test-Path -LiteralPath $parent)) {
    New-Item -ItemType Directory -Force -Path $parent | Out-Null
  }

  [System.IO.File]::AppendAllText($Path, $Content, [System.Text.Encoding]::ASCII)
}

function Get-TestMonoLogRoot {
  $localAppData = [System.Environment]::GetFolderPath([System.Environment+SpecialFolder]::LocalApplicationData)

  if ([string]::IsNullOrWhiteSpace($localAppData)) {
    $localAppData = $env:TEMP
  }

  if ([string]::IsNullOrWhiteSpace($localAppData)) {
    throw 'Could not resolve a writable local log root.'
  }

  return (Join-Path $localAppData 'SandboxToolchains\Logs\test-mono')
}

function Write-TestMonoBootstrapLog {
  param(
    [Parameter(Mandatory = $true)]
    [string]$Message
  )

  $timestamp = [DateTime]::UtcNow.ToString('o')
  $logPath = Join-Path (Get-TestMonoLogRoot) 'electron-vite-bootstrap.log'
  Append-AsciiText -Path $logPath -Content ('[{0}] {1}{2}' -f $timestamp, $Message, [Environment]::NewLine)
}

function Enable-TestMonoComSpecTraceProxy {
  $proxyComSpec = $env:BOXED_CMD_PROXY_EXE
  if ([string]::IsNullOrWhiteSpace($proxyComSpec) -or -not (Test-Path -LiteralPath $proxyComSpec)) {
    Write-TestMonoBootstrapLog -Message 'Shared ComSpec diagnostic proxy is not available in the local boxed shell runtime. Continuing without proxy instrumentation.'
    Write-Warning 'Shared ComSpec diagnostic proxy is not available in the local boxed shell runtime. Continuing without proxy instrumentation.'
    return $false
  }

  $realComSpec = $env:BOXED_CMD_EXE
  if ([string]::IsNullOrWhiteSpace($realComSpec) -or -not (Test-Path -LiteralPath $realComSpec)) {
    throw 'BOXED_CMD_EXE was not initialized to a runnable boxed cmd.exe before enabling the shared ComSpec trace proxy.'
  }

  $comSpecTraceLog = Join-Path (Get-TestMonoLogRoot) 'comspec-wrapper.log'
  $env:BOXED_REAL_COMSPEC = $realComSpec
  $env:BOXED_COMSPEC_TRACE_LOG = $comSpecTraceLog
  $env:BOXED_COMSPEC_DIAGNOSTIC_PROXY = $proxyComSpec
  $env:ComSpec = $proxyComSpec
  $env:COMSPEC = $proxyComSpec

  Write-TestMonoBootstrapLog -Message ('Enabled shared ComSpec diagnostic proxy. Proxy="{0}" RealCmd="{1}" TraceLog="{2}"' -f $proxyComSpec, $realComSpec, $comSpecTraceLog)
  Write-Host "ComSpecDiagnosticProxy: $proxyComSpec"
  Write-Host "ComSpecDiagnosticRealCmd: $realComSpec"
  Write-Host "ComSpecDiagnosticLog: $comSpecTraceLog"
  return $true
}

function Publish-TestMonoComSpecMetadata {
  $realComSpec = $env:BOXED_CMD_EXE
  if (-not [string]::IsNullOrWhiteSpace($realComSpec) -and (Test-Path -LiteralPath $realComSpec)) {
    $env:BOXED_REAL_COMSPEC = $realComSpec
  }

  $proxyComSpec = $env:BOXED_CMD_PROXY_EXE
  if (-not [string]::IsNullOrWhiteSpace($proxyComSpec) -and (Test-Path -LiteralPath $proxyComSpec)) {
    $env:BOXED_COMSPEC_DIAGNOSTIC_PROXY = $proxyComSpec
  }
}

function Resolve-TestMonoElectronExecPath {
  param(
    [Parameter(Mandatory = $true)]
    [string]$RepoPath
  )

  $desktopRoot = Join-Path $RepoPath 'apps\desktop-app'
  $desktopPackageJson = Join-Path $desktopRoot 'package.json'

  if (-not (Test-Path -LiteralPath $desktopPackageJson)) {
    Write-TestMonoBootstrapLog -Message ('Electron exec path resolution skipped because desktop package.json is missing: "{0}"' -f $desktopPackageJson)
    return $null
  }

  $candidateElectronRoots = New-Object System.Collections.Generic.List[string]
  $desktopElectronRoot = Join-Path $desktopRoot 'node_modules\electron'
  if (Test-Path -LiteralPath $desktopElectronRoot) {
    $candidateElectronRoots.Add($desktopElectronRoot)
  }

  $pnpmRoot = Join-Path $RepoPath '.pnpm'
  if (Test-Path -LiteralPath $pnpmRoot) {
    $pnpmElectronDirs = Get-ChildItem -LiteralPath $pnpmRoot -Directory -Filter 'electron@*' -ErrorAction SilentlyContinue |
      Sort-Object Name -Descending

    foreach ($pnpmElectronDir in $pnpmElectronDirs) {
      $candidateElectronRoot = Join-Path $pnpmElectronDir.FullName 'node_modules\electron'
      if (Test-Path -LiteralPath $candidateElectronRoot) {
        $candidateElectronRoots.Add($candidateElectronRoot)
      }
    }
  }

  foreach ($candidateElectronRoot in ($candidateElectronRoots | Select-Object -Unique)) {
    $pathTxt = Join-Path $candidateElectronRoot 'path.txt'
    $distDir = Join-Path $candidateElectronRoot 'dist'
    $defaultExe = Join-Path $distDir 'electron.exe'

    if (Test-Path -LiteralPath $pathTxt) {
      $relativeBinary = [System.IO.File]::ReadAllText($pathTxt).Trim()
      if (-not [string]::IsNullOrWhiteSpace($relativeBinary)) {
        $binaryFromPathTxt = Join-Path $distDir $relativeBinary
        if (Test-Path -LiteralPath $binaryFromPathTxt) {
          return [pscustomobject]@{
            Path = $binaryFromPathTxt
            Source = 'path.txt'
          }
        }
      }
    }

    if (Test-Path -LiteralPath $defaultExe) {
      return [pscustomobject]@{
        Path = $defaultExe
        Source = 'dist-electron.exe'
      }
    }
  }

  Write-TestMonoBootstrapLog -Message ('Electron exec path resolution could not find a runnable electron.exe below desktop node_modules or .pnpm. DesktopRoot="{0}"' -f $desktopRoot)
  return $null
}

function Set-TestMonoElectronExecPath {
  param(
    [Parameter(Mandatory = $true)]
    [string]$RepoPath
  )

  $resolution = Resolve-TestMonoElectronExecPath -RepoPath $RepoPath
  if (-not $resolution -or [string]::IsNullOrWhiteSpace($resolution.Path)) {
    $warningMessage = 'Desktop-app Electron runtime path could not be resolved during bootstrap. Electron launch may fail until the runtime path is repaired.'
    Write-TestMonoBootstrapLog -Message $warningMessage
    Write-Warning $warningMessage
    return $false
  }

  $env:ELECTRON_EXEC_PATH = $resolution.Path
  $env:BOXED_ELECTRON_EXEC_PATH = $resolution.Path

  Write-TestMonoBootstrapLog -Message ('Set ELECTRON_EXEC_PATH to "{0}" via {1}' -f $resolution.Path, $resolution.Source)
  return $true
}

function Ensure-TestMonoElectronViteCmdShim {
  param(
    [Parameter(Mandatory = $true)]
    [string]$RepoPath,

    [Parameter(Mandatory = $true)]
    [string]$FallbackNodeRoot
  )

  $electronViteCmd = Join-Path $RepoPath 'apps\desktop-app\node_modules\.bin\electron-vite.CMD'
  $electronViteJs = Join-Path $RepoPath 'apps\desktop-app\node_modules\electron-vite\bin\electron-vite.js'

  if (-not (Test-Path -LiteralPath $electronViteCmd) -or -not (Test-Path -LiteralPath $electronViteJs)) {
    Write-TestMonoBootstrapLog -Message ('Electron-Vite shim skipped because required files are missing. CmdPath="{0}" JsPath="{1}"' -f $electronViteCmd, $electronViteJs)
    return $false
  }

  $fallbackNodeExe = Join-Path $FallbackNodeRoot 'node.exe'
  $shimContent = @(
    '@echo off'
    'setlocal EnableExtensions'
    'set "_electronViteJs=%~dp0..\electron-vite\bin\electron-vite.js"'
    'set "_shimLogDir=%LOCALAPPDATA%\SandboxToolchains\Logs\test-mono"'
    'if not exist "%_shimLogDir%" mkdir "%_shimLogDir%" >nul 2>nul'
    'set "_shimLog=%_shimLogDir%\electron-vite-shim.log"'
    '>>"%_shimLog%" echo [%DATE% %TIME%] start cwd=%CD% boxed_node_root=%BOXED_NODE_ROOT% args=%*'
    'if defined BOXED_NODE_ROOT if exist "%BOXED_NODE_ROOT%\node.exe" goto :boxedNodeRoot'
    ('if exist "{0}" goto :fallbackNode' -f $fallbackNodeExe)
    'goto :pathNode'
    ':boxedNodeRoot'
    '>>"%_shimLog%" echo [%DATE% %TIME%] using BOXED_NODE_ROOT "%BOXED_NODE_ROOT%\node.exe"'
    '"%BOXED_NODE_ROOT%\node.exe" "%_electronViteJs%" %*'
    'set "_shimExit=%ERRORLEVEL%"'
    '>>"%_shimLog%" echo [%DATE% %TIME%] exit_code=%_shimExit%'
    'exit /b %_shimExit%'
    ':fallbackNode'
    ('>>"%_shimLog%" echo [%DATE% %TIME%] using fallback "{0}"' -f $fallbackNodeExe)
    ('"{0}" "%_electronViteJs%" %*' -f $fallbackNodeExe)
    'set "_shimExit=%ERRORLEVEL%"'
    '>>"%_shimLog%" echo [%DATE% %TIME%] exit_code=%_shimExit%'
    'exit /b %_shimExit%'
    ':pathNode'
    '>>"%_shimLog%" echo [%DATE% %TIME%] using PATH node'
    'node "%_electronViteJs%" %*'
    'set "_shimExit=%ERRORLEVEL%"'
    '>>"%_shimLog%" echo [%DATE% %TIME%] exit_code=%_shimExit%'
    'exit /b %_shimExit%'
  ) -join [Environment]::NewLine

  $currentContent = if (Test-Path -LiteralPath $electronViteCmd) {
    [System.IO.File]::ReadAllText($electronViteCmd)
  }
  else {
    ''
  }

  if ($currentContent -ne $shimContent) {
    Write-AsciiText -Path $electronViteCmd -Content $shimContent
    Write-TestMonoBootstrapLog -Message ('Electron-Vite shim refreshed at "{0}" with fallback node "{1}"' -f $electronViteCmd, $fallbackNodeExe)
  }
  else {
    Write-TestMonoBootstrapLog -Message ('Electron-Vite shim already up to date at "{0}"' -f $electronViteCmd)
  }

  return $true
}

function Test-TestMonoDesktopOutSurface {
  param(
    [Parameter(Mandatory = $true)]
    [string]$RepoPath
  )

  $desktopRoot = Join-Path $RepoPath 'apps\desktop-app'
  $outRoot = Join-Path $desktopRoot 'out'
  $chunksRoot = Join-Path $outRoot 'main\chunks'

  if (-not (Test-Path -LiteralPath $desktopRoot)) {
    Write-TestMonoBootstrapLog -Message ('Electron out preflight skipped because desktop root is missing: "{0}"' -f $desktopRoot)
    return $false
  }

  Write-TestMonoBootstrapLog -Message ('Electron out preflight started. OutRoot="{0}" ChunksRoot="{1}"' -f $outRoot, $chunksRoot)

  try {
    if (Test-Path -LiteralPath $outRoot) {
      Remove-Item -LiteralPath $outRoot -Recurse -Force -ErrorAction Stop
      Write-TestMonoBootstrapLog -Message ('Removed existing Electron out root: "{0}"' -f $outRoot)
    }
    else {
      Write-TestMonoBootstrapLog -Message ('Electron out root was already absent before preflight: "{0}"' -f $outRoot)
    }

    $null = New-Item -ItemType Directory -Force -Path $chunksRoot -ErrorAction Stop
    Write-TestMonoBootstrapLog -Message ('Created Electron chunks root during preflight: "{0}"' -f $chunksRoot)

    Remove-Item -LiteralPath $outRoot -Recurse -Force -ErrorAction Stop
    Write-TestMonoBootstrapLog -Message ('Removed Electron out root after create test: "{0}"' -f $outRoot)
  }
  catch {
    $warningMessage = 'Electron desktop out preflight failed for "{0}". See "{1}" for details. Continuing terminal startup.' -f $chunksRoot, (Join-Path (Get-TestMonoLogRoot) 'electron-vite-bootstrap.log')
    Write-TestMonoBootstrapLog -Message ('Electron out preflight failed. Message="{0}" Type="{1}"' -f $_.Exception.Message, $_.Exception.GetType().FullName)
    Write-TestMonoBootstrapLog -Message $warningMessage
    Write-Warning $warningMessage
    return $false
  }

  return $true
}

$config = & (Join-Path $PSScriptRoot 'Project.Config.ps1')

if (-not $config) {
  throw 'Project config did not return a configuration object.'
}

$resolvedRepoPath = if ([string]::IsNullOrWhiteSpace($RepoPath)) {
  $config.DefaultRepoPath
}
else {
  $RepoPath
}

$logRoot = Get-TestMonoLogRoot
Write-TestMonoBootstrapLog -Message ('Bootstrap start. Action="{0}" RepoPath="{1}" LogRoot="{2}"' -f $Action, $resolvedRepoPath, $logRoot)
$isElectronServeTerminal = $Action -eq 'OpenTerminal' -and $OpenTerminalIntent -eq 'ElectronServe'
$effectiveNxDaemonBootstrapMode = if ($isElectronServeTerminal) {
  $config.Nx.DaemonBootstrapMode
}
else {
  'None'
}

Write-TestMonoBootstrapLog -Message ('OpenTerminalIntent="{0}" EffectiveNxDaemonBootstrapMode="{1}"' -f $OpenTerminalIntent, $effectiveNxDaemonBootstrapMode)

if ($isElectronServeTerminal) {
  $null = Set-TestMonoElectronExecPath -RepoPath $resolvedRepoPath

  $electronViteShimReady = Ensure-TestMonoElectronViteCmdShim `
    -RepoPath $resolvedRepoPath `
    -FallbackNodeRoot $config.Toolchain.NodeRoot

  if ($electronViteShimReady) {
    $electronOutPreflightReady = Test-TestMonoDesktopOutSurface -RepoPath $resolvedRepoPath
    Write-TestMonoBootstrapLog -Message ('Electron out preflight completed={0}' -f $electronOutPreflightReady)
  }
}
else {
  Write-TestMonoBootstrapLog -Message ('Skipping test-mono Electron shell-surface bootstrap for Action="{0}" OpenTerminalIntent="{1}"' -f $Action, $OpenTerminalIntent)
}

$baseScript = Join-Path $config.SharedRoot 'dev\bootstrap\platforms\vscode\Start-VSCodeProjectBase.ps1'

if (-not (Test-Path -LiteralPath $baseScript)) {
  throw "Base project launcher not found: $baseScript"
}

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
  NxDaemonBootstrapMode = $effectiveNxDaemonBootstrapMode
  CmdRoot = $config.Shells.CmdRoot
  PowerShellRoot = $config.Shells.PowerShellRoot
  RegRoot = $config.Shells.RegRoot
  StarshipRoot = $config.Shells.StarshipRoot
  ClinkRoot = $config.Shells.ClinkRoot
  StarshipConfigPath = $config.Shells.StarshipConfigPath
  AdditionalNodeCommands = $config.Toolchain.AdditionalNodeCommands
}

Write-TestMonoBootstrapLog -Message ('Delegating to base project bootstrap: "{0}"' -f $baseScript)
& $baseScript @parameters
Write-TestMonoBootstrapLog -Message ('Base project bootstrap returned. Action="{0}" LastExitCode="{1}"' -f $Action, $LASTEXITCODE)

if ($Action -eq 'OpenTerminal') {
  Publish-TestMonoComSpecMetadata

  if ($EnableComSpecTraceProxy) {
    try {
      $null = Enable-TestMonoComSpecTraceProxy
    }
    catch {
      Write-TestMonoBootstrapLog -Message ('Failed to enable shared ComSpec diagnostic proxy. Message="{0}" Type="{1}"' -f $_.Exception.Message, $_.Exception.GetType().FullName)
      Write-Warning ('Failed to enable shared ComSpec diagnostic proxy. Continuing without it. Message: {0}' -f $_.Exception.Message)
    }
  }
  else {
    Write-TestMonoBootstrapLog -Message 'Shared ComSpec diagnostic proxy not enabled for this terminal.'
  }

  return
}

exit $LASTEXITCODE
```

## `Start-TestMonoElectronTerminal.ps1`

```powershell
param(
  [string]$RepoPath,
  [ValidateSet('VSCode', 'Cursor')]
  [string]$Editor = 'VSCode',
  [switch]$EnableComSpecTraceProxy
)

$ErrorActionPreference = 'Stop'
Set-StrictMode -Version Latest

$launcher = Join-Path $PSScriptRoot 'Start-TestMonoEditor.ps1'

if (-not (Test-Path -LiteralPath $launcher)) {
  throw "Project launcher not found: $launcher"
}

& $launcher `
  -Editor $Editor `
  -Action OpenTerminal `
  -RepoPath $RepoPath `
  -OpenTerminalIntent ElectronServe `
  -EnableComSpecTraceProxy:$EnableComSpecTraceProxy
```

## `Start-TestMonoPnpmInstall.ps1`

This is the sanitized project-box install script.

The PNPM-domain source of truth for the install-script contract now lives here:

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\install.md`

The PNPM-domain source of truth for the clean-reinstall script contract now lives here:

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\clean-reinstall.md`

The complete node-gyp wrapper architecture and full wrapper code example now live here:

- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\msbuild-file-tracking-wrapper.md`

It encodes the governance-approved boxed-owned-toolchain contract:

- the host launches one explicit project-owned PS1
- the script enters the project box through the normal project bootstrap
- the script does **not** choose the PNPM version through host arguments
- the effective PNPM version comes from `Project.Config.ps1`

The current shell-specific requirement is also part of this contract:

- boxed Git Bash is the preferred productive lifecycle `scriptShell`
- boxed PowerShell remains the preferred interactive default shell
- boxed `cmd.exe` remains available as a helper lane for native-build preparation and cleanup fallback
- project bootstrap enables Nx daemon preflight through:
  - `DaemonBootstrapMode = 'ResetAndStart'`
- Git Bash still needs a shell-native `pnpm` wrapper in `bootstrap-bin`, not only `pnpm.cmd`
- Electron post-install verification / repair belongs in its own Electron-domain script and is only called from the PNPM scripts
- native-build preparation projects the governed shared Microsoft build-source trees into their canonical Windows runtime paths before `pnpm install` runs
- the boxed `node-gyp` wrapper is published by bootstrap so Windows build-tracking behavior is adapted without editing downloaded dependency source

```powershell
param(
  [string]$RepoPath = 'C:\Users\yourusername\source\test-mono',

  [ValidateSet('VSCode', 'Cursor')]
  [string]$Editor = 'VSCode'
)

$ErrorActionPreference = 'Stop'
Set-StrictMode -Version Latest

$launcher = Join-Path $PSScriptRoot 'Start-TestMonoEditor.ps1'

if (-not (Test-Path -LiteralPath $launcher)) {
  throw "Project launcher not found: $launcher"
}

& $launcher -Editor $Editor -Action OpenTerminal -RepoPath $RepoPath

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

$scriptExitCode = $LASTEXITCODE
if ($scriptExitCode -ne 0) {
  exit $scriptExitCode
}

$electronPostInstallScript = Join-Path $PSScriptRoot 'Start-TestMonoElectronPostInstall.ps1'
if (-not (Test-Path -LiteralPath $electronPostInstallScript)) {
  throw "Electron post-install script not found: $electronPostInstallScript"
}

& $electronPostInstallScript -Editor $Editor -RepoPath $RepoPath -SkipBootstrap

exit $LASTEXITCODE
```

## `Start-TestMonoElectronPostInstall.ps1`

The full Electron-domain source of truth for the project-owned post-install script now lives here:

- `docs\applications\programming-languages\node\dependencies\frameworks\electron\architectures\boxed-owned-toolchain\scripts\post-install.md`

## `Start-TestMonoPnpmUninstall.ps1`

This is the sanitized project-box uninstall helper used by the clean-reinstall flow.

```powershell
param(
  [string]$RepoPath,
  [ValidateSet('VSCode', 'Cursor')]
  [string]$Editor = 'VSCode',
  [switch]$SkipBootstrap
)

$ErrorActionPreference = 'Stop'
Set-StrictMode -Version Latest

$config = & (Join-Path $PSScriptRoot 'Project.Config.ps1')
$resolvedRepoPath = if ([string]::IsNullOrWhiteSpace($RepoPath)) {
  $config.DefaultRepoPath
}
else {
  $RepoPath
}

if (-not $SkipBootstrap) {
  $launcher = Join-Path $PSScriptRoot 'Start-TestMonoEditor.ps1'
  & $launcher -Editor $Editor -Action OpenTerminal -RepoPath $resolvedRepoPath
}

function Remove-BoxedPathTree {
  param([string]$Path)

  if (-not (Test-Path -LiteralPath $Path)) {
    return
  }

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

$dependencyPaths = New-Object 'System.Collections.Generic.List[string]'

@(
  (Join-Path $resolvedRepoPath '.pnpm'),
  (Join-Path $resolvedRepoPath 'node_modules')
) | ForEach-Object {
  [void]$dependencyPaths.Add($_)
}

foreach ($segment in 'apps', 'libs', 'tools') {
  $segmentRoot = Join-Path $resolvedRepoPath $segment
  if (-not (Test-Path -LiteralPath $segmentRoot)) {
    continue
  }

  Get-ChildItem -LiteralPath $segmentRoot -Directory -ErrorAction SilentlyContinue |
    ForEach-Object {
      [void]$dependencyPaths.Add((Join-Path $_.FullName 'node_modules'))
    }
}

$dependencyPaths |
  Sort-Object -Unique |
  Where-Object { Test-Path -LiteralPath $_ } |
  ForEach-Object {
    Write-Host "Removing $_"
    Remove-BoxedPathTree -Path $_
  }

$modulesYaml = Join-Path $resolvedRepoPath 'node_modules\.modules.yaml'
if (Test-Path -LiteralPath $modulesYaml) {
  Write-Host "Removing $modulesYaml"
  Remove-Item -LiteralPath $modulesYaml -Force -ErrorAction SilentlyContinue
}

@(
  $env:BOXED_PUPPETEER_CACHE_DIR,
  $env:BOXED_PLAYWRIGHT_BROWSERS_PATH
) |
Where-Object { -not [string]::IsNullOrWhiteSpace($_) } |
Sort-Object -Unique |
Where-Object { Test-Path -LiteralPath $_ } |
ForEach-Object {
  Write-Host "Removing $_"
  Remove-BoxedPathTree -Path $_
}
```

## `Start-TestMonoPnpmCleanReinstall.ps1`

This is the sanitized project-box clean-reinstall script.

```powershell
param(
  [string]$RepoPath,

  [ValidateSet('VSCode', 'Cursor')]
  [string]$Editor = 'VSCode'
)

$ErrorActionPreference = 'Stop'
Set-StrictMode -Version Latest

$configScript = Join-Path $PSScriptRoot 'Project.Config.ps1'
if (-not (Test-Path -LiteralPath $configScript)) {
  throw "Project config not found: $configScript"
}

$config = & $configScript
if (-not $config) {
  throw 'Project config did not return a configuration object.'
}

$resolvedRepoPath = if ([string]::IsNullOrWhiteSpace($RepoPath)) {
  $config.DefaultRepoPath
}
else {
  $RepoPath
}

$launcher = Join-Path $PSScriptRoot 'Start-TestMonoEditor.ps1'
if (-not (Test-Path -LiteralPath $launcher)) {
  throw "Project launcher not found: $launcher"
}

& $launcher -Editor $Editor -Action OpenTerminal -RepoPath $resolvedRepoPath

$uninstallScript = Join-Path $PSScriptRoot 'Start-TestMonoPnpmUninstall.ps1'
if (-not (Test-Path -LiteralPath $uninstallScript)) {
  throw "Project uninstall script not found: $uninstallScript"
}

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

Set-Location $resolvedRepoPath

Write-Host "RepoPath: $resolvedRepoPath"
Write-Host "LifecycleShell: $bashExe"
Write-Host "VsRoot: $($nativeBuildRuntime.VSRoot)"
Write-Host "VsDevCmd: $($nativeBuildRuntime.VsDevCmd)"
Write-Host "WindowsSdkRoot: $($nativeBuildRuntime.WindowsSdkRoot)"
Write-Host "WindowsSdkVersion: $($nativeBuildRuntime.WindowsSdkVersion)"
Write-Host 'Configuring PNPM lifecycle shell for boxed Git Bash...'
pnpm config set --location=project scriptShell "$bashExe"

Write-Host 'Running clean pnpm install...'
pnpm install

$scriptExitCode = $LASTEXITCODE
if ($scriptExitCode -ne 0) {
  exit $scriptExitCode
}

$electronPostInstallScript = Join-Path $PSScriptRoot 'Start-TestMonoElectronPostInstall.ps1'
if (-not (Test-Path -LiteralPath $electronPostInstallScript)) {
  throw "Electron post-install script not found: $electronPostInstallScript"
}

& $electronPostInstallScript -Editor $Editor -RepoPath $resolvedRepoPath -SkipBootstrap

exit $LASTEXITCODE
```

### Host command after materializing the clean-reinstall boilerplate above into the shared project bootstrap subtree

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:VS_CODE_TEST_MONO `
  "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
  -NoLogo `
  -NoExit `
  -ExecutionPolicy Bypass `
  -File "C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoPnpmCleanReinstall.ps1" `
  -RepoPath "C:\Users\yourusername\source\test-mono"
```

Cursor variant:

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:CURSOR_TEST_MONO `
  "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
  -NoLogo `
  -NoExit `
  -ExecutionPolicy Bypass `
  -File "C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoPnpmCleanReinstall.ps1" `
  -Editor Cursor `
  -RepoPath "C:\Users\yourusername\source\test-mono"
```

### Host command after materializing the boilerplate above into the shared project bootstrap subtree

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:VS_CODE_TEST_MONO `
  "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
  -NoLogo `
  -NoExit `
  -ExecutionPolicy Bypass `
  -File "C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoPnpmInstall.ps1" `
  -RepoPath "C:\Users\yourusername\source\test-mono"
```

Cursor variant:

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:CURSOR_TEST_MONO `
  "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
  -NoLogo `
  -NoExit `
  -ExecutionPolicy Bypass `
  -File "C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoPnpmInstall.ps1" `
  -Editor Cursor `
  -RepoPath "C:\Users\yourusername\source\test-mono"
```

After you materialize the boilerplate script above into the sanitized shared project subtree, this host command executes it.

The example assumes the sanitized project keeps its repo path visible inside the project box and wants to run a project-owned dependency refresh with the currently validated boxed PNPM lifecycle-shell setup.

The version-provisioning SSOT for moving the project contract to a newer PNPM version lives here:

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\versioning-and-provisioning.md`

## Notes

This boilerplate intentionally keeps `node20` as an additional command because it reflects the validated monorepo example currently used for the architecture.

The optional `PythonRoot`, `StarshipRoot`, `ClinkRoot`, and `StarshipConfigPath` entries reflect the current boxed-owned-toolchain runtime shape where the project bootstrap can initialize:

- Python
- Starship
- `reg.exe` as an explicit boxed helper lane
- the CMD-specific `Clink` adapter
- explicit local PowerShell/CMD shell lanes
- the bootstrap-owned `node-gyp` wrapper surface

in addition to the core Git/Node/pnpm surfaces.

If a project does not need a secondary runtime, remove the `AdditionalNodeCommands` entry entirely.

## Related

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\start.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\scripts.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\msbuild-file-tracking-wrapper.md`
