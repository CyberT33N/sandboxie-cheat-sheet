# Deprecated Nx Wrapper Command Surface

## Status

This document describes the **optional legacy Nx wrapper surface**.

It is:

- still technically available
- no longer the recommended default
- no longer required for the currently validated standard path

## Why this file exists

The repository historically introduced a bootstrap-published plain `nx` surface because earlier boxed runs had multiple Windows shell-selection problems.

That historical wrapper solved two things at once:

1. a stable plain `nx` alias
2. a stable workspace-local Nx resolution path

The current state is different:

- `pnpm exec nx ...` works
- direct `node <resolved nxCli> ...` works
- `pnpm exec tsx ...` works in the real target working directory used by the application runner
- the full application can start without depending on a bootstrap-published plain-`nx` alias

So the wrapper is no longer the recommended baseline.

## Current recommendation

Use this as the default:

```powershell
pnpm exec nx --version
pnpm exec nx show projects
pnpm exec nx run test:serve --no-tui -- --profile=dev-evident
```

Use the lower-level diagnostic path when needed:

```powershell
$nxCli = node -p "try { require.resolve('nx/bin/nx.js') } catch { require.resolve('nx/dist/bin/nx.js') }"
node $nxCli --version
node $nxCli show projects
```

## When the legacy wrapper can still make sense

Keep the optional wrapper only when a team explicitly wants a plain `nx` alias as a convenience surface, for example:

- in PowerShell
- in CMD
- in Git Bash
- or for special terminal profiles that deliberately prefer an alias instead of `pnpm exec nx`

That is a valid preference surface.

It is just not the recommended default architecture anymore.

## Shared implementation location

The optional wrapper implementation now lives here:

```text
C:\shared\sandbox-toolchains\dev\bootstrap\stacks\node\Bootstrap.NxWrapper.psm1
```

The intention is:

- keep the wrapper available
- keep it outside the default Node bootstrap contract
- make opt-in explicit instead of implicit

## What the optional wrapper publishes

When explicitly enabled, it publishes:

- `nx`
- `nx.cmd`
- `nx.ps1`
- `nx-cli.cjs`

into:

- `<boxed bootstrap-bin>`

and exposes:

- `BOXED_NX_LAUNCHER`

for diagnostics.

## Architectural interpretation

This is now a **legacy / optional compatibility surface**, not a first-class baseline.

That means:

- the default architecture should be documented around `pnpm exec nx ...`
- the wrapper should be documented as an additive convenience/compatibility choice
- teams should not be told that they must enable the wrapper just to make Nx work

## Relationship to Git Bash

The wrapper should also **not** be misread as “the Git-Bash-only Nx solution”.

The correct current split is:

- Git Bash remains the preferred **PNPM install/reinstall lifecycle shell**
- boxed `cmd.exe` remains the boxed `ComSpec` / child-process lane
- boxed PowerShell remains the preferred interactive default shell
- the Nx wrapper is a separate optional alias layer on top of those shell choices

So the wrapper is not the root answer to the Git-Bash question.
It is only an optional command-surface choice.

## Related

- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\execution-surfaces.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\bootstrap-integration.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\lifecycle-and-command-surface.md`
- `docs\cli\shell\general.md`
