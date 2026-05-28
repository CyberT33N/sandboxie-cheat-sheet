# Node Monorepo - Install Box + Run Box + Host Materialization

## Architectural status

This is the recommended generic boilerplate for a governed Node / PNPM monorepo when:

- VS Code or Cursor remains on the host
- dependency installation must execute inside Sandboxie
- the resulting dependency tree must still be visible to the host IDE

This is the validated replacement for the older host-installed / host-mirror baseline.

## Shared toolchain root

Recommended shared root:

```text
C:\shared\sandbox-toolchains\node-monorepo-general\
  cache\
    pnpm-store\
    nx-native\
  tools\
    bin\
    <tool>\
      <version>\
```

### Folder responsibilities

- `cache\pnpm-store\` stores the PNPM content-addressable store used by the install box.
- `cache\nx-native\` stores the Nx native cache when root-workspace Nx flows need it.
- `tools\bin\` stores generic external binary entry points if a future tool should be mirrored there.
- `tools\<tool>\<version>\` stores mirrored framework-specific external runtime payloads when they must not remain a fragile repo-local install artifact. A later Electron mirror is the main example.

Unlike the Python toolchain root, this generic Node monorepo root does **not** define a separate `work\input` / `work\output` area by default, because the repository itself remains the canonical work surface.

## PowerShell commands to create the structure

```powershell
$toolRoot = "C:\shared\sandbox-toolchains\node-monorepo-general"

New-Item -ItemType Directory -Force -Path "$toolRoot\cache\pnpm-store"
New-Item -ItemType Directory -Force -Path "$toolRoot\cache\nx-native"
New-Item -ItemType Directory -Force -Path "$toolRoot\tools\bin"
```

## Clean-start requirement

Before validating this architecture, the host dependency tree must be removed so that stale host artifacts cannot fake a successful install.

Recommended PowerShell cleanup command:

```powershell
$root = "C:\git\test\test-mono"

@(
  (Join-Path $root ".pnpm"),
  (Join-Path $root "node_modules")
) + (
  Get-ChildItem -Path (Join-Path $root "apps"), (Join-Path $root "libs"), (Join-Path $root "tools") -Directory -ErrorAction SilentlyContinue |
  ForEach-Object { Join-Path $_.FullName "node_modules" }
) |
Where-Object { Test-Path $_ } |
ForEach-Object { cmd /c rmdir /s /q "$_" }
```

Also clear both boxes before retesting:

- install box
- run box

## Important runtime rules

- Do **not** run `nvm use` inside either box.
- Use the fixed versioned binaries from the real NVM home instead.
- Prefer `pnpm.cmd` over `pnpm.ps1` in boxed workflows.
- Keep dependency installation in the install box and daily execution in the run box.
- Do **not** add `nvm.exe` visibility to the box configs; the boxed workflow uses fixed versioned `node.exe` / `pnpm.cmd` paths directly.
- Do **not** add framework-specific config lookups (for example Angular CLI user config) unless that exact framework actually runs in the box.

Example fixed toolchain commands:

```powershell
& "C:\Users\yourusername\AppData\Local\nvm\v26.2.0\node.exe" -v
& "C:\Users\yourusername\AppData\Local\nvm\v26.2.0\pnpm.cmd" -v
```

## `.vscode\settings.json`

This file makes "Run Script" and new integrated terminals use the run box by default while still exposing a dedicated install-box profile.

```json
{
  "terminal.integrated.automationProfile.windows": {
    "path": "C:\\Tools\\TestRunBoxShell\\cmd.exe",
    "env": {
      "PATH": "C:\\Users\\yourusername\\AppData\\Local\\nvm\\v26.2.0;${env:PATH}",
      "NX_DAEMON": "false",
      "NX_NATIVE_FILE_CACHE_DIRECTORY": "C:\\shared\\sandbox-toolchains\\node-monorepo-general\\cache\\nx-native"
    }
  },
  "terminal.integrated.defaultProfile.windows": "RunBox CMD",
  "terminal.integrated.profiles.windows": {
    "RunBox CMD": {
      "path": "C:\\Tools\\TestRunBoxShell\\cmd.exe",
      "env": {
        "PATH": "C:\\Users\\yourusername\\AppData\\Local\\nvm\\v26.2.0;${env:PATH}",
        "NX_DAEMON": "false",
        "NX_NATIVE_FILE_CACHE_DIRECTORY": "C:\\shared\\sandbox-toolchains\\node-monorepo-general\\cache\\nx-native"
      }
    },
    "InstallBox CMD": {
      "path": "C:\\Tools\\TestInstallBoxShell\\cmd.exe",
      "env": {
        "PATH": "C:\\Users\\yourusername\\AppData\\Local\\nvm\\v26.2.0;${env:PATH}",
        "NX_DAEMON": "false",
        "NX_NATIVE_FILE_CACHE_DIRECTORY": "C:\\shared\\sandbox-toolchains\\node-monorepo-general\\cache\\nx-native"
      }
    }
  }
}
```

## Install box

```ini
[TEST_MONO_INSTALL]
# Install domain:
# - use only for pnpm install / add / update / rebuild
# - do NOT use "nvm use" inside this box
# - use the fixed versioned toolchain path, for example:
#   C:\Users\yourusername\AppData\Local\nvm\v26.2.0\pnpm.cmd
# - this box is the only place where dependency materialization is allowed

Enabled=y
ConfigLevel=10
BorderColor=#e59f00,ttl,6,192,in
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
# Compatibility fixes (console)
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
UseRamDisk=n

# =========================
# Program control (forcing)
# =========================
ForceFolder=C:\Tools\TestInstallBoxShell\

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
# Resource access policy
# =========================

# --- Starship ---
NormalFilePath=starship.exe,C:\Program Files\starship\bin\
NormalFilePath=powershell.exe,C:\Program Files\starship\bin\
ReadFilePath=starship.exe,C:\Users\yourusername\.config\starship.toml

# --- Git (needed by Starship prompt) ---
NormalFilePath=starship.exe,C:\Program Files\Git\
NormalFilePath=git.exe,C:\Program Files\Git\

# --- Node / pnpm toolchain ---
# Visible shim layer
NormalFilePath=node.exe,C:\nvm4w\
NormalFilePath=powershell.exe,C:\nvm4w\
NormalFilePath=cmd.exe,C:\nvm4w\

# Real versioned NVM home
NormalFilePath=node.exe,C:\Users\yourusername\AppData\Local\nvm\
NormalFilePath=powershell.exe,C:\Users\yourusername\AppData\Local\nvm\
NormalFilePath=cmd.exe,C:\Users\yourusername\AppData\Local\nvm\

# pnpm home
NormalFilePath=node.exe,C:\Users\yourusername\AppData\Local\pnpm\
NormalFilePath=powershell.exe,C:\Users\yourusername\AppData\Local\pnpm\
NormalFilePath=cmd.exe,C:\Users\yourusername\AppData\Local\pnpm\
NormalFilePath=pnpm.exe,C:\Users\yourusername\AppData\Local\pnpm\

# --- Dedicated install shell binaries ---
NormalFilePath=powershell.exe,C:\Tools\TestInstallBoxShell\
NormalFilePath=cmd.exe,C:\Tools\TestInstallBoxShell\

# --- Nx (only relevant for workspaces that load Nx native during install/rebuild) ---
OpenFilePath=node.exe,C:\shared\sandbox-toolchains\node-monorepo-general\cache\nx-native\

# --- Workspace command surface ---
NormalFilePath=powershell.exe,C:\git\test\test-mono\
NormalFilePath=cmd.exe,C:\git\test\test-mono\
NormalFilePath=git.exe,C:\git\test\test-mono\

# --- Host-visible repo materialization surface ---
# In PNPM monorepos, workspace commands such as `pnpm add` or `pnpm update`
# may rewrite the shared root lockfile by creating a temporary file at the
# repo root and then renaming/replacing it into `pnpm-lock.yaml`.
# If only selected manifests or `node_modules` paths are open, the final rename
# can still fail with:
# [EPERM] EPERM: operation not permitted, rename
# 'C:\git\test\test-mono\pnpm-lock.yaml.<random>' -> 'C:\git\test\test-mono\pnpm-lock.yaml'
# The validated install-box baseline therefore opens the full example project
# root for `node.exe`.
OpenFilePath=node.exe,C:\git\test\test-mono\

# Dedicated PNPM store
OpenFilePath=node.exe,C:\shared\sandbox-toolchains\node-monorepo-general\cache\pnpm-store\
```

### Why the install box opens the example project root for `node.exe`

In the validated PNPM monorepo flow, the dedicated `--store-dir` solves only the content-addressable store side. It does **not** solve the final workspace commit step at the repo root.

`pnpm add`, `pnpm update`, and similar commands may rewrite the shared lockfile by writing a temporary file such as `pnpm-lock.yaml.<random>` and then renaming/replacing it into `pnpm-lock.yaml`.

During validation, narrower `OpenFilePath` lists that exposed only `.pnpm`, `node_modules`, `package.json`, `pnpm-workspace.yaml`, and `pnpm-lock.yaml*` still produced:

```text
[EPERM] EPERM: operation not permitted, rename
'C:\git\test\test-mono\pnpm-lock.yaml.<random>' -> 'C:\git\test\test-mono\pnpm-lock.yaml'
```

Opening the full example project root for `node.exe` in the install box fixed that failure mode. This is broader than the earlier per-path materialization list, but it remains intentionally limited to:

- the install box
- `node.exe`
- the example repo root

### Native addon overlay - `node-gyp`

If the monorepo contains a Windows native addon that builds through `node-gyp`, apply the dedicated host-sync overlay here:

- `docs\applications\programming-languages\node\dependencies\node-gyp\general.md`

That overlay documents only the additional `node-gyp`-specific requirements on top of this generic monorepo baseline:

- the central shared Python build helper binary
- the host-provided Microsoft Visual Studio Build Tools visibility rules
- the extra install-box `OpenFilePath` lines for `python.exe`, `MSBuild.exe`, and related spawned build tools
- the validated direct `node-gyp` and default `pnpm` rebuild commands

## Run box

```ini
[TEST_MONO_RUN]
# Run domain:
# - use for pnpm run dev / tests / watchers / framework launch commands
# - do NOT use "nvm use" inside this box
# - use the fixed versioned toolchain path or inject the versioned path via VS Code settings
# - this box consumes the materialized host dependency tree read-only

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
UseRamDisk=n

# =========================
# Program control (forcing)
# =========================
ForceFolder=C:\Tools\TestRunBoxShell\

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
# Resource access policy
# =========================

# --- Starship ---
NormalFilePath=starship.exe,C:\Program Files\starship\bin\
NormalFilePath=powershell.exe,C:\Program Files\starship\bin\
ReadFilePath=starship.exe,C:\Users\yourusername\.config\starship.toml

# --- Git (needed by Starship prompt) ---
NormalFilePath=starship.exe,C:\Program Files\Git\
NormalFilePath=git.exe,C:\Program Files\Git\

# --- Node / pnpm toolchain ---
# Visible shim layer
NormalFilePath=node.exe,C:\nvm4w\
NormalFilePath=powershell.exe,C:\nvm4w\
NormalFilePath=cmd.exe,C:\nvm4w\

# Real versioned NVM home
NormalFilePath=node.exe,C:\Users\yourusername\AppData\Local\nvm\
NormalFilePath=powershell.exe,C:\Users\yourusername\AppData\Local\nvm\
NormalFilePath=cmd.exe,C:\Users\yourusername\AppData\Local\nvm\

# pnpm home
NormalFilePath=node.exe,C:\Users\yourusername\AppData\Local\pnpm\
NormalFilePath=powershell.exe,C:\Users\yourusername\AppData\Local\pnpm\
NormalFilePath=cmd.exe,C:\Users\yourusername\AppData\Local\pnpm\
NormalFilePath=pnpm.exe,C:\Users\yourusername\AppData\Local\pnpm\

# --- Dedicated run shell binaries ---
NormalFilePath=powershell.exe,C:\Tools\TestRunBoxShell\
NormalFilePath=cmd.exe,C:\Tools\TestRunBoxShell\

# --- Nx (only relevant for root-workspace Nx flows) ---
OpenFilePath=node.exe,C:\shared\sandbox-toolchains\node-monorepo-general\cache\nx-native\

# --- Runtime dependency surfaces must stay read-only ---
# Root dependency tree
ReadFilePath=node.exe,C:\git\test\test-mono\.pnpm\
ReadFilePath=node.exe,C:\git\test\test-mono\node_modules\

# Package-level dependency trees
ReadFilePath=node.exe,C:\git\test\test-mono\apps\*\node_modules
ReadFilePath=node.exe,C:\git\test\test-mono\apps\*\node_modules\*
ReadFilePath=node.exe,C:\git\test\test-mono\libs\*\node_modules
ReadFilePath=node.exe,C:\git\test\test-mono\libs\*\node_modules\*
ReadFilePath=node.exe,C:\git\test\test-mono\tools\*\node_modules
ReadFilePath=node.exe,C:\git\test\test-mono\tools\*\node_modules\*

# --- Workspace command surface ---
NormalFilePath=powershell.exe,C:\git\test\test-mono\
NormalFilePath=cmd.exe,C:\git\test\test-mono\
NormalFilePath=git.exe,C:\git\test\test-mono\
NormalFilePath=node.exe,C:\git\test\test-mono\
```

## From-scratch workflow

### Step 1 - prepare the shared toolchain root

Create:

- `C:\shared\sandbox-toolchains\node-monorepo-general\cache\pnpm-store\`
- `C:\shared\sandbox-toolchains\node-monorepo-general\cache\nx-native\`
- optional future external tool mirrors under `tools\...`

### Step 2 - prepare the dedicated boxed shells

Create dedicated launcher copies:

- `C:\Tools\TestInstallBoxShell\cmd.exe`
- `C:\Tools\TestRunBoxShell\cmd.exe`

PowerShell launchers can also exist, but the documented baseline prefers `cmd.exe` for boxed package-manager entry to avoid `pnpm.ps1`.

### Step 3 - clear the old host dependency tree

Use the cleanup command documented above.

### Step 4 - run the install from the install box

Example:

```powershell
Set-Location "C:\git\test\test-mono"
# If the workspace uses Nx native during install/rebuild, keep the dedicated
# shared cache active in this shell as well.
$env:NX_DAEMON = "false"
$env:NX_NATIVE_FILE_CACHE_DIRECTORY = "C:\shared\sandbox-toolchains\node-monorepo-general\cache\nx-native"
& "C:\Users\yourusername\AppData\Local\nvm\v26.2.0\pnpm.cmd" install --store-dir "C:\shared\sandbox-toolchains\node-monorepo-general\cache\pnpm-store"
```

### Step 5 - verify host-visible materialization

The following must now exist on the host-visible project path:

- `C:\git\test\test-mono\.pnpm\`
- `C:\git\test\test-mono\node_modules\`
- package-level `node_modules`

### Step 6 - run package scripts from the run box

Examples:

```powershell
Set-Location "C:\git\test\test-mono"
& "C:\Users\yourusername\AppData\Local\nvm\v26.2.0\pnpm.cmd" run dev
```

or package-local:

```powershell
Set-Location "C:\git\test\test-mono\apps\frontend"
& "C:\Users\yourusername\AppData\Local\nvm\v26.2.0\pnpm.cmd" start
```

## Related documents

- `docs\applications\IDE\vscode\methods\host-not-isolated\dependencies-installed-in-box.md`
- `docs\applications\IDE\vscode\methods\host-not-isolated\dependencies-installed-on-host.md`
- `docs\applications\package-manager\pnpm\general.md`
- `docs\applications\programming-languages\node\nvm\general.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\general.md`
- framework-specific overlays such as Electron / Electron-Vite docs when a package needs more than the generic Node monorepo baseline
