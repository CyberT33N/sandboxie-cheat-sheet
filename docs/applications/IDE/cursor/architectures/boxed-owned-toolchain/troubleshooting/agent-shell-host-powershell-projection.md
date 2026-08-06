# Cursor Agent Shell Host-PowerShell Compatibility Projection

## Scope

This document owns the Cursor-specific failure class where:

- the boxed Cursor integrated terminal is healthy;
- box-local CMD and Windows PowerShell can be spawned by boxed Node;
- the Cursor Agent Shell still returns no exit status;
- `cursor-agent-exec` attempts to spawn the canonical host PowerShell path:

```text
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
```

It is not the integrated-terminal ConPTY failure class and it is not a generic
stdout-capture issue.

## Validated failure shape

The relevant distinction is:

```text
boxed Node -> box-local cmd.exe          succeeds
boxed Node -> box-local powershell.exe   succeeds
boxed Node -> host cmd.exe               spawn EPERM
boxed Node -> host powershell.exe        spawn EPERM
Cursor Agent Shell -> host powershell.exe fails
```

`Test-Path` can still return `True` for the host PowerShell path. The failure
is process creation across the strict Sandboxie host-image boundary, not basic
read visibility.

Cursor Agent Shell does not expose a supported setting that replaces this
absolute host PowerShell path with a configured terminal profile. Therefore
`terminal.integrated.defaultProfile.windows` and
`terminal.integrated.automationProfile.windows` are insufficient for this
particular consumer.

## Correct architecture

Do not solve this problem by opening host PowerShell access, disabling
`UseSecurityMode`, or using `DropChildProcessToken` as the normal runtime
contract.

Instead, bootstrap projects the already mirrored, governed boxed PowerShell
tree into the matching **virtual** Windows path inside the sandbox:

```text
box-local source
C:\Program Files\SandboxToolchains\CursorBoxes\<project>\
  execution\toolchain\shells\powershell\<version>\

virtual compatibility target
C:\Windows\System32\WindowsPowerShell\v1.0\
```

The target is materialized in the physical sandbox drive tree, not the host
`C:\Windows` tree. Cursor can therefore resolve its hard-coded canonical path,
while Sandboxie executes the box-owned PowerShell image.

The projection is:

- Cursor-specific and opt-in from the Cursor platform wrappers;
- performed after the regular local shell mirror exists and before Cursor is
  launched;
- guarded so it can run only in a boxed IDE bootstrap context;
- version/hash-marked and refreshed without deleting unrelated Windows files.

## Required current Sandboxie options

The current validated Cursor box keeps these independent compatibility controls:

```ini
UseElectronDetection=n
NoRestartOnPCA=y
DropConHostIntegrity=y
UseWin32kHooks=y
```

Their responsibilities are separate:

- `UseElectronDetection=n` prevents automatic Electron classification from
  injecting `--disable-features=PrintCompositorLPAC` into Cursor workers.
  `SBIE2189` after disabling detection is expected; do not use its automatic
  remediation for this box.
- `NoRestartOnPCA=y` prevents Sandboxie's PCA-job restart behavior. It does
  not remap the Agent Shell's PowerShell path.
- `DropConHostIntegrity=y` addresses `ReadConsoleOutput` integrity failures.
  It does not solve host-image child-process spawning.
- `UseWin32kHooks=y` is the validated Cursor project-box compatibility choice
  when Chromium special-image handling is not used.

## Legacy Terminal Tool

The current validated Cursor Agent Shell path uses Cursor's **Legacy Terminal
Tool**. Keep it enabled as part of the working Cursor-box profile.

The boxed PowerShell canonical-path projection fixes the host-image path that
this executor hard-codes; it does not prove that Cursor's non-legacy executor
has the same process-spawn behavior on Windows. Therefore do not disable Legacy
Terminal Tool during normal operation.

If a future Cursor update claims to fix the non-legacy Windows executor, test
that change in a fresh box session with:

```text
cmd /c echo CURSOR_AGENT_OK
git --version
```

Keep Legacy Terminal Tool enabled unless both commands return output and an exit
status reliably after a full Cursor restart.

## Validation

After a fresh Cursor project launch:

1. confirm the compatibility executable exists at:
   ```text
   C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
   ```
2. confirm it is materialized below the physical sandbox drive root;
3. run an Agent Shell probe:
   ```text
   cmd /c echo CURSOR_AGENT_OK
   ```
4. confirm `cursor-agent-exec` reports an exit status and output;
5. confirm the logs no longer contain:
   - `spawn C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe ENOENT`
   - `spawn EPERM`
   - `bad option: --disable-features=PrintCompositorLPAC`

## Related

- `docs\cli\shell\general.md`
- `docs\troubleshooting\sandboxie\process-spawning\cmd-based-shells.md`
- `docs\applications\operating-systems\windows\build-toolchain\microsoft\architectures\boxed-owned-toolchain\host-image-launch-boundary.md`
- `docs\applications\IDE\cursor\methods\boxed-owned-toolchain\bootstrap\scripts.md`
- `docs\applications\IDE\cursor\architectures\boxed-owned-toolchain\troubleshooting\integrated-terminal-conpty.md`
