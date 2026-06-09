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
3. that install PS1 sets the validated PNPM lifecycle `scriptShell`
4. that install PS1 runs `pnpm install`

The current shell-specific requirement is part of this contract:

- Git Bash must be able to resolve bare `pnpm`
- so the bootstrap provides a shell-native wrapper in `bootstrap-bin`, not only `pnpm.cmd`

## Current real script path

```text
C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-testMonoPnpmInstall.ps1
```

## Current real script body

```powershell
param(
  [string]$RepoPath
)

$ErrorActionPreference = 'Stop'
Set-StrictMode -Version Latest

$launcher = Join-Path $PSScriptRoot 'Start-testMonoVSCode.ps1'

if (-not (Test-Path -LiteralPath $launcher)) {
  throw "Project launcher not found: $launcher"
}

& $launcher -Action OpenTerminal -RepoPath $RepoPath

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

pnpm config set --location=project scriptShell "$bashExe"
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
  -File "C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-testMonoPnpmInstall.ps1" `
  -RepoPath "C:\Users\denni\source\test-mono"
```

## Sanitized boilerplate note

The sanitized project-adapter example still lives here:

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\scripts.md`

That boilerplate remains useful as a reusable example, but PNPM-specific install-script ownership now lives in this PNPM-domain script document.

## Related

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\lifecycle-and-command-surface.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\clean-reinstall.md`
