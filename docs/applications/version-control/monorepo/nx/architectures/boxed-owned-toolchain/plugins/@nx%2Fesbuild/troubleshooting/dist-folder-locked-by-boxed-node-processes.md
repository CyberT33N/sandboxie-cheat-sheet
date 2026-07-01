# `dist` Folder Locked By Boxed `node.exe` Processes

## Scope

This document owns the boxed-owned-toolchain troubleshooting path for a concrete
`@nx/esbuild` failure class:

- the executor tries to remove the existing `dist` directory
- the removal fails only inside the Sandboxie project box
- the same repository and the same Nx command succeed on the host system

This is a plugin-specific troubleshooting note about the Sandboxie execution
surface. It is **not** a reason to change productive project behavior.

## Problem statement

For the current boxed-owned-toolchain `test-mono` posture, the repository has a
validated problem class where:

1. the backend build/serve contract is architecturally valid as-is
2. the same Nx command succeeds on the host
3. the same Nx command fails in the project box during output cleanup
4. terminating stale boxed `node.exe` / `esbuild.exe` processes from the host
   and deleting the backend `dist` tree through a host-orchestrated boxed
   delete step restores the next run

Representative failure shapes:

```text
Error: ENOTEMPTY, Directory not empty: \\?\C:\Users\yourusername\source\test-mono\apps\backend\dist
    at rmSync (node:fs:1206:18)
    at esbuildExecutor (...\@nx\esbuild\dist\src\executors\esbuild\esbuild.impl.js:...)
```

```text
Error: EPERM, Permission denied: \\?\C:\Users\yourusername\source\test-mono\apps\backend\dist
    at rmSync (node:fs:1206:18)
    at esbuildExecutor (...\@nx\esbuild\dist\src\executors\esbuild\esbuild.impl.js:...)
```

The important current-state observation is:

- the failing call is the `rmSync(...)` cleanup inside the local
  `@nx/esbuild` executor implementation
- the failure happens before the normal next backend bundle can proceed
- host-side cleanup of boxed `node.exe` / `esbuild.exe` plus explicit deletion
  of the backend `dist` directory restores a clean next run

## Local executor fact

For this failure class, the current local behavior is explicit:

- `backend:serve` runs through `@nx/js:node`
- `backend:serve` points to `backend:build`
- `backend:build` runs through `@nx/esbuild:esbuild`
- the current local `@nx/esbuild` implementation performs:
  - `rmSync(options.outputPath, { recursive: true, force: true })`
    when `deleteOutputPath` is enabled

That means:

- the `rm` call is executed by the `@nx/esbuild` executor
- `@nx/js:node` is the caller/orchestrator, not the component that performs the
  delete itself

## Architectural constraints

The following current repository constraints are binding for this failure class:

- the productive project code must **not** be changed just to satisfy the
  sandbox
- the current watch/build/runtime topology remains architecturally valid
- the boxed failure must be treated as a Sandboxie execution-surface problem,
  not as proof that the repository command surface is wrong
- the current output cleanup behavior remains correct
- the current `deleteOutputPath=true` posture remains correct
- the fact that `deleteOutputPath` defaults to `true` in `@nx/esbuild` remains
  correct

This means the following are currently **invalid** answers:

- `deleteOutputPath: false`
- disabling normal `dist` cleanup permanently
- removing `watch-deps`
- removing `build-deps`
- disabling the daemon as a repository answer when the workspace graph/watch
  contract still needs it
- moving or weakening the productive project contract only because the sandbox
  execution surface is currently fragile

The host result is the deciding architectural signal:

- if the same repository and the same Nx command succeed on the host, then the
  repository contract is not the first-class thing to change

## Current interpretation

The current repository interpretation is:

- this is a Sandboxie-local process/handle lifecycle problem
- the practical blocker is the boxed `node.exe` family created by the
  daemon/watch lane
- in the current failure class, the boxed `node.exe` family plus the spawned
  `esbuild.exe` process keep the backend `dist` tree in a watched/open state
  that prevents `@nx/esbuild` from deleting it cleanly
- because host-side process cleanup plus a host-orchestrated boxed backend
  `dist` deletion restores the next run, the first-class issue is the boxed
  runtime surface, not the productive build contract

In current repository terms, the working interpretation is therefore:

1. boxed `node.exe` from the daemon/watch lane is the primary process family
   that blocks backend `dist` cleanup
2. `@nx/esbuild` then reaches its normal `deleteOutputPath=true` cleanup step
   and fails with `ENOTEMPTY` / `EPERM`
3. the spawned boxed `esbuild.exe` can remain part of that stale process/handle
   picture
4. the host can still recover the box cleanly by terminating those boxed
   `node.exe` / `esbuild.exe` processes and deleting backend `dist`
5. the next run then succeeds until the boxed runtime enters the stale state
   again

## What was investigated and not accepted as the answer

The following were investigated and are **not** the current repository answer:

- changing productive project values because of the sandbox
- disabling normal backend output cleanup
- using `.nxignore` as the first-class fix
- changing the output path as a sandbox-only workaround
- opening the normal project bootstrap terminal first and only then trying to
  delete `dist` inside that already-bootstrapped session
- turning off the watch/build topology that remains architecturally correct on
  the host
- treating `esbuild.exe` visibility as the primary root cause for this concrete
  failure

The practical reason is simple:

- those changes would mutate a repository contract that already works correctly
  outside the sandbox

## Upstream research

The following upstream material was reviewed for this failure class.

### Official Nx documentation

- [`@nx/esbuild` executor reference](https://nx.dev/docs/technologies/build-tools/esbuild/executors)
  - documents `deleteOutputPath`
  - confirms `deleteOutputPath` defaults to `true`
  - does **not** expose a narrower cleanup mode such as "delete only contents"
- [`@nx/esbuild` introduction](https://nx.dev/docs/technologies/build-tools/esbuild/introduction)
  - documents the standard esbuild plugin surface
- [`@nx/js:node` executor reference](https://nx.dev/docs/technologies/typescript/executors)
  - documents that the Node executor runs a build target
  - this aligns with the current local stack where `@nx/js:node` invokes the
    build target and `@nx/esbuild` performs the delete

### Relevant GitHub issues and discussions

- [`nrwl/nx#35408 - nx locks itself when daemon is enabled`](https://github.com/nrwl/nx/issues/35408)
  - closest current match
  - reports Windows failures where the daemon/watch path keeps `dist` watched
    and later cleanup fails with `ENOTEMPTY`
- [`nrwl/nx#30005 - EPERM: operation not permitted, open daemon.log on Windows`](https://github.com/nrwl/nx/issues/30005)
  - documents Windows daemon/file-lock failures that sometimes require killing
    `node.exe`
- [`nrwl/nx#27180 - Error: EPERM: operation not permitted, unlink '.nx\workspace-data\d\daemon.log'`](https://github.com/nrwl/nx/issues/27180)
  - maintainer response explicitly acknowledges a class of failures where manual
    `node.exe` cleanup is currently required
- [`nrwl/nx#27633 - Failed to process project graph ...`](https://github.com/nrwl/nx/issues/27633)
  - documents stale `node.exe` / IDE-related handle issues on Windows
- [`nrwl/nx#28207 - 19.8.1 leaves dangling child processes on Windows`](https://github.com/nrwl/nx/issues/28207)
  - documents Windows cleanup / orphan-process instability in the Nx task
    lifecycle
- [`nrwl/nx#30368 - Nx processes not quitting on Windows`](https://github.com/nrwl/nx/issues/30368)
  - documents persistent daemon/plugin-worker `node.exe` processes on Windows

The upstream picture is therefore consistent with the repository interpretation:

- Windows + daemon/watch + lingering Nx `node.exe` processes are a known failure
  family
- the upstream material does **not** provide a better plugin-local option than
  `deleteOutputPath`
- the current best recovery remains process cleanup plus output-tree cleanup

## Sandboxie CLI boundary

The current repository research did **not** identify a dedicated Sandboxie CLI
primitive that deletes one arbitrary file or folder inside a box.

The documented Sandboxie CLI surfaces in this workspace are:

- `Start.exe /box:BOX_NAME /listpids`
- `Start.exe /box:BOX_NAME /terminate`
- `Start.exe /box:BOX_NAME <program> <arguments>`
- `Start.exe /box:BOX_NAME /wait <program> <arguments>`

That means the current best Control-Plane answer for deleting one concrete boxed
path is:

- use `Start.exe` from the host
- launch a one-shot boxed child process such as `cmd.exe`
- let that boxed child process delete the logical repo path
- wait for the boxed child process to exit before continuing

## Why delete-inside-bootstrap-terminal failed

The repository validated a weaker path first:

1. host kills stale boxed `node.exe`
2. host opens the normal project bootstrap terminal
3. that bootstrap performs its normal environment setup and Nx daemon preflight
4. only after that, the terminal tries to delete backend `dist`

Why that is weaker:

- the normal project bootstrap intentionally prepares the full boxed Nx
  environment
- that includes daemon preflight in the current contract
- by the time the delete runs inside that already-open bootstrap terminal,
  boxed `node.exe` can already exist again
- the delete then loses the race and still fails with "directory not empty"

So the current best sequence is narrower:

1. host kills stale boxed `node.exe` / `esbuild.exe`
2. host triggers a one-shot boxed delete command **before** the normal project
   bootstrap terminal is opened
3. only after that does the host open the normal project bootstrap terminal and
   run the real `serve` command

## Current best architectural path

Under the current constraints, the best current repository answer is a
**host-orchestrated boxed recovery path**:

1. from the host, terminate the boxed `node.exe` and `esbuild.exe` processes for
   the project box
2. from the host, execute a one-shot boxed delete command for the logical
   backend `dist` path
3. only after that, start a fresh project-box terminal through the existing
   project bootstrap
4. run the real backend `serve` command in that already-governed boxed terminal

Why this is currently the best answer:

- it does not mutate the productive project contract
- it keeps `watch-deps`, `build-deps`, the daemon posture, and normal output
  cleanup intact
- it uses the existing host control plane that is already able to manage the
  project box deterministically
- it avoids reopening the normal Nx daemon/bootstrap lane before the delete step
  has completed
- it treats the failure as a Sandboxie execution-surface problem rather than as
  a repository business-code problem

## Current host-orchestrated recovery and launch command

Run the following PowerShell on the **host system**.

This command:

1. terminates boxed `node.exe` / `esbuild.exe` processes for the project box
2. uses the Sandboxie CLI to run a one-shot boxed `cmd.exe /c rmdir /s /q ...`
   delete against the logical backend `dist` path
3. opens the existing project-box terminal bootstrap
4. runs the real backend `serve` command in that boxed terminal

```powershell
$box = 'VS_CODE_TEST_MONO'
$repoPath = 'C:\Users\yourusername\source\test-mono'
$distPath = Join-Path $repoPath 'apps\backend\dist'

$startExe = 'C:\Program Files\Sandboxie-Plus\Start.exe'
$cmdExe = 'C:\Windows\System32\cmd.exe'
$powerShellExe = 'C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe'
$bootstrapScript = 'C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\Start-TestMonoVSCode.ps1'

$boxPids = @(
  & $startExe /box:$box /listpids |
  Where-Object { $_ -match '^\d+$' } |
  Select-Object -Skip 1 |
  ForEach-Object { [int]$_ }
)

$staleProcesses = @()

if ($boxPids.Count -gt 0) {
  $staleProcesses = @(
    Get-CimInstance Win32_Process |
      Where-Object {
        $boxPids -contains $_.ProcessId -and $_.Name -in @('node.exe', 'esbuild.exe')
      }
  )
}

if ($staleProcesses.Count -gt 0) {
  $stalePids = @($staleProcesses.ProcessId)

  $staleProcesses | ForEach-Object {
    Stop-Process -Id $_.ProcessId -Force
  }

  Wait-Process -Id $stalePids -Timeout 10 -ErrorAction SilentlyContinue
}

Start-Sleep -Milliseconds 750

$deleteCommand = 'if exist "' + $distPath + '" rmdir /s /q "' + $distPath + '"'

& $startExe `
  /box:$box `
  /wait `
  $cmdExe `
  /d /s /c $deleteCommand

if ($LASTEXITCODE -ne 0) {
  throw "The boxed delete command failed for '$distPath'. ExitCode: $LASTEXITCODE"
}

$serveCommand = "& '$bootstrapScript' -Action OpenTerminal -RepoPath '$repoPath'; pnpm exec nx run backend:serve --no-tui --verbose"

& $startExe `
  /box:$box `
  $powerShellExe `
  -NoLogo `
  -NoExit `
  -ExecutionPolicy Bypass `
  -Command $serveCommand
```

## What this command is and is not

This command is:

- a host-side recovery and launch wrapper
- a control-plane answer to a Sandboxie-local failure class
- consistent with the current boxed-owned-toolchain architecture

This command is **not**:

- a reason to change productive project code
- a reason to disable normal backend output cleanup
- a reason to remove `watch-deps`, `build-deps`, or the daemon contract
- a reason to conclude that the sandbox root itself must be deleted directly as
  the preferred first move
- proof that the repository build topology is wrong

## Related

- `docs\cli\process\general.md`
- `docs\cli\terminal\general.md`
- `docs\cli\start\general.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\plugins\@nx%2Fesbuild\overview.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\bootstrap-integration.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\runtime-contract.md`
- `upstream\sandboxie-docs\docs\Content\StartCommandLine.md`
