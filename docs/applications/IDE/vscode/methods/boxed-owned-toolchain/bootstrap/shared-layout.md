# Shared Bootstrap Layout

## Why this document exists

This document is the VS Code method-specific single source of truth for the shared bootstrap tree that currently lives under:

```text
C:\shared\sandbox-toolchains\
```

From a domain-driven and 12-factor perspective, this location and split are intentional:

- reusable bootstrap primitives belong in shared infrastructure
- shared editor-family orchestration belongs in a VSCode-family platform layer
- thin editor-specific wrappers belong in editor-specific platform layers
- Node-specific runtime wiring belongs in a Node stack layer
- shell-specific runtime wiring belongs in a shell stack layer
- prompt/runtime wiring can live in a dedicated stack layer when it must be mirrored and initialized explicitly
- project-specific launchers belong under the specific project subtree

This keeps the implementation split by responsibility instead of by accidental current usage.

## Global references

Generic CLI semantics remain centralized outside the VS Code method:

- `docs\cli\general.md`
- `docs\cli\start\general.md`
- `docs\cli\terminal\general.md`

This document covers only the boxed-owned-toolchain bootstrap implementation and its file layout.

## Shared tree focused on bootstrap-relevant areas

```text
C:\shared\sandbox-toolchains\
  ide\
    vscode\
      runtime\
        1.121.0\
      catalog\
        vscode-user\
          settings.json
          keybindings.json
          snippets\
        seed\
          globalStorage\
          roo\
      extensions\
      maintenance\
        user-data\
    cursor\
      runtime\
        3.9.16\
  dev\
    git\
      2.54.0\
    node\
      26.2.0\
        node-v26.2.0-win-x64\
      20.9.0\
        node-v20.9.0-win-x64\
    pnpm\
      11.7.0\
        package\
          bin\
            pnpm.cjs
    shells\
      cmd\
        10.0.26100.8457\
          cmd.exe
      powershell\
        10.0.26100.8457\
          powershell.exe
      reg\
        10.0.26100.8457\
          reg.exe
      clink\
        1.9.26\
          clink_x64.exe
          clink.bat
      vs-installer\
        3.1.7\
          vswhere.exe
      visual-studio\
        2022\
          BuildTools\
      windows-kits\
        10\
      dotnet-framework\
        Framework\
          v4.0.30319\
        Framework64\
          v4.0.30319\
    starship\
      1.25.1\
        starship.exe
    vscode-extensions\
      vsix\
        CyberT33N\
    bootstrap\
      core\
        Bootstrap.Common.psm1
      platforms\
        vscode-family\
          Bootstrap.VSCodeFamily.psm1
          Start-VSCodeFamilyProjectBase.ps1
          Start-VSCodeFamilyMaintenance.ps1
          Publish-VSCodeFamilyMaintenance.ps1
        vscode\
          Bootstrap.VSCode.psm1
          Start-VSCodeMaintenance.ps1
          Start-VSCodeProjectBase.ps1
          Publish-VSCodeMaintenance.ps1
        cursor\
          Bootstrap.Cursor.psm1
          Start-CursorMaintenance.ps1
          Start-CursorProjectBase.ps1
          Publish-CursorMaintenance.ps1
      stacks\
        microsoft-build\
          Bootstrap.MicrosoftBuild.psm1
        dotnet-framework\
          Bootstrap.DotNetFramework.psm1
        node\
          Bootstrap.Node.psm1
        shells\
          Bootstrap.WindowsShells.psm1
        python\
          Bootstrap.Python.psm1
        starship\
          Bootstrap.Starship.psm1
  projects\
    test-mono\
      bootstrap\
        Project.Config.ps1
        Start-TestMonoEditor.ps1
        Start-TestMonoVSCode.ps1
        Start-TestMonoCursor.ps1
        Start-TestMonoTerminal.ps1
        Start-TestMonoElectronTerminal.ps1
        Start-TestMonoCursorTerminal.ps1
        Start-TestMonoCursorElectronTerminal.ps1
        Start-TestMonoPnpmInstall.ps1
        Start-TestMonoPnpmCleanReinstall.ps1
        Start-TestMonoPnpmUninstall.ps1
        Start-TestMonoElectronPostInstall.ps1
        Start-TestMonoElectronSpawnReplay.ps1
      export\
      runner-input\
```

## Why this split is architecturally correct

### `dev\bootstrap\core\`

This is the bootstrap infrastructure kernel.

It contains only generic primitives that can be reused by future VS Code, Python, runner, or utility flows:

- path existence checks
- directory creation
- file copy helpers
- tree initialization
- tree mirroring
- ASCII file generation
- `PATH` prefix composition

The current file is:

- `C:\shared\sandbox-toolchains\dev\bootstrap\core\Bootstrap.Common.psm1`

### `dev\bootstrap\platforms\vscode-family\`

This is the shared editor-family adapter for the current two-editor architecture.

It contains the orchestration that is shared across `VS Code` and `Cursor`:

- assert the editor runtime layout
- compute box-local family paths
- copy the canonical user catalog into `user-data`
- mirror the shared extension store into the box-local `extensions` directory
- initialize seed-backed paths like `globalStorage` and `.roo`
- run the family CLI path for maintenance actions
- run the direct editor GUI path for project launch

The current files are:

- `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode-family\Bootstrap.VSCodeFamily.psm1`
- `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode-family\Start-VSCodeFamilyMaintenance.ps1`
- `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode-family\Start-VSCodeFamilyProjectBase.ps1`
- `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode-family\Publish-VSCodeFamilyMaintenance.ps1`

### `dev\bootstrap\platforms\vscode\`

This is now the thin VS Code compatibility wrapper layer.

It binds the shared family kernel to the VS Code runtime namespace and preserves the no-breaking public VS Code command surface.

### `dev\bootstrap\platforms\cursor\`

This is the thin Cursor platform wrapper layer.

It binds the shared family kernel to the Cursor runtime namespace and adds the Cursor-specific runtime exclusions and launch semantics.

### `dev\bootstrap\stacks\node\`

This is the Node stack adapter.

It contains Node / pnpm / Git runtime wiring that is reusable across multiple Node-based project boxes:

- assert shared `Git`
- assert primary shared `Node`
- assert shared `pnpm.cjs`
- generate box-local wrapper commands such as `pnpm.cmd`
- generate additional node aliases such as `node20.cmd`
- prepend the relevant toolchain paths into `PATH`

The current file is:

- `C:\shared\sandbox-toolchains\dev\bootstrap\stacks\node\Bootstrap.Node.psm1`

### `dev\bootstrap\stacks\shells\`

This is the shell runtime adapter layer.

It contains shell-specific runtime support that is not owned by the generic Node, Python, or Starship stacks:

- mirror governed shared `cmd.exe` locally into the boxed runtime
- mirror governed shared Windows PowerShell locally into the boxed runtime
- mirror governed shared `Clink` locally when it is provisioned
- generate PowerShell init files for minimal and Starship-enabled sessions
- generate CMD init files for minimal and Starship-enabled sessions
- publish shell-specific environment surfaces such as:
  - `BOXED_CMD_EXE`
  - `BOXED_POWERSHELL_EXE`
  - `BOXED_CLINK_EXE`
  - `BOXED_CMD_STARSHIP_PROFILE`

The current file is:

- `C:\shared\sandbox-toolchains\dev\bootstrap\stacks\shells\Bootstrap.WindowsShells.psm1`

Important current nuance:

- `cmd.exe`, Windows PowerShell, and `Clink` are all now governed shared shell artifacts under `dev\shells\...`
- bootstrap still mirrors them locally into the boxed execution tree
- mutable `Clink` profile/state still remains box-local under `state\shells\cmd\clink\profile\`

### `dev\bootstrap\stacks\python\`

This is the Python stack adapter.

It contains Python runtime mirroring and PATH wiring that can be reused by project or maintenance boxes.

The current file is:

- `C:\shared\sandbox-toolchains\dev\bootstrap\stacks\python\Bootstrap.Python.psm1`

### `dev\bootstrap\stacks\starship\`

This is the Starship prompt/runtime adapter.

It contains prompt-specific runtime support when `Starship` must be mirrored locally into the box execution tree:

- detect whether shared Starship is provisioned
- mirror the shared Starship runtime locally
- prepend the local Starship directory into `PATH`
- generate `bash.minimal.rc`
- generate `bash.starship.rc`

The current file is:

- `C:\shared\sandbox-toolchains\dev\bootstrap\stacks\starship\Bootstrap.Starship.psm1`

### `dev\vscode-extensions\vsix\`

This is the shared source-artifact staging lane for local forked VS Code extension packages.

It is **not** the canonical published extension store.

It exists so locally built VSIX artifacts can be copied into the shared `dev` area first and then installed through the Maintenance Box before the resulting maintenance state is promoted into:

- `C:\shared\sandbox-toolchains\ide\vscode\extensions\`

### `projects\<project>\bootstrap\`

This is the project adapter layer.

It contains only what is specific to one project:

- the project name
- the default repo path
- the exact editor runtime bindings used by the project
- the exact shared toolchain versions used by the project
- any project-specific secondary runtime aliases
- the shared project editor core
- thin host entry points for editor-specific GUI launch and terminal launch
- project-owned install / uninstall / repair wrappers that select the editor without duplicating the whole project bootstrap

Using the sanitized boilerplate project name `test-mono`, the files are:

- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Project.Config.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoEditor.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoVSCode.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoCursor.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoTerminal.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoElectronTerminal.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoCursorTerminal.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoCursorElectronTerminal.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoPnpmInstall.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoPnpmCleanReinstall.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoPnpmUninstall.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoElectronPostInstall.ps1`

## What the current file families do

### Core

`Bootstrap.Common.psm1` still owns the reusable filesystem/bootstrap helpers:

- path assertions
- directory creation
- tree mirroring
- bootstrap-owned ASCII file generation
- `PATH` prefix composition

### Shared family platform

The `vscode-family` layer now owns the shared editor-family runtime behavior:

- family layout assertions
- local runtime mirroring
- shared catalog sync
- shared extension-store mirroring
- seed initialization
- family maintenance publish flow
- family project launch flow

### Thin editor wrappers

The `vscode` and `cursor` layers now stay intentionally thin.

They bind the family kernel to:

- the VS Code runtime namespace
- or the Cursor runtime namespace

without duplicating the shared catalog/toolchain/bootstrap behavior.

### Toolchain and native-build stacks

The stack layers now include:

- `node`
- `microsoft-build`
- `dotnet-framework`
- `shells`
- `python`
- `starship`

That reflects the real current boxed-owned-toolchain contract, where the project bootstrap can prepare:

- Git / Node / pnpm
- Windows shell lanes
- Microsoft build-source projection
- `.NET Framework` compiler projection
- optional Python
- optional Starship

### Project adapter layer

`Project.Config.ps1` is now the real editor-split contract.

It keeps:

- one `VSCode` block
- one `Cursor` block
- one shared `Toolchain` block
- one shared `MicrosoftBuild` block
- one shared `Nx` block
- one shared `Shells` block

`Start-TestMonoEditor.ps1` is the shared project-level editor core.

It chooses:

- which editor was requested
- which platform base script receives the call
- whether the terminal intent is generic or Electron-specific

The remaining project wrappers are then deliberately thin:

- `Start-TestMonoVSCode.ps1`
- `Start-TestMonoCursor.ps1`
- `Start-TestMonoTerminal.ps1`
- `Start-TestMonoElectronTerminal.ps1`
- `Start-TestMonoCursorTerminal.ps1`
- `Start-TestMonoCursorElectronTerminal.ps1`

The non-launch project scripts also stay shared and editor-selectable instead of duplicated:

- `Start-TestMonoPnpmInstall.ps1`
- `Start-TestMonoPnpmCleanReinstall.ps1`
- `Start-TestMonoPnpmUninstall.ps1`
- `Start-TestMonoElectronPostInstall.ps1`

### Runtime contract

The current runtime contract is now:

- shared VSCode-family catalog, seeds, and extension store
- separate `VS Code` and `Cursor` runtime namespaces
- local mirrored runtime execution
- local maintenance authoring state
- explicit publish/promotion
- one shared project bootstrap core with thin editor wrappers
- shared dependency-management and repair scripts that select the editor through parameters instead of duplicating the whole project implementation

## Runtime contract

The current runtime contract is:

- Maintenance Box is the only publisher of shared IDE changes
- Maintenance authoring state is local first
- Project Box is a consumer
- Shared extensions are mirrored into a box-local runtime copy
- project `user-data` is box-local
- shared settings are copied into the local `user-data`
- `.roo` and `globalStorage` are initialized from seeds only when missing
- prompt/runtime helper files such as Bash RC files are generated in `bootstrap-bin`
- Git Bash command surfaces such as `pnpm` are exposed through shell-native wrappers in `bootstrap-bin`, not only through `.cmd` files
- CMD-specific prompt injection state lives box-locally under:
  - `state\shells\cmd\clink\profile\`
- governed shared shell artifacts remain provisioned under:
  - `dev\shells\cmd\...`
  - `dev\shells\powershell\...`
  - `dev\shells\clink\...`

This preserves the architecture contract:

- no live multi-writer extension store
- no project-box writes into the shared canonical extension path
- reproducible local runtime state per project box

## Related

- `docs\applications\IDE\cursor\methods\boxed-owned-toolchain\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\scripts.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\provisioning\shared-artifacts.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\start.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\scripts.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\general.md`
