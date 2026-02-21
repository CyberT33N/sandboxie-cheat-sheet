# Run Terminal is Sandbox
- Man nimmt diese beiden EXEs und zieht sie in einen dedizierten Bereich (wie im Beispiel unten mit Tools, Deathbox, Shell). Anschließend stellt man sicher, dass sie automatisch in der jeweiligen Sandbox gestartet werden, in der man z. B. Applikationen debuggen möchte.

Wenn man z. B. in einem Electron-Projekt arbeitet und die Applikationen darin im „Deathmodus“ debuggen möchte, ist der richtige Weg, die entsprechenden Terminals, die man im Projekt in der Settings-JSON konfiguriert, zusätzlich in der jeweiligen Sandbox einzutragen, damit sie dort mitstarten und in derselben Sandbox laufen.



- C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
- C:\Windows\System32\cmd.exe

.vscode\settings.json
```
{
  "terminal.integrated.automationProfile.windows": {
    "path": "C:\\Tools\\DevBoxShell\\cmd.exe"
  },
  "terminal.integrated.defaultProfile.windows": "DevBox PowerShell",
  "terminal.integrated.profiles.windows": {
    "DevBox PowerShell": {
      "args": [],
      "path": "C:\\Tools\\DevBoxShell\\powershell.exe"
    }
  }
}

```