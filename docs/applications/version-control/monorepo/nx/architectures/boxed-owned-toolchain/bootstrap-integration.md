# Nx Bootstrap Integration

## Scope

This document owns the **current default bootstrap integration** for Nx in the boxed-owned-toolchain architecture.

It records:

- the exact live shared files that implement the current behavior
- the exact project bootstrap environment that Nx inherits
- the recommended standard command surface
- the optional legacy wrapper split

It no longer treats a plain `nx` wrapper as the required default contract.

## Current live implementation files

The current live shared implementation surfaces are:

- `C:\shared\sandbox-toolchains\dev\bootstrap\stacks\node\Bootstrap.Node.psm1`
- `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode\Start-VSCodeProjectBase.ps1`
- `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode\Start-VSCodeMaintenance.ps1`

The current live project adapter chain used in the active project context includes:

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

That means the default Nx runtime contract is not hardcoded in the project adapter.

It is inherited from the generic shared Node/bootstrap layer.

## Current default bootstrap contract

The current default bootstrap contract is:

1. publish the governed local mirrored Node runtime
2. publish the governed local mirrored `pnpm` command surface
3. inject the boxed Nx runtime environment
4. keep the Windows child-process shell contract explicit through boxed `cmd.exe`
5. let Nx run through the workspace-local installation via `pnpm exec nx ...`

This means the current default bootstrap does **not** need to publish a plain `nx` alias in order for Nx to work.

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
```

This is the exact reason the Nx contract belongs to bootstrap:

- the socket path
- native cache path
- temp/cache paths
- and the Windows child-process contract

all come into existence before the developer runs the first Nx command.

## Recommended standard command surface

The current recommended standard command surface is:

```powershell
pnpm exec nx --version
pnpm exec nx show projects
pnpm exec nx run test:serve --no-tui -- --profile=dev-evident
```

This keeps Nx on the workspace-local installation and avoids requiring a bootstrap-published alias for the normal path.

The lower-level diagnostic path remains:

```powershell
$nxCli = node -p "try { require.resolve('nx/bin/nx.js') } catch { require.resolve('nx/dist/bin/nx.js') }"
node $nxCli --version
node $nxCli show projects
```

## Latest validated default result

The latest validated default boxed results showed:

- `pnpm exec nx --version` succeeds
- `pnpm exec nx show projects` succeeds
- `pnpm exec tsx --version` succeeds in the real target working directory used by the application runner
- `pnpm exec tsx tooling/run/cli.ts --help` reaches the real runner surface
- the full application can be started without depending on a bootstrap-published plain-`nx` alias

That means the old wrapper is no longer required for the currently validated standard path.

## Optional legacy wrapper surface

The historical plain-`nx` wrapper has been moved out of the default contract.

It now belongs to a separate optional / legacy surface:

- documentation:
  - `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\deprecated-wrapper-command-surface.md`
- shared implementation:
  - `C:\shared\sandbox-toolchains\dev\bootstrap\stacks\node\Bootstrap.NxWrapper.psm1`

That optional path can still be useful when a team explicitly wants:

- a plain `nx` alias in PowerShell
- a plain `nx` alias in CMD
- a plain `nx` alias in Git Bash

But it is no longer the recommended default and no longer part of the required baseline.

## Maintenance integration

The maintenance bootstrap still calls into the same shared Node bootstrap layer:

```powershell
$nodeRuntime = Initialize-NodeToolchainRuntime `
  -GitRoot $gitRoot `
  -NodeRoot $nodeRoot `
  -PnpmCli $pnpmCli `
  -BootstrapBin $maintenancePaths.BootstrapBin `
  -LocalToolchainRoot $maintenancePaths.LocalToolchainRoot `
  -AdditionalNodeCommands $additionalNodeCommands
```

That keeps the Node/PNPM/runtime contract consistent, without forcing the optional legacy Nx alias into every default maintenance shell.

## Related

- `docs\cli\shell\general.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\execution-surfaces.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\runtime-contract.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\deprecated-wrapper-command-surface.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\scripts.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\general.md`
