# Sandbox Settings

## Not working
- Generelle Optionen -> Restriktionen -> "Verhindere die Beinträchtigung der Benutzeroberfläche"
  - Do not deactivate


# INI Settings
```ini
Enabled=y
BlockNetworkFiles=y
BorderColor=#0423ee,ttl,6,192,in
Template=LingerPrograms
Template=BlockPorts
Template=qWave
Template=SkipHook
Template=GoogleJapaneseIME
Template=GoogleToolbarIE
Template=BlockAccessWMI
Template=BlockLocalConnect
Template=BlockTelemetry
Template=AdobeAcrobat
Template=AdobeDistiller
Template=AdobeAcrobatReader
Template=Chrome_KB5027231_fix
Template=Dragon_Force
Template=Brave_Force
Template=Chrome_Force
Template=IExplore_Force
Template=Maxthon6_Force
Template=LibreWolf_Force
Template=Edge_Force
Template=Opera_Force
Template=Osiris_Force
Template=PaleMoon_Force
Template=SeaMonkey_Force
Template=Slimjet_Force
Template=Iron_Force
Template=Ungoogled_Force
Template=Vivaldi_Force
Template=Waterfox_Force
Template=Yandex_Force
Template=HideInstalledPrograms
ConfigLevel=10
UsePrivacyMode=y
UseSecurityMode=y
UseFileDeleteV2=y
UseRegDeleteV2=y
FakeAdminRights=y
ProtectHostImages=y
CoverBoxedWindows=y
ClosePrintSpooler=y
BlockInterferePower=y
ForceProcess=chrome.exe
ForceRestartAll=y
DisableResourceMonitor=y
HideFirmwareInfo=y
RandomRegUID=y
HideDiskSerialNumber=y
HideNetworkAdapterMAC=y
HideNonSystemProcesses=y
BlockScreenCapture=y
EditAdminOnly=y
MonitorAdminOnly=y
NormalFilePath=chrome.exe,C:\shared\
```



<br><br>

--- 

<br><br>


# Troubleshooting

## Upload Files
- ## Problemstellung — Öffnen der Modal-Box beim Datei-Upload

Ich habe noch nicht herausgefunden, was **DU** machen **MUSST**, damit sich die **Modal-Box** öffnet, wenn **DU** irgendwo eine **Datei** hochladen WILLST.

## Aktueller Stand — Drag & Drop funktioniert primär

Was primär funktioniert, ist **Drag & Drop**.

## Anforderung — Erlaubten Bereich für Drag & Drop sicherstellen
⚠️ **DU MUSST** sicherstellen, dass **DU** einen **erlaubten Bereich** für **Drag & Drop** hast.

## Beispielablauf — Shared-Bereich erstellen und Datei per Drag & Drop einfügen
📌 Zum Beispiel: Einfach auf `C` einen **Shared-Bereich** erstellen. Hier werden die **Dateien** reingeschoben. Danach wird die **Datei** aus dem **Explorer** in die hierbei liegende **Box** (Browser) in den **Drag-&-Drop-Bereich** reingeschoben.

```
NormalFilePath=chrome.exe,C:\shared\
```

## Datein kopieren aus Box
- Du kannst im Hostsystem in den jeweiligen Sandbox-Bereich wechseln und dort navigieren. In den meisten Fällen findest du das unter „User Current“.
  - C:\Sandbox\denni\Browser_Firma_Chrome\user\current\Downloads