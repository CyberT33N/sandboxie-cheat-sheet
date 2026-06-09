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
$env:NX_DAEMON = 'false'
$env:NX_SOCKET_DIR = 'C:\nxs'
$env:NX_ISOLATE_PLUGINS = 'false'
$env:NX_NATIVE_FILE_CACHE_DIRECTORY = $localNxCacheRoot
```

And the socket directory must exist:

```powershell
New-Item -ItemType Directory -Force -Path $env:NX_SOCKET_DIR | Out-Null
```

## What each variable means

### `NX_DAEMON=false`

This disables the long-lived Nx daemon process.

Architecturally, that means:

- no background workspace watcher / graph server
- fewer background processes
- fewer IPC surfaces
- less hidden state between commands

This does **not** disable Nx itself.

It changes the execution model from:

- long-lived performance infrastructure

to:

- direct foreground computation

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

`NX_DAEMON` and `NX_ISOLATE_PLUGINS` are primarily **performance / infrastructure topology controls**, not core monorepo business functionality.

From an enterprise-grade sandbox perspective, fewer background processes and fewer cross-process IPC edges are often preferable because they reduce:

- invisible runtime state
- worker lifecycle fragility
- timeout sensitivity
- cross-process coordination overhead
- sandbox mediation complexity

So in this specific boxed-owned-toolchain environment, the current configuration is not a bad “hack”.

It is a valid, conservative runtime posture:

- smaller execution surface
- simpler trust boundary
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

## Current validated direct Nx surface

The current direct proof surfaces are:

```powershell
nx --version
nx show projects
```

Latest validated boxed output:

- `privyou-tooling`
- `installer`
- `frontend`
- `webpages`
- `backend`
- `privyou`

The lower-level resolved entrypoint also remains valid for diagnosis:

```powershell
$nxCli = node -p "try { require.resolve('nx/bin/nx.js') } catch { require.resolve('nx/dist/bin/nx.js') }"
node $nxCli --version
node $nxCli report
node $nxCli show projects
```

## Current target state

For the boxed-owned-toolchain architecture in this repository, the current Nx target state is:

1. box-local Nx native cache
2. `NX_DAEMON=false`
3. `NX_SOCKET_DIR='C:\nxs'`
4. `NX_ISOLATE_PLUGINS=false`
5. plain `nx` command surface works in the boxed project context

## Related

- `docs\cli\shell\general.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\cache-boundary.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\execution-surfaces.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\bootstrap-integration.md`
