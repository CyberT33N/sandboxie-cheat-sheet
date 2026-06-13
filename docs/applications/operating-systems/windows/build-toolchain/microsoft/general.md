# Microsoft Build Toolchain

## Purpose

This directory is the cross-domain source of truth for the Microsoft native-build infrastructure used in this repository on Windows.

It owns the architecture split for:

- Visual Studio Build Tools
- Windows Kits / Windows SDK
- `vswhere.exe`
- projected `.NET Framework` compiler surfaces

This area belongs under `docs\applications\operating-systems\windows\...` because the capability is:

- Windows-specific
- Microsoft-platform-specific
- cross-language
- and consumed by multiple higher-level domains such as `node-gyp`, bootstrap, and boxed runtime projection

It is **not** owned by the `node-gyp` domain, even though `node-gyp` is a major consumer of it.

## Architecture tracks

### Host sync

- `docs\applications\operating-systems\windows\build-toolchain\microsoft\architectures\host-sync\overview.md`

Owns:

- the host-provided Visual Studio Build Tools / Windows SDK baseline
- the current validated host-sync posture

### Boxed-owned-toolchain

- `docs\applications\operating-systems\windows\build-toolchain\microsoft\architectures\boxed-owned-toolchain\overview.md`

Owns:

- the shared-governed Microsoft build-source roots
- boxed projection into canonical Windows runtime paths
- the `.NET Framework` projection contract used by `Add-Type` / `buildcheck`

## Why this re-anchoring exists

Historically, these topics were documented inside consumer domains such as:

- `node-gyp`
- VS Code boxed bootstrap docs

That made the ownership blurry.

The current architecture boundary is:

- Microsoft build infrastructure is its own Windows platform/toolchain domain
- `node-gyp` consumes that domain
- VS Code/bootstrap docs reference that domain
- install/reinstall scripts pass through that domain

This preserves single-source-of-truth without breaking consumer-specific documentation.

## Related

- `docs\applications\programming-languages\node\dependencies\node-gyp\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\scripts.md`
