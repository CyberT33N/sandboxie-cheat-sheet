# Shell Selection And Command Surface

## Purpose

This document is the single source of truth for cross-cutting shell-selection behavior in this repository.

It belongs under `docs\cli\shell\` because the capability is **not** application-specific.

It applies across:

- VS Code integrated terminals
- boxed PowerShell terminals
- boxed `cmd.exe` terminals
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
- manually opened boxed `cmd.exe` can work
- manually opened boxed PowerShell can work
- `node -> spawn(host cmd.exe)` can fail with `spawn EPERM`
- `node -> spawn(host powershell.exe)` can fail with `spawn EPERM`
- `node -> spawn(box-local mirrored cmd.exe)` can succeed
- `node -> spawn(box-local mirrored powershell.exe)` can succeed
- `node -> spawn(bash)` can still work
- historically, `ComSpec=bash` could make shell-based Nx targets succeed
- boxed `CMD + Starship` requires `Clink`

That means the old broad statement:

- "PowerShell/CMD do not work in the boxed-owned-toolchain method"

is not accurate.

The accurate statement is:

- host/system Windows shell spawn surfaces can fail under strict boxed child-process execution
- explicitly mirrored box-local Windows shell lanes can work
- the bootstrap must decide which lane is used for which purpose

That means the current first-class fix is **not** “loosen the sandbox”.

It is:

- make the boxed shell-selection contract explicit

In the current validated repository state, the explicit shell artifacts are governed under:

- `C:\shared\sandbox-toolchains\dev\shells\cmd\...`
- `C:\shared\sandbox-toolchains\dev\shells\powershell\...`
- `C:\shared\sandbox-toolchains\dev\shells\clink\...`

## Shared shell artifact model

The current repository model now treats the Windows shell lanes as governed shared shell artifacts.

Canonical shared shape:

```text
C:\shared\sandbox-toolchains\dev\shells\
  cmd\
    10.0.26100.8457\
      cmd.exe
  powershell\
    10.0.26100.8457\
      powershell.exe
      ...
  clink\
    1.9.26\
      clink_x64.exe
      clink.bat
      ...
```

That means:

- `cmd.exe` is no longer treated as a runtime that bootstrap should source directly from `C:\Windows\System32\...`
- Windows PowerShell is no longer treated as a runtime that bootstrap should source directly from `C:\Windows\System32\WindowsPowerShell\v1.0\...`
- bootstrap now consumes governed shared shell artifacts and mirrors them locally into the box execution tree

## Host-side provisioning of shared Windows shell artifacts

The initial shared provisioning source for the Windows-bundled shell binaries is still the local OS installation, but the **runtime source of truth** becomes the shared shell artifact tree once provisioned.

Representative host-side provisioning:

```powershell
$ShellsRoot = 'C:\shared\sandbox-toolchains\dev\shells'

$CmdVersion = ((Get-Item "$env:SystemRoot\System32\cmd.exe").VersionInfo.FileVersion -split ' ')[0]
$PowerShellVersion = ((Get-Item "$env:SystemRoot\System32\WindowsPowerShell\v1.0\powershell.exe").VersionInfo.FileVersion -split ' ')[0]

$CmdDest = Join-Path $ShellsRoot "cmd\$CmdVersion"
$PowerShellDest = Join-Path $ShellsRoot "powershell\$PowerShellVersion"

New-Item -ItemType Directory -Force -Path $CmdDest, $PowerShellDest | Out-Null

Copy-Item `
  -LiteralPath "$env:SystemRoot\System32\cmd.exe" `
  -Destination (Join-Path $CmdDest 'cmd.exe') `
  -Force

robocopy `
  "$env:SystemRoot\System32\WindowsPowerShell\v1.0" `
  $PowerShellDest `
  /E /R:1 /W:1
```

This is a provisioning step, not the runtime contract itself.

The runtime contract is:

- bootstrap reads from `C:\shared\sandbox-toolchains\dev\shells\...`
- bootstrap mirrors into the local box execution tree
- integrated terminals use the local mirrored shell lanes

## Current prioritized solution

The current prioritized repository solution is:

1. keep the sandbox strict
2. expose three explicit box-local shell lanes:
   - PowerShell
   - `cmd.exe`
   - Git Bash
3. prefer box-local mirrored PowerShell as the default interactive VS Code shell lane
4. keep child-process shell selection explicit and bootstrap-owned
5. set **both**:
   - `ComSpec`
   - `COMSPEC`
   to the box-local interpreter required by the validated shell-oriented child-process surface
6. keep command resolution explicit through bootstrap-generated wrappers for all relevant shell families
7. treat `Clink` as the CMD-specific runtime adapter for the `CMD + Starship` lane

This is intentionally **not** the same as saying:

- Git Bash is the only shell that can work
- or PowerShell/CMD are impossible in the method

Important distinction:

- the preferred user-facing default shell can be PowerShell
- while the bootstrap-owned child-process contract can still use another explicit shell lane where PNPM / Nx verification proved that path first

In the current preferred productive implementation, that bootstrap-owned `ComSpec` contract points to the box-local mirrored `cmd.exe` lane.

Current bootstrap rule:

```powershell
$boxedComSpec = $windowsShellRuntime.CmdExe
if ([string]::IsNullOrWhiteSpace($boxedComSpec) -or -not (Test-Path -LiteralPath $boxedComSpec)) {
  throw 'Local boxed CMD executable not found for ComSpec override.'
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

At the same time, the repository now distinguishes between:

- the **preferred interactive default shell lane**
  - box-local mirrored PowerShell
- the **explicit alternative Windows shell lane**
  - box-local mirrored `cmd.exe`
- the **explicit Git Bash lane**
  - available for shell-native wrappers and shell-oriented child-process flows
- the **bootstrap-owned child-process contract**
  - currently governed through box-local mirrored `cmd.exe` via `ComSpec`
- the **CMD + Starship adapter**
  - `Clink`

This split is the key architectural correction:

- PowerShell, CMD, and Git Bash are all valid shell lanes
- but the preferred interactive default shell is **not** automatically the same thing as the shell-oriented child-process contract used by Nx / shell-based exec paths

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

For the CMD + Starship lane, there is one additional governed dependency:

- `Clink`

Why this matters:

- once `COMSPEC` points to boxed `cmd.exe`, command resolution must still stay explicit across PowerShell, CMD, and Git Bash
- command-surface behavior should stay explicit instead of depending on fragile host defaults

## Current validated result

After the bootstrap-level `ComSpec` override, the Windows-shell mirroring, and the PowerShell-native wrappers were added, the following were validated in the boxed project shell:

```powershell
nx --version
nx run backend:port-guard --output-style=stream
nx run frontend:port-guard --output-style=stream
nx run test:smoke-electron-runtime --output-style=stream
pnpm exec nx --version
```

All of those commands succeeded.

In addition, the explicit local Windows shell lanes were validated through the bootstrap-provided shell paths:

```powershell
node -e "require('child_process').spawn(process.env.BOXED_CMD_EXE,['/d','/c','echo CMD_CHILD_OK'],{stdio:'inherit'})"
node -e "require('child_process').spawn(process.env.BOXED_POWERSHELL_EXE,['-NoLogo','-NoProfile','-Command','Write-Host PS_CHILD_OK'],{stdio:'inherit'})"
```

Those commands also succeeded.

In the target working directory used by the application runner, the current productive CMD-based contract also validated:

```powershell
Set-Location .\apps\test-tooling
pnpm exec tsx --version
pnpm exec tsx tooling/run/cli.ts --help
```

The `--help` probe reached the runner and failed only with the expected command-schema validation output, which means the command surface was reached successfully.

So the current repository view is:

- boxed `cmd.exe` works
- boxed PowerShell works
- integrated VS Code profiles can work when those binaries are mirrored locally and selected explicitly
- the original blocker was not the shell type itself
- the original blocker was the wrong shell-execution surface
- the preferred productive child-process contract is boxed `cmd.exe`
- Git Bash remains an explicit alternative lane, not the preferred productive `ComSpec` contract

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

For the CMD-specific prompt adapter, read:

- `docs\cli\shell\clink.md`

## Related

- `docs\cli\general.md`
- `docs\cli\shell\clink.md`
- `docs\cli\terminal\general.md`
- `docs\troubleshooting\sandboxie\process-spawning\cmd-based-shells.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\execution-surfaces.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\bootstrap-integration.md`
