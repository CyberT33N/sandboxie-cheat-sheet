# INI Settings:
```ini
# =========================
# Sandbox core / enablement
# =========================
Enabled=y
ConfigLevel=10
BorderColor=#027df7,ttl,6,192,in

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
Template=BlockLocalConnect
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
CallTrace=*
FileTrace=*
PipeTrace=*
KeyTrace=*
IpcTrace=*
GuiTrace=*
ClsidTrace=*
NetFwTrace=*
DnsTrace=y
ApiTrace=y
HookTrace=y
DebugTrace=y
ErrorTrace=y

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

# --- Repo (tooling) ---
NormalFilePath=node.exe,C:\git\test\test-synchronizer\
NormalFilePath=powershell.exe,C:\git\test\test-synchronizer\
NormalFilePath=cmd.exe,C:\git\test\test-synchronizer\
NormalFilePath=git.exe,C:\git\test\test-synchronizer\

# --- Node (nvm4w) ---
NormalFilePath=node.exe,C:\nvm4w\nodejs\
NormalFilePath=powershell.exe,C:\nvm4w\nodejs\
NormalFilePath=cmd.exe,C:\nvm4w\nodejs\

# --- pnpm ---
NormalFilePath=node.exe,C:\Users\denni\AppData\Local\pnpm\
NormalFilePath=powershell.exe,C:\Users\denni\AppData\Local\pnpm\
NormalFilePath=cmd.exe,C:\Users\denni\AppData\Local\pnpm\

# --- Preview app + Electron runtime ---
NormalFilePath=test.exe,C:\test\
NormalFilePath=test.exe,C:\git\test\test-synchronizer\.pnpm\electron@29.4.6\node_modules\electron\dist\

# --- DevBoxShell binaries ---
NormalFilePath=powershell.exe,C:\Tools\DevBoxShell\
NormalFilePath=cmd.exe,C:\Tools\DevBoxShell\
```