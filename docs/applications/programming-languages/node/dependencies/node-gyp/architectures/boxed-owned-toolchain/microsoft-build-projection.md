# Microsoft Build Projection

## Scope

This document owns the **verified Microsoft build-tool projection contract** for boxed-owned-toolchain `node-gyp`.

It documents only the projection behavior that is already implemented and runtime-verified.

It intentionally does **not** track:

- package-specific native build failures
- follow-up experiments
- architecture that is not yet validated

## Shared-governed source roots

The current verified shared Microsoft build sources are:

- `C:\shared\sandbox-toolchains\dev\shells\vs-installer\3.1.7\vswhere.exe`
- `C:\shared\sandbox-toolchains\dev\shells\visual-studio\2022\BuildTools\...`
- `C:\shared\sandbox-toolchains\dev\shells\windows-kits\10\...`

These shared roots are the canonical source of truth for the boxed native-build path.

## Projected boxed runtime paths

The helper projects those shared roots into the canonical Windows paths expected by `vswhere.exe`, `VsDevCmd.bat`, and MSBuild:

- `C:\Program Files (x86)\Microsoft Visual Studio\Installer\vswhere.exe`
- `C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\...`
- `C:\Program Files (x86)\Windows Kits\10\...`

This is a boxed runtime projection, not a live host-path execution contract.

## Verified Visual Studio projection scope

The currently verified Visual Studio projection copies these subtrees:

- `Common7\Tools`
- `MSBuild\Current\Bin`
- `MSBuild\Microsoft\VC`
- `VC\Auxiliary`
- `VC\Tools`

Those subtrees are sufficient for the current verified helper path to:

- resolve `VsDevCmd.bat`
- resolve `cl.exe`
- resolve `MSBuild.exe`

## Verified Windows Kits projection scope

The currently verified Windows Kits projection copies:

- `bin\<version>\x64`
- `Include\<version>`
- `Lib\<version>`
- `DesignTime\CommonConfiguration\Neutral`
- `SDKManifest.xml`

The `DesignTime\CommonConfiguration\Neutral` subtree is critical because MSBuild's Windows SDK validation checks require:

- `DesignTime\CommonConfiguration\Neutral\uCRT.props`
- `DesignTime\CommonConfiguration\Neutral\UAP\<version>\UAP.props`

Without those files, native builds can still fail with `MSB8036` even if `rc.exe`, headers, and libraries are already present.

## Helper-owned environment outputs

The current projection helper publishes:

- `BOXED_VSWHERE_EXE`
- `BOXED_VSROOT`
- `BOXED_VSDEVCMD`
- `BOXED_WINDOWS_SDK_ROOT`
- `BOXED_WINDOWS_SDK_VERSION`

And the broader bootstrap passes through the shared source roots:

- `BOXED_SHARED_VSWHERE_EXE`
- `BOXED_SHARED_VISUAL_STUDIO_ROOT`
- `BOXED_SHARED_WINDOWS_SDK_ROOT`

## Verified runtime results

The following results are already verified in the boxed project shell after `Initialize-NodeGypWindowsBuildEnvironment`:

- `VsDevCmd.bat` resolves from the projected boxed Visual Studio tree
- `VCINSTALLDIR` is set
- `VCToolsInstallDir` is set
- `VSINSTALLDIR` is set
- `WindowsSdkDir` is set
- `WindowsSDKVersion` is set
- `cl.exe` resolves
- `MSBuild.exe` resolves
- `rc.exe` resolves
- `mt.exe` resolves
- `vswhere.exe` resolves and returns the projected Visual Studio Build Tools path

## Marker behavior

The helper uses a projection marker file under the projected Visual Studio root and the projected Windows Kits root:

- `.boxed_projection_complete`

That marker prevents unnecessary full subtree re-copy on later helper calls when the already-projected surface is still complete.

## Current non-goal

This document does **not** claim that every package-specific native build now succeeds.

It documents only that the Microsoft build-tool projection itself is implemented and verified as working.

## Related

- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\current-state.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\runtime-contract.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\bootstrap-integration.md`
