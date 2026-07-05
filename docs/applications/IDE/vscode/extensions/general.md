# VS Code Extensions

## Scope

This folder is the VS Code extension documentation area for this repository.

It owns:

- cross-extension architecture documents for the boxed-owned-toolchain method
- extension inventory and installation boilerplates
- extension-specific documents such as `eslint`

It does **not** replace:

- the boxed-owned-toolchain method overview
- the generic maintenance-box overview
- the generic bootstrap overview

Those areas stay responsible for the broader method.

## Reading order

### Cross-extension architecture

- `docs\applications\IDE\vscode\extensions\architectures\boxed-owned-toolchain\inventory.md`
- `docs\applications\IDE\vscode\extensions\architectures\boxed-owned-toolchain\installation-boilerplate.md`

### Extension-specific overlays

- `docs\applications\IDE\vscode\extensions\eslint\general.md`

## Why this split exists

From a domain-driven perspective, there are two different concerns:

1. cross-extension control-plane architecture
2. extension-specific behavior

Cross-extension architecture includes:

- where extensions are installed
- where local VSIX artifacts are staged
- how maintenance authorship works
- how the canonical shared extension store is published

Extension-specific overlays include:

- runtime settings such as `eslint.runtime`
- per-extension configuration or behavior

Keeping those concerns separate avoids mixing:

- one extension's settings contract
- with the shared installation and publication contract for all extensions

## Related

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\state\extensions-seeds-and-local-state.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boxes\maintenance-box.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\cli\start\general.md`
