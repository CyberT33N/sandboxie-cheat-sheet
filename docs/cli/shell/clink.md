# Clink

## Purpose

This document is the single source of truth for how `Clink` is modeled in this repository.

It belongs under `docs\cli\shell\` because `Clink` is not being treated as:

- a generic package-manager tool
- a VS Code-specific extension
- a prompt binary like `Starship`
- or a standalone terminal application

It is a **CMD-specific shell runtime adapter** that matters for:

- boxed `cmd.exe`
- prompt injection into `cmd.exe`
- Starship support for CMD
- bootstrap-owned shell runtime wiring

## What Clink is in this architecture

`Clink` is not a replacement shell.

In this repository it is modeled as:

- a versioned runtime artifact that extends `cmd.exe`
- a CMD-specific prompt and line-editing adapter
- a bootstrap-consumed dependency for the **CMD + Starship** lane

That means:

- plain boxed `CMD` does **not** require `Clink`
- boxed `PowerShell` does **not** require `Clink`
- boxed Git Bash does **not** require `Clink`
- boxed `CMD + Starship` **does** require `Clink`

## DDD placement decision

### Why the shared artifact belongs under `dev\clink\...`

The correct shared artifact location is:

```text
C:\shared\sandbox-toolchains\dev\clink\<version>\
```

Why this is correct:

- `dev\` already owns versioned developer runtime artifacts such as:
  - `git`
  - `node`
  - `pnpm`
  - `python`
  - `starship`
- `Clink` is also a versioned runtime dependency used by bootstrap
- it is provisioned and mirrored like the other governed developer runtimes

### Why not a new top-level shared folder like `shells\` or `terminals\`

The repository already separates:

- **artifact ownership** in the shared tree
- **domain ownership** in the documentation tree
- **runtime orchestration ownership** in bootstrap stacks

That means the clean split is:

- shared artifact tree:
  - `dev\clink\...`
- documentation domain:
  - `docs\cli\shell\...`
- bootstrap implementation:
  - `dev\bootstrap\stacks\shells\...`

Creating a new top-level shared root such as:

- `C:\shared\sandbox-toolchains\shells\...`
- or `C:\shared\sandbox-toolchains\terminals\...`

would fragment the shared artifact taxonomy without solving a real ownership problem.

So the current repository decision is:

- **no new top-level shared `shells` or `terminals` root**
- **keep versioned shell-adjacent artifacts under `dev\...`**

## Runtime mirror and state split

The local mirrored binary and the local mutable state intentionally live in different places.

### Shared canonical source

```text
C:\shared\sandbox-toolchains\dev\clink\1.9.26\
```

### Local mirrored runtime

```text
C:\Program Files\SandboxToolchains\VSCodeBoxes\<box>\execution\toolchain\shells\clink\1.9.26\
```

### Local mutable Clink profile/state

```text
C:\Program Files\SandboxToolchains\VSCodeBoxes\<box>\state\shells\cmd\clink\profile\
```

This split is important.

Why:

- the binary itself is execution-time runtime content
- the profile directory is mutable state
- `Clink` can write settings, history, logs, and user scripts relative to the profile directory
- mutable `Clink` state must therefore **not** live under `execution\bootstrap-bin`

## Source and provisioning choice

The current preferred provisioning source is the **official portable ZIP** from the Clink release page.

Why the ZIP is preferred:

- it is a versioned artifact
- it avoids machine-global autorun or installer side effects
- it fits the repository's current shared-artifact model
- it is easier to mirror and audit than a mutable host installation

The current governed version is:

```text
1.9.26
```

Official portable asset used:

```text
https://github.com/chrisant996/clink/releases/download/v1.9.26/clink.1.9.26.ac0bbf.zip
```

## Provisioning

```powershell
$ClinkVersion = '1.9.26'
$ClinkZip = Join-Path $env:TEMP "clink-$ClinkVersion.zip"
$ClinkDest = "C:\shared\sandbox-toolchains\dev\clink\$ClinkVersion"

Invoke-WebRequest `
  -Uri 'https://github.com/chrisant996/clink/releases/download/v1.9.26/clink.1.9.26.ac0bbf.zip' `
  -OutFile $ClinkZip

New-Item -ItemType Directory -Force -Path $ClinkDest | Out-Null
Remove-Item "$ClinkDest\*" -Recurse -Force -ErrorAction SilentlyContinue

Expand-Archive -LiteralPath $ClinkZip -DestinationPath $ClinkDest -Force

& "$ClinkDest\clink_x64.exe" --version
```

Expected verification:

```text
Clink v1.9.26
```

## Bootstrap contract

The current bootstrap contract is:

1. consume `Clink` from `dev\clink\<version>\`
2. mirror it locally into:
   - `execution\toolchain\shells\clink\<version>\`
3. prepend the local Clink root into `PATH`
4. generate a box-local Clink profile directory under:
   - `state\shells\cmd\clink\profile\`
5. generate `starship.lua` into that local profile directory
6. use the local Clink executable for:
   - `clink inject --profile "<local profile>"`

Representative environment surfaces:

- `BOXED_CLINK_ROOT`
- `BOXED_CLINK_EXE`
- `BOXED_CLINK_BAT`
- `BOXED_CLINK_AVAILABLE`
- `BOXED_CMD_STARSHIP_PROFILE`

## Why host copy is not the preferred model

Even if a host installation existed, the repository would still prefer the versioned shared ZIP path over copying a mutable host installation.

Why:

- host package-manager state is workstation-specific
- installer state can include autorun choices and user-local drift
- the ZIP path is easier to govern as a canonical artifact
- the method already uses the same pattern for tools such as `Starship`

## Related

- `docs\cli\shell\general.md`
- `docs\applications\terminal\starship\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\state\vscode-runtime-and-catalog.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\shared-layout.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\scripts.md`
