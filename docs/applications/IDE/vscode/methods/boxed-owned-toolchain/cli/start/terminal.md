# Terminal Overlay for the VS Code Method

## Single source of truth

The generic boxed-terminal guidance is centralized here:

- `docs\cli\terminal\general.md`

This document keeps only the VS Code-specific inner commands and validation steps for the boxed-owned-toolchain method.

## Maintenance terminal

After opening a boxed maintenance terminal according to `docs\cli\terminal\general.md`, use it for shared IDE maintenance tasks.

The current maintenance script is:

- `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode\Start-VSCodeMaintenance.ps1`

### Install an extension

```powershell
& "C:\shared\sandbox-toolchains\ide\vscode\runtime\1.121.0\bin\code.cmd" `
  --user-data-dir "C:\shared\sandbox-toolchains\ide\vscode\maintenance\user-data" `
  --extensions-dir "C:\shared\sandbox-toolchains\ide\vscode\extensions" `
  --install-extension RooVeterinaryInc.roo-cline
```

### Verify success

```powershell
$LASTEXITCODE
```

### List installed extensions

```powershell
& "C:\shared\sandbox-toolchains\ide\vscode\runtime\1.121.0\bin\code.cmd" `
  --user-data-dir "C:\shared\sandbox-toolchains\ide\vscode\maintenance\user-data" `
  --extensions-dir "C:\shared\sandbox-toolchains\ide\vscode\extensions" `
  --list-extensions
```

## Project terminal

After opening a boxed project terminal according to `docs\cli\terminal\general.md`, use it to validate the project box bootstrap and toolchain visibility.

The current sanitized example project terminal script is:

- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoTerminal.ps1`

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
