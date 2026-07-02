# TeamViewer in `boxed-owned`

## Verifizierter Stand

Dieses Dokument beschreibt nur den verifizierten boxed Ablauf.

Aktuell ist nur ein eingeschraenkter boxed Zustand verifiziert:

- `TeamViewer` laesst sich in der Box starten.
- Ein boxed Login in den `TeamViewer`-Account funktioniert in diesem Setup derzeit nicht stabil beziehungsweise bleibt haengen.
- Die Direktverbindung ueber `ID` und `Passwort` ist damit in diesem Setup nicht der verifizierte Weg.
- Wenn die Gegenseite eingehende Verbindungen explizit erlaubt und alle benoetigten Rechte vorhanden sind, bleibt eine session-ID-basierte Verbindung der einzige noch in Betracht kommende boxed Pfad.

## Login und Host-SSO

Das boxed Anmelden konnte im hier betrachteten Setup nicht reproduzierbar geloest werden.

`UseSecurityMode=n` veraendert zwar das Anmeldeverhalten, fuehrt in den hier beobachteten Faellen aber zur unerwuenschten Uebernahme des Host-Anmeldezustands. Dieser Zustand gehoert deshalb nicht zu dem hier dokumentierten Zielbild.

## Sichtbarkeit der Setup-Optionen

Unter `Generelle Optionen > Restriktionen` muss `Verhindere die Beeinträchtigung der Benutzeroberfläche` deaktiviert werden.

Das ist weiterhin notwendig, weil sonst die Checkbox beziehungsweise die auswählbare Option in der TeamViewer-Setup-Datei nicht sichtbar ist.

## Zwei Ausführungswege

Es gibt zwei praktikable Wege, um `TeamViewer` in dieser Box bereitzustellen:

1. Präferierter Weg: Nicht installieren, sondern direkt `Nur starten` wählen.
2. Alternativer Weg: Installieren. Dabei können zwar die bereits beobachteten Fehler erscheinen, aber wenn die Installation weiterläuft, sind die Binaries trotzdem vorhanden und können anschließend normal unter `Program Files` gestartet werden.

## Relevante Settings

Fuer die aktuell dokumentierte Baseline, in der `TeamViewer` wieder startet und session-basierte Verbindungsversuche moeglich bleiben, sind aus den hier betrachteten Anpassungen vor allem diese Settings relevant:

- `ProtectHostImages=n`
- `CoverBoxedWindows=n`
- `OpenWndStation=y`

`OpenWndStation=y` wird hier benoetigt, damit der fruehere `OpenDesktop`-bezogene Start- beziehungsweise UI-Block nicht mehr auftritt.

Die boxed Anmeldung wird dadurch jedoch nicht geloest. Zwischenzeitlich getestete Login-Workarounds wie `SpecialImage=chrome,msedgewebview2.exe`, `ExternalManifestHack=msedgewebview2.exe,y` oder eine Lockerung ueber `UseSecurityMode=n` haben keinen stabilen boxed Login-Pfad ergeben, der dem gewuenschten Zielbild entspricht.

## Aktuell getestete Baseline-Config

Diese Config spiegelt den Stand wider, in dem `TeamViewer` wieder startet. Sie beschreibt keinen verifizierten Login-Pfad in der Box.

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
CoverBoxedWindows=n

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

# Allow the current TeamViewer baseline to access the window station.
OpenWndStation=y

# Hide non-system processes from the boxed process view.
HideNonSystemProcesses=y

# Lower ConHost integrity in the boxed console context.
DropConHostIntegrity=y

# Block print spooler access.
ClosePrintSpooler=y

# Block power-state interference from boxed processes.
BlockInterferePower=y

# Block external interference control behavior.

# Block screen capture from the boxed process context.
BlockScreenCapture=y


# ==================================================
# Group: Python runtime compatibility switch
# Purpose: This compatibility profile allows native
# Python packages to load from the dedicated toolchain.
# ==================================================

# Enable host-image protection for this compatibility box.
ProtectHostImages=n
```