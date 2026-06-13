# Microsoft Build Toolchain In Boxed-Owned-Toolchain

## Architectural status

In the current boxed-owned-toolchain architecture, Microsoft build infrastructure is no longer treated as an incidental host path dependency of `node-gyp`.

Instead, the current verified model is:

- shared-governed Microsoft build-source roots under `C:\shared\sandbox-toolchains\dev\shells\...`
- bootstrap-owned projection into the canonical Windows runtime paths expected by Microsoft tools
- consumption through project-owned install / reinstall flows and the boxed native-build helper

This is the current boxed-owned-toolchain source of truth for the Microsoft build-toolchain side of the architecture.

## Domain map

### Microsoft build projection

- `docs\applications\operating-systems\windows\build-toolchain\microsoft\architectures\boxed-owned-toolchain\microsoft-build-projection.md`

Owns:

- `vswhere.exe`
- Visual Studio Build Tools
- Windows Kits / Windows SDK
- projection into canonical `Program Files (x86)` paths inside the box

### .NET Framework projection

- `docs\applications\operating-systems\windows\build-toolchain\microsoft\architectures\boxed-owned-toolchain\dotnet-framework-projection.md`

Owns:

- shared-governed `.NET Framework` compiler roots
- projection into canonical `C:\Windows\Microsoft.NET\...` paths inside the box
- the verified `Add-Type` / `buildcheck` compatibility bridge

## Why this area exists

These topics were previously mixed into consumer-specific documentation such as:

- `node-gyp`
- VS Code boxed bootstrap docs

That created two ownership problems:

1. Microsoft build infrastructure looked like a Node-only concern
2. boxed bootstrap docs carried platform-toolchain truth that should live in a platform-toolchain domain

The current boundary is therefore:

- this Windows Microsoft-build domain owns the infrastructure truth
- `node-gyp` documents how it consumes that truth
- VS Code/bootstrap docs document how they pass it through

## Related

- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\scripts.md`
