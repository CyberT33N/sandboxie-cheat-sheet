# PNPM Clean Reinstall Script

## Scope

This document owns the boxed-owned-toolchain PNPM **clean-reinstall script** contract for the current project-owned workflow.

Use this flow when:

- the workspace dependency tree may be stale or partially materialized
- a normal `pnpm install` returns too quickly to rebuild required runtime artifacts
- a project-specific runtime probe still reports missing packaged binaries after a normal install

## Current cleanup semantics

The current clean-reinstall script removes the **materialized dependency tree** from the repo and then runs a fresh boxed `pnpm install`.

The current scope is:

- remove workspace-root `.pnpm`
- remove workspace-root `node_modules`
- remove package-level `node_modules` under `apps`, `libs`, and `tools`
- remove `node_modules\.modules.yaml` if present
- remove box-local browser caches exposed through:
  - `BOXED_PUPPETEER_CACHE_DIR`
  - `BOXED_PLAYWRIGHT_BROWSERS_PATH`

The Puppeteer-specific meaning of these browser-cache paths is owned here:

- `docs\applications\programming-languages\node\dependencies\puppeteer\boxed-owned-toolchain\overview.md`

Electron-specific follow-up verification and repair after a clean reinstall is owned here:

- `docs\applications\programming-languages\node\dependencies\frameworks\electron\architectures\boxed-owned-toolchain\overview.md`

The current scope does **not** include:

- deleting `package.json`, `pnpm-lock.yaml`, or `pnpm-workspace.yaml`
- deleting the whole box / sandbox contents
- deleting the PNPM store as a baseline step

## Current real script path

```text
C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-testMonoPnpmCleanReinstall.ps1
```

## Current real script body

```powershell
param(
  [string]$RepoPath
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

$launcher = Join-Path $PSScriptRoot 'Start-testMonoVSCode.ps1'
if (-not (Test-Path -LiteralPath $launcher)) {
  throw "Project launcher not found: $launcher"
}

& $launcher -Action OpenTerminal -RepoPath $resolvedRepoPath

if (-not (Test-Path -LiteralPath $resolvedRepoPath)) {
  throw "Repo path not found: $resolvedRepoPath"
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
    cmd /c rmdir /s /q "$_"

    if (Test-Path -LiteralPath $_) {
      Remove-Item -LiteralPath $_ -Recurse -Force -ErrorAction SilentlyContinue
    }
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
  Remove-Item -LiteralPath $_ -Recurse -Force -ErrorAction SilentlyContinue
}

if ([string]::IsNullOrWhiteSpace($env:BOXED_LOCAL_TOOLCHAIN_ROOT)) {
  throw 'BOXED_LOCAL_TOOLCHAIN_ROOT was not initialized by project bootstrap.'
}

$bashExe = Join-Path $env:BOXED_LOCAL_TOOLCHAIN_ROOT 'git\2.54.0\bin\bash.exe'
if (-not (Test-Path -LiteralPath $bashExe)) {
  $bashExe = Join-Path $env:BOXED_LOCAL_TOOLCHAIN_ROOT 'git\2.54.0\usr\bin\bash.exe'
}

if (-not (Test-Path -LiteralPath $bashExe)) {
  throw 'Local boxed Bash executable not found.'
}

Set-Location $resolvedRepoPath

Write-Host "RepoPath: $resolvedRepoPath"
Write-Host "ScriptShell: $bashExe"
Write-Host 'Configuring PNPM lifecycle shell for boxed Git Bash...'
pnpm config set --location=project scriptShell "$bashExe"

Write-Host 'Running clean pnpm install...'
pnpm install

exit $LASTEXITCODE
```

## Current real host command

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:VS_CODE_test_MONO `
  "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
  -NoLogo `
  -NoExit `
  -ExecutionPolicy Bypass `
  -File "C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-testMonoPnpmCleanReinstall.ps1" `
  -RepoPath "C:\Users\denni\source\test-mono"
```

## Expected next validation step

After the clean reinstall finishes, the next direct PNPM-domain validation step is the relevant runtime probe or dependency consumer, for example:

```bash
( cd apps/test-tooling && node scripts/smoke-electron-runtime.mjs )
```

If that probe still reports that Electron was not materialized correctly, switch from the PNPM-domain reinstall document to the Electron-domain troubleshooting document above.

## Related

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\install.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\clean-reinstall.md`
- `docs\applications\programming-languages\node\dependencies\puppeteer\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\dependencies\frameworks\electron\architectures\boxed-owned-toolchain\overview.md`
