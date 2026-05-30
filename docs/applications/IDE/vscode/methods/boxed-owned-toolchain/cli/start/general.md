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
  -ExecutionPolicy Bypass `
  -File "C:\shared\sandbox-toolchains\dev\bootstrap\Start-BoxedVSCode.ps1" `
  -Mode Project `
  -Action LaunchVSCode `
  -ProjectName "test-mono" `
  -RepoPath "C:\Users\denni\source\test-mono"
```

## Recommended maintenance action contract

The preferred target shape is:

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:VS_CODE_MAINTENANCE `
  "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
  -NoLogo `
  -ExecutionPolicy Bypass `
  -File "C:\shared\sandbox-toolchains\dev\bootstrap\Start-BoxedVSCode.ps1" `
  -Mode Maintenance `
  -Action InstallExtension `
  -ExtensionId "RooVeterinaryInc.roo-cline"
```

## Terminal-specific guidance

For generic boxed-terminal behavior, read:

- `docs\cli\terminal\general.md`

For the VS Code-specific terminal overlay, read:

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\cli\start\terminal.md`
