# Python Build Helper For Host-Sync `node-gyp`

## Status

This document belongs to the **host-sync** `node-gyp` overlay.

It is not the Python-domain source of truth anymore.

## Ownership

The shared Python build-helper contract now lives in the Python domain here:

- `docs\applications\programming-languages\python\architectures\host-sync\dev-python-build-helper.md`

That document owns:

- the shared `C:\shared\sandbox-toolchains\dev\python\...` layout
- the `current.txt` version-pointer contract
- the download and extraction commands
- the validation commands

## What remains true for `node-gyp`

For the validated host-sync `node-gyp` workflow:

- `node-gyp` should use the central shared Python build helper under `C:\shared\sandbox-toolchains\dev\python\...`
- the install box should resolve the active Python version through `current.txt`
- `node-gyp` host-sync commands and install-box configuration should reference that shared Python helper explicitly

## Related documents

- `docs\applications\programming-languages\python\architectures\host-sync\dev-python-build-helper.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\install-box-config.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\commands.md`
