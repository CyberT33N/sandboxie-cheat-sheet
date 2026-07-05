# Cursor Project Box

## Role

The Cursor Project Box is the normal boxed authoring environment for exactly one project when that project is opened in `Cursor`.

The generic project-box governance remains canonical here:

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boxes\project-box.md`

This file keeps only the Cursor-specific overlay.

## What stays shared

The Cursor Project Box still consumes the shared VSCode-family canonical surfaces:

- `C:\shared\sandbox-toolchains\ide\vscode\catalog\vscode-user\`
- `C:\shared\sandbox-toolchains\ide\vscode\catalog\seed\globalStorage\`
- `C:\shared\sandbox-toolchains\ide\vscode\catalog\seed\roo\`
- `C:\shared\sandbox-toolchains\ide\vscode\extensions\`
- `C:\shared\sandbox-toolchains\dev\...`

So the Cursor Project Box is still a **consumer**, not a global writer.

## What stays Cursor-specific

The Cursor Project Box uses:

- the Cursor runtime under `C:\shared\sandbox-toolchains\ide\cursor\runtime\...`
- the Cursor project-box family `CursorBoxes`
- the Cursor project bootstrap wrappers

For the current project, the concrete wrapper surfaces are:

- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-testMonoEditor.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-testMonoCursor.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-testMonoCursorTerminal.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-testMonoCursorElectronTerminal.ps1`

## Why this box is separate from the VS Code project box

Even though the canonical family catalog is shared, the live project-box state is still editor-specific:

- local runtime mirror
- local `user-data`
- local `workspaceStorage`
- local caches and logs
- product-window memory
- product-session behavior

That is why the architecture keeps a separate Cursor project-box family instead of letting one live box host both editors.

## Current launch interpretation

The Cursor project-box launch path is:

1. host enters the named Cursor project box
2. the project wrapper selects `Cursor`
3. the shared project core delegates into the Cursor platform wrapper
4. the shared family kernel prepares the local mirrored runtime and toolchain
5. the project launch targets the classic Cursor editor window for repo opening

The product-level reasoning for the final step lives here:

- `docs\applications\IDE\cursor\general.md`

## Shared install and repair scripts

Project-owned dependency and repair scripts remain shared and editor-selectable rather than duplicated.

For the current project, that means:

- `Start-testMonoPnpmInstall.ps1`
- `Start-testMonoPnpmCleanReinstall.ps1`
- `Start-testMonoPnpmUninstall.ps1`
- `Start-testMonoElectronPostInstall.ps1`

use:

- `-Editor VSCode`
- or `-Editor Cursor`

instead of creating a second divergent dependency-management implementation just for Cursor.

## Related

- `docs\applications\IDE\cursor\methods\boxed-owned-toolchain\architecture\target-state.md`
- `docs\applications\IDE\cursor\methods\boxed-owned-toolchain\bootstrap\scripts.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boxes\project-box.md`
