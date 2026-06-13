# Git Long-Path Troubleshooting

## Scope

This document owns the **Git-specific** part of long-path troubleshooting for the boxed-owned-toolchain architecture.

It covers:

- the Git-side symptom
- the Git-side option
- the bootstrap-owned boxed Git wrapper behavior

It does **not** own the Git-independent Windows / Sandboxie path-length policy.

That central source of truth lives here:

- `docs\troubleshooting\filesystem\windows-long-paths.md`

## Why this file exists

The long-path failure class spans more than Git:

- Windows path handling
- sandbox redirection
- filesystem limits
- application-specific toolchains

So the Git domain should own only the Git-specific slice:

- how Git surfaces the error
- how the boxed Git wrapper applies the Git-side option

## Typical Git-side symptom

Representative Git errors:

```text
error: unable to create file <path>: Filename too long
```

or similar checkout / pull failures during:

- `git pull`
- `git checkout`
- `git clone`

## Current Git-specific runtime rule

Inside a normal boxed-owned-toolchain project terminal, the preferred Git command surface is the bootstrap-published `git` wrapper.

Why:

- it automatically applies `core.longpaths=true`
- it stays aligned to the boxed-owned-toolchain local-mirror runtime
- it keeps the Git-side long-path fix inside the Git domain instead of relying on every user to remember an extra ad hoc flag

## Current bootstrap-owned Git wrapper code shape

Representative code shape:

```powershell
$gitExe = Join-Path $GitRoot 'cmd\git.exe'
$gitCredentialManagerExe = Join-Path $GitRoot 'mingw64\bin\git-credential-manager.exe'

if (Test-Path -LiteralPath $gitCredentialManagerExe) {
  $gitCmdContent = @(
    '@echo off'
    ('set "GIT_EXE={0}"' -f $gitExe)
    '"%GIT_EXE%" -c core.longpaths=true -c credential.helper= -c credential.helper=manager-boxed %*'
  ) -join [Environment]::NewLine
}
else {
  $gitCmdContent = @(
    '@echo off'
    ('set "GIT_EXE={0}"' -f $gitExe)
    '"%GIT_EXE%" -c core.longpaths=true %*'
  ) -join [Environment]::NewLine
}

Write-AsciiFile -Path (Join-Path $BootstrapBin 'git.cmd') -Content $gitCmdContent
```

Equivalent `git.ps1` and shell-native `git` wrappers are also published.

## Manual Git-side fallback

If you ever need to bypass the wrapper for diagnosis, the Git-side equivalent is:

```powershell
& "C:\shared\sandbox-toolchains\dev\git\2.54.0\cmd\git.exe" -c core.longpaths=true pull
```

But for the normal boxed-owned-toolchain runtime this manual form should **not** be the primary contract.

The primary contract is:

- use the boxed `git` wrapper

## What this file does not own

This file does **not** claim that `core.longpaths=true` alone solves the whole path-length problem.

That would be inaccurate.

The central long-path problem also includes:

- boxed `LongPathsEnabled=1`
- sandbox redirection length amplification
- `FileRootPath` interpretation

Those cross-domain concerns live here:

- `docs\troubleshooting\filesystem\windows-long-paths.md`

## Related

- `docs\applications\git\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\git\architectures\boxed-owned-toolchain\runtime-contract.md`
- `docs\troubleshooting\filesystem\windows-long-paths.md`
