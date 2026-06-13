# PNPM Install Script

## Scope

This document owns the boxed-owned-toolchain PNPM **install-script contract**.

For this architecture, the source of truth is:

- the host may launch the install flow
- but the install behavior lives in a **project-owned PS1**
- and that PS1 consumes the project contract instead of overriding it through host parameters

This is the PNPM-domain SSOT for install-script behavior. VS Code method documents should re-reference this file instead of restating PNPM install semantics inline.

## Governance

The preferred shape is:

1. the project adapter selects `PnpmCli`
2. a project-owned install PS1 enters the project box through the normal bootstrap
3. that install PS1 initializes the boxed Windows native-build environment helper for `node-gyp`-bearing lifecycle paths
4. bootstrap publishes the boxed `node-gyp` wrapper surface before the install runs
5. that install PS1 resolves the local boxed Git Bash executable from the mirrored Git runtime
6. that install PS1 sets the validated PNPM lifecycle `scriptShell`
7. that install PS1 runs `pnpm install`

The current productive shell-specific requirement is:

- boxed Git Bash is the preferred lifecycle `scriptShell` for install/reinstall
- boxed `cmd.exe` remains available for native-build helper import, uninstall fallback, and other explicit Windows shell surfaces
- boxed PowerShell remains the preferred interactive default shell
- bootstrap publishes:
  - `node-gyp-wrapper.cjs`
  - `node-gyp`
  - `node-gyp.cmd`
  - `node-gyp.ps1`
  - `BOXED_NODE_GYP_JS`
  - `BOXED_NODE_GYP_REAL_JS`
  - `npm_config_node_gyp`

That keeps Windows native-build tracking control outside dependency source.

Architectural interpretation:

- Git Bash is preferred here because it keeps the generic PNPM lifecycle closer to the normal package-manager execution model
- boxed `cmd.exe` is still a first-class helper lane, but using it as the primary install lane pushes the project toward postinstall suppression and manual replay
- that broad manual replay model is not the preferred install contract

If Git Bash is selected explicitly, the bootstrap still needs:

- a shell-native `pnpm` wrapper in `bootstrap-bin`
- not only `pnpm.cmd`

The Puppeteer-specific browser-cache contract for boxed-owned-toolchain installs is owned here:

- `docs\applications\programming-languages\node\dependencies\puppeteer\boxed-owned-toolchain\overview.md`

That document explains why browser-download postinstall hooks must not use host-user-space defaults such as `C:\Users\<user>\.cache\...` in this architecture.

Electron-specific runtime-materialization troubleshooting is owned here:

- `docs\applications\programming-languages\node\dependencies\frameworks\electron\architectures\boxed-owned-toolchain\overview.md`

That document explains how to verify `path.txt` / `dist\electron.exe`, when the package is only partially materialized after `pnpm install`, and how to run the current validated repair sequence.

The architecture-specific troubleshooting interpretation of the Electron failure class lives here:

- `docs\applications\programming-languages\node\dependencies\frameworks\electron\architectures\boxed-owned-toolchain\troubleshooting.md`

The full project-owned Electron post-install script contract now lives here:

- `docs\applications\programming-languages\node\dependencies\frameworks\electron\architectures\boxed-owned-toolchain\scripts\post-install.md`

For the current Git-Bash-based install lane, there are therefore two supported operational choices:

1. run `pnpm install` and trigger the Electron post-install script manually later if the runtime is missing
2. call the Electron post-install script immediately after a successful `pnpm install`

The second option is the preferred automation shape when the project repeatedly lands in the known Electron partial-materialization state.

## Sanitized script path

```text
C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoPnpmInstall.ps1
```

## Sanitized script body

```powershell
param(
  [string]$RepoPath
)

$ErrorActionPreference = 'Stop'
Set-StrictMode -Version Latest

$launcher = Join-Path $PSScriptRoot 'Start-TestMonoVSCode.ps1'

if (-not (Test-Path -LiteralPath $launcher)) {
  throw "Project launcher not found: $launcher"
}

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

$null = Initialize-NodeGypWindowsBuildEnvironment `
  -CmdExe $env:BOXED_CMD_EXE `
  -RegExe $env:BOXED_REG_EXE `
  -PythonExe $env:BOXED_PYTHON_EXE

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

& $electronPostInstallScript -RepoPath $RepoPath -SkipBootstrap

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
  -File "C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoPnpmInstall.ps1" `
  -RepoPath "C:\Users\yourusername\source\test-mono"
```

## Architectural interpretation

This script body is the PNPM-domain proof that the preferred productive path is now:

- bootstrap-owned boxed helper lanes for Python / `reg.exe` / Windows native-build preparation
- project-owned `scriptShell` on explicit boxed Git Bash
- project-owned `pnpm install` launched through one explicit PS1
- project-owned Electron post-install verification / repair called only after `pnpm install` succeeded
- bootstrap-owned `node-gyp` wrapper publication so Windows MSBuild tracking behavior is adapted without editing downloaded dependencies

At the same time:

- bootstrap can still keep separate Windows child-process rules such as `ComSpec` / `COMSPEC`
- boxed `cmd.exe` remains available as a helper lane
- but the install contract itself now deliberately returns to Git Bash as the preferred lifecycle shell
- and Electron-specific repair logic stays in the Electron domain instead of being inlined permanently into the generic PNPM document

## Sanitized boilerplate note

The sanitized project-adapter example still lives here:

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\scripts.md`

That boilerplate remains useful as a reusable example, but PNPM-specific install-script ownership now lives in this PNPM-domain script document.

## Related

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\lifecycle-and-command-surface.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\clean-reinstall.md`
- `docs\applications\programming-languages\node\dependencies\frameworks\electron\architectures\boxed-owned-toolchain\scripts\post-install.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\msbuild-file-tracking-wrapper.md`
- `docs\applications\programming-languages\node\dependencies\puppeteer\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\dependencies\frameworks\electron\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\dependencies\frameworks\electron\architectures\boxed-owned-toolchain\troubleshooting.md`
