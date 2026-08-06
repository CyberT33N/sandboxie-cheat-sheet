# Cursor In The Boxed-Owned-Toolchain

## Status

This is the Cursor-specific overlay for the boxed-owned-toolchain method.

The canonical shared architecture still lives in the VS Code method area because the current implementation uses one shared VSCode-family kernel and only keeps the truly Cursor-specific surfaces separate.

## Scope

This area owns only the parts that are specific to `Cursor`:

- separate Cursor runtime acquisition and installation
- separate Cursor Maintenance Box and Cursor project boxes
- Cursor-specific platform wrappers and project-entry wrappers
- Cursor-specific launch behavior such as classic editor-window forcing
- Cursor-specific troubleshooting such as sign-in return-path behavior and the validated integrated-terminal `conpty` fix

It does **not** duplicate the canonical shared VSCode-family source of truth for:

- catalog governance
- extension-store governance
- seed governance
- shared toolchain governance
- shared bootstrap-kernel responsibilities

## Current model

The current architecture is:

- one shared VSCode-family catalog, seed, and extension surface under `C:\shared\sandbox-toolchains\ide\vscode\...`
- one shared VSCode-family bootstrap kernel under `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode-family\...`
- one separate `VS Code` runtime under `C:\shared\sandbox-toolchains\ide\vscode\runtime\...`
- one separate `Cursor` runtime under `C:\shared\sandbox-toolchains\ide\cursor\runtime\...`
- one separate project-box family per editor
- one separate maintenance-box family per editor

That split preserves single-writer governance for the shared IDE-family state without pretending that `VS Code` and `Cursor` should share one live runtime or one live box state.

## Shared versus Cursor-specific surfaces

### Shared VSCode-family surfaces

These remain canonical and shared between `VS Code` and `Cursor`:

- `C:\shared\sandbox-toolchains\ide\vscode\catalog\vscode-user\`
- `C:\shared\sandbox-toolchains\ide\vscode\catalog\seed\globalStorage\`
- `C:\shared\sandbox-toolchains\ide\vscode\catalog\seed\roo\`
- `C:\shared\sandbox-toolchains\ide\vscode\extensions\`
- `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode-family\`

### Cursor-specific surfaces

These remain Cursor-specific:

- `C:\shared\sandbox-toolchains\ide\cursor\runtime\`
- `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\cursor\`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-testMonoCursor.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-testMonoCursorTerminal.ps1`
- `C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-testMonoCursorElectronTerminal.ps1`

## Reading order

### 1. Shared canonical architecture

Read these first for the family-level source of truth:

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\architecture\target-state.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\architecture\governance.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\state\vscode-runtime-and-catalog.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\state\extensions-seeds-and-local-state.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\shared-layout.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\scripts.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\start.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\scripts.md`

### 2. Cursor-specific method overlays

Then read the Cursor-specific deltas:

- `docs\applications\IDE\cursor\methods\boxed-owned-toolchain\architecture\target-state.md`
- `docs\applications\IDE\cursor\methods\boxed-owned-toolchain\boxes\project-box.md`
- `docs\applications\IDE\cursor\methods\boxed-owned-toolchain\boxes\maintenance-box.md`
- `docs\applications\IDE\cursor\methods\boxed-owned-toolchain\bootstrap\scripts.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\host-entry-wrappers.md`
- `docs\applications\IDE\cursor\methods\boxed-owned-toolchain\provisioning\runtime-installation.md`

### 3. Cursor product and troubleshooting overlays

Then read the product/runtime-specific follow-ups:

- `docs\applications\IDE\cursor\general.md`
- `docs\applications\IDE\cursor\architectures\boxed-owned-toolchain\sign-in.md`
- `docs\applications\IDE\cursor\architectures\boxed-owned-toolchain\troubleshooting\integrated-terminal-conpty.md`
- `docs\applications\IDE\cursor\architectures\boxed-owned-toolchain\troubleshooting\agent-shell-host-powershell-projection.md`

## Why this area exists

From a domain-driven perspective, `Cursor` is now neither:

- a totally separate end-to-end architecture
- nor a purely cosmetic rename of the VS Code method

It is a second IDE platform that:

- reuses the VSCode-family shared kernel
- but still has its own runtime, launcher, box-state, and product-specific failure surfaces

So the correct documentation split is:

- shared truths remain in the VS Code method area
- Cursor-only deltas live here

## Related

- `docs\applications\IDE\cursor\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\general.md`
- `docs\applications\git\architectures\boxed-owned-toolchain\authentication-and-clone.md`
- `docs\applications\programming-languages\node\dependencies\frameworks\electron\architectures\boxed-owned-toolchain\troubleshooting.md`
