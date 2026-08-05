# Cursor Runtime Update In The Boxed-Owned-Toolchain

## Scope

This document defines the complete, repeatable update procedure for the
`Cursor` runtime in the `boxed-owned-toolchain` architecture.

It covers:

- acquiring and validating a new Cursor installer
- installing it only in the dedicated Cursor Maintenance Box
- materializing the validated boxed runtime into the canonical shared runtime
  tree
- switching bootstrap and Sandboxie references to the new version
- validating the maintenance and project launch paths
- cleaning up temporary installer access
- rolling back safely when the previous runtime is still valid

This is a Cursor-specific runtime procedure. It does not redefine the shared
VSCode-family catalog, seed, extension, or toolchain governance.

## Architectural rule

The boxed-owned runtime update has three distinct layers:

```text
1. installer input
   C:\shared\CursorUserSetup-x64-<version>.exe

2. boxed installation result
   C:\Sandbox\yourusername\CURSOR_MAINTENANCE\...

3. canonical shared runtime
   C:\shared\sandbox-toolchains\ide\cursor\runtime\<version>\
```

The layers must not be collapsed.

In particular:

- the installer `.exe` is an **installation input**, not the runtime
- the boxed installer output is the **validated source** for materialization
- the versioned shared runtime is the **canonical runtime source**
- maintenance and project boxes consume local mirrors of that shared runtime
- a normal host installation must never become the runtime source
- a project box must never be used to install or update Cursor
- `VS_CODE_MAINTENANCE` must never be used to install or update Cursor

The authoritative bounded context for acquisition and installation is:

```text
CURSOR_MAINTENANCE
```

## Runtime ownership model

### What is separate

Cursor owns its own versioned executable tree:

```text
C:\shared\sandbox-toolchains\ide\cursor\runtime\<version>\
```

That tree includes, at minimum:

```text
Cursor.exe
resources\app\codeBin\code.cmd
resources\app\bin\cursor.cmd
resources\app\bin\code-tunnel.exe
resources\app\bin\cursor-tunnel.exe
tools\inno_updater.exe
```

Do not merge this tree into the VS Code runtime namespace.

### What remains shared

Cursor continues to consume the existing VSCode-family control plane:

```text
C:\shared\sandbox-toolchains\ide\vscode\catalog\vscode-user\
C:\shared\sandbox-toolchains\ide\vscode\catalog\seed\globalStorage\
C:\shared\sandbox-toolchains\ide\vscode\catalog\seed\roo\
C:\shared\sandbox-toolchains\ide\vscode\extensions\
C:\shared\sandbox-toolchains\dev\...
```

Updating Cursor must not create a second catalog, seed tree, extension store,
or Node/Git/pnpm toolchain.

### What is mirrored automatically

The bootstrap copies the canonical shared runtime into the box-local execution
tree. Do **not** manually copy Cursor into project or maintenance boxes.

The resulting local execution path is conceptually:

```text
C:\Program Files\SandboxToolchains\CursorBoxes\
  maintenance\
    execution\runtime\cursor\<version>\
  <project>\
    execution\runtime\cursor\<version>\
```

The local runtime mirrors deliberately exclude:

```text
resources\app\bin\code-tunnel.exe
resources\app\bin\cursor-tunnel.exe
tools\inno_updater.exe
```

Those files must remain in the canonical shared runtime. The exclusions apply
only to the local execution mirror.

## Before starting

### 1. Choose the target version

Use one version string consistently. This document uses `3.14.7` as an
example:

```powershell
$CursorVersion = '3.14.7'
```

Do not overwrite an existing runtime directory in place. A new Cursor version
gets its own directory:

```text
C:\shared\sandbox-toolchains\ide\cursor\runtime\3.14.7\
```

Versioned directories make validation and rollback explicit.

### 2. Close all affected boxes

Before updating:

1. close Cursor windows
2. terminate all processes in `CURSOR_MAINTENANCE`
3. terminate all processes in each `CURSOR_<PROJECT>` box
4. do not leave a Cursor update helper or tunnel process running

This avoids locked files and prevents a local mirror from being tested against
a partially materialized shared runtime.

### 3. Back up the control-plane files

Run this from a normal host PowerShell session before changing references:

```powershell
$CursorVersion = '3.14.7'
$BackupRoot = Join-Path $env:USERPROFILE "Desktop\cursor-runtime-update-$CursorVersion"

New-Item -ItemType Directory -Force -Path $BackupRoot | Out-Null

Copy-Item `
  -LiteralPath 'C:\Windows\Sandboxie.ini' `
  -Destination (Join-Path $BackupRoot 'Sandboxie.ini.before-update') `
  -Force

Copy-Item `
  -LiteralPath 'C:\shared\sandbox-toolchains\dev\bootstrap\platforms\cursor\Start-CursorMaintenance.ps1' `
  -Destination (Join-Path $BackupRoot 'Start-CursorMaintenance.ps1.before-update') `
  -Force

Copy-Item `
  -LiteralPath 'C:\shared\sandbox-toolchains\projects\your-project\bootstrap\Project.Config.ps1' `
  -Destination (Join-Path $BackupRoot 'Project.Config.ps1.before-update') `
  -Force
```

Replace `your-project` with the actual project directory. Keep an existing,
working versioned runtime until the new version passes all acceptance checks.

### 4. Verify the installer

The installer must be a valid Cursor/Anysphere installer before it enters the
box:

```powershell
$Installer = 'C:\shared\CursorUserSetup-x64-3.14.7.exe'

$Signature = Get-AuthenticodeSignature -FilePath $Installer
$VersionInfo = (Get-Item -LiteralPath $Installer).VersionInfo

[PSCustomObject]@{
  Path           = $Installer
  SignatureState = $Signature.Status
  Signer         = $Signature.SignerCertificate.Subject
  ProductVersion = $VersionInfo.ProductVersion
  FileVersion    = $VersionInfo.FileVersion
  SizeBytes      = (Get-Item -LiteralPath $Installer).Length
} | Format-List
```

Required result:

- `SignatureState` is `Valid`
- the signer is the expected Cursor/Anysphere publisher
- `ProductVersion` matches `$CursorVersion`

Do not execute an unsigned, invalidly signed, or version-mismatched installer.

## Phase 1: stage and install inside `CURSOR_MAINTENANCE`

### 5. Add a temporary, least-privilege installer read rule

The Maintenance Box must be able to read the installer, but it does not need
write access to the shared runtime.

Add this temporary line only in the `[CURSOR_MAINTENANCE]` section of
`C:\Windows\Sandboxie.ini`:

```ini
ReadFilePath=C:\shared\CursorUserSetup-x64-3.14.7.exe
```

Use the real installer name and version.

Do **not** add either of these broad permissions:

```ini
; Do not use these:
ReadFilePath=C:\shared\
OpenFilePath=C:\shared\
OpenFilePath=C:\shared\sandbox-toolchains\ide\cursor\runtime\
```

The shared runtime must remain a controlled, read-only source for normal box
consumption. The installer runs inside the box and writes into boxed state
first; the host materializes the validated result later.

Reload the Sandboxie configuration after saving the temporary rule.

### 6. Start the installer through the Cursor Maintenance Box

Do not use a normal host double-click. Do not use a VS Code box.

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:CURSOR_MAINTENANCE `
  "C:\shared\CursorUserSetup-x64-3.14.7.exe"
```

Use the normal per-user installer flow inside the box. Do not choose the
shared runtime directory as the installer destination.

The `Start-CursorMaintenance.ps1` wrapper is **not** the installer entrypoint.
It validates and mirrors an already-materialized shared runtime, so it cannot
bootstrap a missing runtime.

### 7. Locate the boxed installation result

With the current Sandboxie file-root convention, the physical boxed user
profile is typically:

```text
C:\Sandbox\yourusername\CURSOR_MAINTENANCE\user\current\
```

The expected Cursor application root is:

```text
C:\Sandbox\yourusername\CURSOR_MAINTENANCE\user\current\
  AppData\Local\Programs\cursor\
```

Do not assume that the physical Sandboxie file layout uses
`drive\C\Users\yourusername\...`; inspect the active `FileRootPath` and the
actual sandbox tree when troubleshooting.

Verify the complete runtime contract before copying anything:

```powershell
$BoxedRuntime = 'C:\Sandbox\yourusername\CURSOR_MAINTENANCE\user\current\AppData\Local\Programs\cursor'

$RequiredPaths = @(
  'Cursor.exe',
  'resources\app\codeBin\code.cmd',
  'resources\app\bin\cursor.cmd',
  'resources\app\bin\code-tunnel.exe',
  'resources\app\bin\cursor-tunnel.exe',
  'tools\inno_updater.exe'
)

$MissingPaths = $RequiredPaths | Where-Object {
  -not (Test-Path -LiteralPath (Join-Path $BoxedRuntime $_))
}

if ($MissingPaths) {
  throw ('Boxed Cursor runtime is incomplete: ' + ($MissingPaths -join ', '))
}

$BoxedVersion = ((Get-Item -LiteralPath (Join-Path $BoxedRuntime 'Cursor.exe')).VersionInfo.ProductVersion).Trim()
if ($BoxedVersion -ne '3.14.7') {
  throw "Unexpected boxed Cursor version. Expected 3.14.7, got $BoxedVersion."
}
```

If any required path is missing, stop. Do not combine files from a host
installation and a partial boxed installation to manufacture a runtime.

## Phase 2: materialize the canonical shared runtime

### 8. Copy the complete boxed application tree

Materialize the full application tree into a new versioned shared directory:

```powershell
$CursorVersion = '3.14.7'
$Source = 'C:\Sandbox\yourusername\CURSOR_MAINTENANCE\user\current\AppData\Local\Programs\cursor'
$Destination = "C:\shared\sandbox-toolchains\ide\cursor\runtime\$CursorVersion"

New-Item -ItemType Directory -Force -Path $Destination | Out-Null

robocopy $Source $Destination /E /COPY:DAT /DCOPY:DAT /R:1 /W:1

$RobocopyExitCode = $LASTEXITCODE
if ($RobocopyExitCode -ge 8) {
  throw "Cursor runtime materialization failed. Robocopy ExitCode: $RobocopyExitCode"
}
```

`robocopy` exit codes `0` through `7` are non-failure outcomes. Only `8` or
higher is a materialization failure.

Do not use `/MIR` for this operation. The destination is a new versioned
directory, and an unnecessary destructive mirror is not needed.

### 9. Verify the materialized runtime

```powershell
$RequiredPaths | ForEach-Object {
  $Path = Join-Path $Destination $_
  [PSCustomObject]@{
    Path   = $Path
    Exists = Test-Path -LiteralPath $Path
  }
} | Format-Table -AutoSize

$SharedVersion = ((Get-Item -LiteralPath (Join-Path $Destination 'Cursor.exe')).VersionInfo.ProductVersion).Trim()
if ($SharedVersion -ne $CursorVersion) {
  throw "Unexpected shared Cursor version. Expected $CursorVersion, got $SharedVersion."
}
```

Expected result:

- every required path exists
- `Cursor.exe` reports the selected version
- the shared runtime is complete before any bootstrap reference changes

The installer `.exe` remains an input artifact. It is not copied into the
runtime directory.

## Phase 3: switch all runtime references

### 10. Update the Cursor Maintenance wrapper default

In:

```text
C:\shared\sandbox-toolchains\dev\bootstrap\platforms\cursor\Start-CursorMaintenance.ps1
```

change the default:

```powershell
[string]$CursorVersion = '3.14.7'
```

This makes routine maintenance target the new shared runtime without requiring
a version parameter on every invocation.

### 11. Update each project Cursor contract

In each project:

```text
C:\shared\sandbox-toolchains\projects\your-project\bootstrap\Project.Config.ps1
```

update all Cursor runtime paths together:

```powershell
$cursorConfig = @{
  BoxName = 'CURSOR_YOUR_PROJECT'
  RuntimeNamespace = 'cursor'
  CodeExe = Join-Path $cursorRuntimeRoot 'runtime\3.14.7\Cursor.exe'
  CodeCli = Join-Path $cursorRuntimeRoot 'runtime\3.14.7\resources\app\codeBin\code.cmd'
  CursorCli = Join-Path $cursorRuntimeRoot 'runtime\3.14.7\resources\app\bin\cursor.cmd'
}
```

Do not update only `CodeExe`. The GUI, `code.cmd`, and `cursor.cmd` must all
belong to the same versioned runtime tree.

### 12. Update the Sandboxie runtime access rules

Change the versioned runtime references in both:

- `[CURSOR_MAINTENANCE]`
- `[CURSOR_YOUR_PROJECT]`

Use the new runtime version for the read and deny rules:

```ini
ReadFilePath=C:\shared\sandbox-toolchains\ide\cursor\runtime\3.14.7\

ClosedFilePath=C:\shared\sandbox-toolchains\ide\cursor\runtime\3.14.7\resources\app\bin\code-tunnel.exe
ClosedFilePath=C:\shared\sandbox-toolchains\ide\cursor\runtime\3.14.7\resources\app\bin\cursor-tunnel.exe
ClosedFilePath=C:\shared\sandbox-toolchains\ide\cursor\runtime\3.14.7\tools\inno_updater.exe
```

The `ReadFilePath` rule grants the shared runtime as an input to bootstrap.
The `ClosedFilePath` rules keep update/tunnel helpers from becoming normal
boxed execution paths.

Keep these rules version-specific. Do not widen them to the whole
`runtime\` parent directory unless the security policy is deliberately
changed and reviewed.

Do not add:

```ini
SpecialImage=chrome,Cursor.exe
```

The Cursor maintenance CLI must not inherit Chromium special-image behavior,
because that can inject unsupported flags into `code.cmd`.

### 13. Remove the temporary installer rule

After the shared runtime is validated, remove the temporary installer access:

```ini
ReadFilePath=C:\shared\CursorUserSetup-x64-3.14.7.exe
```

Reload the Sandboxie configuration after all runtime-reference changes are
saved.

## Phase 4: verify the updated system

### 14. Open the Cursor Maintenance terminal

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

The command must complete the shared-layout assertion and create a local
maintenance runtime mirror for the new version.

In the opened terminal, validate the version and resolved paths:

```powershell
$env:BOXED_EDITOR_RUNTIME_ROOT
$env:BOXED_EDITOR_EXE
$env:BOXED_EDITOR_CLI

& $env:BOXED_EDITOR_EXE --version
& $env:BOXED_EDITOR_CLI --version
```

The reported runtime root must contain `cursor\3.14.7`.

### 15. Launch the maintenance GUI

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

Verify:

1. Cursor starts inside the Maintenance Box.
2. Cursor reports the expected version.
3. The maintenance GUI does not start a host-owned runtime.
4. The shared catalog and extension surfaces are still consumed through the
   boxed bootstrap.

### 16. Validate the project box

Launch the project through its Cursor wrapper, not by executing `Cursor.exe`
directly:

```powershell
& "C:\shared\sandbox-toolchains\projects\your-project\bootstrap\Start-YourProjectCursorBoxed.ps1" `
  -Action LaunchCursor `
  -RepoPath "C:\Users\yourusername\source\your-project"
```

Verify:

1. the project box creates a local `cursor\3.14.7` runtime mirror
2. the repository opens in Cursor's classic editor window
3. the canonical shared catalog is mirrored into local `user-data`
4. the shared extension store is mirrored into local `extensions`
5. the project uses the governed Git, Node, pnpm, Python, shell, and Starship
   toolchain

### 17. Verify the integrated terminal

Open `Terminal -> New Terminal` in the boxed Cursor GUI.

The canonical shared VSCode-family settings should contain:

```json
"terminal.integrated.windowsUseConptyDll": true
```

This is the validated mitigation for the Cursor-specific `Cannot launch
conpty` failure class. See
`docs\applications\IDE\cursor\architectures\boxed-owned-toolchain\troubleshooting\integrated-terminal-conpty.md`
for its scope and limits.

### 18. Verify sign-in only when required

Sign-in is a separate acceptance criterion from runtime materialization.

For the documented environment, the validated login fix was:

```ini
ProtectHostImages=n
```

This weakens host-image protection. Apply it only to the relevant Cursor boxes,
only when sign-in is required, and only after accepting that security tradeoff.

See:

```text
docs\applications\IDE\cursor\architectures\boxed-owned-toolchain\sign-in.md
```

Do not treat browser, protocol, named-pipe, or window-class exceptions as
substitutes for that documented finding.

## Acceptance checklist

The runtime update is complete only when all of the following are true:

- [ ] installer signature and version were verified
- [ ] installer ran through `CURSOR_MAINTENANCE`
- [ ] boxed runtime contains all required files
- [ ] boxed `Cursor.exe` has the target version
- [ ] complete boxed application tree was materialized into a new shared,
      versioned runtime directory
- [ ] shared `Cursor.exe`, `code.cmd`, and `cursor.cmd` all resolve below the
      same version directory
- [ ] maintenance wrapper default points to the target version
- [ ] every project `Project.Config.ps1` points to the target version
- [ ] Cursor Maintenance and Cursor Project Sandboxie rules point to the
      target version
- [ ] temporary installer `ReadFilePath` access was removed
- [ ] the Maintenance Box creates a local runtime mirror and starts Cursor
- [ ] the project box creates a local runtime mirror and opens the repository
- [ ] the integrated terminal remains usable
- [ ] sign-in was tested separately if it is required

## Rollback

Rollback is safe only when the previous versioned shared runtime was retained
and passed its own validation.

To roll back:

1. stop all Cursor boxes
2. restore the previous version in:
   - `Start-CursorMaintenance.ps1`
   - each project `Project.Config.ps1`
   - both Cursor Sandboxie sections
3. reload Sandboxie configuration
4. start the Maintenance Box and project box again
5. verify that their local mirrors resolve to the previous version

Do not use an incomplete runtime directory as a rollback target. If the old
runtime lacked `Cursor.exe`, `code.cmd`, or another required path, it was not
a valid runtime and cannot provide a rollback guarantee.

## Related

- `docs\applications\IDE\cursor\methods\boxed-owned-toolchain\provisioning\runtime-installation.md`
- `docs\applications\IDE\cursor\methods\boxed-owned-toolchain\boxes\maintenance-box.md`
- `docs\applications\IDE\cursor\methods\boxed-owned-toolchain\bootstrap\scripts.md`
- `docs\applications\IDE\cursor\architectures\boxed-owned-toolchain\sign-in.md`
- `docs\applications\IDE\cursor\architectures\boxed-owned-toolchain\troubleshooting\integrated-terminal-conpty.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\shared-layout.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\provisioning\shared-artifacts.md`
