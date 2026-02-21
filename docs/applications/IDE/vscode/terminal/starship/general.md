
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

# Troubleshooting
- docs\troubleshooting\IDE\vscode\terminal\starship\general.md

## Debug
- docs\troubleshooting\IDE\vscode\terminal\debug.md