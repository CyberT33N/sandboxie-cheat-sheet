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
3. bootstrap projects the productive `ComSpec` / `COMSPEC` lane
4. that install PS1 sets the validated PNPM lifecycle `scriptShell`
5. that install PS1 runs `pnpm install`

The current productive shell-specific requirement is:

- boxed `cmd.exe` is the preferred lifecycle `scriptShell`
- boxed `cmd.exe` is the preferred productive child-process lane for PNPM command execution

The historical Git-Bash-based variant remains only as an alternative compatibility lane, not as the preferred productive path.

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

if ([string]::IsNullOrWhiteSpace($env:BOXED_CMD_EXE)) {
  throw 'BOXED_CMD_EXE was not initialized by project bootstrap.'
}

$cmdExe = $env:BOXED_CMD_EXE
if (-not (Test-Path -LiteralPath $cmdExe)) {
  throw 'Local boxed CMD executable not found.'
}

pnpm config set --location=project scriptShell "$cmdExe"
pnpm install

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

- bootstrap-owned `COMSPEC` / `ComSpec` on boxed `cmd.exe`
- project-owned `scriptShell` on boxed `cmd.exe`
- project-owned `pnpm install` launched through one explicit PS1

That means the install contract no longer depends on the historical Git-Bash-based lifecycle shell.

## Sanitized boilerplate note

The sanitized project-adapter example still lives here:

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\scripts.md`

That boilerplate remains useful as a reusable example, but PNPM-specific install-script ownership now lives in this PNPM-domain script document.

## Related

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\lifecycle-and-command-surface.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\clean-reinstall.md`
- `docs\applications\programming-languages\node\dependencies\puppeteer\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\dependencies\frameworks\electron\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\dependencies\frameworks\electron\architectures\boxed-owned-toolchain\troubleshooting.md`
