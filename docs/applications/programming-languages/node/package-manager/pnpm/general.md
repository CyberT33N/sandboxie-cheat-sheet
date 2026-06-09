# PNPM

## Scope

This folder is the PNPM-domain documentation area for this repository.

PNPM-specific concerns belong here, for example:

- package-manager architecture choices
- store placement
- lifecycle-shell behavior
- architecture-specific runtime contracts

## Architecture split

### Preferred

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\overview.md`

The boxed-owned-toolchain entrypoint is now a TOC / SSOT map. The detailed PNPM truth is split there by concern, including:

- runtime contract
- lifecycle and command surface
- versioning and provisioning
- scripts

### Secondary / legacy reference

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\host-sync\overview.md`

## Related

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\pnpm.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\install.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\clean-reinstall.md`
