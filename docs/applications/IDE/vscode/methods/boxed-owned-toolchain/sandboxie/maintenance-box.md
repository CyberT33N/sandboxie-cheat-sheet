# Sandboxie Rules For The Maintenance Box

## Scope

This document captures the Sandboxie-specific rules that are required for the Maintenance Box in the boxed-owned-toolchain method.

## Required removals

For CLI-driven extension operations, the Maintenance Box must **not** keep:

- `Template=Chrome_KB5027231_fix`
- `SpecialImage=chrome,Code.exe`

## Why this is required

When those Chromium-oriented rules were active, `code.cmd`-based maintenance commands broke with injected Chromium flags such as:

```text
bad option: --disable-features=PrintCompositorLPAC
```

The Maintenance Box must support reliable CLI-driven maintenance and publish operations, so those rules are incompatible with the method.

## Functional result

With those rules removed, the Maintenance Box can reliably perform:

- extension installation
- extension listing
- other `code.cmd`-driven maintenance actions
- publish/promote approved maintenance state back to shared canonical surfaces

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
ReadFilePath=C:\shared\sandbox-toolchains\dev\bootstrap\
ReadFilePath=C:\shared\sandbox-toolchains\ide\vscode\runtime\1.121.0\
ReadFilePath=C:\shared\sandbox-toolchains\dev\git\2.54.0\
ReadFilePath=C:\shared\sandbox-toolchains\dev\node\26.2.0\
ReadFilePath=C:\shared\sandbox-toolchains\dev\node\20.9.0\
ReadFilePath=C:\shared\sandbox-toolchains\dev\pnpm\
ReadFilePath=C:\shared\sandbox-toolchains\dev\python\
ReadFilePath=C:\shared\sandbox-toolchains\dev\shells\
ReadFilePath=C:\shared\sandbox-toolchains\dev\starship\

OpenFilePath=C:\shared\sandbox-toolchains\ide\vscode\catalog\vscode-user\
OpenFilePath=C:\shared\sandbox-toolchains\ide\vscode\catalog\seed\globalStorage\
OpenFilePath=C:\shared\sandbox-toolchains\ide\vscode\catalog\seed\roo\
OpenFilePath=C:\shared\sandbox-toolchains\ide\vscode\extensions\

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

The current maintenance runtime itself is expected to execute locally from the mirrored box paths under:

- `C:\Program Files\SandboxToolchains\VSCodeBoxes\maintenance\state\...`
- `C:\Program Files\SandboxToolchains\VSCodeBoxes\maintenance\execution\...`

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

- PNPM often needs security updates
- a version-specific Sandboxie rule would otherwise have to be changed repeatedly in the box config
- the maintenance box is expected to work against the governed PNPM inventory under `dev\pnpm\...`
- the broader folder rule keeps visibility scoped to PNPM while avoiding needless operational churn

So the correct split is:

- **project or maintenance contract** = exact selected PNPM version
- **Sandboxie box rule** = broad `dev\pnpm\` visibility

The same governance principle applies to shell-specific runtime artifacts:

- **project or maintenance contract** = exact selected shell runtime roots such as `CmdRoot`, `PowerShellRoot`, and `ClinkRoot`
- **Sandboxie box rule** = broad `dev\shells\` visibility

The wildcard trace lines are intentionally shown commented out above because they are a debug surface, not part of the normal day-to-day box configuration.

## Related

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boxes\maintenance-box.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\sandboxie\observations-and-signals.md`
- `docs\performance\filesystem\sandboxie-debug-tracing.md`
