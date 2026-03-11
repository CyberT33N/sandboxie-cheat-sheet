# Mono Repo (nx + electron-vite + pnpm)

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
ForceFolder=C:\Tools\DevBoxShell

# =========================
# Monitoring / tracing
# =========================
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

# --- nx ---
OpenFilePath=node.exe,C:\shared\nx-native-cache\

# --- Cursor ---
# Needed for debugging in electron
ReadFilePath=electron.exe,C:\Users\denni\AppData\Local\Programs\cursor\resources\app\

# =========================
# [ELECTRON] - test
# =========================
# needed by electron
NormalFilePath=esbuild.exe,C:\git\test\test-mono\apps\test\

NormalFilePath=node.exe,C:\git\test\test-mono\apps\test\
NormalFilePath=powershell.exe,C:\git\test\test-mono\apps\test\
NormalFilePath=cmd.exe,C:\git\test\test-mono\apps\test\
NormalFilePath=git.exe,C:\git\test\test-mono\apps\test\
NormalFilePath=electron.exe,C:\git\test\test-mono\apps\test\

NormalFilePath=test.exe,C:\test\
NormalFilePath=test.exe,C:\git\test\test-mono\node_modules\.pnpm\electron@29.4.6\node_modules\electron\dist\
NormalFilePath=test.exe,C:\git\test\test-mono\apps\test\dist\

NormalFilePath=electron.exe,C:\git\test\test-mono\node_modules\.pnpm\electron@29.4.6\node_modules\electron\dist\
NormalFilePath=electron.exe,C:\git\test\test-mono\

# Program control (forcing)
ForceProcess=test.exe
ForceProcess=electron.exe

ForceFolder=C:\git\test\test-mono\node_modules\.pnpm\electron@29.4.6\node_modules\electron\dist\

# [test] - [ELECTRON-VITE] - Wir müssen die gebauten Artefakte freigeben für Cursor, weil es nicht boxed läuft damit wir debuggen können
OpenFilePath=node.exe,C:\git\test\test-mono\apps\test\out\
# Das Gleiche gilt für die Electron-Exe. Sonst hat die Electron-Exe in der Sandbox keinen Zugriff auf die Artefakte, die auf dem Host-System liegen.
OpenFilePath=node.exe,C:\git\test\test-mono\apps\privyou\out\

# =========================
# [test] - [MONO] - Frontend
# =========================
# Needed by angular CLI
NormalFilePath=esbuild.exe,C:\git\test\test-mono\apps\frontend\
NormalFilePath=powershell.exe,C:\git\test\test-mono\apps\frontend\
NormalFilePath=cmd.exe,C:\git\test\test-mono\apps\frontend\
NormalFilePath=node.exe,C:\git\test\test-mono\apps\frontend\

# =========================
# [test] - [MONO] - Backend
# =========================
# needed by tsx
NormalFilePath=esbuild.exe,C:\git\test\test-mono\apps\backend\
NormalFilePath=powershell.exe,C:\git\test\test-mono\apps\backend\
NormalFilePath=cmd.exe,C:\git\test\test-mono\apps\backend\
NormalFilePath=node.exe,C:\git\test\test-mono\apps\backend\

# =========================
# [test] - [MONO] - Workspace
# =========================
NormalFilePath=esbuild.exe,C:\git\test\test-mono\
NormalFilePath=powershell.exe,C:\git\test\test-mono\
NormalFilePath=cmd.exe,C:\git\test\test-mono\
NormalFilePath=node.exe,C:\git\test\test-mono\


```