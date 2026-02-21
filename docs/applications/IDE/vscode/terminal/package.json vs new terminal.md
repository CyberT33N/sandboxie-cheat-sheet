


# package.json vs new terminal


### Was passiert da?
**Ja.** „Run Script“ (Hover in `package.json`) läuft in VS Code als **Task/NPM‑Runner** und nutzt dafür ein **separates Task/Debug-Terminal‑Profil** (Automation Profile) – nicht zwingend dein normales Terminal-Profil. Das ist so dokumentiert bei VS Code Terminal Profiles (Task/Debug Profile) und es gibt genau in dieser Ecke bekannte Probleme, wenn das Standard-Terminal mit „Startup/Init‑Command“ arbeitet ([VS Code: Terminal Profiles](https://code.visualstudio.com/docs/terminal/profiles), [vscode #177550](https://github.com/microsoft/vscode/issues/177550)).

Dein `DevBox PowerShell` Profil hat beim Start einen `-Command ... Invoke-Expression ...` (Starship‑Init). Das ist für **interaktives Terminal** ok – aber Tasks können dadurch beim Start „aus dem Tritt“ kommen und du siehst dann genau so einen `Invoke-Expression` Binding‑Fehler.

---

### Fix (Best Practice): Interaktiv = PowerShell+Starship, Tasks/RunScript = clean `cmd.exe`
Das ist der stabilste Weg **ohne** deine Sandbox-Regeln zu ändern.

#### Schritt 1 — Starship‑Init im interaktiven Profil robuster machen
Ändere in deinem Profil den `-Command` auf die Pipeline‑Variante (robuster als Klammern):

```json
"-Command",
"& 'C:\\Program Files\\starship\\bin\\starship.exe' init powershell --print-full-init | Out-String | Invoke-Expression"
```

#### Schritt 2 — Automation Profile für „Run Script“ setzen (in die Sandbox, aber ohne Starship)
In **derselben** `.vscode/settings.json` ergänzen:

```json
"terminal.integrated.automationProfile.windows": {
  "path": "C:\\Tools\\DevBoxShell\\cmd.exe"
}
```

Full .vscode\settings.json:
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
        "& 'C:\\Program Files\\starship\\bin\\starship.exe' init powershell --print-full-init | Out-String | Invoke-Expression"
      ],
      "env": {
        "PATH": "C:\\Program Files\\starship\\bin;${env:PATH}",
        "STARSHIP_CONFIG": "C:\\Users\\denni\\.config\\starship.toml"
      }
    }
  },
  "terminal.integrated.automationProfile.windows": {
    "path": "C:\\Tools\\DevBoxShell\\cmd.exe"
  }
}
```


Warum `cmd.exe`?
- Tasks brauchen keine Prompt‑Init.
- `cmd` triggert auch nicht dein `pnpm.ps1` ExecutionPolicy-Thema, weil er typischerweise `pnpm.cmd` nimmt.
- Da deine `cmd.exe` im `C:\Tools\DevBoxShell\` liegt, wird sie durch Sandboxie bei dir auch sauber in die Box gezogen.

#### Schritt 3 — VS Code neu laden
`Ctrl+Shift+P` → **Developer: Reload Window**

#### Ergebnis
- **Terminal → New Terminal**: weiterhin **DevBox PowerShell** mit Starship.
- **package.json → Run Script**: öffnet ein **Task-Terminal (cmd)** und führt das Script ohne Starship‑Init‑Fehler aus.

Wenn du willst, dass „Run Script“ *unbedingt* PowerShell bleibt, sag kurz Bescheid – dann bauen wir ein separates „Task‑PowerShell ohne Starship“ Setup.


















<br><br>

---

<br><br>


# related
- docs\applications\IDE\vscode\terminal\debug.md
- docs\applications\terminal\starship\general.md

- docs\troubleshooting\IDE\vscode\terminal\starship\general.md
- docs\troubleshooting\IDE\vscode\terminal\debug.md
- docs\troubleshooting\terminal\logging.md