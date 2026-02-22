





# VS Code

## Debug
- docs\troubleshooting\IDE\vscode\terminal\debug.md

- **WICHTIG** - Man kann natürlich einfach allgemein „electron.exe“ nehmen und einer Sandbox zuweisen; dann startet es auch in der Sandbox. Aus Entwicklersicht macht das aber wenig Sinn, weil man

- nicht debuggen kann und
- in den meisten Fällen keinen Zugriff auf die Logs hat, weil es in einer neuen Instanz läuft.

Fürs reine Starten funktioniert es also, ist aber nicht wirklich zukunftsorientiert.

Zukunftsorientiert ist es, wenn man alles sauber einrichtet: Das Projekt hat eine korrekte Terminal-Instanz, die der jeweiligen Box zugewiesen ist, und man startet dann diese Terminal-Instanz. Damit hat man auch Logging.

Wenn man debuggen will, muss man dem Guide folgen, den Inspector-Port im geboxten Terminalfenster öffnen und kann dann innerhalb von VS Code darauf zugreifen.

Unterm Strich gibt es zwei Optionen:

1) Wie oben beschrieben: Terminal-Instanz korrekt der Box zuweisen und darüber starten (inkl. Logging; Debugging via Inspector-Port).
2) Das komplette VS Code-Fenster in einer Sandbox starten. Dann muss man diese isolierte Thematik (einzelne Bereiche freischalten) nicht machen, weil der ganze Prozess innerhalb der Sandbox läuft.



- Wenn man diesem Debugging Guide folgt, muss man auch darauf achten, dass die jeweiligen EXEs, die man rüberkopiert und in der Sandbox startet, anschließend wieder im normalen Modus erlaubt werden.


docs\applications\IDE\vscode\terminal\debug.md

















<br><br>

---

<br><br>


# Troubleshooting

## Box Types

### Red

#### Allow electron applications
- Wie in der normalen Anleitung für Electron-Applikationen (aus Nicht-Entwickler-Sicht) gilt auch im Red-Box-Typ, wo alles isoliert ist: Man muss die Electron-Applikation entsprechend freigeben.

Das heißt konkret: Wenn man z. B. mit Electron oder Electronite arbeitet, liegt innerhalb des MPM (bzw. wenn man mit PMPM arbeitet) die Distribution-Binary, also die Electron-Exe, in dem unten genannten Pfad. Diese muss man dann genauso explizit erlauben, wie man auch eine normale Applikation im Red-Box-Modus erlauben würde.

- docs\applications\electron\general.md

- **WICHTIG** - Man kann natürlich grundsätzlich alle Electron-Executables einer Sandbox zuweisen. Das Problem dabei ist: Wenn man wirklich spezifisch arbeiten will und mehrere Electron-Projekte hat – oder im produktiven Umfeld ggf. Dateien ebenfalls „Electron.exe“ heißen – bekommt man schnell eine Problemstellung.

Deshalb sollte man schauen, dass man gezielt nur die jeweiligen Bereiche/Verzeichnisse erlaubt, in denen man tatsächlich arbeitet.

Zusätzlich muss man jedes Mal eigene Terminal-Fenster starten, die innerhalb der Sandbox laufen, um dann das jeweilige Run-Script auszuführen.


settings.ini:
```
NormalFilePath=test.exe,C:\git\test\test-project\.pnpm\electron@29.4.6\node_modules\electron\dist\
```






<br><br>


## Localhost
- docs\troubleshooting\applications\development\localhost.md


<br><br>









## Terminal

### Loging
- https://github.com/CyberT33N/sandboxie-cheat-sheet/tree/main/docs/troubleshooting/terminal

### ReadConsoleOutput “Access denied (0x5)” / broken STDOUT
- docs\troubleshooting\operating-systems\windows\terminal\powershell.md