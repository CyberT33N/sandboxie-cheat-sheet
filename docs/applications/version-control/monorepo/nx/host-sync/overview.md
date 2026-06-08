# Host-Sync NX Overview

## Architectural status

This document belongs to the **host-sync** architecture only.

It is **not** the preferred NX method in this repository.

The preferred method now lives here:

- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\overview.md`

This host-sync write-up is kept as a legacy / reference description for environments where:

- the IDE remains on the host
- execution is delegated into boxes
- and earlier host-visible cache or materialization patterns still matter historically

## Shared native cache: technical but anti-pattern

The old host-sync write-up externalized the Nx native file cache onto the host.

That is no longer the preferred recommendation.

From a security and governance perspective, a host-shared native cache is an anti-pattern because:

1. native artifacts are cached outside the box boundary
2. the cache can become reusable across box contexts
3. one compromised cache surface can spread across multiple runs or boxes
4. the trust boundary is widened beyond the box-local execution plane

So for the current preferred posture:

- **preferred**: each box keeps its own native Nx cache locally
- **not preferred**: a shared host-visible Nx native cache

## Why this came up historically

This legacy pattern existed because native Nx bindings were previously correlated with failures involving:

- host-visible user-space paths
- app-data-based temp/cache locations
- permission problems on native file loading

That history is now better explained by the later architecture work:

- native paths below host user space are fragile under `UsePrivacyMode=y`
- box-root-aligned local paths are the correct direction

Reference:

- `docs\troubleshooting\sandboxie\privacy-mode\host-user-space-vs-box-root.md`

## Legacy directions

There are two legacy directions worth preserving as historical reference.

### Legacy direction A: host-shared native cache

This is the old host-sync-style pattern:

```powershell
$env:NX_NATIVE_FILE_CACHE_DIRECTORY = 'C:\shared\sandbox-toolchains\node-monorepo-general\cache\nx-native'
$env:NX_DAEMON = 'false'
pnpm install
```

or:

```powershell
$env:NX_NATIVE_FILE_CACHE_DIRECTORY = 'C:\shared\sandbox-toolchains\node-monorepo-general\cache\nx-native'
$env:NX_DAEMON = 'false'
pnpm rebuild
```

Status:

- technically plausible
- historically discussed
- **legacy / not preferred**
- **not the recommended baseline**

### Legacy direction B: short box-local / sandbox-root-aligned cache

A second legacy direction is to keep the cache local to the box, but move it out of host-user-space-like path classes into a shorter sandbox-root-aligned location.

That direction is architecturally much better than:

- `%LOCALAPPDATA%\Temp\...`
- `AppData\...`
- or other host-user-space cache targets

because it stays:

- box-local
- shorter
- and independent from the privacy-mode user-space path family

## What should not be used

Do **not** use broad host temp exceptions such as:

```ini
OpenFilePath=node.exe,%LOCALAPPDATA%\Temp\nx-native-file-cache-*
```

That is too broad, too host-coupled, and too risky.

## Why Boxed-Own-Toolchain is preferred

The preferred architecture is now the boxed-owned-toolchain variant because it keeps:

- runtime binaries local to the box
- caches local to the box
- mutable state local to the box
- and promotion explicit instead of live against shared host state

Read that preferred write-up here:

- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\overview.md`

## Related

- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\host-sync\overview.md`
- `docs\troubleshooting\sandboxie\privacy-mode\host-user-space-vs-box-root.md`
- `docs\applications\programming-languages\node\dependencies\frameworks\electron\electron-vite\general.md`
- `docs\applications\version-control\monorepo\nx\debugging.md`




