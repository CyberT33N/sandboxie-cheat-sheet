# Start a Program in a Specific Box

## Scope

This document is the single source of truth for the generic CLI pattern that starts a new process in a specific Sandboxie box.

This applies across workloads:

- VS Code
- maintenance shells
- project shells
- runner flows
- diagnostics
- future non-VS-Code applications

## Primary tool

Use `Start.exe` when the goal is:

- choose a specific box
- start a child program inside that box
- optionally pass startup arguments to that child program

Use `SbieIni.exe` when the goal is:

- query configuration
- update configuration
- inspect configured box settings

## Official CLI shape

The generic form is:

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" /box:BOX_NAME <program> <arguments>
```

This is the primary programmatic mechanism for binding:

- a box identity
- a launched child process

## Core semantics

`Start.exe` can:

- start a new program in a specific box
- pass arguments to the newly started child process
- optionally wait for that child process to finish

`Start.exe` cannot:

- inject commands later into an already-open terminal session
- act as a remote-control channel for an existing shell window

That boundary is important for automation design.

For cross-cutting shell-selection behavior such as `ComSpec` / `COMSPEC`, read:

- `docs\cli\shell\general.md`

## Minimal examples

### Start a program in a named box

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:MY_BOX `
  notepad.exe
```

### Start PowerShell in a named box

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:MY_BOX `
  "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
  -NoLogo
```

### Start a one-shot command in a named box

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:MY_BOX `
  "C:\Windows\System32\cmd.exe" `
  /c echo hello
```

## Wait for completion

If the caller needs the launched program's exit code, use `/wait`:

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:MY_BOX `
  /wait `
  "C:\Windows\System32\cmd.exe" `
  /c exit 9
```

This is useful for:

- wrappers
- health checks
- CI-like validation
- reproducible automation

## Design recommendation

For enterprise-grade automation, prefer:

- explicit wrapper actions
- explicit named parameters

instead of long free-form passthrough chains across multiple parsing layers.

Good pattern:

```text
Start.exe -> boxed PowerShell -> wrapper script -> explicit action -> target process
```

Prefer script contracts like:

- `-Action LaunchVSCode`
- `-Action InstallExtension`
- `-Action OpenTerminal`
- `-Action HealthCheck`

and explicit payload parameters like:

- `-ExtensionId`
- `-RepoPath`
- `-ProjectName`

This reduces quoting ambiguity and makes the startup contract auditable.

## When to switch to terminal-specific guidance

If the goal is specifically:

- open an interactive shell in a box
- keep the shell visible
- inspect stdout/stderr
- run commands manually after launch

then continue with:

- `docs\cli\terminal\general.md`

## Related

- `docs\cli\shell\general.md`
- `docs\cli\terminal\general.md`
