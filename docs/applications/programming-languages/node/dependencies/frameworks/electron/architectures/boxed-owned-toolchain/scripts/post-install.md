# Electron Post-Install Script

## Scope

This document owns the **project-owned Electron post-install script** for the boxed-owned-toolchain architecture.

It exists because the Electron-specific runtime materialization and repair logic belongs in the Electron domain, not in the generic PNPM install-domain write-up and not in the normal VS Code start scripts.

## Why this belongs here

The current repository architecture separates:

- generic package-manager lifecycle orchestration
- Electron-specific runtime verification and repair
- normal VS Code / bootstrap start flows

That means:

- the PNPM install and clean-reinstall scripts may **call** an Electron post-install step
- but they should not own the full Electron-specific repair code inline
- the normal VS Code start scripts should **not** mutate the workspace just to compensate for a missing Electron runtime

## Two supported operational shapes

For the current boxed-owned-toolchain architecture, there are two supported ways to handle the Electron follow-up surface after `pnpm install`:

1. **manual trigger later**
   - use the explicit Electron post-install script only when the runtime is missing
2. **integrated post-install**
   - let the project-owned PNPM install / clean-reinstall scripts call the Electron post-install script immediately after a successful `pnpm install`

The second option is the preferred automation shape when the project repeatedly lands in the known Electron partial-materialization state.

## Sanitized script path

```text
C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoElectronPostInstall.ps1
```

## Sanitized script body

```powershell
param(
  [string]$RepoPath,
  [string]$ElectronPackagePath = 'apps\test',
  [string]$RuntimeProbeWorkingDirectory = 'apps\test-tooling',
  [switch]$SkipBootstrap
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

if (-not $SkipBootstrap) {
  $launcher = Join-Path $PSScriptRoot 'Start-TestMonoVSCode.ps1'
  if (-not (Test-Path -LiteralPath $launcher)) {
    throw "Project launcher not found: $launcher"
  }

  & $launcher -Action OpenTerminal -RepoPath $resolvedRepoPath
}

if (-not (Test-Path -LiteralPath $resolvedRepoPath)) {
  throw "Repo path not found: $resolvedRepoPath"
}

if ([string]::IsNullOrWhiteSpace($env:BOXED_ELECTRON_INSTALL_WRAPPER) -or -not (Test-Path -LiteralPath $env:BOXED_ELECTRON_INSTALL_WRAPPER)) {
  throw 'Electron install wrapper was not initialized by project bootstrap.'
}

if ([string]::IsNullOrWhiteSpace($env:BOXED_ELECTRON_CACHE_DIR)) {
  throw 'BOXED_ELECTRON_CACHE_DIR was not initialized by project bootstrap.'
}

if ([string]::IsNullOrWhiteSpace($env:BOXED_NODE20_EXE) -or -not (Test-Path -LiteralPath $env:BOXED_NODE20_EXE)) {
  throw 'BOXED_NODE20_EXE was not initialized by project bootstrap.'
}

$env:ELECTRON_CACHE = $env:BOXED_ELECTRON_CACHE_DIR
$env:electron_config_cache = $env:BOXED_ELECTRON_CACHE_DIR

Set-Location $resolvedRepoPath

$electronAppRoot = Join-Path $resolvedRepoPath $ElectronPackagePath
$electronPackageJson = Join-Path $electronAppRoot 'package.json'
if (-not (Test-Path -LiteralPath $electronPackageJson)) {
  throw "Electron package.json not found: $electronPackageJson"
}

$electronCli = node -e "const path=require('node:path'); const {createRequire}=require('node:module'); const appRoot=path.resolve(process.argv[1]); const req=createRequire(path.resolve(appRoot,'package.json')); process.stdout.write(req.resolve('electron/cli.js'));" "$electronAppRoot"
if ([string]::IsNullOrWhiteSpace($electronCli) -or -not (Test-Path -LiteralPath $electronCli)) {
  throw 'Electron CLI could not be resolved after pnpm install.'
}

$electronDir = Split-Path -Parent $electronCli
Write-Host "RepoPath: $resolvedRepoPath"
Write-Host "ElectronPackagePath: $ElectronPackagePath"
Write-Host "ElectronCli: $electronCli"
Write-Host "ElectronDir: $electronDir"
Write-Host "ElectronCacheDir: $env:BOXED_ELECTRON_CACHE_DIR"
Write-Host "ElectronInstallWrapper: $env:BOXED_ELECTRON_INSTALL_WRAPPER"
Write-Host "ElectronNode20: $env:BOXED_NODE20_EXE"
Write-Host 'Running Electron post-install verification/repair...'

$scriptExitCode = 0

Push-Location $electronDir
try {
  node "$env:BOXED_ELECTRON_INSTALL_WRAPPER"
  $scriptExitCode = $LASTEXITCODE
}
finally {
  Pop-Location
}

if ($scriptExitCode -ne 0) {
  exit $scriptExitCode
}

$runtimeProbeRoot = Join-Path $resolvedRepoPath $RuntimeProbeWorkingDirectory
$runtimeProbeScript = Join-Path $runtimeProbeRoot 'scripts\smoke-electron-runtime.mjs'
if (-not (Test-Path -LiteralPath $runtimeProbeScript)) {
  throw "Electron runtime probe not found: $runtimeProbeScript"
}

Write-Host "RuntimeProbeRoot: $runtimeProbeRoot"
Write-Host 'Running Electron runtime smoke probe...'

Push-Location $runtimeProbeRoot
try {
  node 'scripts/smoke-electron-runtime.mjs'
  $scriptExitCode = $LASTEXITCODE
}
finally {
  Pop-Location
}

exit $scriptExitCode
```

## Sanitized manual host command

Use this when you choose the **manual trigger later** option:

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:VS_CODE_TEST_MONO `
  "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
  -NoLogo `
  -NoExit `
  -ExecutionPolicy Bypass `
  -File "C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoElectronPostInstall.ps1" `
  -RepoPath "C:\Users\yourusername\source\test-mono"
```

## Integration rule

If the project chooses the **integrated post-install** option, the project-owned PNPM scripts should:

1. run `pnpm install`
2. stop immediately if `pnpm install` fails
3. call the Electron post-install script only after the install succeeded

That keeps the separation clean:

- PNPM scripts own lifecycle orchestration
- the Electron domain owns Electron runtime verification and repair

## What should not own this logic

This logic should **not** primarily live in:

- generic VS Code start scripts
- generic project launch scripts
- normal IDE startup preflight

Why:

- Electron repair mutates workspace-local package state
- that mutation belongs to install/reinstall or an explicit maintenance step
- it should not burden every normal VS Code launch

## Related

- `docs\applications\programming-languages\node\dependencies\frameworks\electron\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\dependencies\frameworks\electron\architectures\boxed-owned-toolchain\troubleshooting.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\install.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\clean-reinstall.md`
