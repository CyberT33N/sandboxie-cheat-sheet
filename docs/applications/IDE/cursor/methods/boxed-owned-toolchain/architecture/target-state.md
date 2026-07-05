# Cursor Target State In The Boxed-Owned-Toolchain

## Scope

This document explains the Cursor-specific target-state overlay for the boxed-owned-toolchain method.

The canonical shared architecture still lives in the VS Code method area. This file records only the parts that are different once `Cursor` is introduced as a second IDE platform.

## Shared family kernel

The current architecture deliberately keeps the following surfaces shared across `VS Code` and `Cursor`:

- `settings.json`
- `keybindings.json`
- snippets
- seed-backed `globalStorage`
- seed-backed `.roo`
- the canonical extension store
- the shared toolchain under `C:\shared\sandbox-toolchains\dev\...`
- the shared VSCode-family bootstrap kernel under `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\vscode-family\...`

That shared family kernel is the single source of truth for:

- catalog ownership
- extension ownership
- seed ownership
- toolchain wiring
- local mirror / local publish bootstrap behavior

## What remains separate for Cursor

The following surfaces remain deliberately separate for `Cursor`:

- the runtime tree under `C:\shared\sandbox-toolchains\ide\cursor\runtime\...`
- the platform wrapper layer under `C:\shared\sandbox-toolchains\dev\bootstrap\platforms\cursor\...`
- the maintenance-box family `CursorBoxes`
- the project-box family `CursorBoxes`
- the Cursor-specific project launcher wrappers in `projects\test-mono\bootstrap\...`

## Why Cursor does not share the VS Code runtime

`Cursor` is not modeled as a cosmetic alias of `VS Code`.

It has its own:

- executable surface
- CLI surface
- runtime tree layout
- product-window behavior
- tunnel/update helper files
- product-specific troubleshooting classes

So the architecture does **not** try to fake one merged runtime.

Instead, it uses:

- a shared VSCode-family state/control-plane layer
- but a separate `Cursor` runtime namespace

## Why Cursor does not share the VS Code boxes

The architecture also deliberately avoids one combined live box for `VS Code` and `Cursor`.

That is required because the live box state includes more than the shared catalog:

- box-local `user-data`
- box-local `workspaceStorage`
- box-local logs and caches
- product-session state
- product-window memory
- product-specific auth/session behavior
- product-specific integrated-terminal behavior

Even when the canonical shared `settings.json` is identical, the **live** state is not the same bounded context.

So the correct split is:

- shared canonical family state
- separate live runtime state per editor
- separate live box state per editor

## Why separate maintenance boxes still matter

`VS Code` and `Cursor` both write to the same canonical shared family surfaces:

- `C:\shared\sandbox-toolchains\ide\vscode\catalog\...`
- `C:\shared\sandbox-toolchains\ide\vscode\extensions\...`

But they still need separate maintenance boxes because the maintenance workflow is not only "write settings".

It also needs to isolate:

- runtime installation and validation
- CLI behavior
- GUI launch behavior
- login/session state
- updater/tunnel helper surfaces
- editor-specific diagnostics

That is why the architecture keeps both:

- `VS_CODE_MAINTENANCE`
- `CURSOR_MAINTENANCE`

instead of collapsing everything into one maintenance runtime.

## Shared family tree with Cursor added

```text
C:\shared\sandbox-toolchains\
  ide\
    vscode\
      runtime\
      catalog\
      extensions\
    cursor\
      runtime\
  dev\
    bootstrap\
      platforms\
        vscode-family\
        vscode\
        cursor\
  projects\
    test-mono\
      bootstrap\
```

The important interpretation is:

- `ide\vscode\...` is the current VSCode-family shared state surface
- `ide\cursor\runtime\...` is a separate runtime surface
- `platforms\vscode-family\...` is the shared bootstrap kernel
- `platforms\cursor\...` is the Cursor-specific wrapper layer

## Current launch contract

The current project launch contract for `Cursor` is:

- shared project core chooses the editor
- Cursor-specific project wrapper delegates into the family kernel
- the family kernel forces the classic editor-window mode for project launches

That means a normal repo-open contract in `Cursor` must target the classic editor surface, not the agent/chat surface.

The product-level reasoning for that lives here:

- `docs\applications\IDE\cursor\general.md`

The bootstrap-specific wrapper paths live here:

- `docs\applications\IDE\cursor\methods\boxed-owned-toolchain\bootstrap\scripts.md`

## Installation-model difference from VS Code

The other major architectural difference is runtime provisioning:

- `VS Code` uses the existing shared runtime provisioning flow in the VS Code method area
- `Cursor` currently requires a dedicated installation lane through the Cursor Maintenance Box and then a separate shared runtime materialization

That installation delta lives here:

- `docs\applications\IDE\cursor\methods\boxed-owned-toolchain\provisioning\runtime-installation.md`

## What this document does not redefine

This document does **not** redefine the shared family truths for:

- catalog governance
- extension governance
- seed governance
- toolchain wiring
- local mirrored runtime state

Those remain canonical in the VS Code method area and are re-referenced instead of copied.

## Related

- `docs\applications\IDE\cursor\methods\boxed-owned-toolchain\general.md`
- `docs\applications\IDE\cursor\methods\boxed-owned-toolchain\boxes\project-box.md`
- `docs\applications\IDE\cursor\methods\boxed-owned-toolchain\boxes\maintenance-box.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\architecture\target-state.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\architecture\governance.md`
