# Boxed-Owned-Toolchain Nx

## Status

This is the preferred Nx architecture path in this repository.

## Why this file exists

The Nx area is being normalized toward an explicit `architectures\...` split and consistent `boxed-owned-toolchain` naming.

## Current source of truth

The detailed current Nx write-up still lives here:

- `docs\applications\version-control\monorepo\nx\boxed-own-toolchain\overview.md`

Use that document for the currently documented:

- box-local native cache posture
- `NX_DAEMON=false`
- `NX_SOCKET_DIR='C:\nxs'`
- `NX_ISOLATE_PLUGINS=false`
- direct `nx` entrypoint validation

## Related

- `docs\applications\version-control\monorepo\nx\general.md`
- `docs\applications\version-control\monorepo\nx\architectures\host-sync\overview.md`
