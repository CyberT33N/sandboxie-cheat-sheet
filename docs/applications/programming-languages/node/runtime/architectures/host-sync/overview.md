# Host-Sync Node Runtime

## Status

This is the secondary / legacy Node-runtime architecture path in this repository.

It is not the preferred default.

## Why this file exists

The Node runtime area is being normalized toward an explicit `architectures\...` split.

This file is the architecture entrypoint for host-driven or host-managed Node runtime assumptions.

## Current reference surface

The legacy host-managed runtime assumptions are currently represented mainly through:

- `docs\applications\programming-languages\node\nvm\general.md`
- host-sync PNPM and install-box documents that still assume host-visible runtime surfaces

## Related

- `docs\applications\programming-languages\node\runtime\general.md`
- `docs\applications\programming-languages\node\runtime\architectures\boxed-owned-toolchain\overview.md`
