# Cursor Bootstrap Wrappers

## Scope

This document records the Cursor-specific wrapper layer for the boxed-owned-toolchain method.

It does **not** replace the canonical shared bootstrap source of truth under:

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\shared-layout.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\scripts.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\scripts.md`

This file keeps only the Cursor-specific wrapper surfaces and the way they bind into the shared family kernel.

## Current wrapper inventory

### Shared family kernel

The shared kernel that both editors use is:

- `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode-family\Bootstrap.VSCodeFamily.psm1`
- `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode-family\Start-VSCodeFamilyProjectBase.ps1`
- `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode-family\Start-VSCodeFamilyMaintenance.ps1`
- `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode-family\Publish-VSCodeFamilyMaintenance.ps1`

### Cursor platform wrappers

The Cursor-specific wrapper layer is:

- `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\cursor\Bootstrap.Cursor.psm1`
- `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\cursor\Start-CursorProjectBase.ps1`
- `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\cursor\Start-CursorMaintenance.ps1`
- `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\cursor\Publish-CursorMaintenance.ps1`

### Current project wrappers

For the current project, the Cursor-facing wrappers are:

- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Project.Config.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-testMonoEditorBoxed.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-testMonoCursorBoxed.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-testMonoEditor.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-testMonoCursor.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-testMonoCursorTerminal.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-testMonoCursorElectronTerminal.ps1`

### Shared project scripts with editor selection

The current project also keeps shared host-facing dependency wrappers and
in-box non-launch scripts that accept `-Editor Cursor`:

- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-testMonoPnpmInstallBoxed.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-testMonoPnpmCleanReinstallBoxed.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-testMonoPnpmUninstallBoxed.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-testMonoPnpmInstall.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-testMonoPnpmCleanReinstall.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-testMonoPnpmUninstall.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-testMonoElectronPostInstall.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-testMonoElectronSpawnReplay.ps1`

## Responsibility split

## `Bootstrap.Cursor.psm1`

This is the Cursor-specific adapter layer.

It does **not** reimplement the family logic.

Instead, it:

- reuses the VSCode-family layout assertions
- reuses the VSCode-family catalog sync
- reuses the VSCode-family seed sync
- reuses the VSCode-family extension mirror
- reuses the VSCode-family maintenance publish flow
- applies the Cursor runtime exclusions

The validated current runtime exclusions are:

- `resources\app\bin\code-tunnel.exe`
- `resources\app\bin\cursor-tunnel.exe`
- `tools\inno_updater.exe`

The Cursor platform wrappers also opt into the boxed PowerShell canonical-path
compatibility projection. This is required because Cursor Agent Shell can
hard-code:

```text
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
```

even when the local boxed PowerShell profile has been detected correctly.
Before Cursor launches, the shared family bootstrap projects the local governed
PowerShell mirror into that virtual Windows path inside the box. This preserves
the strict host-image boundary instead of allowing Cursor to spawn host
PowerShell.

The full failure record, required Sandboxie options, and verification contract
live here:

- `docs\applications\IDE\cursor\architectures\boxed-owned-toolchain\troubleshooting\agent-shell-host-powershell-projection.md`

## `Start-CursorProjectBase.ps1`

This is a thin project-wrapper layer over:

- `Start-VSCodeFamilyProjectBase.ps1`

It fixes the Cursor-specific contract:

- `EditorId = Cursor`
- `EditorDisplayName = Cursor`
- `BoxFamilyName = CursorBoxes`
- `RuntimeNamespace = cursor`

and forwards the shared toolchain/project contract into the family kernel.

The complete forwarding call includes the Cursor-only compatibility projection
root:

```powershell
& $familyScript `
  -Action $familyAction `
  -EditorId 'Cursor' `
  -EditorDisplayName 'Cursor' `
  -BoxFamilyName 'CursorBoxes' `
  -RuntimeNamespace 'cursor' `
  -ProjectName $ProjectName `
  -RepoPath $RepoPath `
  -CodeExe $CodeExe `
  -CodeCli $CodeCli `
  -CatalogUserRoot $CatalogUserRoot `
  -SharedExtensionsRoot $SharedExtensionsRoot `
  -SeedGlobalStorageRoot $SeedGlobalStorageRoot `
  -SeedRooRoot $SeedRooRoot `
  -GitRoot $GitRoot `
  -NodeRoot $NodeRoot `
  -PnpmCli $PnpmCli `
  -PythonRoot $PythonRoot `
  -VsWhereExe $VsWhereExe `
  -VisualStudioRoot $VisualStudioRoot `
  -WindowsSdkRoot $WindowsSdkRoot `
  -DotNetFrameworkRoot $DotNetFrameworkRoot `
  -DotNetFramework64Root $DotNetFramework64Root `
  -CmdRoot $CmdRoot `
  -PowerShellRoot $PowerShellRoot `
  -WindowsPowerShellCompatibilityProjectionRoot 'C:\Windows\System32\WindowsPowerShell\v1.0' `
  -RegRoot $RegRoot `
  -StarshipRoot $StarshipRoot `
  -StarshipConfigPath $StarshipConfigPath `
  -ClinkRoot $ClinkRoot `
  -NxDaemonBootstrapMode $NxDaemonBootstrapMode `
  -RuntimeCtlShimExe $RuntimeCtlShimExe `
  -McpFilesystemExtended $McpFilesystemExtended `
  -Ugrep $Ugrep `
  -RuntimeExcludeRelativePaths @(
    'resources\app\bin\code-tunnel.exe',
    'resources\app\bin\cursor-tunnel.exe',
    'tools\inno_updater.exe'
  ) `
  -AdditionalNodeCommands $AdditionalNodeCommands
```

## `Start-CursorMaintenance.ps1`

This is the maintenance-wrapper equivalent over:

- `Start-VSCodeFamilyMaintenance.ps1`

It fixes the same Cursor-specific identity values and the same runtime exclusions while still targeting the shared catalog, seed, and extension surfaces.

Its complete family invocation uses the same projection root:

```powershell
& $familyScript `
  -Action $familyAction `
  -ExtensionId $ExtensionId `
  -EditorId 'Cursor' `
  -EditorDisplayName 'Cursor' `
  -BoxFamilyName 'CursorBoxes' `
  -RuntimeNamespace 'cursor' `
  -CodeExe $codeExe `
  -CodeCli $codeCli `
  -CatalogUserRoot $catalogUserRoot `
  -SharedSeedGlobalStorageRoot $sharedSeedGlobalStorageRoot `
  -SharedSeedRooRoot $sharedSeedRooRoot `
  -SharedExtensionsRoot $sharedExtensionsRoot `
  -GitRoot $gitRoot `
  -NodeRoot $nodeRoot `
  -PnpmCli $pnpmCli `
  -PythonRoot $pythonRoot `
  -VsWhereExe $vswhereExe `
  -VisualStudioRoot $visualStudioRoot `
  -WindowsSdkRoot $windowsSdkRoot `
  -DotNetFrameworkRoot $dotNetFrameworkRoot `
  -DotNetFramework64Root $dotNetFramework64Root `
  -CmdRoot $cmdRoot `
  -PowerShellRoot $powerShellRoot `
  -WindowsPowerShellCompatibilityProjectionRoot 'C:\Windows\System32\WindowsPowerShell\v1.0' `
  -RegRoot $regRoot `
  -StarshipRoot $starshipRoot `
  -ClinkRoot $clinkRoot `
  -StarshipConfigPath $starshipConfigPath `
  -PromotionScript $promotionScript `
  -RuntimeExcludeRelativePaths @(
    'resources\app\bin\code-tunnel.exe',
    'resources\app\bin\cursor-tunnel.exe',
    'tools\inno_updater.exe'
  ) `
  -AdditionalNodeCommands $additionalNodeCommands
```

## `Project.Config.ps1`

The current project config is now the real editor split point.

It keeps:

- one `VSCode` block
- one `Cursor` block
- one shared `Toolchain` block
- one shared `MicrosoftBuild` block
- one shared `Nx` block
- one shared `Shells` block

That is the current single-source-of-truth project contract.

## `Start-testMonoEditor.ps1`

This is the shared project-level editor core.

It resolves:

- which editor was requested
- which base platform script should receive the call
- whether the terminal intent is generic or Electron-specific
- which shared toolchain and shell contract should be projected

So the current project architecture is:

- one shared project core
- thin editor wrappers on top

not:

- two separate duplicated project bootstrap implementations

## Current host-facing Cursor commands

### Launch the Cursor GUI for the project box

```powershell
& "C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoCursorBoxed.ps1" `
  -Action LaunchCursor `
  -RepoPath "C:\Users\denni\source\test-mono"
```

### Open the generic Cursor project terminal

```powershell
& "C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoCursorBoxed.ps1" `
  -Action OpenTerminal `
  -RepoPath "C:\Users\denni\source\test-mono"
```

### Open the Cursor Electron terminal

```powershell
& "C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoCursorBoxed.ps1" `
  -Action OpenTerminal `
  -OpenTerminalIntent ElectronServe `
  -RepoPath "C:\Users\denni\source\test-mono"
```

### Open the Cursor maintenance terminal

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

### Launch the Cursor maintenance GUI

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

## Important current launch nuance

The shared family project kernel now forces the classic editor-window contract for Cursor project launches.

That means:

- Cursor repo-open is still a direct GUI launch path
- but explicit project launches add the classic editor-window flags

The product-level reasoning for that lives here:

- `docs\applications\IDE\cursor\general.md`

## Host entry boundary

After the boxed `.NET Framework` projection exists, normal Cursor project
launches and terminals must use the project host wrapper rather than starting
host Windows PowerShell directly through `Start.exe`.

The shared host-wrapper contract lives here:

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\host-entry-wrappers.md`

## What stays canonical elsewhere

This file does **not** become the new source of truth for:

- shared family catalog and extension governance
- shared toolchain contract
- full sanitized boilerplate script bodies

Those stay canonical in the VS Code method area and are re-referenced from here.

## Related

- `docs\applications\IDE\cursor\methods\boxed-owned-toolchain\general.md`
- `docs\applications\IDE\cursor\methods\boxed-owned-toolchain\architecture\target-state.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\scripts.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\host-entry-wrappers.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\scripts.md`
