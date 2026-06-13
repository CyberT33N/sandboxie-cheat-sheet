# Git Runtime Contract

## Scope

This document owns the **boxed runtime contract** for Git in the boxed-owned-toolchain architecture.

It covers:

- local Git projection into the box
- the bootstrap-published `git` command surface
- the bootstrap-published boxed Git Credential Manager helper command
- the Git-specific long-path runtime option

It does **not** own:

- PortableGit download and extraction
- initial authentication ceremony details
- Git-independent long-path policy

## Current runtime model

The runtime contract is split into two layers:

1. **governed shared Git root**
   - `C:\shared\sandbox-toolchains\dev\git\2.54.0\`
2. **boxed local mirrored Git runtime**
   - projected into the project box execution tree

Representative local boxed root:

```text
C:\Program Files\SandboxToolchains\VSCodeBoxes\test\execution\toolchain\git\2.54.0\
```

The boxed-owned-toolchain runtime principle is:

- Shared stays the governed Golden Source
- the box executes the local mirrored runtime
- bootstrap publishes explicit command surfaces instead of relying on raw shared executables directly

## Why the raw shared executable is not the final contract

The bare shared executable:

```text
C:\shared\sandbox-toolchains\dev\git\2.54.0\cmd\git.exe
```

is sufficient for provisioning checks, but it is not the full boxed execution contract.

The boxed execution contract must additionally solve:

- automatic `core.longpaths=true`
- helper execution inside the box
- helper execution without whitespace-sensitive absolute helper paths
- wrapper parity across PowerShell, CMD, and Git Bash

## Current bootstrap-published Git command surface

The current bootstrap publishes:

```text
git
git.cmd
git.ps1
git-credential-manager-boxed
git-credential-manager-boxed.cmd
git-credential-manager-boxed.ps1
```

These commands are published into:

```text
<boxed bootstrap-bin>
```

and resolved from `PATH` inside the boxed project terminal.

## Why `git-credential-manager-boxed` exists

The current repository hit a concrete failure where the helper path was effectively parsed as:

```text
C:/Program
```

instead of the full path to:

```text
C:/Program Files/SandboxToolchains/VSCodeBoxes/.../git-credential-manager.exe
```

That happened because a whitespace-containing absolute helper path is a fragile `credential.helper` surface.

The boxed-owned-toolchain answer is therefore:

- do **not** rely on a raw helper path with spaces
- publish a boxed helper command with a stable name
- let Git resolve that command from `PATH`

That is why the runtime contract now uses:

```text
credential.helper=manager-boxed
```

instead of an inline absolute helper path with spaces.

## Current bootstrap code shape

The current bootstrap contract is represented by code of this shape:

```powershell
$gitExe = Join-Path $GitRoot 'cmd\git.exe'
$gitCredentialManagerExe = Join-Path $GitRoot 'mingw64\bin\git-credential-manager.exe'

if (Test-Path -LiteralPath $gitCredentialManagerExe) {
  $gcmCmdContent = @(
    '@echo off'
    ('set "GCM_EXE={0}"' -f $gitCredentialManagerExe)
    '"%GCM_EXE%" %*'
  ) -join [Environment]::NewLine

  Write-AsciiFile -Path (Join-Path $BootstrapBin 'git-credential-manager-boxed.cmd') -Content $gcmCmdContent

  $gcmPs1Content = @(
    ('$gcmExe = ''{0}''' -f $gitCredentialManagerExe)
    '& $gcmExe @args'
    'exit $LASTEXITCODE'
  ) -join [Environment]::NewLine

  Write-AsciiFile -Path (Join-Path $BootstrapBin 'git-credential-manager-boxed.ps1') -Content $gcmPs1Content

  $gcmShellContent = (@(
    '#!/bin/sh'
    ('GCM_EXE_WIN=''{0}''' -f $gitCredentialManagerExe)
    ''
    'if command -v cygpath >/dev/null 2>&1; then'
    '  GCM_EXE="$(cygpath -u "$GCM_EXE_WIN")"'
    'else'
    '  GCM_EXE="$GCM_EXE_WIN"'
    'fi'
    ''
    'exec "$GCM_EXE" "$@"'
  ) -join "`n") + "`n"

  Write-AsciiFile -Path (Join-Path $BootstrapBin 'git-credential-manager-boxed') -Content $gcmShellContent

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

Equivalent PowerShell and shell-native wrappers are published alongside it.

## Long-path responsibility split

Git owns only the **Git-specific** long-path piece:

- `core.longpaths=true`

The Git-independent long-path policy is a broader Windows / filesystem troubleshooting concern and lives here:

- `docs\troubleshooting\filesystem\windows-long-paths.md`

That central document owns:

- boxed `LongPathsEnabled=1`
- sandbox path amplification
- `FileRootPath` interpretation

The Git-specific troubleshooting view lives here:

- `docs\applications\git\architectures\boxed-owned-toolchain\troubleshooting\long-paths.md`

## Exported runtime metadata

Representative runtime metadata now includes:

```powershell
$env:BOXED_GIT_ROOT = $GitRoot
$env:BOXED_GIT_EXE = $gitExe
$env:BOXED_GIT_CREDENTIAL_MANAGER_EXE = $gitCredentialManagerExe
```

These are diagnostic and integration surfaces for later boxed scripts.

## Current architectural interpretation

From a domain-driven and 12-factor perspective:

- Git authentication remains a Git concern
- the Git helper command surface remains a Git concern
- the boxed wrapper contract belongs in bootstrap because it is runtime configuration
- the local mirrored helper execution path is preferable to a drifting shared absolute helper path

That gives a runtime contract that is:

- explicit
- reviewable
- reproducible
- aligned to the local-mirror execution model

## Related

- `docs\applications\git\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\git\architectures\boxed-owned-toolchain\provisioning.md`
- `docs\applications\git\architectures\boxed-owned-toolchain\authentication-and-clone.md`
- `docs\applications\git\architectures\boxed-owned-toolchain\troubleshooting\long-paths.md`
- `docs\troubleshooting\filesystem\windows-long-paths.md`
