# Sandbox Settings



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
Template=HideInstalledPrograms
Template=Firefox_Force
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
ForceProcess=firefox.exe
NormalFilePath=firefox.exe,C:\shared\
```



<br><br>

--- 

<br><br>


# Troubleshooting


## Updates

```text
firefox.exe: SBIE1305 Blocked loading a sandboxed image - \drive\C\Program Files\Mozilla Firefox\mozglue.dll
```

**Meaning:** Firefox attempted to load a *boxed* (sandboxed) copy of `mozglue.dll` from inside the sandbox. When `ProtectHostImages=y` is enabled, Sandboxie blocks host-installed programs from loading DLLs from the sandbox.


This issue is commonly triggered by running **Firefox updates inside the sandbox**. Updates (or repair operations) can create/modify boxed copies under:

- `C:\Sandbox\denni\Browser_t33n_Firefox\drive\C\Program Files\Mozilla Firefox\`

After that, Firefox may try to load those boxed binaries and hit `SBIE1305`.

### Verified — Option 1: Delete boxed Mozilla Firefox folder (quick fix)

1) Delete the boxed Mozilla Firefox folder inside the sandbox:

- `C:\Sandbox\denni\Browser_t33n_Firefox\drive\C\Program Files\Mozilla Firefox\`

2) Start Firefox in the sandbox again.

This reliably fixes the `SBIE1305` startup block.

**Why this is not the right long-term approach:** once you start touching updates while Firefox is running under Sandboxie, you can run into update-related warnings like:

```text
firefox.exe: SBIE2191 Firefox should not be updated when it is run with Sandboxie.
firefox.exe: SBIE2192 Do not update the program under the control of Sandboxie.
```

### Recommended — Option 2 (architecture-correct)

Delete the **entire** box content and then start Firefox again.

⚠️ If you rely on persistent data in this sandbox, you MUST back it up first and then copy it back to the correct sandbox paths after recreating the box.



















<br><br>
<br><br>




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
NormalFilePath=firefox.exe,C:\shared\
```

## Datein kopieren aus Box
- Du kannst im Hostsystem in den jeweiligen Sandbox-Bereich wechseln und dort navigieren. In den meisten Fällen findest du das unter „User Current“.
  - C:\Sandbox\denni\Browser_Firefox\user\current\Downloads