# Boxed-Owned-Toolchain PNPM Overview

## Architectural status

This is the **preferred PNPM architecture path** in this repository.

Use this document when the architecture is:

- fully boxed authoring
- local runtime mirroring
- local mutable working state
- explicit promotion instead of live authoring against shared state

The host-sync PNPM write-up remains only as a secondary reference here:

- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\host-sync\overview.md`

## Current runtime contract

The governed PNPM runtime surface is:

```text
C:\shared\sandbox-toolchains\dev\pnpm\11.2.2\package\bin\pnpm.cjs
```

It is hosted by the governed shared Node runtime rather than by `@pnpm/exe`.

## Provisioning

```powershell
$Node26Root = "C:\shared\sandbox-toolchains\dev\node\26.2.0\node-v26.2.0-win-x64"
$PnpmVersion = "11.2.2"
$PnpmRoot = "C:\shared\sandbox-toolchains\dev\pnpm\$PnpmVersion"

New-Item -ItemType Directory -Force -Path $PnpmRoot | Out-Null
Remove-Item "$PnpmRoot\*" -Recurse -Force -ErrorAction SilentlyContinue

Push-Location $PnpmRoot
& "$Node26Root\npm.cmd" pack "pnpm@$PnpmVersion"
tar -xf "pnpm-$PnpmVersion.tgz"
Pop-Location
```

## Preferred operational posture

For the current boxed-owned-toolchain architecture:

- use the locally mirrored `pnpm` command surface prepared by bootstrap
- keep the default per-box PNPM store
- do **not** introduce a host-shared PNPM content-addressable store as the baseline

Why:

1. a host-shared store widens the blast radius across boxes
2. cached package payloads become reusable across trust boundaries
3. one compromised or drifted shared store surface can spread to multiple boxes

So the preferred baseline is:

- normal `pnpm install`
- no shared `--store-dir`
- one store per box

## The actual boxed failure class

The main validated blocker was **not** PNPM package resolution itself.

The blocked surface was the Windows lifecycle shell path under Sandboxie.

Validated behavior in the boxed project shell:

1. `cmd.exe` worked when launched manually
2. `node -> spawn(local node.exe)` worked
3. `node -> spawn(cmd.exe)` failed with `spawn EPERM`
4. `node -> spawn(powershell.exe)` failed with `spawn EPERM`
5. `node -> spawn(box-local bash.exe)` worked

That means the root problem was:

- not a general Node child-process failure
- not a broken local toolchain mirror
- not a generic PNPM command failure
- but a shell-spawn problem for host system shells during Windows lifecycle execution

The cross-cutting Sandboxie write-up for that issue lives here:

- `docs\troubleshooting\sandboxie\process-spawning\cmd-based-shells.md`

## Validated working lifecycle fix

The validated project-shell sequence was:

### 1. Resolve the box-local Bash executable

```powershell
$bashExe = Join-Path $env:BOXED_LOCAL_TOOLCHAIN_ROOT 'git\2.54.0\bin\bash.exe'
if (-not (Test-Path $bashExe)) {
  $bashExe = Join-Path $env:BOXED_LOCAL_TOOLCHAIN_ROOT 'git\2.54.0\usr\bin\bash.exe'
}

$bashExe
Test-Path $bashExe
& $bashExe -lc 'echo BASH_OK'
```

### 2. Configure PNPM to use the box-local shell for project scripts

```powershell
pnpm config set --location=project scriptShell "$bashExe"
pnpm config get scriptShell
```

### 3. Run the install

```powershell
pnpm install
```

That sequence validated successfully in the boxed project shell.

## Why `--location=project` matters

The setting is written at project scope so that:

- the fix is explicit and reviewable
- the project keeps its own lifecycle-shell contract
- the rest of the machine is not silently mutated globally

## Important nuance: `pnpm exec`

`pnpm exec` remains a separate execution surface from lifecycle shell execution.

Observed boxed result:

- `pnpm exec nx --version` still failed with `spawn EPERM`

Architectural interpretation:

- the `scriptShell` fix resolved lifecycle execution during `pnpm install`
- but `pnpm exec` still uses its own Windows exec path
- so `pnpm exec` must not be treated as the primary proof surface for boxed-owned-toolchain validation

For current Nx proof-path and environment contract, read:

- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\overview.md`

## Current reference truth

The current boxed-owned-toolchain PNPM reference truth is:

1. governed shared `pnpm.cjs`
2. local mirrored command surface from bootstrap
3. default per-box store
4. box-local Bash as the validated lifecycle shell path
5. no shared PNPM store as the default baseline

## Related

- `docs\applications\programming-languages\node\package-manager\pnpm\general.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\host-sync\overview.md`
- `docs\troubleshooting\sandboxie\process-spawning\cmd-based-shells.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\overview.md`
