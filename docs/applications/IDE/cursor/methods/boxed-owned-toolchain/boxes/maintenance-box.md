# Cursor Maintenance Box

## Role

The Cursor Maintenance Box is the Cursor-specific control-plane box for:

- Cursor runtime installation and validation
- Cursor maintenance-terminal diagnostics
- Cursor maintenance-GUI diagnostics
- Cursor-specific extension CLI checks

The family-level shared-state governance remains canonical here:

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boxes\maintenance-box.md`

This file keeps only the Cursor-specific overlay.

## Shared family write surfaces still remain the same

The Cursor Maintenance Box still targets the same canonical shared VSCode-family state surfaces:

- `C:\shared\sandbox-toolchains\ide\vscode\catalog\vscode-user\`
- `C:\shared\sandbox-toolchains\ide\vscode\catalog\seed\globalStorage\`
- `C:\shared\sandbox-toolchains\ide\vscode\catalog\seed\roo\`
- `C:\shared\sandbox-toolchains\ide\vscode\extensions\`

So the maintenance authorship split does **not** change:

- shared catalog, seeds, and extensions are still single-writer assets
- project boxes are still consumers

## What is Cursor-specific in maintenance

The Cursor Maintenance Box still has to stay separate from the VS Code Maintenance Box because it owns editor-specific runtime behavior:

- `Cursor.exe`
- Cursor CLI layout under `resources\app\codeBin\code.cmd`
- product-specific helper files such as:
  - `resources\app\bin\code-tunnel.exe`
  - `resources\app\bin\cursor-tunnel.exe`
  - `tools\inno_updater.exe`
- product-specific login/session behavior
- product-specific CLI behavior and troubleshooting

## Why this box must exist even though catalog and extensions are shared

The maintenance flow is not only "write settings into a shared folder".

It also needs to separate:

- runtime installation
- runtime verification
- CLI diagnostics
- GUI diagnostics
- editor-specific updater/tunnel exclusions

Without a separate Cursor maintenance context, the architecture would mix two different IDE runtimes into one control-plane box and blur failure ownership.

## Current wrapper surface

The current Cursor maintenance wrapper is:

- `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\cursor\Start-CursorMaintenance.ps1`

It delegates into the shared family maintenance kernel while fixing the Cursor-specific parameters:

- `EditorId = Cursor`
- `EditorDisplayName = Cursor`
- `BoxFamilyName = CursorBoxes`
- `RuntimeNamespace = cursor`

and the Cursor runtime exclusions:

- `resources\app\bin\code-tunnel.exe`
- `resources\app\bin\cursor-tunnel.exe`
- `tools\inno_updater.exe`

## Current operation lanes

The current maintenance lanes are:

- `OpenTerminal`
- `LaunchCursor`
- `ListExtensions`
- `InstallExtension`

The current architecture still treats the shared catalog and shared extension store as the canonical write surface even when those actions are invoked through the Cursor wrapper.

## Important current nuance

The repository currently distinguishes between:

- the valid shared-state governance model
- and a separate Cursor-specific maintenance CLI failure surface

That CLI-specific failure surface is not the same thing as the shared-state ownership model.

So:

- the existence of a Cursor-specific `ListExtensions` or `InstallExtension` issue does **not** invalidate the shared VSCode-family catalog and extension architecture
- it only means the Cursor maintenance wrapper still owns its own diagnostics lane

## Runtime installation difference

The Cursor Maintenance Box is also the correct place to stage the current Cursor runtime installation flow, because `Cursor` does not currently follow the same archive-style runtime provisioning path as `VS Code`.

That provisioning delta lives here:

- `docs\applications\IDE\cursor\methods\boxed-owned-toolchain\provisioning\runtime-installation.md`

## Related

- `docs\applications\IDE\cursor\methods\boxed-owned-toolchain\architecture\target-state.md`
- `docs\applications\IDE\cursor\methods\boxed-owned-toolchain\bootstrap\scripts.md`
- `docs\applications\IDE\cursor\methods\boxed-owned-toolchain\provisioning\runtime-installation.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boxes\maintenance-box.md`
