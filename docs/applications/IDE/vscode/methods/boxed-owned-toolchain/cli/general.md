# CLI in the VS Code Method

## Single source of truth

The generic CLI control-plane guidance is centralized outside the VS Code area:

- `docs\cli\general.md`
- `docs\cli\start\general.md`
- `docs\cli\terminal\general.md`

That is the architectural single source of truth for:

- starting a program in a specific box
- opening a terminal in a specific box
- understanding generic command-passing behavior

## What stays in the VS Code area

This folder now keeps only the VS Code-specific overlay for the boxed-owned-toolchain method.

That includes:

- how the Maintenance Box is used for shared IDE maintenance
- how the Project Box is used for boxed authoring
- how `code.cmd` is used for extension operations
- how the VS Code bootstrap contract should expose explicit actions

## Rule

Do not duplicate generic `Start.exe` or generic boxed-terminal guidance here.

Reference the global CLI documents first, then document only the VS Code-specific behavior.

## Local overlay documents

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\cli\start\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\cli\start\terminal.md`
