# Extension Inventory In The Boxed-Owned-Toolchain

## Scope

This document records the complete extension inventory for the current boxed-owned-toolchain VS Code family setup.

It covers:

- the currently present canonical shared extension baseline
- the required marketplace-backed extensions
- the required local forked extensions that must be installed from VSIX artifacts

It does **not** duplicate:

- the generic extension-store governance
- the generic maintenance-box governance

Those stay canonical here:

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\state\extensions-seeds-and-local-state.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boxes\maintenance-box.md`

## Architecture summary

The current extension architecture is:

1. the Maintenance Box is the only global author
2. extensions are installed into the local maintenance authoring state first
3. approved changes are promoted into the canonical shared extension store
4. project boxes consume the canonical shared store through a local mirrored runtime copy

That means this inventory is about the **canonical shared VSCode-family extension set**, not about one project box's live runtime copy.

## Current canonical shared-store baseline

The current shared extension store snapshot already contains:

- `dbaeumer.vscode-eslint` at version `3.0.24`

Evidence surface:

- `C:\shared\sandbox-toolchains\ide\vscode\extensions\extensions.json`

## Required full inventory

### Marketplace-backed extensions

These extensions should be installed through the Maintenance Box by gallery identifier:

- `dbaeumer.vscode-eslint`
- `eamodio.gitlens`
- `golang.Go`
- `Nuxt.mdc`
- `ms-azuretools.vscode-containers`
- `ms-vscode-remote.remote-containers`
- `GitHub.vscode-github-actions`
- `ms-playwright.playwright`
- `ms-python.vscode-python-envs`
- `vitest.explorer`
- `ms-azuretools.vscode-docker`
- `redhat.vscode-yaml`
- `docker.docker`
- `ms-vscode.PowerShell`
- `ms-python.vscode-pylance`
- `ms-python.python`
- `ms-python.debugpy`
- `bradlc.vscode-tailwindcss`
- `ms-vscode-remote.remote-wsl`

### Local forked extensions installed from VSIX

These extensions are not currently consumed from the public VS Code Marketplace in this repository flow.

They should be staged from the local development workspace into the shared `dev` area as VSIX artifacts and then installed through the same Maintenance Box contract.

- `CyberT33N.code-navigation`
  - repo: `C:\Projects\development-platform\vs-code\extensions\code-navigation`
  - current VSIX artifact: `C:\Projects\development-platform\vs-code\extensions\code-navigation\artifacts\vsix\code-navigation-1.0.0.vsix`
- `CyberT33N.pretty-ts-errors`
  - repo: `C:\Projects\development-platform\vs-code\extensions\pretty-ts-errors`
  - current VSIX artifact: `C:\Projects\development-platform\vs-code\extensions\pretty-ts-errors\artifacts\vsix\pretty-ts-errors-1.5.1.vsix`
- `CyberT33N.errorlens`
  - repo: `C:\Projects\development-platform\vs-code\extensions\vscode-error-lens`
  - current VSIX artifact: `C:\Projects\development-platform\vs-code\extensions\vscode-error-lens\artifacts\vsix\errorlens-4.1.0.vsix`
- `CyberT33N.symbols-secured`
  - repo: `C:\Projects\development-platform\vs-code\extensions\vscode-symbols`
  - current VSIX artifact: `C:\Projects\development-platform\vs-code\extensions\vscode-symbols\artifacts\vsix\symbols-secured-0.0.27.vsix`
- `CyberT33N.dotenv`
  - repo: `C:\Projects\development-platform\vs-code\extensions\vscode-dotenv`
  - current VSIX artifact: `C:\Projects\development-platform\vs-code\extensions\vscode-dotenv\artifacts\vsix\dotenv-1.0.1.vsix`
- `CyberT33N.background`
  - repo: `C:\Projects\development-platform\vs-code\extensions\vscode-background`
  - current VSIX artifact: `C:\Projects\development-platform\vs-code\extensions\vscode-background\artifacts\vsix\background-3.0.0.vsix`
- `CyberT33N.rainbow-csv`
  - repo: `C:\Projects\development-platform\vs-code\extensions\vscode_rainbow_csv`
  - current VSIX artifact: `C:\Projects\development-platform\vs-code\extensions\vscode_rainbow_csv\artifacts\vsix\rainbow-csv-4.0.0.vsix`

## Recommended shared staging path for local VSIX artifacts

The current architecture needs a dedicated **source-artifact staging lane** that is distinct from the canonical published extension store.

The recommended shared staging root is:

```text
C:\shared\sandbox-toolchains\dev\vscode-extensions\vsix\CyberT33N\
```

Why this path is correct:

- it is in the shared `dev` area as requested
- it keeps local source VSIX artifacts separate from the canonical published extension store
- it stays reusable for both `VS Code` and `Cursor`, because both consume the same published VSCode-family extension store
- it avoids misusing `ide\vscode\extensions\` as both:
  - source-artifact staging
  - and the published canonical store

## Scope boundary

This inventory intentionally excludes legacy placeholder examples such as:

- `RooVeterinaryInc.roo-cline`

when they appear only as generic command examples in older documents and are not part of the required inventory above.

## Total inventory size

The complete current required inventory documented here is:

- `19` gallery-installed extensions
- `7` VSIX-installed local forked extensions
- `26` total required extensions

## Related

- `docs\applications\IDE\vscode\extensions\architectures\boxed-owned-toolchain\installation-boilerplate.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\state\extensions-seeds-and-local-state.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boxes\maintenance-box.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\cli\start\general.md`
