# Boxed-Owned-Toolchain PNPM

## Architectural status

This is the **preferred PNPM architecture path** in this repository.

Use this architecture when the operating model is:

- fully boxed authoring
- local runtime mirroring
- local mutable working state
- explicit promotion instead of live authoring against shared state

The host-sync PNPM write-up remains only as a secondary reference here:

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\host-sync\overview.md`

## Role of this file

This file is now the **TOC / entrypoint** for the boxed-owned-toolchain PNPM source of truth.

The previous high-density overview has been split by concern so the PNPM domain can stay:

- single-source-of-truth oriented
- easier to scale by concern
- easier to re-reference from VS Code method documents without duplicating package-manager detail

No previously established boxed-owned-toolchain PNPM truth is intentionally dropped by this split. The content now lives in the documents below.

## Current reference snapshot

The current boxed-owned-toolchain PNPM reference truth is:

1. governed shared `pnpm.cjs`
2. project-selected PNPM version via the project bootstrap contract
3. local mirrored command surface from bootstrap
4. default per-box PNPM store
5. box-local Bash as the validated lifecycle shell path
6. project-owned host-triggered install / clean-reinstall scripts
7. no shared PNPM store as the default baseline

## Domain map

### Runtime contract

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\runtime-contract.md`

Owns:

- the governed runtime surface
- why `@pnpm/exe` is not the contract
- per-box store posture
- effective version selection
- Sandboxie visibility range

### Lifecycle and command surface

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\lifecycle-and-command-surface.md`

Owns:

- the validated Sandboxie lifecycle-shell failure class
- the box-local Bash fix
- Git Bash command-name resolution requirements
- `pnpm exec` as a separate failure surface

### Versioning and provisioning

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\versioning-and-provisioning.md`

Owns:

- provisioning versioned PNPM binaries under `dev\pnpm\...`
- updating the project contract to a newer PNPM version
- why `pnpm self-update` / `npm install -g pnpm` are not the architecture center
- override stance and upgrade workflow

### Scripts

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\install.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\clean-reinstall.md`

Owns:

- project-owned PNPM install scripts
- project-owned PNPM clean-reinstall scripts
- current real `test-mono` script paths and host launch commands

## Cross-domain references

- Sandboxie shell-spawn troubleshooting:
  `docs\troubleshooting\sandboxie\process-spawning\cmd-based-shells.md`
- Central shell-selection contract:
  `docs\cli\shell\general.md`
- Boxed-owned-toolchain Nx execution contract:
  `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\overview.md`
- Boxed-owned-toolchain Nx execution-surface split:
  `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\execution-surfaces.md`
- Puppeteer-specific browser-cache and postinstall contract:
  `docs\applications\programming-languages\node\dependencies\puppeteer\boxed-owned-toolchain\overview.md`
- Electron-specific runtime-materialization and repair contract:
  `docs\applications\programming-languages\node\dependencies\frameworks\electron\architectures\boxed-owned-toolchain\overview.md`
- VS Code method orchestration view:
  `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\pnpm.md`

## Related

- `docs\applications\programming-languages\node\package-manager\pnpm\general.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\host-sync\overview.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\boxed-owned-toolchain\overview.md`
