# .NET Framework Projection

## Compatibility note

The detailed boxed-owned-toolchain source of truth for `.NET Framework` projection has moved into the Windows platform/toolchain domain:

- `docs\applications\operating-systems\windows\build-toolchain\microsoft\architectures\boxed-owned-toolchain\dotnet-framework-projection.md`

This `node-gyp` path remains only as a compatibility landing page because `node-gyp` consumes that infrastructure, but it no longer owns it.

## What still belongs to `node-gyp`

`node-gyp` still owns:

- how the boxed helper passes `.NET Framework` projection inputs into the build environment
- how that projected compiler surface is consumed by `buildcheck` / `Add-Type`-dependent native build flows

But the projection truth itself now lives in the Windows Microsoft build-toolchain area above.

## Related

- `docs\applications\operating-systems\windows\build-toolchain\microsoft\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\boxed-owned-toolchain\overview.md`
