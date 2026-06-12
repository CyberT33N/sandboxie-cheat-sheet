# PNPM Lifecycle And Command Surface

## Scope

This document owns the current boxed-owned-toolchain PNPM execution contract for:

- lifecycle execution during `pnpm install`
- lifecycle execution during clean reinstall
- `pnpm exec ...`
- the relationship between PNPM and the bootstrap-owned Windows shell contract

## The actual boxed failure class

The main validated blocker was **not** PNPM package resolution itself.

The blocked surface was the Windows shell-selection boundary under strict Sandboxie policy.

Validated historical behavior in the boxed project shell:

1. `cmd.exe` worked when launched manually
2. `node -> spawn(local node.exe)` worked
3. `node -> spawn(host/system cmd.exe)` failed with `spawn EPERM`
4. `node -> spawn(host/system powershell.exe)` failed with `spawn EPERM`
5. `node -> spawn(box-local bash.exe)` worked

That means the root problem was:

- not a general Node child-process failure
- not a broken local toolchain mirror
- not a generic PNPM command failure
- but a shell-spawn problem for the command-interpreter surface used by PNPM and related tooling

The cross-cutting shell-selection write-ups live here:

- `docs\cli\shell\general.md`
- `docs\troubleshooting\sandboxie\process-spawning\cmd-based-shells.md`

## Historical fallback: Git Bash

Git Bash was the earlier validated workaround for parts of the boxed PNPM lifecycle problem.

That historical sequence was:

```powershell
$bashExe = Join-Path $env:BOXED_LOCAL_TOOLCHAIN_ROOT 'git\2.54.0\bin\bash.exe'
if (-not (Test-Path -LiteralPath $bashExe)) {
  $bashExe = Join-Path $env:BOXED_LOCAL_TOOLCHAIN_ROOT 'git\2.54.0\usr\bin\bash.exe'
}

if (-not (Test-Path -LiteralPath $bashExe)) {
  throw 'Local boxed Bash executable not found.'
}

pnpm config set --location=project scriptShell "$bashExe"
pnpm install
```

That path is still important historically and Git Bash remains an explicit alternative shell lane.

However, it is **no longer** the preferred productive PNPM contract for the boxed-owned-toolchain architecture.

## Current preferred productive contract

The current preferred productive path is:

1. boxed PowerShell stays the preferred interactive VS Code shell
2. boxed `cmd.exe` becomes the productive PNPM lifecycle and child-process lane
3. bootstrap projects that lane explicitly through `ComSpec` / `COMSPEC`
4. project-owned install scripts set PNPM `scriptShell` to the boxed CMD binary

### Bootstrap-owned `COMSPEC` / `ComSpec` contract

The current productive bootstrap contract is:

```powershell
$boxedComSpec = $windowsShellRuntime.CmdExe
if ([string]::IsNullOrWhiteSpace($boxedComSpec) -or -not (Test-Path -LiteralPath $boxedComSpec)) {
  throw 'Local boxed CMD executable not found for ComSpec override.'
}

$env:ComSpec = $boxedComSpec
$env:COMSPEC = $boxedComSpec
$env:BOXED_COMSPEC = $boxedComSpec
```

This code belongs in the shared bootstrap entrypoints, not in ad hoc terminal overrides.

### Project-owned install-script contract

The current productive install-script contract is:

```powershell
if ([string]::IsNullOrWhiteSpace($env:BOXED_CMD_EXE)) {
  throw 'BOXED_CMD_EXE was not initialized by project bootstrap.'
}

$cmdExe = $env:BOXED_CMD_EXE
if (-not (Test-Path -LiteralPath $cmdExe)) {
  throw 'Local boxed CMD executable not found.'
}

pnpm config set --location=project scriptShell "$cmdExe"
pnpm install
```

The clean-reinstall flow uses the same `scriptShell` contract before it runs the reinstall.

### Why boxed CMD is the preferred productive lane

The current architectural preference is boxed CMD because it is the only lane that was fully validated across the productive PNPM command surfaces:

- `COMSPEC` / `ComSpec` pointing to boxed CMD succeeded
- `pnpm exec nx --version` succeeded in a fresh boxed session
- `pnpm exec tsx --version` succeeded from the actual target working directory
- `pnpm exec tsx tooling/run/cli.ts --help` reached the application runner and returned only the expected CLI-schema validation output

By contrast, a fresh boxed session with:

- `COMSPEC` / `ComSpec` pointing to boxed PowerShell

failed immediately on `pnpm exec` with a PowerShell command-resolution / module-loading error.

That means:

- boxed PowerShell remains the preferred **interactive** shell
- boxed CMD is the preferred **productive PNPM lifecycle and child-process** lane

## Current validated fresh-session proof

After the shared bootstrap and project install scripts were aligned to boxed CMD, a fresh boxed project terminal validated:

```powershell
$env:COMSPEC
$env:ComSpec
pnpm config get scriptShell
pnpm exec nx --version

Set-Location .\apps\test-tooling
pnpm exec tsx --version
pnpm exec tsx tooling/run/cli.ts --help
```

Validated interpretation:

- `COMSPEC` and `ComSpec` both resolved to the boxed CMD executable
- `scriptShell` resolved to the boxed CMD executable
- `pnpm exec nx --version` was green
- `pnpm exec tsx --version` was green in the target working directory
- `pnpm exec tsx tooling/run/cli.ts --help` reached the runner and failed only with the expected command-schema validation message

That is the current proof that the Git-Bash-based productive contract is no longer the active PNPM blocker.

## Git Bash still remains available

Git Bash is still a supported alternative shell lane.

That matters for:

- explicit Git Bash profiles in VS Code
- shell-native wrapper testing
- compatibility experiments that specifically need Bash semantics

If Git Bash is selected directly, the boxed-owned-toolchain contract still requires:

- a Windows wrapper such as `pnpm.cmd`
- a shell-native wrapper such as `pnpm`
- Bash RC startup that prepends `bootstrap-bin` into the Git Bash `PATH`

Without that, Git Bash can still show:

```text
bash: pnpm: command not found
```

So Git Bash remains available, but it is no longer the preferred productive PNPM lifecycle lane.

## Why `--location=project` matters

The `scriptShell` setting is written at project scope so that:

- the fix is explicit and reviewable
- the project keeps its own lifecycle-shell contract
- the rest of the machine is not silently mutated globally

## Important nuance: `pnpm exec`

`pnpm exec` remains a separate proof surface from plain lifecycle execution.

That is exactly why the current contract had to be proven with:

- `pnpm exec nx --version`
- target-cwd `pnpm exec tsx --version`
- target-cwd `pnpm exec tsx tooling/run/cli.ts --help`

and not only with `pnpm install`.

Also note:

- repo-root `pnpm exec ...` and target-cwd `pnpm exec ...` are not automatically equivalent
- the productive proof must use the same working-directory shape that the real project target uses

For the Nx-specific command-surface split, read:

- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\execution-surfaces.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\runtime-contract.md`

## Related

- `docs\cli\shell\general.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\install.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\clean-reinstall.md`
- `docs\troubleshooting\sandboxie\process-spawning\cmd-based-shells.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\execution-surfaces.md`
