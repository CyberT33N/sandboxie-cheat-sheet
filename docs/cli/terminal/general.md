# Open a Terminal in a Specific Box

## Scope

This document is the single source of truth for the generic terminal-launch pattern in this repository.

Use this area when the goal is:

- open an interactive terminal in a specific box
- keep the terminal open
- see stdout/stderr directly
- run commands manually inside the box
- understand command-passing boundaries

## Relationship to `Start.exe`

A boxed terminal is still started through `Start.exe`, but terminal usage is treated as its own subdomain because it adds operational behavior beyond simple process launch.

Read first:

- `docs\cli\start\general.md`

## Core rule

`Start.exe` can pass arguments to the new shell process it creates.

`Start.exe` cannot later push new commands into an already-open terminal session.

That means the two stable patterns are:

1. launch the terminal, then type or paste commands inside it
2. launch a one-shot shell command at startup

## Open PowerShell in a specific box

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:MY_BOX `
  "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
  -NoLogo `
  -NoExit
```

Use `-NoExit` when the shell should remain visible after the command flow finishes.

## Open CMD in a specific box

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:MY_BOX `
  "C:\Windows\System32\cmd.exe"
```

## Recommended debugging pattern

When output visibility matters, prefer:

1. open the boxed terminal
2. run the real command inside that terminal
3. inspect stdout/stderr directly
4. check the shell exit status

For PowerShell:

```powershell
$LASTEXITCODE
```

This pattern is the most reliable for diagnosing wrapper, quoting, or toolchain problems.

## One-shot command pattern

If interactive diagnosis is not needed, launch a command immediately:

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:MY_BOX `
  "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
  -NoLogo `
  -ExecutionPolicy Bypass `
  -Command "Get-Date"
```

Or with `cmd.exe`:

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:MY_BOX `
  "C:\Windows\System32\cmd.exe" `
  /c dir
```

## Returning an exit code through `Start.exe`

If the caller needs the child exit code, use `/wait`:

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:MY_BOX `
  /wait `
  "C:\Windows\System32\cmd.exe" `
  /c exit 9
```

This is useful for:

- wrappers
- scripted validation
- automated checks

## Architectural recommendation

Keep these two concerns separate on purpose:

- general box/child-process launch
- terminal-specific interaction and visibility

Do not overload a single generic passthrough mechanism for both.

Application-specific areas should reference this document and then add only their workload-specific inner commands.
