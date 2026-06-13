# .NET Framework Projection

## Scope

This document owns the **verified `.NET Framework` projection contract** for boxed-owned-toolchain `node-gyp`.

It documents only the runtime behavior that is already implemented and verified.

It intentionally excludes:

- open package-specific native build issues
- unverified alternative projection strategies

## Shared-governed source roots

The current verified shared `.NET Framework` compiler sources are:

- `C:\shared\sandbox-toolchains\dev\shells\dotnet-framework\Framework\v4.0.30319`
- `C:\shared\sandbox-toolchains\dev\shells\dotnet-framework\Framework64\v4.0.30319`

Each shared source root is verified to contain:

- `csc.exe`
- `cvtres.exe`

## Projected boxed runtime paths

The helper projects those shared roots into the canonical runtime paths expected by PowerShell `Add-Type` and other hard-coded compiler consumers:

- `C:\Windows\Microsoft.NET\Framework\v4.0.30319`
- `C:\Windows\Microsoft.NET\Framework64\v4.0.30319`

This boxed projection is the current compatibility bridge for `Add-Type` and `buildcheck`.

## Helper-owned environment outputs

The projection publishes:

- `BOXED_DOTNET_FRAMEWORK_SOURCE_ROOT`
- `BOXED_DOTNET_FRAMEWORK64_SOURCE_ROOT`
- `BOXED_DOTNET_FRAMEWORK_ROOT`
- `BOXED_DOTNET_FRAMEWORK64_ROOT`
- `BOXED_DOTNET_FRAMEWORK_VERSION`
- `BOXED_DOTNET_FRAMEWORK_CSC_EXE`
- `BOXED_DOTNET_FRAMEWORK64_CSC_EXE`
- `BOXED_DOTNET_FRAMEWORK_CVTRES_EXE`
- `BOXED_DOTNET_FRAMEWORK64_CVTRES_EXE`
- `BOXED_DOTNET_FRAMEWORK_AVAILABLE`

The projected x86 and x64 framework directories are also prepended into `PATH`.

## Marker behavior

The projection uses per-root marker files:

- `C:\Windows\Microsoft.NET\Framework\v4.0.30319\.boxed_projection_complete`
- `C:\Windows\Microsoft.NET\Framework64\v4.0.30319\.boxed_projection_complete`

Those markers avoid unnecessary re-copy when the projected framework surface is already complete.

## Verified runtime results

The following results are already verified in the boxed project shell after `Initialize-NodeGypWindowsBuildEnvironment`:

- `Test-Path $env:BOXED_DOTNET_FRAMEWORK_CSC_EXE` returns `True`
- `Test-Path $env:BOXED_DOTNET_FRAMEWORK64_CSC_EXE` returns `True`
- `Get-Command csc.exe` resolves
- `Get-Command cvtres.exe` resolves
- PowerShell `Add-Type` succeeds when invoked through the boxed PowerShell lane

Verified smoke-test result:

```text
OK
```

That confirms the projected boxed `.NET Framework` compiler chain is usable for the current `Add-Type` surface.

## Current non-goal

This document does **not** claim that every package-specific native build is fully resolved.

It documents only that the `.NET Framework` compiler projection itself is working.

## Related

- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\current-state.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\runtime-contract.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\bootstrap-integration.md`
