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
- prefer PowerShell-native recursive deletion and only fall back to the explicit boxed `cmd.exe` lane if needed

The Puppeteer-specific meaning of these browser-cache paths is owned here:

- `docs\applications\programming-languages\node\dependencies\puppeteer\boxed-owned-toolchain\overview.md`

Electron-specific follow-up verification and repair after a clean reinstall is owned here:

- `docs\applications\programming-languages\node\dependencies\frameworks\electron\architectures\boxed-owned-toolchain\overview.md`

The full project-owned Electron post-install script contract now lives here:

- `docs\applications\programming-languages\node\dependencies\frameworks\electron\architectures\boxed-owned-toolchain\scripts\post-install.md`

The current scope does **not** include:

- deleting `package.json`, `pnpm-lock.yaml`, or `pnpm-workspace.yaml`
- deleting the whole box / sandbox contents
- deleting the PNPM store as a baseline step

## Sanitized script path

```text
C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoPnpmCleanReinstall.ps1
```

## Sanitized script body

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

$launcher = Join-Path $PSScriptRoot 'Start-TestMonoVSCode.ps1'
if (-not (Test-Path -LiteralPath $launcher)) {
  throw "Project launcher not found: $launcher"
}

& $launcher -Action OpenTerminal -RepoPath $resolvedRepoPath

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

& $electronPostInstallScript -RepoPath $resolvedRepoPath -SkipBootstrap

exit $LASTEXITCODE
```

## Sanitized host command

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

## Architectural interpretation

The clean-reinstall path now matches the productive boxed-owned-toolchain PNPM contract:

- bootstrap publishes the boxed `node-gyp` wrapper surface
- the project-owned script initializes the boxed Windows native-build helper before the reinstall
- the project-owned script sets `scriptShell` to boxed Git Bash
- the reinstall happens inside the normal project bootstrap context
- after a successful reinstall, the project-owned script calls the Electron post-install verification / repair step

Boxed `cmd.exe` remains relevant for:

- the boxed `VsDevCmd` import inside the native-build helper
- cleanup fallback in the uninstall helper
- separate Windows child-process surfaces outside the install/reinstall lifecycle lane

But the clean-reinstall contract itself now deliberately returns to Git Bash as the preferred lifecycle shell, because the boxed-CMD-only path otherwise collapses toward postinstall suppression and manual replay.

## Expected next validation step

For projects that do not want this integrated automation, the alternative remains:

- finish the reinstall
- run the Electron post-install script manually later only when the runtime is missing

After the clean reinstall finishes, the next direct PNPM-domain validation step is the relevant runtime probe or dependency consumer, for example:

```bash
( cd apps/test-tooling && node scripts/smoke-electron-runtime.mjs )
```

If that probe still reports that Electron was not materialized correctly, switch from the PNPM-domain reinstall document to the Electron-domain troubleshooting document above.

The architecture-specific troubleshooting interpretation of that failure class lives here:

- `docs\applications\programming-languages\node\dependencies\frameworks\electron\architectures\boxed-owned-toolchain\troubleshooting.md`

## Related

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\install.md`
- `docs\applications\programming-languages\node\dependencies\frameworks\electron\architectures\boxed-owned-toolchain\scripts\post-install.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\msbuild-file-tracking-wrapper.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\clean-reinstall.md`
- `docs\applications\programming-languages\node\dependencies\puppeteer\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\dependencies\frameworks\electron\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\dependencies\frameworks\electron\architectures\boxed-owned-toolchain\troubleshooting.md`
