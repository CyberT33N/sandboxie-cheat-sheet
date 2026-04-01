
# Python


## Troubleshooting
- docs\applications\programming-languages\python\troubleshooting.md

## Python Manager

#### pyenv
- docs\applications\programming-languages\python\python-manager\pyenv\general.md

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
ReadFilePath=C:\Users\denni\.pyenv\pyenv-win\versions\3.9.13\Scripts\
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

# Execute python cli tools

Wichtig: In der Box sollte man bei `pyenv-win` nicht über die Shims gehen, wenn das CLI zuverlässig starten soll.

Der Grund ist, dass `docling.bat` aus `shims` intern wieder `pyenv` benutzt. Dafür müssen nicht nur `PATH`, sondern auch die `pyenv`-Auflösung und die gesetzte Python-Version innerhalb der Box sauber funktionieren. Genau daran scheitert es hier.

Der robustere Weg ist deshalb: Das CLI direkt aus der echten Python-Version aufrufen und die Shims komplett umgehen.

Dafür muss in `settings.ini` mindestens dieser Pfad zusätzlich freigegeben werden:

`ReadFilePath=C:\Users\denni\.pyenv\pyenv-win\versions\3.9.13\Scripts\`

Starte PowerShell in der Box und füge am Anfang Folgendes ein:
```
$env:PATH = "C:\Users\denni\.pyenv\pyenv-win\versions\3.9.13;C:\Users\denni\.pyenv\pyenv-win\versions\3.9.13\Scripts;" + $env:PATH
Get-Command docling
docling --help
```

Wenn `docling` in `C:\Users\denni\.pyenv\pyenv-win\versions\3.9.13\Scripts\` installiert ist, wird es damit direkt gefunden, ohne dass `pyenv` oder die Shims benötigt werden.

Falls du es ganz explizit ohne PATH-Test starten willst, geht auch das:

```
C:\Users\denni\.pyenv\pyenv-win\versions\3.9.13\Scripts\docling.exe --help
```

Der entscheidende Einstieg am Anfang ist also nicht der Shim-Pfad, sondern der echte Versionspfad:

```powershell
$env:PATH = "C:\Users\denni\.pyenv\pyenv-win\versions\3.9.13;C:\Users\denni\.pyenv\pyenv-win\versions\3.9.13\Scripts;" + $env:PATH
```


---

## Powershell Script starten mit Python

Starte Powershell in Box und dann:
```
$env:PATH = "C:\ProgramData\chocolatey\bin;" + $env:PATH

cd C:\Projects\utils\ai\voice\VoiceTyper
powershell -ExecutionPolicy Bypass -File "start.ps1"
```


