# Boxed-Owned-Toolchain PNPM Overview

## Architectural status

This is the **preferred PNPM method** in this repository.

Use this document first when the architecture is:

- fully boxed authoring
- box-local runtime mirroring
- box-local mutable working state
- explicit promotion instead of live writing back into shared state

The host-sync PNPM write-up remains only as a secondary reference here:

- `docs\applications\programming-languages\node\package-manager\pnpm\host-sync\overview.md`

## Current recommendation

For the current boxed-owned-toolchain architecture:

- use the mirrored local `pnpm` command surface prepared by bootstrap
- keep the default per-box PNPM store
- do **not** introduce a host-shared PNPM content-addressable store as the baseline

Why:

1. a host-shared store widens the blast radius across boxes
2. cached package payloads become reusable across trust boundaries
3. one compromised or drifted store surface can spread to multiple boxes

So the preferred baseline is:

- normal `pnpm install`
- no shared `--store-dir`
- one store per box

## The actual problem that blocked `pnpm install`

The key failure was **not** the package-manager itself.

The blocked surface was the Windows lifecycle shell path.

Validated behavior in the boxed project shell:

1. `cmd.exe` worked when launched manually
2. `node -> spawn(local node.exe)` worked
3. `node -> spawn(cmd.exe)` failed with `spawn EPERM`
4. `node -> spawn(powershell.exe)` failed with `spawn EPERM`
5. `node -> spawn(box-local bash.exe)` worked

That means the problem was:

- not a general Node child-process failure
- not a broken local toolchain mirror
- not a generic `pnpm` command failure
- but a shell-spawn problem for host system shells during Windows lifecycle execution

The cross-cutting Sandboxie write-up for that class of issue lives here:

- `docs\troubleshooting\sandboxie\process-spawning\cmd-based-shells.md`

## Why this affects `pnpm install`

On Windows, PNPM lifecycle scripts normally use `cmd.exe` as their shell surface.

In this boxed-owned-toolchain setup, that shell-spawn path failed even though:

- the interactive project terminal worked
- the local mirrored Node runtime worked
- the local mirrored `pnpm` command worked

So `pnpm install` succeeded only after lifecycle execution was redirected to a **box-local** shell executable that Node could actually spawn.

## Validated working commands

The following commands are the exact working project-shell sequence that validated the fix.

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

That is the correct governance direction for this repository.

## Optional cleanup command

If you intentionally want to remove the setting again later:

```powershell
pnpm config delete --location=project scriptShell
```

## What this fix does *not* mean

This does **not** mean:

- that every package issue is solved
- that every optional native dependency will now succeed
- that the boxed architecture should move back to a shared PNPM store

It means only this:

- the main lifecycle shell blocker for `pnpm install` was resolved

Any remaining failed optional native installs must now be analyzed individually.

## Current follow-up verification

After the lifecycle-shell fix, the next mandatory validation steps are:

1. inspect optional native installs that were skipped
2. verify `nx` commands directly under the new shell setup:
   - `pnpm exec nx --version`
   - `pnpm exec nx report`
   - `pnpm exec nx show projects`

## Related

- `docs\troubleshooting\sandboxie\process-spawning\cmd-based-shells.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\pnpm.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\node.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\git.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\host-sync\overview.md`
