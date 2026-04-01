


# pyenv
- docs\applications\programming-languages\python\frameworks\pyenv\general.md

#chocolatey
- docs\applications\operating-systems\windows\dependency-manager\chocolatey\general.md

# ffmpeg


## Powershell Script starten mit Python
Starte Powershell in Box und dann:
```
$env:PATH = "C:\ProgramData\chocolatey\bin;" + $env:PATH
cd C:\Projects\utils\ai\voice\VoiceTyper
powershell -ExecutionPolicy Bypass -File "start.ps1"
```







# Troubleshooting

## Internet not working
```
# Must be disabled or internet not working
# Template=BlockLocalConnect
```