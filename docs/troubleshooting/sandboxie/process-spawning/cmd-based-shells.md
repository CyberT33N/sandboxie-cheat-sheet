# CMD-Based Shell Spawns Under Sandboxie

## Scope

This is a cross-cutting Sandboxie troubleshooting note.

It is intentionally:

- not package-manager-specific
- not framework-specific
- not tied to one IDE

The issue documented here applies whenever a program inside the box asks the runtime to spawn a Windows shell such as `cmd.exe` or `powershell.exe`.

## Problem statement

In strict boxed workflows, there can be a difference between:

1. launching a shell manually in the interactive terminal
2. spawning that shell from another runtime such as Node via `child_process.spawn(...)`

That means a setup can show this combination:

- manual `cmd.exe` works
- manual PowerShell works
- but `node -> spawn(cmd.exe)` fails with `spawn EPERM`
- and `node -> spawn(powershell.exe)` also fails with `spawn EPERM`

## Why this matters

Many tools on Windows do not execute their lifecycle or helper commands directly.

Instead they delegate to a shell layer, often:

- `cmd.exe`
- `powershell.exe`
- or another configured script shell

So if the shell-spawn surface is blocked, the visible failure may first appear inside:

- `pnpm install`
- `npm install`
- `rebuild` flows
- framework postinstall hooks
- package lifecycle scripts
- any tool that shells out through Node or another host runtime

## Validated diagnostic pattern

The validated diagnostic split is:

### Manual shell

```powershell
cmd.exe /d /s /c echo CMD_OK
```

This can succeed.

### Runtime spawning the host shell

Example symptom:

```text
Error: spawn EPERM
  code: 'EPERM'
  syscall: 'spawn'
```

That can still fail when the shell is spawned from inside Node or another runtime.

### Runtime spawning a box-local executable

The same setup may still allow:

- `node -> spawn(local node.exe)`
- `node -> spawn(local git bash.exe)`

That distinction is the key architectural signal.

## Architectural interpretation

If:

- the local mirrored runtime works
- the local mirrored toolchain works
- but host system shells fail when spawned as child processes

then the problem is not a general runtime failure.

It is a **shell-spawn boundary problem**.

In other words:

- interactive shell launch and runtime child-process launch are not the same mediation surface
- Sandboxie can allow one while still denying or constraining the other

## Correct architectural response

Do **not** immediately widen the box with broad host-shell exceptions just to make lifecycle scripts pass.

That would normalize the wrong boundary.

Prefer, in this order:

1. keep using box-local mirrored runtimes
2. use a box-local shell executable if a script shell is required
3. keep the configuration explicit and as narrow as possible

Typical example:

- use a box-local mirrored Git Bash `.exe`
- configure the tool to use that shell explicitly
- avoid depending on spawned host `cmd.exe` or host `powershell.exe` when the strict box policy blocks them

## What this is *not*

This document does **not** claim that every failure around shells is caused by Sandboxie.

It documents one specific class of issue:

- a spawned Windows shell fails from inside another runtime
- while the interactive shell itself still works

That pattern should be treated as a first-class troubleshooting category in boxed workflows.

## Application-independent consequence

Yes, this can affect more than `pnpm`.

Any application, framework, or runtime can be affected if it:

- shells out through `cmd.exe`
- shells out through `powershell.exe`
- or otherwise assumes that spawned host shells are available under the same rules as the interactive terminal

## Important PNPM-specific nuance

In validated boxed-owned-toolchain testing, there was an important split:

- setting PNPM `scriptShell` to a box-local Bash `.exe` fixed lifecycle execution during `pnpm install`
- but `pnpm exec ...` still remained a separate failure surface

There was also a second shell-surface nuance in the integrated Git Bash terminal:

- a generated `pnpm.cmd` wrapper was not by itself enough to make bare `pnpm` resolution work in Git Bash
- the bootstrap also needed to expose a shell-native `pnpm` wrapper and ensure the local `bootstrap-bin` directory was exported into the Bash `PATH`

So when troubleshooting PNPM on Windows under Sandboxie, distinguish between:

1. lifecycle script execution during install / rebuild
2. integrated Git Bash command-name resolution for `pnpm`
3. `pnpm exec` command execution

They are related, but they are not necessarily fixed by the same configuration change.

## Related

- `docs\troubleshooting\sandboxie\privacy-mode\host-user-space-vs-box-root.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\overview.md`
