# PNPM Lifecycle And Command Surface

## The actual boxed failure class

The main validated blocker was **not** PNPM package resolution itself.

The blocked surface was the Windows lifecycle shell path under Sandboxie.

Validated behavior in the boxed project shell:

1. `cmd.exe` worked when launched manually
2. `node -> spawn(local node.exe)` worked
3. `node -> spawn(cmd.exe)` failed with `spawn EPERM`
4. `node -> spawn(powershell.exe)` failed with `spawn EPERM`
5. `node -> spawn(box-local bash.exe)` worked

Current refined understanding:

- the failing entries above were the problematic **host/system-shell spawn surfaces**
- this must not be generalized into "PowerShell/CMD are impossible"
- the later validated box-local mirrored `cmd.exe` / PowerShell lanes prove that those shells can work when they are projected explicitly into the boxed runtime

That means the root problem was:

- not a general Node child-process failure
- not a broken local toolchain mirror
- not a generic PNPM command failure
- but a shell-spawn problem for host system shells during Windows lifecycle execution

The cross-cutting Sandboxie write-up for that issue lives here:

- `docs\troubleshooting\sandboxie\process-spawning\cmd-based-shells.md`
- `docs\cli\shell\general.md`

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

## Additional integrated Git Bash command-surface fix

After the project contract, lifecycle-shell setting, and boxed terminal startup were already working, one more shell-specific failure was validated:

```text
bash: pnpm: command not found
```

This appeared in the integrated Git Bash terminal even though:

- the correct shared `PnpmCli` had been selected
- the Node runtime was available
- `pnpm.cmd` had been generated in `bootstrap-bin`

The reason was:

1. a `.cmd` wrapper alone is not a sufficient bare-command surface for Git Bash
2. the Bash startup files also need the local `bootstrap-bin` directory on the shell `PATH`

So the current boxed-owned-toolchain contract now requires both:

- a Windows wrapper such as `pnpm.cmd`
- a shell-native wrapper such as `pnpm`

and the Bash RC files must prepend `bootstrap-bin` before interactive Git Bash commands are resolved.

## Why this must be done

Without this fix, the architecture becomes inconsistent:

- PowerShell/CMD can resolve the bootstrap-generated command surface
- but the integrated Git Bash shell, which remains the default boxed VS Code shell-oriented lane, cannot

That would make the boxed Git Bash terminal an incomplete toolchain surface even though the project contract itself is correct.

So the fix is not cosmetic. It is required so that `pnpm` is actually available in the default integrated Git Bash lane the architecture currently uses for shell-oriented task execution.

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

For the current Nx proof-path and environment contract, read:

- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\execution-surfaces.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\runtime-contract.md`

## Related

- `docs\cli\shell\general.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\scripts\install.md`
- `docs\troubleshooting\sandboxie\process-spawning\cmd-based-shells.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\execution-surfaces.md`
