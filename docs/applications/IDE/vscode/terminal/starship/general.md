
# INI Settings
docs\applications\terminal\starship\general.md


.vscode\settings.json
```json
{
  "terminal.integrated.defaultProfile.windows": "DevBox PowerShell",
  "terminal.integrated.profiles.windows": {
    "DevBox PowerShell": {
      "path": "C:\\Tools\\DevBoxShell\\powershell.exe",
      "args": [
        "-NoExit",
        "-ExecutionPolicy",
        "Bypass",
        "-Command",
        "Invoke-Expression (& 'C:\\Program Files\\starship\\bin\\starship.exe' init powershell)"
      ],
      "env": {
        "PATH": "C:\\Program Files\\starship\\bin;${env:PATH}",
        "STARSHIP_CONFIG": "C:\\Users\\denni\\.config\\starship.toml"
      }
    }
  }
}
```



















<br><br

---

<br><br>










### Warum Starship **nicht als Prompt** erscheint (obwohl `starship --version` geht)
Du hast aktuell nur den **Binary‑Zugriff/PATH** gelöst. Starship wird aber **nicht automatisch** zum Prompt, solange PowerShell nicht beim Start **Starship “init”** ausführt.

Das erwartete Prompt‑Layout bekommst du erst, wenn in der PowerShell‑Session einmal ausgeführt wurde:

```powershell
Invoke-Expression (& "C:\Program Files\starship\bin\starship.exe" init powershell)
```

---

### Option 1 (schnell, für die aktuelle PowerShell‑Session)
Im Sandbox‑PowerShell‑Fenster:

```powershell
Invoke-Expression (& "C:\Program Files\starship\bin\starship.exe" init powershell)
```

Danach sollte der Prompt sofort „Starship‑Style“ anzeigen.

---

### Option 2 (best practice): automatisch in **VS Code Terminal** aktivieren
Ergänze in `C:\git\test\test-synchronizer\.vscode\settings.json` dein Terminal‑Profil so, dass PowerShell Starship beim Start initialisiert.

```json
{
  "terminal.integrated.defaultProfile.windows": "DevBox PowerShell",
  "terminal.integrated.profiles.windows": {
    "DevBox PowerShell": {
      "path": "C:\\Tools\\DevBoxShell\\powershell.exe",
      "args": [
        "-NoExit",
        "-ExecutionPolicy",
        "Bypass",
        "-Command",
        "Invoke-Expression (& 'C:\\Program Files\\starship\\bin\\starship.exe' init powershell)"
      ],
      "env": {
        "PATH": "C:\\Program Files\\starship\\bin;${env:PATH}",
        "STARSHIP_CONFIG": "C:\\Users\\denni\\.config\\starship.toml"
      }
    }
  }
}
```

- `STARSHIP_CONFIG` ist optional, macht das Config‑Lookup aber eindeutig.
- `ExecutionPolicy Bypass` hilft dir auch bei `pnpm.ps1`‑Shims.

---

### Option 3: automatisch in einem **normalen (boxed) PowerShell-Fenster**
Du willst das gleiche Verhalten auch außerhalb VS Code? Dann starte PowerShell künftig mit einem Start‑Command:

```powershell
"C:\Tools\DevBoxShell\powershell.exe" -NoExit -ExecutionPolicy Bypass -Command "Invoke-Expression (& 'C:\Program Files\starship\bin\starship.exe' init powershell)"
```

Das kannst du z. B. als Shortcut speichern und immer darüber öffnen.










<br><br

---

<br><br>

# Troubleshooting

## Debug
- docs\troubleshooting\IDE\vscode\terminal\debug.md