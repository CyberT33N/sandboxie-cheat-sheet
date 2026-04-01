
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

Bitte den folgenden Beitrag lesen, wenn man Versionen wechselt.

Man muss sicherstellen, dass die Dependencies, die man ausführt, natürlich auch immer installiert sind, und zwar in der jeweiligen Version, in der man sich befindet, entweder auf dem Hostsystem oder in der Box.

docs\applications\programming-languages\python\versioning.md