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

# Enable host-image protection for this compatibility box.
ProtectHostImages=y


# ==================================================
# Group: Process forcing
# Purpose: Force Python processes into this sandbox.
# ==================================================

# Force python.exe to start inside this sandbox.
ForceProcess=python.exe

# ==================================================
# Group: Dedicated python-general toolchain root
# Purpose: Treat the complete shared toolchain root as
# the canonical runtime, dependency, cache, and work area.
# ==================================================

# Open the dedicated Python General toolchain root.
OpenFilePath=C:\shared\sandbox-toolchains\python-general\
```