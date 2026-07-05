# Cursor Runtime Installation

## Scope

This document records the Cursor-specific runtime-installation model for the boxed-owned-toolchain architecture.

The shared tree provisioning source of truth for the family method still lives in the VS Code method area. This file keeps only the Cursor-specific installation delta.

## Why Cursor installation differs from VS Code

The current repository already has a validated shared-runtime acquisition path for `VS Code`.

`Cursor` currently differs in one important way:

- the repository does **not** treat Cursor as another archive/runtime that is provisioned through the existing VS Code runtime flow
- instead, the validated current method is to install the Cursor `.exe` through the dedicated Cursor Maintenance Box and then keep the resulting runtime under the separate Cursor runtime namespace

So the important split is:

- `VS Code` runtime provisioning stays in the existing VS Code method flow
- `Cursor` runtime installation stays in a dedicated Cursor installation flow

## Canonical runtime target

The current shared runtime target for Cursor is:

```text
C:\shared\sandbox-toolchains\ide\cursor\runtime\3.9.16\
```

The validated runtime surface there includes:

- `Cursor.exe`
- `resources\app\codeBin\code.cmd`
- `resources\app\bin\cursor.cmd`
- `resources\app\bin\code-tunnel.exe`
- `resources\app\bin\cursor-tunnel.exe`
- `tools\inno_updater.exe`

## Why the installation must happen through the Cursor Maintenance Box

The dedicated Cursor Maintenance Box exists so the runtime installation stays isolated from:

- the VS Code runtime
- the VS Code maintenance runtime state
- project-box live state
- shared family catalog and extension authorship

That isolation matters because a Cursor installation or update touches:

- a different executable tree
- different helper binaries
- different login/session behavior
- different product-specific diagnostics

So the correct installation bounded context is:

- `CURSOR_MAINTENANCE`

not:

- a VS Code project box
- the VS Code maintenance box
- or the host outside the maintained boxed flow

## Current installation flow

The validated current architectural flow is:

1. open the `CURSOR_MAINTENANCE` box
2. stage the Cursor installer `.exe` into that maintenance context
3. run the installer there
4. validate that the Cursor runtime files now exist
5. keep the resulting runtime under `C:\shared\sandbox-toolchains\ide\cursor\runtime\3.9.16\`
6. continue to consume the shared VSCode-family catalog, seed, and extension surfaces from:
   - `C:\shared\sandbox-toolchains\ide\vscode\...`

Important current nuance:

- the exact temporary installer staging path is **not** the architecture source of truth
- the architecture source of truth is the **boxed maintenance installation model** plus the final shared Cursor runtime target

## Current maintenance entrypoint

The current maintenance wrapper is:

```text
C:\shared\sandbox-toolchains\dev\bootstrap\platforms\cursor\Start-CursorMaintenance.ps1
```

The current standard terminal entrypoint is:

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:CURSOR_MAINTENANCE `
  "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
  -NoLogo `
  -NoExit `
  -ExecutionPolicy Bypass `
  -File "C:\shared\sandbox-toolchains\dev\bootstrap\platforms\cursor\Start-CursorMaintenance.ps1" `
  -Action OpenTerminal
```

The current standard GUI entrypoint is:

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:CURSOR_MAINTENANCE `
  "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
  -NoLogo `
  -NoExit `
  -ExecutionPolicy Bypass `
  -File "C:\shared\sandbox-toolchains\dev\bootstrap\platforms\cursor\Start-CursorMaintenance.ps1" `
  -Action LaunchCursor
```

## Relationship to shared family state

Installing Cursor does **not** create a second canonical catalog or extension store.

The current architecture still keeps these shared:

- `C:\shared\sandbox-toolchains\ide\vscode\catalog\vscode-user\`
- `C:\shared\sandbox-toolchains\ide\vscode\catalog\seed\globalStorage\`
- `C:\shared\sandbox-toolchains\ide\vscode\catalog\seed\roo\`
- `C:\shared\sandbox-toolchains\ide\vscode\extensions\`

So the installation difference is:

- separate runtime acquisition
- separate runtime namespace
- separate maintenance box

not:

- duplicated family settings
- duplicated extension store
- duplicated seeds

## Why the runtime must not be merged into the VS Code namespace

The architecture intentionally avoids pretending that:

- `Cursor.exe`
- `code.cmd`
- `cursor.cmd`
- updater/tunnel helpers

are interchangeable with the VS Code runtime tree.

Keeping a separate Cursor runtime namespace preserves:

- runtime ownership clarity
- simpler maintenance diagnostics
- cleaner Sandboxie allow/deny rules
- a smaller blast radius for product-specific issues

## Related

- `docs\applications\IDE\cursor\methods\boxed-owned-toolchain\general.md`
- `docs\applications\IDE\cursor\methods\boxed-owned-toolchain\boxes\maintenance-box.md`
- `docs\applications\IDE\cursor\architectures\boxed-owned-toolchain\sign-in.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\provisioning\shared-artifacts.md`
