# Sandboxie Rules For Project Boxes

## Scope

This document captures the Sandboxie-specific rules that are required for project-authoring boxes in the boxed-owned-toolchain method.

## Required removals

Project-authoring boxes must **not** keep:

- `Template=Chrome_KB5027231_fix`
- `SpecialImage=chrome,Code.exe`

## Required compatibility choice

Project-authoring boxes use:

```ini
UseWin32kHooks=y
```

as the validated compatibility setting.

## Why this is required

`Code.exe` is used by VS Code for both:

- the GUI process
- internal server / worker processes

When `Code.exe` is classified as a Chromium special image, Chromium flags are injected into contexts where those flags are fatal to the internal server processes.

That is why the project box must avoid the `SpecialImage` path.

`UseWin32kHooks=y` is the validated compatibility concession used instead.

## Shared access model

Project boxes should consume the shared canonical surfaces without becoming global writers.

That means:

- shared runtime readable
- shared catalog readable
- shared canonical extensions readable
- local runtime copy used for the live project `--extensions-dir`

## Sanitized target config example

The following is a sanitized target configuration example for a project-authoring box:

```ini
[VS_CODE_TEST_MONO]
Enabled=y
ConfigLevel=10
BorderColor=#027df7,ttl,6,192,in
ForceRestartAll=y

Template=AutoRecoverIgnore
Template=LingerPrograms
Template=BlockPorts
Template=FileCopy
Template=SkipHook
Template=AdobeDistiller
Template=AdobeAcrobatReader
Template=AdobeAcrobat
Template=Thunderbird
Template=BlockTelemetry
Template=NotepadPlusPlus_fix
Template=OneDrive
Template=HideInstalledPrograms

UseFileDeleteV2=y
UseRegDeleteV2=y

UseSecurityMode=y
UsePrivacyMode=y
DropAdminRights=y
FakeAdminRights=n
EditAdminOnly=y
ProtectHostImages=y
ClosePrintSpooler=y
BlockInterferePower=y
HideFirmwareInfo=y
RandomRegUID=y
HideDiskSerialNumber=y
HideNetworkAdapterMAC=y
HideNonSystemProcesses=y
MonitorAdminOnly=y

NoAddProcessToJob=y
DropConHostIntegrity=y
BlockNetworkFiles=y
UseRamDisk=n

# debug-only tracing lines; do not keep them enabled for normal operation
# FileTrace=*
# PipeTrace=*
# KeyTrace=*
# IpcTrace=*
# GuiTrace=*
# ClsidTrace=*

# no Template=Chrome_KB5027231_fix
# no SpecialImage=chrome,Code.exe
UseWin32kHooks=y

ReadFilePath=C:\shared\sandbox-toolchains\
ReadFilePath=C:\shared\sandbox-toolchains\dev\bootstrap\
ReadFilePath=C:\shared\sandbox-toolchains\projects\test-mono\bootstrap\

ReadFilePath=C:\shared\sandbox-toolchains\ide\vscode\runtime\1.121.0\
ReadFilePath=C:\shared\sandbox-toolchains\ide\vscode\catalog\
ReadFilePath=C:\shared\sandbox-toolchains\ide\vscode\extensions\

ReadFilePath=C:\shared\sandbox-toolchains\dev\git\2.54.0\
ReadFilePath=C:\shared\sandbox-toolchains\dev\node\26.2.0\
ReadFilePath=C:\shared\sandbox-toolchains\dev\node\20.19.6\
ReadFilePath=C:\shared\sandbox-toolchains\dev\pnpm\
ReadFilePath=C:\shared\sandbox-toolchains\dev\python\
ReadFilePath=C:\shared\sandbox-toolchains\dev\starship\

ClosedFilePath=C:\shared\sandbox-toolchains\ide\vscode\runtime\1.121.0\bin\code-tunnel.exe
ClosedFilePath=C:\shared\sandbox-toolchains\ide\vscode\runtime\1.121.0\tools\inno_updater.exe

ReadFilePath=starship.exe,C:\Users\yourusername\.config\starship.toml
```

## Why this differs from older configs

This target example intentionally omits:

- host Git / Node / pnpm / nvm paths
- box-specific terminal shell copies
- write access to the canonical shared extension store

Those belonged to earlier or transitional states, not to the final method contract.

The wildcard trace lines are intentionally shown commented out above because they are a debug surface, not part of the normal day-to-day box configuration.

The live runtime is expected to execute locally from mirrored paths such as:

- `C:\Program Files\SandboxToolchains\VSCodeBoxes\test-mono\state\...`
- `C:\Program Files\SandboxToolchains\VSCodeBoxes\test-mono\execution\...`

## PNPM box-visibility governance

For Sandboxie visibility rules, PNPM should normally be opened at the **folder level**:

```ini
ReadFilePath=C:\shared\sandbox-toolchains\dev\pnpm\
```

not as a version-pinned rule such as:

```ini
ReadFilePath=C:\shared\sandbox-toolchains\dev\pnpm\11.5.0\
```

Why this is the recommended rule:

- the project contract still selects one exact `PnpmCli` version
- but PNPM itself often needs security updates
- a version-pinned Sandboxie allow rule would otherwise have to be changed every time PNPM is upgraded
- the broader folder rule avoids repetitive box-config churn while still keeping visibility scoped to the PNPM subtree

So the correct split is:

- **project contract** = exact selected PNPM version
- **Sandboxie box rule** = broad `dev\pnpm\` visibility

## Related

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boxes\project-box.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\state\extensions-seeds-and-local-state.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\sandboxie\observations-and-signals.md`
- `docs\performance\filesystem\sandboxie-debug-tracing.md`
