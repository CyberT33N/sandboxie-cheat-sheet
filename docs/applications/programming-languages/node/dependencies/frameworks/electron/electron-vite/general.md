# INI Settings:
```ini
# =========================
# Sandbox core / enablement
# =========================
Enabled=y
ConfigLevel=10
BorderColor=#027df7,ttl,6,192,in
ForceRestartAll=y

# =========================
# Templates (compat / defaults)
# =========================
Template=AutoRecoverIgnore
Template=LingerPrograms
Template=BlockPorts
Template=FileCopy
Template=SkipHook
Template=Chrome_KB5027231_fix
Template=AdobeDistiller
Template=AdobeAcrobatReader
Template=AdobeAcrobat
Template=Thunderbird
Template=BlockTelemetry

# Not working for localhost connections
# Template=BlockLocalConnect

Template=NotepadPlusPlus_fix
Template=OneDrive
Template=HideInstalledPrograms

# =========================
# File/Registry deletion behavior
# =========================
UseFileDeleteV2=y
UseRegDeleteV2=y

# =========================
# Security / isolation controls
# =========================
UseSecurityMode=y
UsePrivacyMode=y
FakeAdminRights=y
EditAdminOnly=y
ProtectHostImages=y
CoverBoxedWindows=y
BlockScreenCapture=y
ClosePrintSpooler=y
BlockInterferePower=y
HideFirmwareInfo=y
RandomRegUID=y
HideDiskSerialNumber=y
HideNetworkAdapterMAC=y
HideNonSystemProcesses=y
MonitorAdminOnly=y

# =========================
# Compatibility fixes (Electron/console)
# =========================
NoAddProcessToJob=y
DropConHostIntegrity=y

# =========================
# Network restrictions
# =========================
BlockNetworkFiles=y

# =========================
# Storage / performance
# =========================
UseRamDisk=y

# =========================
# Program control (forcing)
# =========================
ForceProcess=test.exe
ForceFolder=C:\Tools\DevBoxShell
ForceFolder=C:\git\test\test-synchronizer\.pnpm\electron@29.4.6\node_modules\electron\dist

# =========================
# Monitoring / tracing
# =========================
DisableResourceMonitor=y
FileTrace=*
PipeTrace=*
KeyTrace=*
IpcTrace=*
GuiTrace=*
ClsidTrace=*

# =========================
# Resource access policy (least privilege, program-scoped)
# =========================

# --- Starship ---
NormalFilePath=starship.exe,C:\Program Files\starship\bin\
NormalFilePath=powershell.exe,C:\Program Files\starship\bin\
ReadFilePath=starship.exe,C:\Users\denni\.config\starship.toml

# --- Git (needed by Starship prompt) ---
NormalFilePath=starship.exe,C:\Program Files\Git\
NormalFilePath=git.exe,C:\Program Files\Git\

# --- Node (nvm4w) ---
NormalFilePath=node.exe,C:\nvm4w\nodejs\
NormalFilePath=powershell.exe,C:\nvm4w\nodejs\
NormalFilePath=cmd.exe,C:\nvm4w\nodejs\

# --- pnpm ---
NormalFilePath=node.exe,C:\Users\denni\AppData\Local\pnpm\
NormalFilePath=powershell.exe,C:\Users\denni\AppData\Local\pnpm\
NormalFilePath=cmd.exe,C:\Users\denni\AppData\Local\pnpm\

# --- DevBoxShell binaries ---
NormalFilePath=powershell.exe,C:\Tools\DevBoxShell\
NormalFilePath=cmd.exe,C:\Tools\DevBoxShell\

# =========================
# Dev deps for start-fe-be.ps1 (forms backend + forms)
# =========================

# Forms backend (Port 3000)
NormalFilePath=powershell.exe,C:\git\test\test-forms-backend\
NormalFilePath=cmd.exe,C:\git\test\test-forms-backend\
NormalFilePath=node.exe,C:\git\test\test-forms-backend\

# Forms frontend (Port 4200)
NormalFilePath=powershell.exe,C:\git\test\test-forms\
NormalFilePath=cmd.exe,C:\git\test\test-forms\
NormalFilePath=node.exe,C:\git\test\test-forms\

# =========================
# [ELECTRON] - test-synchronizer
# =========================
# needed by electron
NormalFilePath=esbuild.exe,C:\git\test\test-synchronizer\

# TD dump directory (readable)
ReadFilePath=C:\TDAMP\

NormalFilePath=node.exe,C:\git\test\test-synchronizer\
NormalFilePath=powershell.exe,C:\git\test\test-synchronizer\
NormalFilePath=cmd.exe,C:\git\test\test-synchronizer\
NormalFilePath=git.exe,C:\git\test\test-synchronizer\
NormalFilePath=electron.exe,C:\git\test\test-synchronizer\

NormalFilePath=test.exe,C:\test\
NormalFilePath=test.exe,C:\git\test\test-synchronizer\.pnpm\electron@29.4.6\node_modules\electron\dist\

# =========================
# [FE] - test-forms
# =========================
# Needed by angular CLI
NormalFilePath=esbuild.exe,C:\git\test\test-forms\

# =========================
# [BE] - test-forms-backend
# =========================
# needed by tsx
NormalFilePath=esbuild.exe,C:\git\test\test-forms-backend\

```