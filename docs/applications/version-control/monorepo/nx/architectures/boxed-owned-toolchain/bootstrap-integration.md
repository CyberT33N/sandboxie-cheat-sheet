# Nx Bootstrap Integration

## Scope

This document owns the Nx-specific bootstrap integration for the boxed-owned-toolchain architecture.

It records:

- the exact live shared files that implement the current behavior
- the exact current code that generates the Nx command surface
- the exact project bootstrap environment that Nx inherits
- how the command surface is called and validated

## Current live implementation files

The current live shared implementation surfaces are:

- `C:\shared\sandbox-toolchains\dev\bootstrap\stacks\node\Bootstrap.Node.psm1`
- `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode\Start-VSCodeProjectBase.ps1`
- `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode\Start-VSCodeMaintenance.ps1`

The current live project adapter chain used in the active project context includes:

- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoVSCode.ps1`

The sanitized documentation-safe boilerplate equivalent remains:

- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoVSCode.ps1`

## Current call chain

The live project adapter delegates into the generic base launcher like this:

```powershell
$config = & (Join-Path $PSScriptRoot 'Project.Config.ps1')
$baseScript = Join-Path $config.SharedRoot 'dev\bootstrap\platforms\vscode\Start-VSCodeProjectBase.ps1'

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
  CmdRoot = $config.Shells.CmdRoot
  PowerShellRoot = $config.Shells.PowerShellRoot
  StarshipRoot = $config.Shells.StarshipRoot
  ClinkRoot = $config.Shells.ClinkRoot
  StarshipConfigPath = $config.Shells.StarshipConfigPath
  AdditionalNodeCommands = $config.Toolchain.AdditionalNodeCommands
}

& $baseScript @parameters
```

That means the Nx command surface is **not** hardcoded in the project adapter.

It is published by the generic shared Node bootstrap layer.

## Exact current Nx wrapper implementation

The current live Nx wrapper code is generated in:

- `C:\shared\sandbox-toolchains\dev\bootstrap\stacks\node\Bootstrap.Node.psm1`

The relevant current code is:

```powershell
$primaryNodeExe = Join-Path $NodeRoot 'node.exe'
$nxLauncherPath = Join-Path $BootstrapBin 'nx-cli.cjs'

$nxLauncherContent = @'
'use strict';

const { existsSync } = require('node:fs');
const { spawnSync } = require('node:child_process');
const { createRequire } = require('node:module');
const path = require('node:path');

function findWorkspaceAnchor(startDirectory) {
  let currentDirectory = startDirectory;

  while (true) {
    const packageJsonPath = path.join(currentDirectory, 'package.json');
    const pnpmWorkspacePath = path.join(currentDirectory, 'pnpm-workspace.yaml');
    const nxJsonPath = path.join(currentDirectory, 'nx.json');

    if (existsSync(packageJsonPath) || existsSync(pnpmWorkspacePath) || existsSync(nxJsonPath)) {
      return currentDirectory;
    }

    const parentDirectory = path.dirname(currentDirectory);
    if (parentDirectory === currentDirectory) {
      return startDirectory;
    }

    currentDirectory = parentDirectory;
  }
}

const workspaceAnchor = findWorkspaceAnchor(process.cwd());
const requireFromWorkspace = createRequire(path.join(workspaceAnchor, '__boxed_nx_resolver__.cjs'));

let nxCliPath;

try {
  nxCliPath = requireFromWorkspace.resolve('nx/bin/nx.js');
} catch (firstError) {
  try {
    nxCliPath = requireFromWorkspace.resolve('nx/dist/bin/nx.js');
  } catch (secondError) {
    console.error('[boxed nx] Could not resolve the local Nx CLI from the current workspace.');
    console.error(`[boxed nx] cwd=${process.cwd()}`);
    console.error(`[boxed nx] workspaceAnchor=${workspaceAnchor}`);
    console.error('[boxed nx] Checked: nx/bin/nx.js and nx/dist/bin/nx.js');
    process.exit(1);
  }
}

const result = spawnSync(process.execPath, [nxCliPath, ...process.argv.slice(2)], {
  stdio: 'inherit',
  env: process.env,
});

if (typeof result.status === 'number') {
  process.exit(result.status);
}

if (result.error) {
  console.error('[boxed nx] Failed to start Nx.', result.error);
  process.exit(1);
}

process.exit(1);
'@

Write-AsciiFile -Path $nxLauncherPath -Content $nxLauncherContent

$nxCmdContent = @(
  '@echo off'
  ('set "NODE_EXE={0}"' -f $primaryNodeExe)
  ('set "NX_LAUNCHER={0}"' -f $nxLauncherPath)
  '"%NODE_EXE%" "%NX_LAUNCHER%" %*'
) -join [Environment]::NewLine

Write-AsciiFile -Path (Join-Path $BootstrapBin 'nx.cmd') -Content $nxCmdContent

$nxPs1Content = @(
  ('$nodeExe = ''{0}''' -f $primaryNodeExe)
  ('$nxLauncher = ''{0}''' -f $nxLauncherPath)
  '& $nodeExe $nxLauncher @args'
  'exit $LASTEXITCODE'
) -join [Environment]::NewLine

Write-AsciiFile -Path (Join-Path $BootstrapBin 'nx.ps1') -Content $nxPs1Content

$nxShellContent = (@(
  '#!/bin/sh'
  ('NODE_EXE_WIN=''{0}''' -f $primaryNodeExe)
  ('NX_LAUNCHER_WIN=''{0}''' -f $nxLauncherPath)
  ''
  'if command -v cygpath >/dev/null 2>&1; then'
  '  NODE_EXE="$(cygpath -u "$NODE_EXE_WIN")"'
  '  NX_LAUNCHER="$(cygpath -u "$NX_LAUNCHER_WIN")"'
  'else'
  '  NODE_EXE="$NODE_EXE_WIN"'
  '  NX_LAUNCHER="$NX_LAUNCHER_WIN"'
  'fi'
  ''
  'exec "$NODE_EXE" "$NX_LAUNCHER" "$@"'
) -join "`n") + "`n"

Write-AsciiFile -Path (Join-Path $BootstrapBin 'nx') -Content $nxShellContent

$env:BOXED_NX_LAUNCHER = $nxLauncherPath
```

## Why this implementation is correct

This implementation is correct for the current architecture because it:

1. resolves the local workspace Nx CLI dynamically instead of pinning one hardcoded path
2. keeps the plain `nx` command surface available in PowerShell, CMD, and Git Bash
3. keeps Nx hosted by the already-governed local mirrored Node runtime
4. avoids dependence on a global host Nx install

The current wrapper set now explicitly includes:

- `nx`
- `nx.cmd`
- `nx.ps1`
- `nx-cli.cjs`

## Exact current project environment contract

The generic project bootstrap currently sets the Nx runtime environment like this:

```powershell
$localNxSocketRoot = 'C:\nxs'
if (-not (Test-Path -LiteralPath $localNxSocketRoot)) {
  New-Item -ItemType Directory -Force -Path $localNxSocketRoot | Out-Null
}
$boxedComSpec = $windowsShellRuntime.CmdExe
if ([string]::IsNullOrWhiteSpace($boxedComSpec) -or -not (Test-Path -LiteralPath $boxedComSpec)) {
  throw 'Local boxed CMD executable not found for ComSpec override.'
}
$env:NX_DAEMON = 'false'
$env:NX_SOCKET_DIR = $localNxSocketRoot
$env:NX_ISOLATE_PLUGINS = 'false'
$env:NX_NATIVE_FILE_CACHE_DIRECTORY = $localNxCacheRoot
$env:TEMP = $localTempRoot
$env:TMP = $localTempRoot
$env:TMPDIR = $localTempRoot
$env:PUPPETEER_CACHE_DIR = $localPuppeteerCacheRoot
$env:PLAYWRIGHT_BROWSERS_PATH = $localPlaywrightBrowsersRoot
$env:ComSpec = $boxedComSpec
$env:COMSPEC = $boxedComSpec

$env:BOXED_VSCODE_MODE = 'Project'
$env:BOXED_PROJECT_NAME = $ProjectName
$env:BOXED_REPO_PATH = $RepoPath
$env:BOXED_VSCODE_USERDATA = $projectPaths.UserData
$env:BOXED_LOCAL_EXTENSIONS = $projectPaths.ExtensionsDir
$env:BOXED_SHARED_EXTENSIONS = $SharedExtensionsRoot
$env:BOXED_VSCODE_RUNTIME_ROOT = $localRuntime.RuntimeRoot
$env:BOXED_CODE_EXE = $localRuntime.CodeExe
$env:BOXED_CODE_CLI = $localRuntime.CodeCli
$env:BOXED_LOCAL_TOOLCHAIN_ROOT = $projectPaths.LocalToolchainRoot
$env:BOXED_NX_NATIVE_CACHE_ROOT = $localNxCacheRoot
$env:BOXED_LOCAL_TEMP_ROOT = $localTempRoot
$env:BOXED_PUPPETEER_CACHE_DIR = $localPuppeteerCacheRoot
$env:BOXED_PLAYWRIGHT_BROWSERS_PATH = $localPlaywrightBrowsersRoot
$env:BOXED_NX_SOCKET_DIR = $localNxSocketRoot
$env:BOXED_COMSPEC = $boxedComSpec
$env:BOXED_NODE_ROOT = $nodeRuntime.NodeRoot
$env:BOXED_PNPM_CLI = $nodeRuntime.PnpmCli
$env:BOXED_NODE_EXTRA_COMMANDS = (($AdditionalNodeCommands.Keys | Sort-Object) -join ',')
$env:BOXED_STARSHIP_ROOT = $starshipRuntime.StarshipRoot
$env:BOXED_STARSHIP_EXE = $starshipRuntime.StarshipExe
$env:BOXED_STARSHIP_CONFIG = $starshipRuntime.ConfigPath
$env:BOXED_BASH_MINIMAL_RC = $starshipRuntime.BashMinimalRc
$env:BOXED_BASH_STARSHIP_RC = $starshipRuntime.BashStarshipRc
$env:BOXED_STARSHIP_AVAILABLE = if ($starshipRuntime.Available) { 'true' } else { 'false' }
```

This is the exact reason the Nx contract belongs to bootstrap:

- the socket path
- native cache path
- wrapper publication
- and local mirrored command surface

all come into existence before the developer types the first Nx command.

The current shell-selection contract belongs here as well, because `nx:run-commands` is sensitive to the Windows command-interpreter environment.

## Maintenance integration

The maintenance bootstrap also calls into the same Node bootstrap layer:

```powershell
$nodeRuntime = Initialize-NodeToolchainRuntime `
  -GitRoot $gitRoot `
  -NodeRoot $nodeRoot `
  -PnpmCli $pnpmCli `
  -BootstrapBin $maintenancePaths.BootstrapBin `
  -LocalToolchainRoot $maintenancePaths.LocalToolchainRoot `
  -AdditionalNodeCommands $additionalNodeCommands
```

That means the current Nx wrapper generation is not project-only.
It is part of the shared shared-toolchain bootstrap layer.

## How it is currently called

### Host-side project terminal launch

The current live project-terminal launch shape is:

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:VS_CODE_TEST_MONO `
  "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
  -NoLogo `
  -NoExit `
  -ExecutionPolicy Bypass `
  -File "C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoVSCode.ps1" `
  -Action OpenTerminal `
  -RepoPath "C:\Users\yourusername\source\test-mono"
```

### Plain boxed Nx command surface

Inside the bootstrapped project terminal, the expected entrypoints are now:

```powershell
nx --version
nx show projects
```

### Diagnostic wrapper verification

The current diagnostic verification commands are:

```powershell
$env:BOXED_BOOTSTRAP_BIN
$env:BOXED_NX_LAUNCHER
Get-ChildItem "$env:BOXED_BOOTSTRAP_BIN\nx*"
where.exe nx
Get-Command nx | Format-List Name,Source,Definition,CommandType
```

## Latest validated output

Latest validated boxed output showed:

- `nx`
- `nx-cli.cjs`
- `nx.cmd`
- `nx.ps1`

under:

- `C:\Program Files\SandboxToolchains\VSCodeBoxes\test-mono\execution\bootstrap-bin`

And `Get-Command nx -All` showed:

- `nx.ps1`
- `nx.cmd`
- `nx`

with PowerShell resolving `nx` from:

- `C:\Program Files\SandboxToolchains\VSCodeBoxes\test-mono\execution\bootstrap-bin\nx.ps1`

And:

```powershell
nx show projects
```

returned:

- `test-tooling`
- `installer`
- `frontend`
- `webpages`
- `backend`
- `test`

The latest validated boxed environment also showed:

```powershell
$env:COMSPEC
$env:ComSpec
```

returning:

```text
C:\Program Files\SandboxToolchains\VSCodeBoxes\test-mono\execution\toolchain\shells\cmd\10.0.26100.8457\cmd.exe
```

That means the default Windows shell surface used by `nx:run-commands` is now redirected by bootstrap to the boxed-CMD lane for the currently validated command set.

## Related

- `docs\cli\shell\general.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\execution-surfaces.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\runtime-contract.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\scripts.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\general.md`
