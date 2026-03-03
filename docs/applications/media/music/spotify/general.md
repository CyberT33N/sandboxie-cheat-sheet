

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
ConfigLevel=10
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
HideNonSystemProcesses=n
BlockScreenCapture=y
EditAdminOnly=y
MonitorAdminOnly=y
HideNonSystemProcesses=y
UsePrivacyMode=y
ForceProcess=Spotify.exe
NormalFilePath=C:\Users\denni\AppData\Roaming\Spotify\
```




<br><br>

---

<br><br>




# Troubleshooting

## SBIE1305 (blocked loading a sandboxed image)
- Kann ausgelöst werden, wenn auf dem Hostsystem ein Update stattfindet, zum Beispiel.

```text
Spotify.exe: SBIE1305 Blocked loading a sandboxed image - \user\current\AppData\Roaming\Spotify\chrome_elf.dll
```

**Fix:** Delete the **entire** contents of the Spotify box and then start Spotify again. After the box is cleared, Spotify will recreate what it needs and the app starts normally.

If you need to keep any files from this sandbox, back them up before deleting the box contents and copy them back afterwards.