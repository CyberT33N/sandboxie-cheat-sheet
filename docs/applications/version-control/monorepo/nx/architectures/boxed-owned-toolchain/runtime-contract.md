# Nx Runtime Contract

## Scope

This document owns the current boxed-owned-toolchain runtime contract for Nx.

It explains:

- which environment variables are part of the contract
- what they mean
- why they belong in bootstrap
- what the current validated Nx runtime surface looks like

## Current validated environment contract

The current boxed-owned-toolchain Nx environment is:

```powershell
$env:NX_DAEMON = 'true'
$env:NX_SOCKET_DIR = 'C:\nxs'
$env:NX_ISOLATE_PLUGINS = 'false'
$env:NX_NATIVE_FILE_CACHE_DIRECTORY = $localNxCacheRoot
```

And the socket directory must exist:

```powershell
New-Item -ItemType Directory -Force -Path $env:NX_SOCKET_DIR | Out-Null
```

## What each variable means

### `NX_DAEMON=true`

This enables the long-lived Nx daemon process.

Architecturally, that means:

- the workspace can use daemon-backed watch behavior
- the project graph and watch surfaces stay available for long-running serve flows
- the daemon becomes part of the boxed runtime contract instead of an ambient host default

This does **not** disable Nx itself.

It changes the execution model toward:

- a daemon-backed local workspace process
- plus explicit bootstrap-owned daemon hygiene before the developer starts working

## Bootstrap-owned daemon preflight

The current boxed-owned-toolchain direction for this repository is not just:

- `NX_DAEMON=true`

It is also:

- reset stale Nx daemon/cache state at bootstrap time
- then start a fresh daemon explicitly

Representative bootstrap-owned sequence:

```powershell
pnpm exec nx reset
pnpm exec nx daemon --start
```

Why this is part of the runtime contract:

- the project already proved that watch-oriented Nx commands need the daemon
- a stale daemon/cache state produced the later `Io error. Look inside err_kind for more details.` failure class
- doing the reset/start inside bootstrap keeps daemon lifecycle explicit instead of turning it into hidden workstation state

This also means the current repository guidance is:

- the default should be `NX_DAEMON=true`
- and the boxed project bootstrap should own the reset/start preflight for installed Nx workspaces

### `NX_SOCKET_DIR`

This controls where Nx places its sockets.

In the validated boxed setup, the default path became too long, so a very short path was required:

```powershell
$env:NX_SOCKET_DIR = 'C:\nxs'
```

Why this is correct:

- the path is short enough for the validated Nx socket limit
- it stays local to the boxed execution surface
- it avoids the long nested runtime/cache path that previously broke plugin loading

### `NX_ISOLATE_PLUGINS=false`

This forces Nx inference plugins to run **in-process** instead of in isolated worker processes.

What changes:

- plugin worker isolation is turned off
- plugin logic still runs
- but it runs inside the main Nx process rather than via separate plugin worker processes

What this does **not** mean:

- it does not disable the plugins
- it does not remove project graph functionality
- it does not remove normal Nx command capability

### `NX_NATIVE_FILE_CACHE_DIRECTORY`

This places the native Nx file cache into the box-local execution tree.

What this means:

- the native cache remains local to the project box
- cache artifacts do not become a shared host baseline
- the cache stays aligned with the local mirrored runtime model

## Why this is still enterprise-correct

This is the key architectural point:

`NX_DAEMON`, `NX_ISOLATE_PLUGINS`, and the daemon preflight sequence are primarily **runtime topology controls**.

From an enterprise-grade sandbox perspective, the important part is not “never run a daemon”.

The important part is:

- if the real project serve/watch contract needs the daemon, bootstrap must own it explicitly
- and bootstrap must keep the daemon lifecycle deterministic instead of reusing stale background state blindly

The current conservative posture therefore remains:

- daemon enabled because the validated serve/watch path needs it
- short socket path
- box-local native cache
- `NX_ISOLATE_PLUGINS=false`
- bootstrap-owned reset/start hygiene

That still reduces:

- invisible runtime state
- worker lifecycle fragility
- timeout sensitivity
- cross-process coordination overhead
- sandbox mediation complexity

So in this specific boxed-owned-toolchain environment, the current configuration is not a bad “hack”.

It is a valid, conservative runtime posture:

- explicit daemon lifecycle
- box-local state ownership
- more deterministic command behavior

## Why bootstrap owns this contract

These variables belong in bootstrap, not in editor settings.

Why bootstrap is the correct place:

- these are runtime/toolchain environment controls
- they belong to process startup, not editor UI preferences
- every shell and every child process launched through the bootstrap should inherit the same contract

This is also the correct twelve-factor reading:

- runtime configuration is injected via environment
- the command surface does not rely on hidden workstation state
- project startup stays explicit and reviewable

## Current validated standard Nx surface

The current preferred standard proof surfaces are:

```powershell
pnpm exec nx --version
pnpm exec nx show projects
```

Latest validated boxed output:

- `test-tooling`
- `installer`
- `frontend`
- `webpages`
- `backend`
- `test`

The lower-level resolved entrypoint remains valid for diagnosis:

```powershell
$nxCli = node -p "try { require.resolve('nx/bin/nx.js') } catch { require.resolve('nx/dist/bin/nx.js') }"
node $nxCli --version
node $nxCli report
node $nxCli show projects
```

## Current target state

For the boxed-owned-toolchain architecture in this repository, the current Nx target state is:

1. box-local Nx native cache
2. `NX_DAEMON=true`
3. `NX_SOCKET_DIR='C:\nxs'`
4. `NX_ISOLATE_PLUGINS=false`
5. bootstrap-owned `pnpm exec nx reset` followed by `pnpm exec nx daemon --start` for installed Nx workspaces
6. the preferred standard path is `pnpm exec nx ...`
7. direct `node <resolved nxCli> ...` remains the diagnostic baseline
8. the historical plain-`nx` wrapper surface is optional legacy compatibility, not part of the recommended default contract

## Related

- `docs\cli\shell\general.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\cache-boundary.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\execution-surfaces.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\bootstrap-integration.md`
