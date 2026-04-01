
# Python

## Single source of truth

This document is the primary architectural reference for Python inside Sandboxie in this repository.

It defines:

- the supported architecture options
- the recommended sandbox model
- the configuration baseline for the recommended compatibility sandbox
- the documentation split between main, workflow, dependency, versioning, and legacy materials

## Documentation map

### Main architecture references
- docs\applications\programming-languages\python\general.md
- docs\applications\programming-languages\python\cli.md
- docs\applications\programming-languages\python\dependencies.md
- docs\applications\programming-languages\python\versioning.md

### Legacy host-virtualization reference
- docs\applications\programming-languages\python\python-manager\pyenv\general.md

### Troubleshooting
- docs\applications\programming-languages\python\troubleshooting.md

### Dependency manager references
- docs\applications\operating-systems\windows\dependency-manager\chocolatey\general.md
- docs\applications\tools\media\ffmpeg\general.md

---

## Architectural problem statement

Python in Sandboxie can be operated through multiple architectural paths, but these paths do not have the same security profile or operational stability.

The key problem is not only how to start `python.exe`, but how the entire runtime stack behaves when it loads:

- Python launchers
- generated CLI executables
- native `.pyd` modules
- DLL-backed dependencies
- cache directories
- work directories

This becomes especially important for packages such as `docling` that use native modules, parsing libraries, OCR components, and other Windows-facing dependencies.

---

## Architecture options

### Option 1 — install installers directly inside the sandbox

This option is not the recommended baseline for this environment.

In this repository context, MSI, MSIX, and related installer flows did not operate cleanly enough inside the sandbox without relaxing additional protections. That makes this option operationally unstable for the current setup.

### Option 2 — install on the host and rely on host virtualization

This option is a legacy pattern and is explicitly not recommended.

Why it is an anti-pattern:

- package installation runs unsandboxed on the host
- compromised dependencies would affect the host installation path directly
- the runtime becomes dependent on host-side `pyenv`, host-side user paths, and implicit virtualization behavior
- the model is harder to reason about and less reproducible

This option may work in some historical scenarios, but it is architecturally weaker than the dedicated toolchain-root approach.

### Option 3 — dedicated shared toolchain root with copied runtime binaries

This is the recommended architecture for this repository.

The idea is to:

- copy the real Python runtime into a dedicated shared root
- install Python packages from inside the sandbox into a sandbox-owned user base
- keep cache, runtime, and work data under one clearly named toolchain area
- avoid host-side package installation as the primary operating model

---

## Recommended sandbox split

The documentation must differentiate between two sandbox profiles.

### Profile A — compatibility sandbox

Use this profile for packages that load native DLL or `.pyd` modules, for example:

- document conversion stacks
- OCR toolchains
- image libraries
- torch-based packages

This profile uses `ProtectHostImages=n` because the dedicated shared toolchain root is intentionally treated as the runtime boundary for that sandbox.

### Profile B — strict sandbox

Use this profile for simpler Python workloads that do not require problematic native Windows-facing dependency behavior.

Examples:

- simple calculation scripts
- pure-Python utilities
- internal automation scripts with minimal native dependency load

This profile keeps `ProtectHostImages=y` and should only be used when the dependency stack is known to work under the stricter DLL-loading model.

---

## Recommended folder structure

The shared toolchain root should remain generic and reusable instead of being tool-specific.

```text
C:\shared\sandbox-toolchains\python-general\
  python\
    3.12.9\
      python.exe
  userbase\
  cache\
    pip\
    huggingface\
    torch\
  work\
    input\
    output\
```

### Folder responsibilities

- `python\` stores the copied runtime binaries for the selected Python version.
- `userbase\` stores packages and generated CLI launchers from `pip install --user`.
- `cache\` stores package-manager caches and model caches.
- `work\input\` stores input data that the sandboxed tools should consume.
- `work\output\` stores generated outputs.

### PowerShell commands to create the structure

```powershell
$toolRoot = "C:\shared\sandbox-toolchains\python-general"

New-Item -ItemType Directory -Force -Path "$toolRoot\python\3.12.9"
New-Item -ItemType Directory -Force -Path "$toolRoot\userbase"
New-Item -ItemType Directory -Force -Path "$toolRoot\cache\pip"
New-Item -ItemType Directory -Force -Path "$toolRoot\cache\huggingface"
New-Item -ItemType Directory -Force -Path "$toolRoot\cache\torch"
New-Item -ItemType Directory -Force -Path "$toolRoot\work\input"
New-Item -ItemType Directory -Force -Path "$toolRoot\work\output"
```

---

## `settings.ini` — recommended compatibility profile

This is the current working baseline for the recommended compatibility sandbox.

```ini
# ==================================================
# Group: Core box activation and baseline behavior
# Purpose: Enable the sandbox and keep the standard
# baseline restrictions that are part of this box.
# ==================================================

# Enable the sandbox definition.
Enabled=y

# Block direct use of network file paths from inside the box.
BlockNetworkFiles=y

# Apply the visual border to boxed windows.
BorderColor=#0423ee,ttl,6,192,in,6


# ==================================================
# Group: Template-driven application hardening
# Purpose: Keep the currently validated template set
# that this sandbox uses as its baseline policy.
# ==================================================

# Keep lingering program handling enabled.
Template=LingerPrograms

# Keep qWave-related handling enabled.
Template=qWave

# Keep hook-skipping compatibility enabled.
Template=SkipHook

# Keep Google Japanese IME compatibility enabled.
Template=GoogleJapaneseIME

# Keep Google Toolbar IE compatibility enabled.
Template=GoogleToolbarIE

# Block WMI access for stronger isolation.
Template=BlockAccessWMI

# Keep telemetry blocking enabled.
Template=BlockTelemetry

# Keep Adobe Acrobat compatibility rules enabled.
Template=AdobeAcrobat

# Keep Adobe Distiller compatibility rules enabled.
Template=AdobeDistiller

# Keep Adobe Acrobat Reader compatibility rules enabled.
Template=AdobeAcrobatReader

# Keep the Chromium KB5027231 compatibility fix enabled.
Template=Chrome_KB5027231_fix

# Hide installed programs from boxed applications.
Template=HideInstalledPrograms

# Keep Nitro PDF 5 compatibility rules enabled.
Template=NitroPDF5

# Keep Pdf995 compatibility rules enabled.
Template=Pdf995

# Keep Nitro PDF 6 compatibility rules enabled.
Template=NitroPDF6


# ==================================================
# Group: Isolation mode and administrative controls
# Purpose: Keep the sandbox in the validated hardened
# operating mode for this compatibility profile.
# ==================================================

# Keep the configuration level at the current validated value.
ConfigLevel=10

# Enable the security-hardened sandbox mode.
UseSecurityMode=y

# Keep file delete version 2 enabled.
UseFileDeleteV2=y

# Keep registry delete version 2 enabled.
UseRegDeleteV2=y

# Present fake administrative rights inside the sandbox.
FakeAdminRights=y

# Cover boxed windows to make boxed UI state visible.
CoverBoxedWindows=y

# Restart all boxed processes together when required.
ForceRestartAll=y

# Hide firmware details from boxed processes.
HideFirmwareInfo=y

# Randomize the boxed registry identity.
RandomRegUID=y

# Hide the disk serial number from boxed processes.
HideDiskSerialNumber=y

# Hide the network adapter MAC address from boxed processes.
HideNetworkAdapterMAC=y

# Restrict editing of the sandbox to administrators.
EditAdminOnly=y

# Restrict monitoring of the sandbox to administrators.
MonitorAdminOnly=y

# Enable privacy mode for stronger data protection.
UsePrivacyMode=y


# ==================================================
# Group: Additional hardening and anti-interference
# Purpose: Add stricter runtime protections that are
# not Python-specific but are part of the validated box.
# ==================================================

# Allow boxed job objects when required by the runtime.
AllowBoxedJobs=y

# Hide non-system processes from the boxed process view.
HideNonSystemProcesses=y

# Lower ConHost integrity in the boxed console context.
DropConHostIntegrity=y

# Block print spooler access.
ClosePrintSpooler=y

# Block power-state interference from boxed processes.
BlockInterferePower=y

# Block external interference control behavior.
BlockInterferenceControl=y

# Block screen capture from the boxed process context.
BlockScreenCapture=y


# ==================================================
# Group: Python runtime compatibility switch
# Purpose: This compatibility profile allows native
# Python packages to load from the dedicated toolchain.
# ==================================================

# Disable host-image protection for this compatibility box.
ProtectHostImages=n


# ==================================================
# Group: Process forcing
# Purpose: Force Python processes into this sandbox.
# ==================================================

# Force python.exe to start inside this sandbox.
ForceProcess=python.exe


# ==================================================
# Group: External helper binaries
# Purpose: Keep direct read-only access to helper tools
# that remain host-managed for this runtime.
# ==================================================

# Make Chocolatey shim binaries readable.
ReadFilePath=C:\ProgramData\chocolatey\bin\

# Make the Chocolatey ffmpeg toolchain readable.
ReadFilePath=C:\ProgramData\chocolatey\lib\ffmpeg\tools\ffmpeg\bin\


# ==================================================
# Group: Dedicated python-general toolchain root
# Purpose: Treat the complete shared toolchain root as
# the canonical runtime, dependency, cache, and work area.
# ==================================================

# Open the dedicated Python General toolchain root.
OpenFilePath=C:\shared\sandbox-toolchains\python-general\
```

### Strict profile delta

The strict profile should reuse the same baseline and change only the compatibility switch unless a concrete workload proves that additional deltas are required.

```ini
# Keep host-image protection enabled in the strict profile.
ProtectHostImages=y
```

---

## Workflow summary

### Step 1 — copy the runtime into the toolchain root

Copy the real runtime binaries for the chosen version into:

```text
C:\shared\sandbox-toolchains\python-general\python\3.12.9\
```

### Step 2 — start PowerShell inside the sandbox

Use a terminal inside the sandbox when you want direct logs and a stable boxed environment.

### Step 3 — set the session variables

```powershell
$toolRoot = "C:\shared\sandbox-toolchains\python-general"
$pythonExe = "$toolRoot\python\3.12.9\python.exe"
$env:PYTHONUSERBASE = "$toolRoot\userbase"
$env:PIP_CACHE_DIR = "$toolRoot\cache\pip"
$env:HF_HOME = "$toolRoot\cache\huggingface"
$env:TORCH_HOME = "$toolRoot\cache\torch"
$env:PATH = "$toolRoot\userbase\Python312\Scripts;" + $env:PATH
```

### Step 4 — install dependencies into the sandbox-owned user base

```powershell
& $pythonExe -m pip install --user <package-name>
```

### Step 5 — execute the CLI or script from the dedicated toolchain

Generic pattern:

```powershell
& "$toolRoot\userbase\Python312\Scripts\<tool>.exe" <arguments>
```

---

## Operational notes

- If a package depends on native `.pyd` or DLL-backed modules, prefer the compatibility sandbox profile.
- If a workload is simple and does not require the relaxed compatibility path, use the strict sandbox profile.
- Host installation remains a legacy method and is intentionally not the recommended architecture.

---

## Related documents

- docs\applications\programming-languages\python\cli.md
- docs\applications\programming-languages\python\dependencies.md
- docs\applications\programming-languages\python\versioning.md
- docs\applications\programming-languages\python\python-manager\pyenv\general.md


