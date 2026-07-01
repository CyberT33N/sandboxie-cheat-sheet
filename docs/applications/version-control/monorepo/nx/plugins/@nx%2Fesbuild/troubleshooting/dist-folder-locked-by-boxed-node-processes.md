# `dist` Folder Locked By Boxed `node.exe` Processes

## Scope

This document owns the boxed-owned-toolchain troubleshooting path for a concrete `@nx/esbuild` failure class:

- the executor tries to remove the existing `dist` directory
- the removal fails because a boxed `node.exe` process still holds files or handles below that output tree

This is a plugin-specific troubleshooting note, not a reason to change productive project behavior.

## Symptom

Representative failure shape:

```text
Error: ENOTEMPTY, Directory not empty: \\?\C:\Users\denni\source\privadent-mono\apps\privadent\backend\dist
    at rmSync (node:fs:1206:18)
    at esbuildExecutor (...\@nx\esbuild\dist\src\executors\esbuild\esbuild.impl.js:...)
```

## Current interpretation

When this happens only inside the boxed workflow while the same project runs normally on the host, the first interpretation in this repository is:

- stale boxed `node.exe` processes are still alive
- one of those processes still keeps the output tree open
- `@nx/esbuild` then fails during output cleanup before the next serve/build cycle can continue

This is treated as a boxed process-lifecycle problem.

It is **not** the first-class answer to:

- change productive `@nx/esbuild` behavior
- stop deleting `dist`
- weaken the normal project build contract only to satisfy the sandbox

## Required boxed recovery step

Before retrying the Nx command, terminate the stale boxed `node.exe` processes for the affected sandbox from the **host system**.

Representative command for the project box `VS_CODE_PRIVADENT_MONO`:

```powershell
$box='VS_CODE_PRIVADENT_MONO'; $boxPids = (& "C:\Program Files\Sandboxie-Plus\Start.exe" /box:$box /listpids | Where-Object { $_ -match '^\d+$' } | Select-Object -Skip 1 | ForEach-Object { [int]$_ }); Get-CimInstance Win32_Process -Filter "Name='node.exe'" | Where-Object { $boxPids -contains $_.ProcessId } | ForEach-Object { Stop-Process -Id $_.ProcessId -Force }
```

## Optional preview step

If you want to inspect which boxed `node.exe` processes would be terminated first:

```powershell
$box='VS_CODE_PRIVADENT_MONO'; $boxPids = (& "C:\Program Files\Sandboxie-Plus\Start.exe" /box:$box /listpids | Where-Object { $_ -match '^\d+$' } | Select-Object -Skip 1 | ForEach-Object { [int]$_ }); Get-CimInstance Win32_Process -Filter "Name='node.exe'" | Where-Object { $boxPids -contains $_.ProcessId } | Select-Object ProcessId, Name, CommandLine
```

## Coarse fallback

If selective termination is not sufficient and you intentionally want to stop **all** programs in that sandbox, Sandboxie documents this host-side command:

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" /box:VS_CODE_PRIVADENT_MONO /terminate
```

This is broader than the preferred `node.exe`-only recovery step.

## What to do next

After the stale boxed `node.exe` processes are terminated:

1. reopen a fresh boxed terminal or boxed VS Code session if needed
2. rerun the affected Nx command
3. only if the problem still reproduces, continue with deeper plugin/runtime diagnosis

## What not to conclude

Do **not** conclude from this failure alone that the correct repository answer is:

- `deleteOutputPath: false`
- disabling normal `dist` cleanup permanently
- changing the productive project build contract only because the sandbox kept a process alive

The current repository interpretation is narrower:

- first restore a clean boxed process state
- then rerun the normal build/serve path

## Related

- `docs\applications\version-control\monorepo\nx\plugins\@nx%2Fesbuild\overview.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\bootstrap-integration.md`
- `docs\cli\start\general.md`
- `upstream\sandboxie-docs\docs\Content\StartCommandLine.md`
