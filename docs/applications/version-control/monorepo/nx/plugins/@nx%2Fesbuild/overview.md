# `@nx/esbuild`

## Scope

This folder owns plugin-specific documentation for the Nx `@nx/esbuild` plugin in this repository.

Use this area when the topic is:

- specific to the `@nx/esbuild` executor behavior
- not a generic Nx runtime-contract topic
- not a generic Sandboxie shell/runtime topic

## Current plugin-specific topic

- troubleshooting:
  - `docs\applications\version-control\monorepo\nx\plugins\@nx%2Fesbuild\troubleshooting\dist-folder-locked-by-boxed-node-processes.md`

## Why this area exists

The Nx architecture documents own:

- runtime contract
- daemon behavior
- cache/socket topology
- bootstrap integration

But when a problem is tied to one concrete Nx plugin, the plugin-specific write-up should live under `plugins\...` instead of being mixed into the generic Nx architecture pages.

Generic boxed process termination semantics do not live here. They live in:

- `docs\cli\process\general.md`

## Related

- `docs\applications\version-control\monorepo\nx\general.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\overview.md`
- `docs\cli\process\general.md`
