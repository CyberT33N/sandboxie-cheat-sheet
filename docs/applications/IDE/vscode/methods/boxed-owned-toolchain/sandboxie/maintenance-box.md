# Sandboxie Rules For The Maintenance Box

## Scope

This document captures the Sandboxie-specific rules that are required for the Maintenance Box in the boxed-owned-toolchain method.

## Required removals

For CLI-driven extension operations, the Maintenance Box must **not** keep:

- `Template=Chrome_KB5027231_fix`
- `SpecialImage=chrome,Code.exe`

## Why this is required

When those Chromium-oriented rules were active, `code.cmd`-based extension commands broke with injected Chromium flags such as:

```text
bad option: --disable-features=PrintCompositorLPAC
```

The Maintenance Box must support reliable CLI-driven `code.cmd` operations against the shared extension store, so those rules are incompatible with the method.

## Functional result

With those rules removed, the Maintenance Box can reliably perform:

- extension installation
- extension listing
- other `code.cmd`-driven shared IDE maintenance actions

## Sanitized target config example

The following is a sanitized target configuration example for the Maintenance Box:

```ini
[VS_CODE_MAINTENANCE]
Enabled=y
ConfigLevel=10
BorderColor=#e59f00,ttl,6,192,in
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

ReadFilePath=C:\shared\sandbox-toolchains\

NormalFilePath=C:\shared\sandbox-toolchains\ide\vscode\runtime\1.121.0\
NormalFilePath=C:\shared\sandbox-toolchains\dev\git\2.54.0\
NormalFilePath=C:\shared\sandbox-toolchains\dev\node\26.2.0\
NormalFilePath=C:\shared\sandbox-toolchains\dev\node\20.19.6\
NormalFilePath=C:\shared\sandbox-toolchains\dev\pnpm\11.2.2\
ReadFilePath=C:\shared\sandbox-toolchains\dev\starship\

OpenFilePath=C:\shared\sandbox-toolchains\ide\vscode\catalog\
OpenFilePath=C:\shared\sandbox-toolchains\ide\vscode\extensions\
OpenFilePath=C:\shared\sandbox-toolchains\ide\vscode\maintenance\

ClosedFilePath=C:\shared\sandbox-toolchains\ide\vscode\runtime\1.121.0\bin\code-tunnel.exe
ClosedFilePath=C:\shared\sandbox-toolchains\ide\vscode\runtime\1.121.0\tools\inno_updater.exe

ReadFilePath=starship.exe,C:\Users\yourusername\.config\starship.toml
```

## Why this differs from older configs

This target example intentionally omits:

- host Git / Node / pnpm / nvm paths
- box-specific terminal shell copies
- Chromium special-image handling for `Code.exe`

Those belonged to earlier or transitional states, not to the final method contract.

The wildcard trace lines are intentionally shown commented out above because they are a debug surface, not part of the normal day-to-day box configuration.

## Related

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boxes\maintenance-box.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\sandboxie\observations-and-signals.md`
- `docs\performance\filesystem\sandboxie-debug-tracing.md`
