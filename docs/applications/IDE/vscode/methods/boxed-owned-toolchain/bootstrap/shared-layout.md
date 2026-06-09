# Shared Bootstrap Layout

## Why this document exists

This document is the VS Code method-specific single source of truth for the shared bootstrap tree that currently lives under:

```text
C:\shared\sandbox-toolchains\
```

From a domain-driven and 12-factor perspective, this location and split are intentional:

- reusable bootstrap primitives belong in shared infrastructure
- VS Code-specific orchestration belongs in a VS Code platform layer
- Node-specific runtime wiring belongs in a Node stack layer
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
  dev\
    git\
      2.54.0\
    node\
      26.2.0\
        node-v26.2.0-win-x64\
      20.19.6\
        node-v20.19.6-win-x64\
    pnpm\
      11.2.2\
        package\
          bin\
            pnpm.cjs
      11.5.0\
        package\
          bin\
            pnpm.cjs
    starship\
      1.25.1\
        starship.exe
    bootstrap\
      core\
        Bootstrap.Common.psm1
      platforms\
        vscode\
          Bootstrap.VSCode.psm1
          Start-VSCodeMaintenance.ps1
          Start-VSCodeProjectBase.ps1
      stacks\
        node\
          Bootstrap.Node.psm1
        python\
          Bootstrap.Python.psm1
        starship\
          Bootstrap.Starship.psm1
  projects\
    test-mono\
      bootstrap\
        Project.Config.ps1
        Start-TestMonoVSCode.ps1
        Start-TestMonoTerminal.ps1
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

### `dev\bootstrap\platforms\vscode\`

This is the VS Code platform adapter.

It contains the orchestration that is specific to VS Code as a platform:

- assert `Code.exe` / `code.cmd`
- compute box-local VS Code paths
- copy the canonical user catalog into `user-data`
- mirror the shared extension store into the box-local `extensions` directory
- initialize seed-backed paths like `globalStorage` and `.roo`
- run `code.cmd` for CLI actions
- run `Code.exe` for GUI launch

The current files are:

- `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode\Bootstrap.VSCode.psm1`
- `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode\Start-VSCodeMaintenance.ps1`
- `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode\Start-VSCodeProjectBase.ps1`

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

### `projects\<project>\bootstrap\`

This is the project adapter layer.

It contains only what is specific to one project:

- the project name
- the box name
- the default repo path
- the exact VS Code runtime bindings used by the project
- the exact shared toolchain versions used by the project
- any project-specific secondary runtime aliases
- thin host entry points for GUI launch and terminal launch

Using the sanitized boilerplate project name `test-mono`, the files are:

- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Project.Config.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoVSCode.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoTerminal.ps1`

## What each file currently does

### `Bootstrap.Common.psm1`

Provides the generic reusable helpers:

- `Assert-PathExists`
- `Ensure-Directory`
- `Copy-FileIfExists`
- `Test-DirectoryHasContent`
- `Sync-TreeMirror`
- `Initialize-TreeIfMissing`
- `Copy-TreeContents`
- `Initialize-TreeCopyIfMissing`
- `Write-AsciiFile`
- `Prepend-PathEntries`

### `Bootstrap.Node.psm1`

Provides the Node stack wiring:

- validates the shared `Git`, `Node`, and `pnpm` surfaces
- creates `pnpm.cmd`
- creates a shell-native `pnpm` wrapper for Git Bash
- creates additional node wrappers such as `node20.cmd`
- creates shell-native additional wrappers such as `node20`
- prepends `bootstrap-bin`, `git\cmd`, and the primary `Node` root to `PATH`

### `Bootstrap.Python.psm1`

Provides the Python stack wiring:

- validates the shared Python surface
- mirrors the selected Python version locally
- prepends the local Python version root into `PATH`

### `Bootstrap.Starship.psm1`

Provides the Starship prompt/runtime wiring:

- validates whether shared Starship is present
- mirrors Starship locally when it is provisioned
- prepends the local Starship directory into `PATH`
- generates `bash.minimal.rc`
- generates `bash.starship.rc`

Those RC files are also responsible for prepending the local `bootstrap-bin` directory into the Bash `PATH` so Git Bash can resolve bootstrap-generated shell wrappers such as `pnpm` and `node20`.

### `Bootstrap.VSCode.psm1`

Provides the VS Code platform wiring:

- validates `Code.exe` and `code.cmd`
- computes box-local `user-data` and `extensions` paths
- copies canonical `settings.json` / `keybindings.json`
- mirrors snippets
- mirrors shared extensions to the box-local runtime copy
- initializes seed-backed runtime state if missing
- wraps `code.cmd --install-extension`
- wraps `code.cmd --list-extensions`
- wraps `Code.exe`

### `Start-VSCodeMaintenance.ps1`

Provides the maintenance control-plane entry point:

- local maintenance authoring state under `C:\Program Files\SandboxToolchains\VSCodeBoxes\maintenance\...`
- local mirrored runtime/toolchain execution
- explicit publish/promotion back into shared canonical surfaces
- CLI-first actions:
  - `InstallExtension`
  - `ListExtensions`
  - `OpenTerminal`
  - `LaunchVSCode`

### `Start-VSCodeProjectBase.ps1`

Provides the reusable project-box orchestration:

- validates the repo path
- computes box-local VS Code paths
- synchronizes shared extensions into a box-local extension runtime copy
- initializes seeds
- initializes the Node toolchain layer
- initializes the Python toolchain layer when configured
- initializes the Starship prompt/runtime layer
- supports:
  - `LaunchVSCode`
  - `OpenTerminal`

### `Project.Config.ps1`

Provides the example `test-mono` project contract:

- box name: `VS_CODE_TEST_MONO`
- default repo path
- VS Code runtime paths
- shared extension path
- seed paths
- shared Git root
- primary `Node 26.2.0`
- optional shared Python root
- optional shared Starship root
- shared `pnpm.cjs`
- additional `node20` command

### `Start-TestMonoVSCode.ps1`

This is the project-specific host entry point for:

- `-Action LaunchVSCode`
- `-Action OpenTerminal`

It loads `Project.Config.ps1`, resolves the repo path, and calls the generic `Start-VSCodeProjectBase.ps1`.

### `Start-TestMonoTerminal.ps1`

This is the thin convenience wrapper that forwards directly to:

- `Start-TestMonoVSCode.ps1 -Action OpenTerminal`

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

This preserves the architecture contract:

- no live multi-writer extension store
- no project-box writes into the shared canonical extension path
- reproducible local runtime state per project box

## Related

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\scripts.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\provisioning\shared-artifacts.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\start.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\scripts.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\general.md`
