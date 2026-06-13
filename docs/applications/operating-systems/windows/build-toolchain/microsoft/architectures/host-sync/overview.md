# Microsoft Build Toolchain In Host Sync

## Architectural status

For the current validated host-sync architecture in this repository:

- Visual Studio Build Tools remain host-provided
- Windows Kits / Windows SDK remain host-provided
- the install box consumes them through explicit Sandboxie visibility rules
- the install-box shell imports the developer environment before native builds run

This host-sync baseline does **not** depend on the boxed-owned-toolchain projection model.

## Current scope

The current validated host-sync Microsoft build-toolchain reference owned here is:

- `docs\applications\operating-systems\windows\build-toolchain\microsoft\architectures\host-sync\visual-studio-build-tools.md`

That page covers both:

- Visual Studio Build Tools
- Windows SDK / Windows Kits

## Important boundary

The `.NET Framework` projection work documented for boxed-owned-toolchain is **not** part of the current host-sync baseline.

Host-sync remains the host-provided model.

## Related

- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\general.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\install-box-config.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\commands.md`
