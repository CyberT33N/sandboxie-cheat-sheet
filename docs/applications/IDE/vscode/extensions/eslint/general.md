# ESLint Extension

## Scope

This folder is the ESLint-extension documentation area for VS Code in this repository.

It is the correct domain owner for:

- extension-specific settings contracts
- extension-specific runtime binding decisions
- architecture-specific ESLint behavior inside VS Code

## Why this split exists

From a domain-driven perspective, the ESLint extension is not the same concern as:

- the generic VS Code method overview
- the generic Node toolchain overview
- the generic bootstrap layer

Those areas explain the broad method.

This area explains the part that is specific to the ESLint extension itself.

## Current documented architecture

The current architecture-specific ESLint write-up is:

- `docs\applications\IDE\vscode\extensions\eslint\architectures\boxed-owned-toolchain\settings-json.md`

## Related

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\state\vscode-runtime-and-catalog.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\provisioning\shared-artifacts.md`
