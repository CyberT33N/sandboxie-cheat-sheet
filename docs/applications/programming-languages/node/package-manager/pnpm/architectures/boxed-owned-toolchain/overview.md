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

The governed PNPM runtime surface shape is:

```text
C:\shared\sandbox-toolchains\dev\pnpm\<version>\package\bin\pnpm.cjs
```

It is hosted by the governed shared Node runtime rather than by `@pnpm/exe`.

The current project-specific example contract now points at `11.5.0`.

## Provisioning

```powershell
$Node26Root = "C:\shared\sandbox-toolchains\dev\node\26.2.0\node-v26.2.0-win-x64"
$PnpmVersion = "11.5.0"
$PnpmRoot = "C:\shared\sandbox-toolchains\dev\pnpm\$PnpmVersion"

New-Item -ItemType Directory -Force -Path $PnpmRoot | Out-Null
Remove-Item "$PnpmRoot\*" -Recurse -Force -ErrorAction SilentlyContinue

Push-Location $PnpmRoot
& "$Node26Root\npm.cmd" pack "pnpm@$PnpmVersion"
tar -xf "pnpm-$PnpmVersion.tgz"
Pop-Location
```

## Host-side provisioning of another project-specific version

The shared `dev\pnpm\` area may keep multiple PNPM versions side by side.

If one project contract now requires `11.5.0`, provision it as an additional governed version instead of mutating older PNPM subtrees in place.

Sanitized host-side example:

```powershell
$SharedRoot = "C:\shared\sandbox-toolchains"
$Node26Root = Join-Path $SharedRoot "dev\node\26.2.0\node-v26.2.0-win-x64"
$PnpmVersion = "11.5.0"
$PnpmRoot = Join-Path $SharedRoot "dev\pnpm\$PnpmVersion"

New-Item -ItemType Directory -Force -Path $PnpmRoot | Out-Null
Remove-Item "$PnpmRoot\*" -Recurse -Force -ErrorAction SilentlyContinue

Push-Location $PnpmRoot
& "$Node26Root\npm.cmd" pack "pnpm@$PnpmVersion"
tar -xf "pnpm-$PnpmVersion.tgz"
Pop-Location

& "$Node26Root\node.exe" "$PnpmRoot\package\bin\pnpm.cjs" --version
```

Expected verification:

```text
11.5.0
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
- but the integrated Git Bash shell, which is now the preferred boxed VS Code terminal, cannot

That would make the boxed Git Bash terminal an incomplete toolchain surface even though the project contract itself is correct.

So the fix is not cosmetic. It is required so that `pnpm` is actually available in the shell the architecture chose as the normal integrated terminal.

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

## How the effective PNPM version is actually selected

In the current boxed-owned-toolchain contract, the generic bootstrap does **not** choose a PNPM version by itself.

Instead:

1. the shared `dev\pnpm\<version>\package\bin\pnpm.cjs` inventory provides the available governed versions
2. the project adapter selects one exact `PnpmCli` path
3. the reusable bootstrap mirrors that selected CLI locally and generates the boxed `pnpm.cmd` wrapper from it

That means the effective project-box PNPM version is currently selected through the **project bootstrap contract**, not by running `pnpm self-update` or `npm install -g pnpm` inside the box.

Architecturally:

- shared `dev\pnpm\...` = provisioned binary inventory
- project adapter = project-specific version contract
- generic bootstrap = contract materialization and local wrapper generation

## Sandboxie visibility range

When the PNPM version is changed in the project contract, the relevant box would also have to be updated **if** the Sandboxie rule were pinned to one exact PNPM version directory.

That is why the recommended Sandboxie visibility rule is **not**:

```ini
ReadFilePath=C:\shared\sandbox-toolchains\dev\pnpm\11.5.0\
```

but:

```ini
ReadFilePath=C:\shared\sandbox-toolchains\dev\pnpm\
```

Why this is the better governance rule:

1. PNPM often needs security updates
2. version-pinned box rules would otherwise require repeated manual config churn
3. the project contract already selects the exact `PnpmCli` version
4. the broader PNPM subtree rule keeps visibility narrow enough while avoiding needless box-rule edits

So the intended split is:

- **exact version selection** = bootstrap/project contract
- **broad PNPM visibility range** = Sandboxie box rule

## What to change when a project must move to a newer PNPM version

The current boxed-owned-toolchain sequence is:

1. provision `11.5.0` in `C:\shared\sandbox-toolchains\dev\pnpm\11.5.0\...`
2. update the project adapter so `PnpmCli` points to `dev\pnpm\11.5.0\package\bin\pnpm.cjs`
3. restart the project box through the normal bootstrap launcher

After that, the boxed `pnpm` command resolves to the newly selected shared CLI because the bootstrap-generated wrapper will be rebuilt from the updated project contract.

## Why `pnpm self-update` or `npm install -g pnpm` did not fix the boxed command

In this architecture, the boxed `pnpm` command is not supposed to be the machine-global PNPM installation.

It is the bootstrap-generated command surface for the currently selected shared `PnpmCli`.

So commands such as:

- `pnpm self-update 11.5.0`
- `npm install -g pnpm@11.5.0`

do not change the project-box contract by themselves when the project bootstrap still points at some other governed PNPM version.

## Architectural recommendation

For a one-box-one-repo boxed-owned-toolchain contract, the more correct architecture is:

- provision multiple governed PNPM versions side by side under `dev\pnpm\...`
- declare the selected version in the project adapter / box contract
- let the generic bootstrap materialize that declared contract

This is preferable to making the everyday host launch command choose the PNPM version ad hoc, because:

1. the project contract stays explicit and reviewable
2. one box keeps one deterministic runtime/toolchain contract
3. restart behavior is reproducible
4. the box does not silently drift because of caller-provided launch arguments

## Governance for install automation

The same rule applies to host-triggered dependency installation:

- the host may launch the install flow
- but the install behavior should live in a **project-owned PS1**
- and that PS1 should consume the project contract instead of overriding it through host parameters

So the preferred shape is:

1. project adapter selects `PnpmCli`
2. project-owned install PS1 enters the project box through the normal bootstrap
3. that install PS1 sets the validated PNPM lifecycle `scriptShell`
4. that install PS1 runs `pnpm install`

The single source of truth for the sanitized project-box install PS1 body and its host launch command is documented here:

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\scripts.md`

## Update workflow summary

When a project wants to move to a newer PNPM version, the repeatable workflow is:

1. provision the new governed PNPM binary in `dev\pnpm\...` using the provisioning script above
2. update the project contract so `PnpmCli` points at the new version
3. restart the project box through the normal bootstrap launcher
4. run the project-owned install PS1 documented in the boilerplate scripts area

This document owns **step 1**.

The project-box install script document owns **step 4**:

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\scripts.md`

## Optional override stance

An argument-based host override can exist as a **break-glass or migration tool**, but it should not be the normal contract.

If such an override is ever introduced, it should be treated as:

- exceptional
- explicit
- validated against the project contract
- and preferably rejected unless the caller intentionally opts into a temporary override mode

That keeps the default architecture deterministic while still leaving room for controlled maintenance workflows.

## Related

- `docs\applications\programming-languages\node\package-manager\pnpm\general.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\host-sync\overview.md`
- `docs\troubleshooting\sandboxie\process-spawning\cmd-based-shells.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\overview.md`
