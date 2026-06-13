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

## Current install-lane decision

For `pnpm install` and `pnpm` clean reinstall, the current preferred lifecycle lane is explicit box-local Git Bash.

Representative contract:

```powershell
if ([string]::IsNullOrWhiteSpace($env:BOXED_GIT_ROOT)) {
  throw 'BOXED_GIT_ROOT was not initialized by project bootstrap.'
}

$bashExe = Join-Path $env:BOXED_GIT_ROOT 'bin\bash.exe'
if (-not (Test-Path -LiteralPath $bashExe)) {
  $bashExe = Join-Path $env:BOXED_GIT_ROOT 'usr\bin\bash.exe'
}

if (-not (Test-Path -LiteralPath $bashExe)) {
  throw 'Local boxed Git Bash executable not found.'
}

pnpm config set --location=project scriptShell "$bashExe"
pnpm install
```

This does **not** mean that Git Bash is the only valid shell in the boxed-owned-toolchain method.

It means only that the lifecycle surface inside `pnpm install` / clean reinstall is currently healthier on explicit Git Bash than on the boxed-CMD lane.

## Why Git Bash is preferred here

The current repository evidence and handoff chain support this interpretation:

- boxed PowerShell was already not the validated generic lifecycle lane
- boxed `cmd.exe` remains useful interactively and as an explicit helper lane
- but using boxed `cmd.exe` as the primary PNPM lifecycle shell pushed the install path toward shell-oriented hangs and manual recovery patterns
- the realistic boxed-CMD escape hatch is to suppress lifecycle/postinstall work broadly and replay required steps manually
- that is not the preferred architecture because it turns the project into a custom lifecycle orchestrator
- Git Bash lets the normal lifecycle run much further and narrows the remaining failures to smaller package/runtime-specific follow-up surfaces

## What boxed CMD means now

Boxed `cmd.exe` is **not** removed from the architecture.

It still matters for:

- bootstrap-owned `ComSpec` / `COMSPEC` on separate Windows child-process surfaces
- `Initialize-NodeGypWindowsBuildEnvironment` via boxed `cmd.exe`
- cleanup fallback in uninstall scripts
- explicit CMD terminal profiles

But as the **primary** PNPM install/reinstall lane it is now treated as:

- diagnostic or fallback-only
- only realistically viable if lifecycle/postinstall steps are suppressed and replayed explicitly
- therefore **not** the preferred lifecycle contract

## Current expected post-install interpretation

Git Bash is the preferred install/reinstall lane, but it is not a blanket green guarantee.

The current documented follow-up surfaces are:

- optional/native packages such as `cpu-features`, `canvas`, `msgpackr-extract`, and `lmdb` can still surface as skipped or require separate verification
- Electron can still end in a partially materialized state and must then be verified and repaired explicitly

That is still a better failure class than boxed CMD, because the generic lifecycle has run and the remaining work is narrower and traceable.

For the Electron-specific surface, the current documentation now supports two shapes:

1. trigger the Electron post-install script manually later
2. wire that post-install script directly into the project-owned install / clean-reinstall scripts

The Electron-domain source of truth for that code now lives here:

- `docs\applications\programming-languages\node\dependencies\frameworks\electron\architectures\boxed-owned-toolchain\scripts\post-install.md`

## Relationship to `ComSpec` / `COMSPEC`

The bootstrap-owned `ComSpec` / `COMSPEC` contract remains a **separate** Windows child-process surface.

It can still point to boxed `cmd.exe` for other validated command paths.

PNPM lifecycle execution does **not** have to inherit that same lane, because project-owned `scriptShell` takes precedence for the install/reinstall surface.

## Git Bash command-surface requirements

If Git Bash is selected directly, the boxed-owned-toolchain contract still requires:

- a Windows wrapper such as `pnpm.cmd`
- a shell-native wrapper such as `pnpm`
- Bash RC startup that prepends `bootstrap-bin` into the Git Bash `PATH`

Without that, Git Bash can still show:

```text
bash: pnpm: command not found
```

## Why `--location=project` matters

The `scriptShell` setting is written at project scope so that:

- the fix is explicit and reviewable
- the project keeps its own lifecycle-shell contract
- the rest of the machine is not silently mutated globally

## Important nuance: `pnpm exec`

`pnpm exec` remains a separate proof surface from plain lifecycle execution.

Returning install/reinstall to Git Bash does **not** by itself settle every broader `pnpm exec` / Nx / child-process lane.

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
