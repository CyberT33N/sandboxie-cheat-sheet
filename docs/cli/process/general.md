# Terminate Processes in a Specific Box

## Scope

This document is the single source of truth for **programmatic process termination** in Sandboxie boxes for this repository.

Use this area when the goal is:

- stop all processes in a specific box
- inspect process IDs in a specific box
- terminate only selected processes, such as `node.exe`, in a specific box
- do this from the host system or from a boxed terminal

## Why this lives under `docs\cli`

This capability is cross-cutting:

- it is not specific to VS Code
- it is not specific to Nx
- it is not specific to one project box
- it can apply to maintenance, project, runner, or future utility boxes

That makes it a CLI/control-plane concern rather than an application-domain concern.

## Relationship to other CLI documents

Read these alongside this document when needed:

- `docs\cli\start\general.md`
- `docs\cli\terminal\general.md`
- `docs\cli\shell\general.md`

The split is:

- `start\` owns how a new process is launched into a box
- `terminal\` owns how an interactive shell is opened and used
- `process\` owns how already-running boxed processes are enumerated and terminated

## Sandboxie primitives

Sandboxie documents two relevant `Start.exe` primitives here:

- `Start.exe /box:BOX_NAME /listpids`
- `Start.exe /box:BOX_NAME /terminate`

`/listpids` returns:

1. first line = number of sandboxed processes
2. remaining lines = one PID per line

`/terminate` stops **all** processes in the named sandbox.

## Preferred selective pattern

The preferred repository pattern is:

1. ask Sandboxie which PIDs belong to the box
2. filter those PIDs down to the target process name
3. terminate only that narrowed set

This is preferable to terminating the whole sandbox when only one runtime family, such as `node.exe`, is stale.

## Host-side selective `node.exe` termination

Representative host-side command for the project box `VS_CODE_PRIVADENT_MONO`:

```powershell
$box='VS_CODE_PRIVADENT_MONO'; $boxPids = (& "C:\Program Files\Sandboxie-Plus\Start.exe" /box:$box /listpids | Where-Object { $_ -match '^\d+$' } | Select-Object -Skip 1 | ForEach-Object { [int]$_ }); Get-CimInstance Win32_Process -Filter "Name='node.exe'" | Where-Object { $boxPids -contains $_.ProcessId } | ForEach-Object { Stop-Process -Id $_.ProcessId -Force }
```

## Host-side preview before termination

If you want to inspect the matching boxed `node.exe` processes first:

```powershell
$box='VS_CODE_PRIVADENT_MONO'; $boxPids = (& "C:\Program Files\Sandboxie-Plus\Start.exe" /box:$box /listpids | Where-Object { $_ -match '^\d+$' } | Select-Object -Skip 1 | ForEach-Object { [int]$_ }); Get-CimInstance Win32_Process -Filter "Name='node.exe'" | Where-Object { $boxPids -contains $_.ProcessId } | Select-Object ProcessId, Name, CommandLine
```

## Box-terminal selective `node.exe` termination

The same selective pattern can also be run **inside** a boxed PowerShell terminal, as long as `Start.exe` is available on the machine:

```powershell
$box='VS_CODE_PRIVADENT_MONO'; $boxPids = (& "C:\Program Files\Sandboxie-Plus\Start.exe" /box:$box /listpids | Where-Object { $_ -match '^\d+$' } | Select-Object -Skip 1 | ForEach-Object { [int]$_ }); Get-CimInstance Win32_Process -Filter "Name='node.exe'" | Where-Object { $boxPids -contains $_.ProcessId } | ForEach-Object { Stop-Process -Id $_.ProcessId -Force }
```

This is the preferred in-box command when you want to kill stale boxed Node runtimes without killing every process in the sandbox.

## Broad sandbox termination fallback

If selective process termination is not sufficient and you deliberately want to stop **all** programs in the sandbox:

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" /box:VS_CODE_PRIVADENT_MONO /terminate
```

This is broader than the preferred `node.exe`-only path.

## Architectural interpretation

The repository interpretation is:

- use selective termination first when only one runtime family is stale
- use full sandbox termination only when the whole box state must be reset
- do not change productive project behavior just because a stale boxed process kept files or ports open

## Related

- `docs\cli\general.md`
- `docs\cli\start\general.md`
- `docs\cli\terminal\general.md`
- `upstream\sandboxie-docs\docs\Content\StartCommandLine.md`
