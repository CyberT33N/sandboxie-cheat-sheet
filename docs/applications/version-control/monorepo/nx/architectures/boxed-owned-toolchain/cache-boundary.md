# Nx Cache Boundary

## Scope

This document owns the cache and local-state boundary for Nx in the boxed-owned-toolchain architecture.

It explains:

- where the Nx native cache belongs
- where the Nx socket surface belongs
- why those paths are local execution state rather than shared source of truth

## Preferred posture

For the current boxed-owned-toolchain architecture:

- keep the Nx native file cache box-local
- keep the Nx socket directory box-local
- do **not** externalize the native cache to a host-shared path as the baseline

## Why host-shared native cache is not the baseline

The native cache contains execution-relevant artifacts.

That means host-shared reuse widens the trust boundary in exactly the wrong direction:

1. cached native outputs become reusable across boxes
2. the blast radius grows beyond the current project box
3. the execution surface becomes harder to reason about
4. one compromised cache surface can influence later runs outside the immediate box boundary

So the correct current baseline is:

- box-local runtime mirror
- box-local native cache
- box-local socket path
- no shared host-visible Nx native cache as the default

## Current local surfaces

The current boxed project bootstrap sets:

```powershell
$env:NX_NATIVE_FILE_CACHE_DIRECTORY = $localNxCacheRoot
$env:NX_SOCKET_DIR = 'C:\nxs'
```

And it also exposes the corresponding diagnostic variables:

```powershell
$env:BOXED_NX_NATIVE_CACHE_ROOT = $localNxCacheRoot
$env:BOXED_NX_SOCKET_DIR = 'C:\nxs'
```

## Why the socket path is intentionally short

The validated boxed setup hit a socket-path-length failure.

So the current contract intentionally uses:

```powershell
$env:NX_SOCKET_DIR = 'C:\nxs'
```

This is not a hacky special case.

It is the correct runtime choice for the current environment because it:

- avoids the previously validated path-length break
- stays local to the box execution context
- keeps the socket surface explicit and reviewable

## Domain-driven interpretation

The Nx cache does **not** belong to the shared toolchain catalog.

It belongs to the **monorepo execution bounded context**.

That means:

- shared toolchain assets remain canonical source artifacts
- the Nx native cache remains execution-derived local state

This separation is important because a cache is not source of truth.
It is an execution optimization artifact.

Treating it as shared canonical infrastructure would collapse two different bounded contexts:

- versioned toolchain truth
- ephemeral orchestration state

The current architecture deliberately does not collapse those concerns.

## Twelve-factor interpretation

The cache and socket surfaces are runtime state, not code and not canonical config.

That means:

- configuration decides **where** they live
- but the cache contents themselves are disposable local runtime artifacts

This is consistent with a twelve-factor reading:

- environment defines runtime placement
- ephemeral execution state should not become shared release truth
- local caches should remain replaceable rather than quietly accumulating system-wide authority

## Current target state

For the boxed-owned-toolchain Nx architecture in this repository, the current target state is:

1. box-local Nx native cache
2. short box-local Nx socket path
3. no shared host-visible native-cache baseline
4. no broad temp-path exceptions as the preferred posture

## Related

- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\runtime-contract.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\bootstrap-integration.md`
- `docs\applications\version-control\monorepo\nx\architectures\host-sync\overview.md`
- `docs\troubleshooting\sandboxie\privacy-mode\host-user-space-vs-box-root.md`
