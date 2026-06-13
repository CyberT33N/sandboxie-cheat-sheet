# Microsoft Build Projection

## Compatibility note

The detailed boxed-owned-toolchain source of truth for Microsoft build projection has moved into the Windows platform/toolchain domain:

- `docs\applications\operating-systems\windows\build-toolchain\microsoft\architectures\boxed-owned-toolchain\microsoft-build-projection.md`

This `node-gyp` path remains only as a compatibility landing page because `node-gyp` consumes that infrastructure, but it no longer owns it.

## What still belongs to `node-gyp`

`node-gyp` still owns:

- how the boxed helper consumes the projected Microsoft build surface
- how `Initialize-NodeGypWindowsBuildEnvironment` prepares the shell
- how the wrapper / runtime contract interacts with native builds

But the platform-toolchain projection truth now lives in the Windows Microsoft build-toolchain area above.

## Related

- `docs\applications\operating-systems\windows\build-toolchain\microsoft\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\overview.md`
