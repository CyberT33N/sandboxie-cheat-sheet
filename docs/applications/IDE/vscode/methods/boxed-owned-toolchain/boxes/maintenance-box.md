# Maintenance Box

## Role

The Maintenance Box is the global control-plane box for shared IDE assets.

It is the only global writer in the method.

## Responsibilities

Its responsibilities are:

- install, update, remove, and list shared extensions
- maintain the canonical `settings.json`
- maintain the canonical `keybindings.json`
- maintain snippets
- maintain seed material for `globalStorage`
- maintain seed material for `.roo`
- validate the shared extension store
- validate the shared VS Code runtime

## Authoritative write surfaces

The Maintenance Box is the only box that should author and publish changes for:

- `C:\shared\sandbox-toolchains\ide\vscode\catalog\...`
- `C:\shared\sandbox-toolchains\ide\vscode\extensions\...`

## Local authoring state

The current maintenance workflow does not live-author directly against those shared surfaces.

Instead, the Maintenance Box authors local working state under the box execution root, for example:

- `C:\Program Files\SandboxToolchains\VSCodeBoxes\maintenance\state\user-data`
- `C:\Program Files\SandboxToolchains\VSCodeBoxes\maintenance\state\extensions`
- `C:\Program Files\SandboxToolchains\VSCodeBoxes\maintenance\state\catalog\...`

Approved changes are then promoted back into the canonical shared surfaces.

## Operating model

The Maintenance Box is:

- CLI-first by default
- GUI-capable when diagnosis or curation is useful
- not the normal day-to-day authoring environment

## Typical commands

Typical maintenance actions are:

- `InstallExtension`
- `ListExtensions`
- `OpenTerminal`
- optional GUI launch for manual inspection
- publish/promote approved maintenance state

## Why this box exists

Without a dedicated maintenance author, shared IDE assets would become vulnerable to:

- accidental overwrite by project boxes
- unclear authorship
- cross-project drift
- difficult rollback and audit behavior

The Maintenance Box solves this by concentrating global authorship in a single bounded context.

## Related

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\architecture\governance.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\sandboxie\maintenance-box.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\cli\start\general.md`
