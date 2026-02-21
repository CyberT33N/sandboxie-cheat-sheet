
# Sandbox Settings

## May not working
- Netzwerk Optionen -> Other Options -> "Block DNS, UDP Port 53"
  - Das hängt von der Electron-Applikation ab, die du verwendest. Es kann sein, dass der Zugriff hier blockiert wird.

- Erweiterte Optionen -> Prozhesse -> "Hindere sandgeboxte Prozesse daran, über WMI aus Systemdetails zuzugreifen"
  - Das hängt von der jeweiligen Elektron-Applikation ab und davon, was sie genau macht. Wenn bestimmte Einstellungen relevant sind (z. B. weil etwas angezeigt werden muss), kann es sein, dass hier unter Umständen Dinge nicht funktionieren.

## Not working
- Generelle Optionen -> Restriktionen -> "Verhindere die Beinträchtigung der Benutzeroberfläche"
  - Do not deactivate

- Sicherheit Options → Job Objekt
   - Deaktiviere "„Füge sandbox-/geboxte Prozesse zu Job‑Objekten hinzu“"
     - https://github.com/CyberT33N/sandboxie-cheat-sheet/blob/main/docs/troubleshooting/electron.md





<br><br>

---

<br><br>


# VS Code




<br><br>

---

<br><br>


# Troubleshooting

## Terminal

### Loging
- https://github.com/CyberT33N/sandboxie-cheat-sheet/tree/main/docs/troubleshooting/terminal

### ReadConsoleOutput “Access denied (0x5)” / broken STDOUT
- docs\troubleshooting\operating-systems\windows\terminal\powershell.md