
# Python


## Troubleshooting
- docs\applications\programming-languages\python\troubleshooting.md

## Python Manager

#### pyenv
- docs\applications\programming-languages\python\frameworks\pyenv\general.md

## Dependency Manager

### chocolatey
- docs\applications\operating-systems\windows\dependency-manager\chocolatey\general.md


---

## Dependencies


#### ffmpeg
docs\applications\tools\media\ffmpeg\general.md


---


settings.ini
```ini
Enabled=y
BlockNetworkFiles=y
BorderColor=#0423ee,ttl,6,192,in,6
Template=LingerPrograms
Template=qWave
Template=SkipHook
Template=GoogleJapaneseIME
Template=GoogleToolbarIE
Template=BlockAccessWMI
# Must be disabled or internet not working
# Template=BlockLocalConnect
Template=BlockTelemetry
Template=AdobeAcrobat
Template=AdobeDistiller
Template=AdobeAcrobatReader
Template=Chrome_KB5027231_fix
Template=HideInstalledPrograms
Template=NitroPDF5
Template=Pdf995
Template=NitroPDF6
ConfigLevel=10
UseSecurityMode=y
UseFileDeleteV2=y
UseRegDeleteV2=y
FakeAdminRights=y
ProtectHostImages=y
CoverBoxedWindows=y
ForceRestartAll=y
HideFirmwareInfo=y
RandomRegUID=y
HideDiskSerialNumber=y
HideNetworkAdapterMAC=y
EditAdminOnly=y
MonitorAdminOnly=y
UsePrivacyMode=y
NoAddProcessToJob=y

ReadFilePath=C:\Users\denni\.pyenv\pyenv-win\versions\3.9.13\
ReadFilePath=C:\ProgramData\chocolatey\bin\
ReadFilePath=C:\ProgramData\chocolatey\lib\ffmpeg\tools\ffmpeg\bin\
ForceProcess=python.exe

# Dieser Bereich ist dafür da, wenn man Dateien mit Python-Tools bearbeiten will.
NormalFilePath=python.exe,C:\shared\
```
- Generell gilt das auch für alle anderen Sandboxes: Wenn man mit Python irgendetwas bearbeiten will, sagen wir, man möchte mit einem Python-Skript Dateien konvertieren, dann müssen diese natürlich auch in der Box verfügbar sein.

Der einfachste Weg ist dementsprechend, einen Shared-Bereich zu erstellen.

Alternativ kann man natürlich auch die jeweiligen Bereiche über die normalen Properties freigeben, wie OpenFilePath oder ReadFilePath.


---

## Powershell Script starten mit Python
Starte Powershell in Box und dann:
```
$env:PATH = "C:\ProgramData\chocolatey\bin;" + $env:PATH
cd C:\Projects\utils\ai\voice\VoiceTyper
powershell -ExecutionPolicy Bypass -File "start.ps1"
```




