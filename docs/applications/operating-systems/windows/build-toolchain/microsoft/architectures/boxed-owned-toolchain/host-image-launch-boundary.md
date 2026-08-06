# Host Image Protection And Boxed PowerShell Launch

## Scope

This document owns the Windows/Sandboxie compatibility boundary between:

- the boxed `.NET Framework` projection under `C:\Windows\Microsoft.NET\...`
- `ProtectHostImages=y`
- the initial PowerShell process that starts a project bootstrap

It does not redefine the Node-Gyp build contract. Node-Gyp consumes the
Microsoft build projection documented in this Windows build-toolchain domain.

## The compatibility problem

The boxed-owned-toolchain projects the governed `.NET Framework` tree into
the canonical Windows paths required by `Add-Type`, `buildcheck`, `csc.exe`,
`cvtres.exe`, and related native-build consumers:

```text
C:\Windows\Microsoft.NET\Framework\v4.0.30319\
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\
```

That projection intentionally contains CLR runtime files, including:

```text
mscoreei.dll
clr.dll
clrjit.dll
```

With `ProtectHostImages=y`, Sandboxie blocks a **host-image** process from
loading one of those boxed images. Therefore this host-side entrypoint is not
valid after the boxed `.NET Framework` projection exists:

```text
Start.exe /box:<project-box> C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe ...
```

The failure appears as `SBIE1305`, for example for boxed `mscoreei.dll`,
`clr.dll`, or `clrjit.dll`.

## What must remain unchanged

Do not solve this failure by deleting or excluding individual CLR files from
the `.NET Framework` projection.

The full projected tree is part of the validated Microsoft/Node-Gyp runtime
contract. In particular, the following must remain governed shared sources
and boxed projections:

- `.NET Framework` compiler roots
- `csc.exe`
- `cvtres.exe`
- `Add-Type` / `buildcheck` compatibility files
- the required CLR runtime dependencies

`ProtectHostImages=y` also remains the preferred project-box security setting.

## Correct launch boundary

The first managed process for a project box with the `.NET` projection must be
the already mirrored **boxed PowerShell**, not the host PowerShell image.

The required process chain is:

```text
Host launcher
  -> Sandboxie Start.exe
    -> boxed cmd.exe
      -> boxed PowerShell
        -> existing in-box project bootstrap
```

The boxed PowerShell executable is located in the project execution tree:

```text
C:\Program Files\SandboxToolchains\<box-family>\<project>\
  execution\toolchain\shells\powershell\<version>\powershell.exe
```

Because this executable is itself a boxed image, it can load the boxed CLR
projection without violating `ProtectHostImages=y`.

## First-bootstrap fallback

Before the local boxed PowerShell mirror exists, the host launcher may use
host PowerShell once to create the initial local runtime/toolchain mirror.

That fallback is valid only before the boxed `.NET Framework` projection is
present. All normal later project launches and terminals must use boxed CMD
followed by boxed PowerShell.

## Ownership split

### Windows build-toolchain domain

Owns:

- full `.NET Framework` projection
- CLR image compatibility boundary
- `ProtectHostImages=y` interaction
- the requirement for a boxed initial PowerShell image

### Node-Gyp domain

Owns:

- when native-build preparation is requested
- use of `Initialize-NodeGypWindowsBuildEnvironment`
- use of projected Microsoft compiler/build surfaces

### IDE/project bootstrap domain

Owns:

- host-facing project launch wrappers
- editor selection (`VSCode` or `Cursor`)
- selection of the matching project box and box family
- invocation of the existing in-box project script

## Validation

After a full `.NET Framework` projection exists:

1. terminate all processes in the project box
2. start through the project host launcher
3. confirm that the launcher uses boxed CMD and boxed PowerShell
4. open a terminal successfully with `ProtectHostImages=y`
5. run the Node-Gyp build verification separately

Do not validate this boundary by starting host
`C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe` directly in the
project box.

## Non-configurable in-box consumers

The normal host-entry solution controls the first PowerShell image explicitly.
Some in-box third-party consumers cannot be configured and can later
hard-code the same canonical host PowerShell path as a child-process target.

That is the Cursor Agent Shell failure class. The correct response remains
box-owned execution: bootstrap projects the local boxed PowerShell tree into
the corresponding **virtual** Windows path before the consumer launches.
It must not become a host-PowerShell access exception.

See:

- `docs\applications\IDE\cursor\architectures\boxed-owned-toolchain\troubleshooting\agent-shell-host-powershell-projection.md`

## Related

- `docs\applications\operating-systems\windows\build-toolchain\microsoft\architectures\boxed-owned-toolchain\dotnet-framework-projection.md`
- `docs\applications\operating-systems\windows\build-toolchain\microsoft\architectures\boxed-owned-toolchain\microsoft-build-projection.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\runtime-contract.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\host-entry-wrappers.md`
