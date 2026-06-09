# `Start.exe` in the VS Code Method

## Single source of truth

The generic `Start.exe` launch behavior is documented here:

- `docs\cli\start\general.md`

This document keeps only the VS Code-specific overlay for the boxed-owned-toolchain method.

## Maintenance Box

The Maintenance Box is the control-plane box for shared IDE assets.

Its responsibilities are broader than "install one extension":

- install, update, remove, and list shared extensions
- validate the shared extension store
- validate the shared VS Code runtime
- maintain the canonical VS Code user-data seed
- maintain seed material for shared `globalStorage`
- maintain seed material for `.roo`

Architecturally, this means:

- CLI-first operation is the default
- GUI launch is optional and secondary
- the Maintenance Box is not the normal daily authoring environment

## Project Box

The Project Box is the normal authoring plane for one project.

This is the place where the method should launch the VS Code GUI against the boxed repo and use the bootstrap-provided toolchain in the integrated terminal.

## Recommended wrapper contract

The tested behavior shows that a generic free-form passthrough such as:

```powershell
-CodeArgs --new-window
```

or:

```powershell
-CodeArgs --install-extension RooVeterinaryInc.roo-cline
```

is too fragile across multiple parsing layers.

For this VS Code method, the preferred bootstrap contract is:

- explicit action names
- explicit payload parameters

Recommended actions:

- `-Action LaunchVSCode`
- `-Action InstallExtension`
- `-Action ListExtensions`
- `-Action OpenTerminal`

Recommended payload parameters:

- `-ProjectName`
- `-RepoPath`
- `-ExtensionId`

## Recommended project start contract

The preferred target shape is:

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:VS_CODE_TEST_MONO `
  "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
  -NoLogo `
  -NoExit `
  -ExecutionPolicy Bypass `
  -File "C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoVSCode.ps1" `
  -Action LaunchVSCode `
  -RepoPath "C:\Users\yourusername\source\test-mono"
```

## Recommended project terminal contract

For terminal-only workflows, use the same project bootstrap with `-Action OpenTerminal`.

This keeps the launch path auditable:

- `Start.exe`
- normal Windows `powershell.exe`
- project bootstrap
- explicit `-Action OpenTerminal`

It does **not** require project-specific shell copies or shared terminal binaries.

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:VS_CODE_TEST_MONO `
  "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
  -NoLogo `
  -NoExit `
  -ExecutionPolicy Bypass `
  -File "C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoVSCode.ps1" `
  -Action OpenTerminal `
  -RepoPath "C:\Users\yourusername\source\test-mono"
```

Private Git authentication, helper selection, device-code sign-in, and the initial boxed clone before the repo exists are documented here:

- `docs\applications\git\architectures\boxed-owned-toolchain\overview.md`

## Recommended maintenance action contract

The preferred target shape is:

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

For list-only validation, use the same wrapper contract with `-Action ListExtensions`:

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

These host-driven maintenance actions are the preferred standard workflow.
An already-open maintenance terminal remains useful for troubleshooting, but routine extension installation and extension listing should normally be invoked through the explicit wrapper actions above.

## Host-driven dependency install pattern

For the boxed-owned-toolchain method, the preferred governance is:

- keep the selected PNPM version in the project contract
- keep the install flow in a project-owned PS1
- do **not** make the host launch command itself choose the PNPM version

The project-owned install PS1 is the preferred shape because it preserves the deterministic project/box contract and avoids smuggling toolchain-version choices into host launch parameters.

For the PNPM-domain source of truth for install-script ownership, read:

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\install.md`

For the PNPM-domain source of truth for clean reinstall flows, read:

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\clean-reinstall.md`

For the PNPM-domain source of truth for provisioning or updating the governed PNPM binary before the project contract changes, read:

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\versioning-and-provisioning.md`

## Example boilerplate launchers

Using the sanitized example project name `test-mono`, the concrete launchers are:

- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoVSCode.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoTerminal.ps1` (optional thin convenience wrapper)
- `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode\Start-VSCodeMaintenance.ps1`

For the full shared bootstrap tree and the sanitized boilerplate start flow, read:

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\shared-layout.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\scripts.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\start.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\scripts.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\install.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\clean-reinstall.md`

## Terminal-specific guidance

For generic boxed-terminal behavior, read:

- `docs\cli\terminal\general.md`

For the VS Code-specific terminal overlay, read:

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\cli\start\terminal.md`
