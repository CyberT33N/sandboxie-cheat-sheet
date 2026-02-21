
# Debug


# Run Terminal is Sandbox
- Man nimmt diese beiden EXEs und zieht sie in einen dedizierten Bereich (wie im Beispiel unten mit Tools, Deathbox, Shell). Anschließend stellt man sicher, dass sie automatisch in der jeweiligen Sandbox gestartet werden, in der man z. B. Applikationen debuggen möchte.

Wenn man z. B. in einem Electron-Projekt arbeitet und die Applikationen darin im „Deathmodus“ debuggen möchte, ist der richtige Weg, die entsprechenden Terminals, die man im Projekt in der Settings-JSON konfiguriert, zusätzlich in der jeweiligen Sandbox einzutragen, damit sie dort mitstarten und in derselben Sandbox laufen.



- C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
- C:\Windows\System32\cmd.exe




# INI Settings
```
# --- DevBoxShell binaries ---
NormalFilePath=powershell.exe,C:\Tools\DevBoxShell\
NormalFilePath=cmd.exe,C:\Tools\DevBoxShell\
```













<br><br>

---

<br><br>



# Related

## Enable starship
- docs\troubleshooting\IDE\vscode\terminal\starship\general.md