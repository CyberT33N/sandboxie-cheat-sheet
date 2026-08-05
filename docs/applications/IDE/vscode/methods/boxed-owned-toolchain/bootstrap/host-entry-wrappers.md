# Project Host Entry Wrappers

## Scope

This document defines the host-facing project launch layer for the
boxed-owned-toolchain method.

It covers:

- selecting the correct project box
- starting boxed CMD followed by boxed PowerShell
- delegating into the existing in-box `VS Code` or `Cursor` project wrapper
- provisioning the independent `ugrep` Dev binary into the local project
  toolchain and compatible ProgramData path

It does not replace editor-specific in-box bootstrap scripts.

## Why host wrappers are required

The project bootstrap scripts run **inside** a sandbox. They cannot choose the
first PowerShell image that enters the box because PowerShell must already be
running before they can execute.

After the Microsoft `.NET Framework` projection exists, starting host
PowerShell directly in a project box can fail with `SBIE1305` when
`ProtectHostImages=y`.

Therefore the normal project entrypoint is:

```text
Host wrapper
  -> Start.exe /box:<project-box>
    -> boxed cmd.exe
      -> boxed PowerShell
        -> existing in-box project wrapper
```

The Windows projection rationale belongs here:

- `docs\applications\operating-systems\windows\build-toolchain\microsoft\architectures\boxed-owned-toolchain\host-image-launch-boundary.md`

## Wrapper responsibilities

### Shared editor host wrapper

Each project should expose one host-facing editor wrapper with:

```text
Start-<Project>EditorBoxed.ps1
```

It:

1. resolves the selected editor (`VSCode` or `Cursor`)
2. resolves the editor's project box and box-family directory
3. resolves the already mirrored boxed PowerShell executable
4. starts boxed CMD and then boxed PowerShell
5. delegates to the existing editor-specific in-box project script

### Thin editor wrappers

Each project should additionally expose thin wrappers:

```text
Start-<Project>VSCodeBoxed.ps1
Start-<Project>CursorBoxed.ps1
```

They fix only the editor identity and delegate to
`Start-<Project>EditorBoxed.ps1`.

This keeps one host-entry implementation instead of duplicating launch,
Sandboxie, and local-PowerShell logic per editor.

### Explicit workload entries

When a project-owned in-box script needs a host entrypoint but has no dedicated
thin wrapper yet, it must still delegate through `Start-<Project>EditorBoxed.ps1`;
do not start host PowerShell directly through `Start.exe`.

Use its explicit workload parameters:

```text
-EntryScriptName
-EntryArguments
-KeepOpen
```

For example, an Electron repair can select its in-box script while the shared
host wrapper still resolves boxed CMD and the already mirrored boxed PowerShell.

### Host-facing dependency wrappers

Project-owned dependency scripts that can trigger native builds must also enter
through the boxed host boundary. The project should expose wrappers such as:

```text
Start-<Project>PnpmInstallBoxed.ps1
Start-<Project>PnpmCleanReinstallBoxed.ps1
Start-<Project>PnpmUninstallBoxed.ps1
```

They delegate through the same host entry layer before invoking their existing
in-box PNPM scripts. A dependency wrapper may expose `-KeepOpen` and forward
it to the shared host wrapper when it must preserve the former direct-launch
`-NoExit` behavior.

### Existing in-box wrappers remain required

Do **not** replace:

```text
Start-<Project>Editor.ps1
Start-<Project>VSCode.ps1
Start-<Project>Cursor.ps1
```

They still own:

- local runtime mirroring
- user-data and extension setup
- shell/toolchain initialization
- project terminal preparation
- editor GUI launch

The host wrappers only solve the pre-bootstrap host-to-box process boundary.

## Sanitized commands

### Open a Cursor project terminal

```powershell
& "C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoCursorBoxed.ps1" `
  -Action OpenTerminal `
  -RepoPath "C:\Users\yourusername\source\test-mono"
```

### Launch Cursor for a project

```powershell
& "C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoCursorBoxed.ps1" `
  -Action LaunchCursor `
  -RepoPath "C:\Users\yourusername\source\test-mono"
```

### Open a VS Code project terminal

```powershell
& "C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoVSCodeBoxed.ps1" `
  -Action OpenTerminal `
  -RepoPath "C:\Users\yourusername\source\test-mono"
```

### Launch VS Code for a project

```powershell
& "C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoVSCodeBoxed.ps1" `
  -Action LaunchVSCode `
  -RepoPath "C:\Users\yourusername\source\test-mono"
```

Do not use host `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`
as the normal project-box entrypoint after the `.NET Framework` projection
exists.

### Run boxed PNPM install

```powershell
& "C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoPnpmInstallBoxed.ps1" `
  -Editor Cursor `
  -RepoPath "C:\Users\yourusername\source\test-mono" `
  -KeepOpen
```

## `ugrep` runtime contract

`ugrep` is a normal governed Dev binary, independent of editor extensions or
MCP server implementation details.

The host-side provisioning source is the real Chocolatey package payload:

```text
C:\ProgramData\chocolatey\lib\ugrep\tools\bin\ugrep.exe
```

The shared governed target is:

```text
C:\shared\sandbox-toolchains\dev\ugrep\<version>\ugrep.exe
```

Bootstrap copies it into:

```text
C:\Program Files\SandboxToolchains\<box-family>\<project>\
  execution\toolchain\ugrep\<version>\ugrep.exe
```

It also projects the same governed binary into the compatible boxed path:

```text
C:\ProgramData\chocolatey\bin\ugrep.exe
```

The local `ugrep` directory is prepended to `PATH`, and bootstrap exports:

```text
BOXED_UGREP_ROOT
BOXED_UGREP_EXE
UGREP_EXECUTABLE_PATH
```

The binary is verified by checksum during projection.

Each project box that consumes `ugrep` must be able to read the governed shared
source:

```ini
ReadFilePath=C:\shared\sandbox-toolchains\dev\ugrep\
```

## Related

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\shared-layout.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\scripts.md`
- `docs\applications\IDE\cursor\methods\boxed-owned-toolchain\bootstrap\scripts.md`
- `docs\applications\operating-systems\windows\dependency-manager\chocolatey\general.md`
