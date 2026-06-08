# Boxed-Own-Toolchain NX Overview

## Architectural status

This is the **preferred NX method** in this repository.

Use this document when the architecture is:

- fully boxed authoring
- box-local runtime mirroring
- box-local mutable working state
- explicit promotion instead of live shared authoring

The legacy host-sync write-up remains only as a secondary reference here:

- `docs\applications\version-control\monorepo\nx\host-sync\overview.md`

## Preferred cache posture

For the current boxed-own-toolchain architecture:

- keep the Nx native file cache box-local
- do **not** externalize the native cache to a host-shared path as the baseline

Why:

1. the native cache contains execution-relevant native artifacts
2. host-shared reuse widens the blast radius across box boundaries
3. per-box isolation is stronger and simpler to reason about
4. there is no strong architectural need for multiple monorepos or boxes to share the same Nx native cache

So the correct baseline is:

- box-local Nx native cache
- box-local runtime mirror
- no shared host-visible Nx native cache as the default

## The problem chain we actually hit

The validated boxed project workflow exposed multiple distinct layers of failure:

1. `pnpm install` first failed because PNPM lifecycle execution on Windows tried to use spawned host shell surfaces such as `cmd.exe`
2. switching PNPM to a box-local Bash `.exe` fixed that lifecycle shell problem
3. `pnpm exec nx ...` still failed with `spawn EPERM`
4. direct `nx` execution via Node worked, but `nx report` / `nx show projects` then failed because the socket path was too long
5. after shortening the socket path, isolated Nx plugin workers still timed out / exited unexpectedly
6. disabling plugin isolation made `nx report` and `nx show projects` succeed

That means the current boxed-owned-toolchain answer is not one single flag, but a small governed environment contract.

## Validated environment contract

The current validated boxed-owned-toolchain environment is:

```powershell
$env:NX_DAEMON = 'false'
$env:NX_SOCKET_DIR = 'C:\nxs'
$env:NX_ISOLATE_PLUGINS = 'false'
```

And the socket directory must exist:

```powershell
New-Item -ItemType Directory -Force -Path $env:NX_SOCKET_DIR | Out-Null
```

## What each variable means

### `NX_DAEMON=false`

This disables the long-lived Nx daemon process.

Architecturally, that means:

- no background workspace watcher/graph server
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
- under Sandboxie this remains within the box's local execution boundary unless explicitly opened outward
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

That is proven by the validated result:

- `node $nxCli report` succeeded
- `node $nxCli show projects` returned real projects

## Why this is still enterprise-correct

This is the key architectural point:

`NX_DAEMON` and `NX_ISOLATE_PLUGINS` are primarily **performance / infrastructure topology controls**, not core monorepo business functionality.

From an enterprise-grade sandbox perspective, fewer background processes and fewer cross-process IPC edges are often preferable because they reduce:

- invisible runtime state
- worker lifecycle fragility
- timeout sensitivity
- cross-process coordination overhead
- sandbox mediation complexity

So in this specific boxed-owned-toolchain environment, the current configuration is not a bad "hack".

It is a valid, conservative runtime posture:

- smaller execution surface
- simpler trust boundary
- more deterministic command behavior

## Why "workaround" should be read carefully

`NX_ISOLATE_PLUGINS=false` can be called a workaround in the sense that it avoids a worker-isolation failure path.

But it is **not** the same as:

- disabling Nx
- removing project-graph functionality
- or accepting a broken architecture

It is better understood as:

- an officially supported compatibility / fallback mode

for environments where the isolated plugin worker model is less reliable than in-process execution.

## Validated direct Nx entrypoint

The validated direct entrypoint was:

```powershell
$nxCli = node -p "try { require.resolve('nx/bin/nx.js') } catch { require.resolve('nx/dist/bin/nx.js') }"
$nxCli
```

Then:

```powershell
node $nxCli --version
node $nxCli report
node $nxCli show projects
```

With the environment contract above active, the following commands were validated:

```powershell
node $nxCli report
node $nxCli show projects
```

and `show projects` returned:

- `installer`
- `frontend`
- `webpages`
- `backend`
- `test`

## Important nuance: `pnpm exec nx`

`pnpm exec nx ...` remained a separate failure surface during validation.

So for this architecture, direct `nx` verification via the resolved Node entrypoint is currently the validated proof path.

That should be treated separately from:

- PNPM lifecycle shell execution
- `pnpm exec`
- and direct Nx execution

## Bootstrap ownership

Yes, the bootstrap scripts should own these variables.

Why bootstrap is the correct place:

- these are runtime/toolchain environment controls
- they belong to process startup, not editor UI preferences
- every shell and every child process launched through the bootstrap should inherit the same contract

Why `settings.json` is not the main source of truth:

- terminal env in editor settings is weaker and more incidental
- it only covers terminals launched inside that editor context
- it is not the canonical process bootstrap layer

So for the boxed-own-toolchain architecture:

- **bootstrap script** = source of truth
- `settings.json` env = optional secondary overlay only if ever needed

## Bootstrap implementation

The validated bootstrap-level implementation is to set these values during shell startup before Nx commands are executed:

```powershell
$localNxSocketRoot = 'C:\nxs'
if (-not (Test-Path -LiteralPath $localNxSocketRoot)) {
  New-Item -ItemType Directory -Force -Path $localNxSocketRoot | Out-Null
}

$env:NX_DAEMON = 'false'
$env:NX_SOCKET_DIR = $localNxSocketRoot
$env:NX_ISOLATE_PLUGINS = 'false'
```

The current bootstrap scripts were updated to apply this at startup so that:

- the boxed terminal inherits the right Nx contract
- package-manager-driven shell flows inherit the same contract
- later manual Nx commands do not depend on ad-hoc terminal setup

## Current target state

For the boxed-own-toolchain architecture in this repository, the current target state is:

1. box-local Nx native cache
2. `NX_DAEMON=false`
3. `NX_SOCKET_DIR='C:\nxs'`
4. `NX_ISOLATE_PLUGINS=false`
5. direct Nx execution works in the boxed project context

## Related

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\overview.md`
- `docs\troubleshooting\sandboxie\process-spawning\cmd-based-shells.md`
- `docs\troubleshooting\sandboxie\privacy-mode\host-user-space-vs-box-root.md`
- `docs\applications\version-control\monorepo\nx\host-sync\overview.md`
