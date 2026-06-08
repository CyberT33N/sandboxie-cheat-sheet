# Boxed-Owned-Toolchain `node-gyp`

## Architectural status

This document captures the explored but **not validated** `node-gyp` architecture track where Microsoft Visual Studio Build Tools would be installed from inside a sandbox instead of being consumed from the host through the documented host-sync model.

This track is currently:

- exploratory
- not validated
- not recommended as the primary method for this repository

The currently documented comparison baseline remains:

- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\general.md`

## What this path means

In this architecture, the install box (or a duplicated playground box derived from the install-box baseline) would own the Microsoft C++ build toolchain instead of consuming host-installed Build Tools.

The intended model was:

- host-side download of the official Visual Studio Build Tools bootstrapper
- host-side creation of an offline layout under the shared toolchain area
- execution of the installer from inside the sandbox
- installation either:
  - into a host-visible shared path, or
  - into a box-local path used only inside the same sandbox

## Current outcome

The explored track is **not validated**.

Current bottom line:

- offline layout creation on the host succeeded
- installation attempts from inside the hardened sandbox did **not** materialize a usable Build Tools tree
- the repository must therefore **not** describe this path as working

## Why this remains secondary

From the currently documented evidence:

- the Microsoft installer chain behaves like a heavy Windows installer stack rather than like a portable copied binary
- the hardened sandbox baseline interacts badly with that installer behavior
- the exact minimum permission set for a working boxed installer flow is still unknown

So this path remains exploratory only.

## Legacy detailed notes

The earlier detailed exploratory notes are still preserved here:

- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\box-owned-toolchain\general.md`

That legacy path should be treated as preserved notes, not as the preferred architecture name.

## Related

- `docs\applications\programming-languages\node\dependencies\node-gyp\general.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\general.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\visual-studio-build-tools.md`
