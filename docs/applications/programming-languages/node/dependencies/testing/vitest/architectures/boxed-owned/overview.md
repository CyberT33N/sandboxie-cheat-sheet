# Vitest Typechecking And Windows CMD Child Processes

## Status

This document is the canonical architectural reference for the Vitest
typechecking child-process failure class in a boxed-owned toolchain.

It is intentionally independent of:

- a particular monorepo;
- a particular workstation;
- a particular Sandboxie box name;
- a particular Node, PNPM, or Vitest version;
- a particular application framework.

The concrete implementation is owned by the bootstrap/control plane. Project
test configuration is not the preferred place to solve this runtime boundary.

## Problem statement

A boxed test command can show this combination:

1. all test files and assertions pass;
2. TypeScript itself is installed and usable;
3. Vitest reports an unhandled error;
4. the error is:

```text
Error: spawn EPERM
code: 'EPERM'
syscall: 'spawn'
```

5. the stack includes the Vitest typechecker and a child-process launcher such
   as `tinyexec`.

This is not a TypeScript diagnostic and it is not proof that typechecking
completed successfully. The TypeScript child process did not start.

It must not be solved by disabling typechecking merely to make the test run
appear green.

## The relevant execution chain

The validated Windows shape is:

```text
Vitest typechecker
  -> default checker command: tsc
    -> Windows package-manager command shim: tsc.cmd
      -> child-process helper
        -> cmd.exe /d /s /c ...
          -> TypeScript JavaScript entrypoint
```

The exact package-manager shim and helper may vary, but the architectural
property is the same: a runtime that otherwise works asks Windows to start a
shell as a child process.

For the validated Vitest implementation, the default typecheck checker is
`tsc`. Its Windows command-launch helper may invoke the literal command
`cmd.exe` when it resolves a `.cmd` command surface.

That distinction matters:

- `ComSpec` / `COMSPEC` govern shell-aware APIs such as Windows `exec` paths;
- a library that directly calls `spawn('cmd.exe', ...)` uses `PATH` resolution;
- setting only `ComSpec` is therefore insufficient for the latter case.

## What the failure is not

Do not conflate this error with these separate concerns:

- a TypeScript source error;
- a Vitest assertion failure;
- a test fixture or application temporary-directory failure;
- coverage-report output;
- a general inability to run Node;
- a general inability to run `cmd.exe` interactively;
- a reason to widen host access or use `OpenFilePath`.

Coverage can legitimately need separate treatment because it traverses and
writes many report paths. That concern is documented in
`troubleshooting.md`; it does not explain a failure whose syscall is
`spawn`.

## Unverified coverage-output lifecycle hypothesis

Coverage output can create a separate boxed filesystem-lifecycle failure class.
The current working hypothesis is that a first coverage run can create its
report tree successfully, while a later run fails because stale boxed
`node.exe` or `esbuild.exe` processes still retain output-tree handles or
state from the earlier execution.

This is not yet a verified Vitest conclusion and must not be conflated with
the `tsc.cmd` child-process failure documented above. In particular, no
process-cleanup code belongs in this Vitest architecture reference until the
coverage case has been independently reproduced and validated.

The related verified Electron-Vite process-lifecycle recovery is documented
under the related rules below. It is evidence for an investigation direction,
not proof that Vitest coverage has the same root cause.

## Architectural diagnosis

The decisive diagnostic split is:

```text
node -> spawn('cmd.exe')                 => EPERM
node -> spawn(process.env.BOXED_CMD_EXE) => success
```

If adding the local CMD root to the front of `PATH` makes the first command
succeed, the root cause is confirmed:

```text
bare cmd.exe resolved to a host/system command surface
instead of the explicitly mirrored box-local CMD lane
```

This is a shell-selection and command-surface-resolution failure. It is not a
filesystem permission request.

## Preferred solution: complete the bootstrap-owned CMD contract

The preferred solution is to adapt the shared bootstrap script that owns the
Windows shell runtime.

The contract has two required parts:

1. publish the box-local CMD executable through `ComSpec` and `COMSPEC`;
2. prepend the directory containing that executable to `PATH`.

The first part supports shell-aware consumers. The second part supports
libraries that directly spawn the literal executable name `cmd.exe`.

Both parts must use the locally mirrored runtime, never a host `cmd.exe`.

### 1. Mirror the governed CMD runtime locally

The shared shell bootstrap creates the local command surface from a governed
shared artifact. The relevant implementation is:

```powershell
$localShellRoot = Join-Path $LocalToolchainRoot 'shells'
$cmdVersion = Split-Path -Leaf $CmdRoot
$localCmdRoot = Join-Path (Join-Path $localShellRoot 'cmd') $cmdVersion
$localCmdExe = Join-Path $localCmdRoot 'cmd.exe'

if (-not (Test-Path -LiteralPath $localCmdExe)) {
  if (Test-Path -LiteralPath $localCmdRoot) {
    Remove-Item -LiteralPath $localCmdRoot -Recurse -Force -ErrorAction SilentlyContinue
  }

  Copy-FileIfExists `
    -Source (Join-Path $CmdRoot 'cmd.exe') `
    -Destination $localCmdExe
}

Assert-PathExists -Path $localCmdExe -Label 'Local mirrored CMD executable'
```

This is the relevant **CMD adapter/shim layer**. It is not a mutation of the
package-manager-owned `tsc.cmd` file. The package-manager shim remains
unchanged; bootstrap supplies the explicit interpreter required to execute it.

### 2. Publish the local CMD lane for both resolution mechanisms

`Initialize-WindowsShellRuntime` must publish the local CMD root before any
host/system command directories on `PATH`:

```powershell
$pathEntries = New-Object 'System.Collections.Generic.List[string]'

# Some Windows runners, including command-launch helpers used by test tooling,
# spawn the literal command "cmd.exe" instead of consulting ComSpec. Keep the
# box-local CMD lane first on PATH so those child-process calls do not resolve
# host cmd.exe.
[void]$pathEntries.Add($localCmdRoot)

if ($clinkAvailable) {
  [void]$pathEntries.Add($localClinkRoot)
}

if ($regAvailable) {
  [void]$pathEntries.Add($localRegRoot)
}

Prepend-PathEntries -Entries $pathEntries.ToArray() | Out-Null
```

The explicit environment contract remains required:

```powershell
$boxedComSpec = $windowsShellRuntime.CmdExe

if ([string]::IsNullOrWhiteSpace($boxedComSpec) -or
    -not (Test-Path -LiteralPath $boxedComSpec)) {
  throw 'Local boxed CMD executable not found for ComSpec override.'
}

$env:ComSpec = $boxedComSpec
$env:COMSPEC = $boxedComSpec
$env:BOXED_COMSPEC = $boxedComSpec

$env:BOXED_CMD_ROOT = $windowsShellRuntime.CmdRoot
$env:BOXED_CMD_EXE = $windowsShellRuntime.CmdExe
```

The `PATH` addition is not a broad host exception. It is an explicit command
resolution rule for an already box-local, versioned, bootstrap-governed
runtime.

## Why this belongs in bootstrap

The failure is outside application business logic and outside the semantic
definition of a test suite:

- Vitest is entitled to run its configured typechecker;
- TypeScript is entitled to use its package-manager command surface;
- the project must not need machine-specific executable paths in its test
  configuration;
- the shell selection belongs to runtime composition.

The bootstrap/control plane already owns:

- local runtime mirroring;
- `PATH` composition;
- `ComSpec` / `COMSPEC`;
- Node child-process diagnostics;
- explicit local toolchain lanes.

The fix therefore belongs beside those responsibilities rather than in a
project's production Vitest configuration.

## Validation protocol

Open a newly bootstrapped project terminal. Do not reuse a terminal whose
`PATH` was manually changed for diagnosis.

### Verify the absolute local lane

```powershell
node -e "const {spawn}=require('node:child_process'); const p=spawn(process.env.BOXED_CMD_EXE,['/d','/s','/c','exit 0']); p.on('error',console.error); p.on('exit',c=>console.log(c))"
```

Expected output:

```text
0
```

### Verify bare command resolution

```powershell
node -e "const {spawn}=require('node:child_process'); const p=spawn('cmd.exe',['/d','/s','/c','exit 0']); p.on('error',console.error); p.on('exit',c=>console.log(c))"
```

Expected output:

```text
0
```

### Verify the real test surface

Run the canonical test command with:

- Typechecking enabled;
- coverage configured according to the workload policy;
- no manual `PATH` patch;
- no test-config checker override.

Success requires both:

```text
Type Errors  no errors
Errors       0 errors
```

## Existing shims and adapters

### Package-manager `.cmd` shims

Commands such as `tsc.cmd` are dependency/package-manager artifacts. Bootstrap
must not overwrite them as the normal solution. It must make the box-local
interpreter reachable when those shims are used.

### Electron-Vite shim

The existing Electron-Vite `.cmd` shim and direct-Node spawn rewrite are a
separate, framework-specific example of the same general class:

```text
cmd.exe /d /s /c electron-vite.CMD ...
  -> local node.exe electron-vite.js ...
```

It is not the primary fix for Vitest typechecking. It demonstrates the
accepted control-plane pattern for a narrowly identified final spawn edge.

### Child-process spawn tracer

The bootstrap-generated Node tracer is the preferred diagnostic tool:

- it records the command, arguments, cwd, and selected environment;
- it records synchronous `spawn` throws and asynchronous child errors;
- it can implement a narrow, evidence-based rewrite when the generic
  box-local CMD lane cannot serve a proven exceptional command surface.

Do not use the diagnostic CMD proxy as a permanent default command interpreter.
It is an observability surface, not the productive baseline.

## Optional second solution: explicit direct TypeScript checker

If a narrowly scoped project policy deliberately chooses not to expose the
boxed CMD lane to bare command resolution, Vitest can be configured to call
the TypeScript JavaScript entrypoint directly:

```ts
test: {
  typecheck: {
    enabled: true,
    checker: './node_modules/typescript/bin/tsc'
  }
}
```

The TypeScript entrypoint has a Node shebang and avoids the `tsc.cmd ->
cmd.exe` indirection.

This keeps typechecking enabled, but it is the second-choice solution because
it places a boxed-runtime adaptation into project test configuration. Prefer
the bootstrap-owned local-CMD command surface when that surface is a normal
part of the toolchain contract.

## Non-goals

This architecture does not authorize:

- disabling typechecking;
- adding `OpenFilePath` for the project or host system;
- allowing host `C:\Windows\System32\cmd.exe` as the preferred child-process
  shell;
- replacing every `.cmd` command with a custom project shim;
- applying an Electron-Vite-specific rewrite to unrelated commands without
  evidence;
- changing application business code to solve a bootstrap boundary.

## Related rules

- `docs\cli\shell\general.md`
- `docs\troubleshooting\sandboxie\process-spawning\cmd-based-shells.md`
- `docs\troubleshooting\sandboxie\process-spawning\nested-child-process-orchestration.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\scripts.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\shared-layout.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\scripts.md`
- `docs\applications\version-control\monorepo\nx\architectures\boxed-owned-toolchain\execution-surfaces.md`
- `docs\applications\programming-languages\node\package-manager\pnpm\architectures\boxed-owned-toolchain\lifecycle-and-command-surface.md`
- `docs\applications\programming-languages\node\dependencies\frameworks\electron\electron-vite\architectures\boxed-owned-toolchain\overview.md` — verified host-side stale-process recovery for repeated Electron-Vite output creation
- `docs\applications\programming-languages\node\dependencies\testing\vitest\architectures\boxed-owned\troubleshooting.md`
