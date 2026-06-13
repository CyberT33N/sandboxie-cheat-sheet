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
2. the first historical workaround switched PNPM lifecycle execution to a box-local Bash `.exe`
3. `pnpm exec nx ...` still failed with `spawn EPERM`
4. direct Nx execution via the resolved Node entrypoint worked, but `nx report` / `nx show projects` then needed the boxed Nx environment contract
5. direct guard scripts succeeded without Nx orchestration
6. `node -> spawn(cmd.exe)` failed with `spawn EPERM`
7. `node -> spawn(bash)` succeeded
8. `node -> spawn(bash -> node scripts/port-guard.mjs)` succeeded
9. manually setting `ComSpec=bash` made `nx run backend:port-guard` and `nx run frontend:port-guard` succeed as a historical intermediate workaround

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

cd ../test-tooling
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

That later manual shell-selection validation has now been superseded by the current productive bootstrap-level solution:

- bootstrap now sets `ComSpec` / `COMSPEC` to the box-local boxed-CMD executable
- bootstrap now publishes the standard multi-shell command surfaces that are still needed, such as `pnpm.ps1`
- bootstrap keeps Git Bash as an explicit alternative compatibility lane
- the boxed project terminal now validates:
  - `nx run backend:port-guard`
  - `nx run frontend:port-guard`
  - `nx run test:smoke-electron-runtime`
  - `pnpm exec nx --version`

## Current command-surface split

### Plain `nx`

Current status:

- no longer the recommended default surface
- should be treated as an optional legacy convenience lane only
- if a plain `nx` alias is still desired, it should be enabled explicitly instead of assumed as part of the default bootstrap contract

### Direct `node <resolved nxCli>`

Current status:

- validated
- still the lowest-level proof path
- useful for diagnosis when the higher command surfaces are in doubt

### `pnpm exec nx`

Current status:

- this is now the preferred standard execution surface
- currently validated as green under the boxed-CMD `COMSPEC` contract
- must still be verified separately whenever the shell-selection contract changes
- must not be confused with the historical plain-`nx` wrapper surface

### `nx:run-commands` / `command`

Current status:

- sensitive to the Windows shell-selection path by nature
- historically unblocked by a box-local Git-Bash `ComSpec` workaround
- now validated with the bootstrap-owned `ComSpec` / `COMSPEC` override to boxed `cmd.exe`
- that historical/current child-process fix does **not** redefine the preferred interactive default shell, which is boxed PowerShell
- no longer dependent on manually typing `ComSpec=bash ...` in the shell for the currently tested target set

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

## Current prioritized solution

The current prioritized repository solution is:

1. keep the box strict
2. treat `pnpm exec nx ...` as the preferred standard Nx execution path
3. keep the child-process shell contract explicit and bootstrap-owned
4. for the current productive `run-commands` fix, keep `ComSpec` / `COMSPEC` mapped to the box-local boxed-CMD executable
5. keep PowerShell-native wrappers, CMD wrappers, and shell-native wrappers side by side where they are still actually needed
6. treat boxed PowerShell as the preferred interactive default shell
7. keep Git Bash available as an explicit alternative compatibility lane and as the preferred PNPM install/reinstall lifecycle lane
8. keep the currently validated individual Nx `run-commands` target set aligned with that shell contract
9. treat the historical plain-`nx` wrapper as optional legacy compatibility, not as a required default

Latest validated boxed result:

- `COMSPEC` points to:
  - `C:\Program Files\SandboxToolchains\VSCodeBoxes\test-mono\execution\toolchain\shells\cmd\10.0.26100.8457\cmd.exe`
- individual Nx targets succeed:
  - `backend:port-guard`
  - `frontend:port-guard`
  - `test:smoke-electron-runtime`
- `pnpm exec nx --version` succeeds
- direct `node <resolved nxCli>` succeeds
- `pnpm exec tsx ...` succeeds in the real target working directory used by the application runner
- this means the historical wrapper is no longer required for the current validated standard path

## Related

- `docs\cli\shell\general.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\runtime-contract.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\bootstrap-integration.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\lifecycle-and-command-surface.md`
- `docs\troubleshooting\sandboxie\process-spawning\cmd-based-shells.md`
