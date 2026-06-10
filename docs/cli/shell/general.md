# Shell Selection And Command Surface

## Purpose

This document is the single source of truth for cross-cutting shell-selection behavior in this repository.

It belongs under `docs\cli\shell\` because the capability is **not** application-specific.

It applies across:

- VS Code integrated terminals
- boxed PowerShell terminals
- boxed Git Bash terminals
- child-process execution from Node
- Nx `run-commands`
- PNPM lifecycle and command execution
- bootstrap-generated command wrappers

## Why this does not belong under `applications\terminal`

The `applications\terminal` area is consumer-specific and currently Starship-focused.

This problem is wider than a terminal application or prompt tool.

The current capability is:

- choose which shell or command interpreter a boxed workflow should use
- decide how child processes discover that interpreter
- decide how commands like `nx` and `pnpm` resolve across PowerShell, CMD, and Git Bash

That makes this a **CLI control-plane concern**, not a terminal-application concern.

From a domain-driven perspective:

- `applications\terminal\...` is for application-domain documentation such as Starship
- `docs\cli\shell\...` is for the cross-cutting shell and command-interpreter contract

## Problem classes that must be separated

This repository now treats the following as separate problem classes:

1. **interactive terminal shell**
   - what shell the developer sees and types into
2. **child-process shell selection**
   - what shell a runtime such as Node uses when it performs shell-based execution
3. **command-surface resolution**
   - how `nx`, `pnpm`, `node20`, and similar commands are discovered in PowerShell, CMD, and Git Bash
4. **console-host / terminal-attach behavior**
   - PCA / conhost / integrated-terminal specifics

Those surfaces are related, but they are not the same.

## What `ComSpec` / `COMSPEC` is

On Windows, `ComSpec` is the environment variable that points to the default command interpreter.

Typical host default:

```text
C:\Windows\System32\cmd.exe
```

Why this matters:

- many Windows-oriented tools do not directly `spawn()` an executable
- they use a shell-oriented execution path instead
- Node `child_process.exec()` on Windows uses the command interpreter selected through `ComSpec`
- Nx `run-commands` / `command` targets ultimately sit on that kind of shell path

So if the boxed environment leaves `ComSpec` pointing at a Windows shell that cannot be spawned as a child process under strict Sandboxie policy, shell-based tools can fail even when the underlying command logic is healthy.

## Validated current repository finding

The validated boxed result is:

- direct script execution can work
- `node -> spawn(cmd.exe)` can still fail with `spawn EPERM`
- `node -> spawn(bash)` can still work
- `ComSpec=bash` can make shell-based Nx targets succeed

That means the current first-class fix is **not** “loosen the sandbox”.

It is:

- make the boxed shell-selection contract explicit

## Current prioritized solution

The current prioritized repository solution is:

1. keep the sandbox strict
2. keep using box-local mirrored Git Bash as the trusted shell runtime
3. set **both**:
   - `ComSpec`
   - `COMSPEC`
   to the box-local Git Bash executable during bootstrap
4. keep command resolution explicit through bootstrap-generated wrappers for all relevant shell families

Current bootstrap rule:

```powershell
$boxedComSpec = Join-Path $nodeRuntime.GitRoot 'bin\bash.exe'
if (-not (Test-Path -LiteralPath $boxedComSpec)) {
  $boxedComSpec = Join-Path $nodeRuntime.GitRoot 'usr\bin\bash.exe'
}
if (-not (Test-Path -LiteralPath $boxedComSpec)) {
  throw 'Local boxed Git Bash executable not found for ComSpec override.'
}

$env:ComSpec = $boxedComSpec
$env:COMSPEC = $boxedComSpec
$env:BOXED_COMSPEC = $boxedComSpec
```

This keeps the command-interpreter selection:

- explicit
- local to the box
- bootstrap-owned
- independent from host fallback

## Why wrappers are still required

The `ComSpec` override is necessary, but not sufficient on its own.

The current validated state also requires bootstrap-generated wrappers:

- shell-native wrappers for Git Bash
- `.cmd` wrappers for CMD compatibility surfaces
- `.ps1` wrappers for PowerShell command resolution

Current examples:

- `nx`
- `nx.cmd`
- `nx.ps1`
- `pnpm`
- `pnpm.cmd`
- `pnpm.ps1`
- `node20`
- `node20.cmd`
- `node20.ps1`

Why this matters:

- once `COMSPEC` points to Git Bash, PowerShell must not be forced to rely only on `.cmd` wrappers
- command-surface behavior should stay explicit instead of depending on fragile host defaults

## Current validated result

After the bootstrap-level `ComSpec` override and PowerShell-native wrappers were added, the following were validated in the boxed project shell:

```powershell
nx --version
nx run backend:port-guard --output-style=stream
nx run frontend:port-guard --output-style=stream
nx run test:smoke-electron-runtime --output-style=stream
```

All of those commands succeeded.

That is why this is now the **prioritized documented solution**.

## What this solution is not

This is **not**:

- a Sandboxie.ini option that officially remaps the default Windows command interpreter
- a signal to weaken isolation
- a justification for switching to Green Box / Application Compartment mode as the baseline
- a reason to refactor project orchestration away from legitimate Nx usage

## Official-option status

At the current documented state:

- there is **no** official Sandboxie documentation page for a config key that directly remaps child-process shell selection to a custom interpreter like Git Bash
- there is **no** official documented Sandboxie.ini equivalent of “set command interpreter to Bash for boxed child processes”

Related but different official / quasi-official knobs:

- `NoRestartOnPCA=y`
  - known from official Sandboxie GitHub issues
  - relevant to VS Code integrated-terminal attach / popup behavior
  - **not** the same as `ComSpec` shell remapping
- `DropConHostIntegrity=y`
  - relevant to console-host / `ReadConsoleOutput` issues
  - **not** the same as shell-selection remapping
- `NoSecurityIsolation`, `DropChildProcessToken`
  - compatibility levers that weaken security isolation
  - **not** the preferred answer for this repository

## Twelve-factor reading

This repository treats shell selection as runtime configuration.

That means the correct place for it is:

- bootstrap environment injection

not:

- hidden workstation defaults
- scattered editor-only settings
- implicit host shell behavior

So this central shell contract is a valid twelve-factor-style runtime rule:

- explicit
- environment-driven
- reproducible
- reviewable

## Relationship to troubleshooting notes

The troubleshooting note here:

- `docs\troubleshooting\sandboxie\process-spawning\cmd-based-shells.md`

remains useful for diagnosing the failure class.

But the **current preferred repository answer** now lives here in `docs\cli\shell\general.md`, because it is no longer just a symptom log.

It is now a governed control-plane rule.

## Related

- `docs\cli\general.md`
- `docs\cli\terminal\general.md`
- `docs\troubleshooting\sandboxie\process-spawning\cmd-based-shells.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\execution-surfaces.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\bootstrap-integration.md`
