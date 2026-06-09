# Nx Execution Surfaces

## Scope

This document owns the current execution-surface analysis for Nx in the boxed-owned-toolchain architecture.

It explains the split between:

- plain `nx`
- direct `node <resolved nxCli>`
- `pnpm exec nx`
- and `nx:run-commands` / `command` targets

## The validated failure chain

The current boxed project workflow exposed multiple distinct layers of failure:

1. `pnpm install` first failed because PNPM lifecycle execution on Windows tried to use spawned host shell surfaces such as `cmd.exe`
2. switching PNPM lifecycle execution to a box-local Bash `.exe` fixed that install-time shell problem
3. `pnpm exec nx ...` still failed with `spawn EPERM`
4. direct Nx execution via the resolved Node entrypoint worked, but `nx report` / `nx show projects` then needed the boxed Nx environment contract
5. direct guard scripts succeeded without Nx orchestration
6. `node -> spawn(cmd.exe)` failed with `spawn EPERM`
7. `node -> spawn(bash)` succeeded
8. `node -> spawn(bash -> node scripts/port-guard.mjs)` succeeded
9. manually setting `ComSpec=bash` made `nx run backend:port-guard` and `nx run frontend:port-guard` succeed

That means the current blocker is not “Nx is broken”.

It is a shell-selection / child-process boundary problem on Windows inside the strict boxed environment.

## Direct evidence

### Direct guard path

These direct commands were validated successfully:

```powershell
cd apps/backend
node scripts/port-guard.mjs

cd ../frontend
node scripts/port-guard.mjs

cd ../privyou-tooling
node scripts/smoke-electron-runtime.mjs
```

That proves:

- the guards themselves are healthy
- the Electron smoke guard is healthy
- the failure does not originate in the guard business logic

### Direct Node child-process split

The validated split was:

```powershell
node -> spawn(node.exe)         => success
node -> spawn(cmd.exe)          => spawn EPERM
node -> spawn(bash.exe / bash)  => success
node -> spawn(bash -> guard)    => success
```

That is the key diagnostic boundary.

### Nx through `run-commands`

This command failed in the boxed terminal:

```powershell
node "$nxCli" run backend:port-guard --output-style=stream
```

with:

```text
spawn EPERM
```

But the same target succeeded once the shell selection was overridden:

```powershell
ComSpec=bash COMSPEC=bash node "$nxCli" run backend:port-guard --output-style=stream
ComSpec=bash COMSPEC=bash node "$nxCli" run frontend:port-guard --output-style=stream
```

So the current evidence points to the Windows shell path, not to the target logic itself.

## Current command-surface split

### Plain `nx`

Current status:

- validated
- bootstrap-generated wrapper surface now exists
- intended for ordinary developer usage in the boxed terminal

### Direct `node <resolved nxCli>`

Current status:

- validated
- still the lowest-level proof path
- useful for diagnosis when the higher command surfaces are in doubt

### `pnpm exec nx`

Current status:

- still a separate failure surface
- must not be confused with plain `nx`
- must not be treated as the primary proof path

### `nx:run-commands` / `command`

Current status:

- still sensitive to the Windows shell-selection path
- currently proven to succeed when the shell is forced away from `cmd.exe`
- not yet fully repaired at bootstrap baseline level

## Why this matters architecturally

The current answer is not to pretend all Nx command surfaces are equivalent.

They are not.

The boxed-owned-toolchain method must treat them separately:

1. direct execution surface
2. wrapper execution surface
3. package-manager execution surface
4. Nx shell-command orchestration surface

This separation is critical because otherwise one green command can hide another still-broken runtime lane.

## Accepted solution-space boundary

For the current architecture discussion, the first-class solution space is:

- adapt bootstrap and shell/runtime selection so the box can execute the project's legitimate Nx command surfaces

The first-class answer is **not**:

- refactor the project away from Nx `run-commands`
- rewrite targets just to avoid the current shell-spawn boundary

That boundary is explicitly part of the current accepted architecture direction.

## Current unresolved point

At the latest validated state:

- the bootstrap-generated `nx` command works
- `COMSPEC` / `ComSpec` still points to `C:\WINDOWS\system32\cmd.exe`

So the current plain `nx` command surface is healthy, but the default Windows shell surface used by `run-commands` is still not yet redirected by bootstrap.

## Related

- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\runtime-contract.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\bootstrap-integration.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\lifecycle-and-command-surface.md`
- `docs\troubleshooting\sandboxie\process-spawning\cmd-based-shells.md`
