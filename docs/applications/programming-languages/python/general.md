
# Python

## Single source of truth

This document is the primary architectural reference for Python inside Sandboxie in this repository.

It defines:

- the supported architecture options
- the recommended toolchain-root model
- the currently validated compatibility configuration
- the split between runtime, dependency, versioning, GPU, troubleshooting, and legacy documents

## Documentation map

### Core architecture
- docs/applications/programming-languages/python/general.md
- docs/applications/programming-languages/python/cli.md
- docs/applications/programming-languages/python/dependencies.md
- docs/applications/programming-languages/python/versioning.md

### GPU area
- docs/applications/programming-languages/python/gpu/general.md
- docs/applications/programming-languages/python/gpu/install.md
- docs/applications/programming-languages/python/gpu/uninstall.md
- docs/applications/programming-languages/python/gpu/troubleshooting.md

### Legacy host-virtualization reference
- docs/applications/programming-languages/python/python-manager/pyenv/general.md

### Troubleshooting
- docs/applications/programming-languages/python/troubleshooting.md

### Related external references
- docs/applications/operating-systems/windows/dependency-manager/chocolatey/general.md
- docs/applications/tools/media/ffmpeg/general.md

---

## Architectural problem statement

Python inside Sandboxie is not just about launching `python.exe`.

The effective runtime boundary includes:

- the copied interpreter
- generated CLI launchers
- installed packages
- native `.pyd` modules
- DLL-backed dependencies
- caches
- working directories

This becomes critical for packages such as `docling`, `torch`, OCR stacks, and image libraries, because these workloads often use native extensions and Windows-facing runtime behavior.

---

## Architecture options

### Option 1 — install installers directly inside the sandbox

This option is not the recommended baseline for this repository.

In the tested repository context, MSI, MSIX, and related installer flows did not operate cleanly enough inside the sandbox without weakening additional protections.

### Option 2 — install on the host and rely on host virtualization

This option is a legacy pattern and is explicitly not recommended.

Why it is an anti-pattern:

- package installation runs unsandboxed on the host
- compromised dependencies would affect the host directly
- the runtime becomes dependent on host-managed Python state
- the model is harder to reproduce and harder to reason about

### Option 3 — dedicated shared toolchain root with copied runtime binaries

This is the recommended architecture for this repository.

The model is:

- copy the real runtime binaries into a dedicated shared root
- install packages from inside the sandbox into a sandbox-owned user base
- keep runtime, packages, caches, temp files, and work directories under one explicit toolchain area
- avoid host-side package installation as the primary operating model

---

## Recommended sandbox split

The Python area must distinguish between two sandbox profiles.

### Profile A — compatibility sandbox

Use this profile for packages that load native DLL or `.pyd` modules, for example:

- document conversion stacks
- OCR toolchains
- image libraries
- torch-based packages

This profile uses `ProtectHostImages=n` because the dedicated toolchain root is intentionally treated as the runtime boundary for that box.

### Profile B — strict sandbox

Use this profile for simpler workloads that do not require problematic native Windows-facing dependency behavior.

Typical examples:

- simple calculation scripts
- pure-Python utilities
- lightweight internal automation tasks

This profile keeps `ProtectHostImages=y` and should only be used when the package stack is known to work under the stricter DLL-loading model.

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
    tmp\
  work\
    input\
    output\
```

### Folder responsibilities

- `python\` stores the copied runtime binaries for the selected Python version.
- `userbase\` stores packages and generated CLI launchers from `pip install --user`.
- `cache\pip\` stores pip download cache data.
- `cache\huggingface\` stores model and hub cache data.
- `cache\torch\` stores torch-related cache data.
- `cache\tmp\` stores temporary unpack and reinstall artifacts for large wheels.
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
New-Item -ItemType Directory -Force -Path "$toolRoot\cache\tmp"
New-Item -ItemType Directory -Force -Path "$toolRoot\work\input"
New-Item -ItemType Directory -Force -Path "$toolRoot\work\output"
```

---

## `settings.ini` — validated compatibility profile

The following configuration block reflects the currently validated compatibility profile that worked with the dedicated `python-general` toolchain root.

```ini
# ==================================================
# Group: Core box activation and baseline behavior
# Purpose: Enable the sandbox and keep the base box
# behavior active.
# ==================================================

# Enable the sandbox definition.
Enabled=y

# Block direct network file usage from inside the box.
BlockNetworkFiles=y

# Apply the boxed window border.
BorderColor=#0423ee,ttl,6,192,in,6


# ==================================================
# Group: Template-driven baseline hardening
# Purpose: Keep the validated template set that this
# box relies on.
# ==================================================

# Keep lingering program handling enabled.
Template=LingerPrograms

# Keep qWave handling enabled.
Template=qWave

# Keep hook-skipping compatibility enabled.
Template=SkipHook

# Keep Google Japanese IME compatibility enabled.
Template=GoogleJapaneseIME

# Keep Google Toolbar IE compatibility enabled.
Template=GoogleToolbarIE

# Block WMI access.
Template=BlockAccessWMI

# Keep telemetry blocking enabled.
Template=BlockTelemetry

# Keep Adobe Acrobat compatibility rules enabled.
Template=AdobeAcrobat

# Keep Adobe Distiller compatibility rules enabled.
Template=AdobeDistiller

# Keep Adobe Acrobat Reader compatibility rules enabled.
Template=AdobeAcrobatReader

# Keep the Chromium KB5027231 fix enabled.
Template=Chrome_KB5027231_fix

# Hide installed programs from boxed processes.
Template=HideInstalledPrograms

# Keep Nitro PDF 5 compatibility rules enabled.
Template=NitroPDF5

# Keep Pdf995 compatibility rules enabled.
Template=Pdf995

# Keep Nitro PDF 6 compatibility rules enabled.
Template=NitroPDF6


# ==================================================
# Group: Isolation mode and administrative controls
# Purpose: Keep the validated isolation and admin
# control baseline for this box.
# ==================================================

# Keep the configuration level at the validated value.
ConfigLevel=10

# Enable file-delete version 2 handling.
UseFileDeleteV2=y

# Enable registry-delete version 2 handling.
UseRegDeleteV2=y

# Present fake administrative rights in the box.
FakeAdminRights=y

# Visually cover boxed windows.
CoverBoxedWindows=y

# Restart all boxed processes together when required.
ForceRestartAll=y

# Hide firmware information from boxed processes.
HideFirmwareInfo=y

# Randomize the boxed registry identity.
RandomRegUID=y

# Hide the disk serial number.
HideDiskSerialNumber=y

# Hide the network adapter MAC address.
HideNetworkAdapterMAC=y

# Restrict editing of the box to administrators.
EditAdminOnly=y

# Restrict monitoring of the box to administrators.
MonitorAdminOnly=y

# Keep privacy mode enabled for the validated compatibility profile.
UsePrivacyMode=y


# ==================================================
# Group: Additional runtime hardening
# Purpose: Add non-Python-specific protections that
# were validated together with this box.
# ==================================================

# Allow boxed job objects when required by the runtime.
AllowBoxedJobs=y

# Hide non-system processes from the boxed process view.
HideNonSystemProcesses=y

# Lower ConHost integrity in the boxed console context.
DropConHostIntegrity=y

# Block print spooler access.
ClosePrintSpooler=y

# Block power-state interference.
BlockInterferePower=y

# Block interference-control behavior.
BlockInterferenceControl=y

# Block screen capture from the boxed process context.
BlockScreenCapture=y


# ==================================================
# Group: Python compatibility switch
# Purpose: Allow native Python package loading from
# the dedicated toolchain root.
# ==================================================

# Disable host-image protection for the compatibility profile.
ProtectHostImages=n


# ==================================================
# Group: Dedicated python-general toolchain root
# Purpose: Treat the complete shared toolchain root as
# the canonical runtime, dependency, cache, and work area.
# ==================================================

# Open the complete python-general toolchain root.
OpenFilePath=C:\shared\sandbox-toolchains\python-general\
```

### Strict profile delta

The strict profile should reuse the same baseline and change only the compatibility switch unless a concrete workload proves that more changes are required.

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

Use PowerShell inside the sandbox when you want direct logs and stable boxed execution.

If the session starts in `cmd.exe`, switch into PowerShell first before using `$toolRoot`, `$env:...`, or `New-Item` syntax.

### Step 3 — set the session variables

```powershell
$toolRoot = "C:\shared\sandbox-toolchains\python-general"
$pythonExe = "$toolRoot\python\3.12.9\python.exe"
$env:PYTHONUSERBASE = "$toolRoot\userbase"
$env:PIP_CACHE_DIR = "$toolRoot\cache\pip"
$env:HF_HOME = "$toolRoot\cache\huggingface"
$env:TORCH_HOME = "$toolRoot\cache\torch"
$env:TEMP = "$toolRoot\cache\tmp"
$env:TMP = "$toolRoot\cache\tmp"
$env:PATH = "$toolRoot\userbase\Python312\Scripts;" + $env:PATH
```

### Step 4 — install dependencies into the sandbox-owned user base

```powershell
& $pythonExe -m pip install --user <package-name>
```

### Step 5 — execute the CLI from the dedicated toolchain

Generic pattern:

```powershell
& "$toolRoot\userbase\Python312\Scripts\<tool>.exe" <arguments>
```

---

## GPU note

GPU-capable workloads require their own documentation set because the deciding factor is not Sandboxie alone.

The exact Python version, the exact `torch` build, and the toolchain-specific reinstall flow must all line up.

See:

- docs/applications/programming-languages/python/gpu/general.md
- docs/applications/programming-languages/python/gpu/install.md
- docs/applications/programming-languages/python/gpu/uninstall.md
- docs/applications/programming-languages/python/gpu/troubleshooting.md

---

## Operational notes

- If a package depends on native `.pyd` or DLL-backed modules, prefer the compatibility sandbox profile.
- If a workload is simple and does not require the relaxed compatibility path, use the strict sandbox profile.
- Host installation remains a legacy method and is intentionally not the recommended architecture.

---

## Related documents

- docs/applications/programming-languages/python/cli.md
- docs/applications/programming-languages/python/dependencies.md
- docs/applications/programming-languages/python/versioning.md
- docs/applications/programming-languages/python/python-manager/pyenv/general.md


