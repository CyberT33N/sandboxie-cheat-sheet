# Microsoft Visual Studio Build Tools

## Compatibility note

The detailed host-sync source of truth for Microsoft Visual Studio Build Tools and Windows SDK has been moved into the Windows platform/toolchain domain:

- `docs\applications\operating-systems\windows\build-toolchain\microsoft\architectures\host-sync\visual-studio-build-tools.md`

This `node-gyp` path remains only as a compatibility landing page because `node-gyp` consumes that infrastructure, but it no longer owns it.

## What still belongs to `node-gyp`

`node-gyp` still owns:

- how the host-sync install shell consumes the Microsoft build-toolchain surface
- how that environment is used before `node-gyp configure` / `node-gyp rebuild`

But the Microsoft build-toolchain itself now lives in the dedicated Windows build-toolchain area above.

## Related

- `docs\applications\operating-systems\windows\build-toolchain\microsoft\architectures\host-sync\overview.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\general.md`
