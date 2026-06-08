# Terminal Overlay for the VS Code Method

## Single source of truth

The generic boxed-terminal guidance is centralized here:

- `docs\cli\terminal\general.md`

This document keeps only the VS Code-specific inner commands and validation steps for the boxed-owned-toolchain method.

## Maintenance terminal

The maintenance terminal is a valid boxed shell for diagnosis, but it is **not** the preferred day-to-day path for routine extension maintenance.

The preferred operational path is:

- host starts the maintenance wrapper through `Start.exe`
- the wrapper receives an explicit action such as `InstallExtension` or `ListExtensions`
- the wrapper runs the correct inner `code.cmd` flow inside the maintenance box

The current maintenance script is:

- `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode\Start-VSCodeMaintenance.ps1`

### Preferred: install an extension from the host

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:VS_CODE_MAINTENANCE `
  "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
  -NoLogo `
  -NoExit `
  -ExecutionPolicy Bypass `
  -File "C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode\Start-VSCodeMaintenance.ps1" `
  -Action InstallExtension `
  -ExtensionId "RooVeterinaryInc.roo-cline"
```

### Preferred: list extensions from the host

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:VS_CODE_MAINTENANCE `
  "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
  -NoLogo `
  -NoExit `
  -ExecutionPolicy Bypass `
  -File "C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode\Start-VSCodeMaintenance.ps1" `
  -Action ListExtensions
```

### Diagnostic fallback: run `code.cmd` manually inside an already-open maintenance terminal

Use this only when you are debugging wrapper behavior, quoting, or bootstrap state.

```powershell
& $env:BOXED_CODE_CLI `
  --user-data-dir $env:BOXED_VSCODE_USERDATA `
  --extensions-dir $env:BOXED_LOCAL_EXTENSIONS `
  --list-extensions
```

This fallback intentionally uses the already-initialized local maintenance state, not the old shared live-writer path.

### Verify success

```powershell
$LASTEXITCODE
```

## Project terminal

After opening a boxed project terminal according to `docs\cli\terminal\general.md`, use it to validate the project box bootstrap and toolchain visibility.

The preferred sanitized example project terminal entrypoint is:

- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoVSCode.ps1 -Action OpenTerminal`

This keeps the launch path on:

- `Start.exe`
- a bootstrap-selected locally mirrored shell runtime
- bootstrap-selected toolchain wiring

and avoids project-specific shell copies.

### Verify the project toolchain

```powershell
git --version
node --version
pnpm --version
node20 --version
```

### Why these checks matter

These checks confirm that the project box inherited the intended bootstrap-provided toolchain contract rather than silently falling back to unrelated host tooling.

## Related

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\cli\start\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\shared-layout.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\scripts.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\start.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\scripts.md`
- `docs\applications\operating-systems\windows\terminal\powershell\general.md`
