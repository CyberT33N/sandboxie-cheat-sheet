# VS Code







# Method #1 -  Vscode auf Host, Toolchain in starker Box, Debug via Attach‑Inspect (Empfohlen)

- 📌 Empfohlene Variante (mit Sandboxie)
Die beste Variante ist, dass Cursor oder VSCode auf dem Host ganz normal läuft und die jeweiligen Unterprozesse, die man startet, in einer eigenen Box laufen. Dazu gehören z. B.:

- die Electron-Binary im PNPM-Ordner
- die Terminals, die man für PowerShell und CMD nutzt
- die Node-Executables

Damit ist die komplette Toolchain isoliert, aber weiterhin bleibt alles möglich. Man kann auch weiterhin über den Attached Inspect debuggen. Das ist aus meiner Sicht die beste Variante, die man mit Sandboxie erreichen kann.

Methode 2 (nicht vollständig isoliert)
Es gibt natürlich noch Methode 2, aber die ist nicht komplett isoliert, weil die Sicherheitsisolation wegfällt. Dadurch bringt es eigentlich nichts: Man hat dann im Wesentlichen nur virtualisierte Rechte und ein paar weitere Vorteile, aber keinen echten Mehrwert.

Fazit
Von der Gewichtung her ist es besser, wenn VSCode auf dem Host-System normal läuft und alles, was zur Toolchain gehört bzw. was man daraus aufruft, konsequent in einer eigenen Box isoliert läuft.

- docs\applications\IDE\vscode\terminal\debug\general.md
- docs\applications\IDE\vscode\terminal\package.json vs new terminal.md

## Limitierung — IDE-Debug-Script in `package.json` funktioniert nicht

Eine starke **Limitierung**, die **DU** hast, ist das klassische **Debug-Script**, das **DU** in deiner **IDE** kennst: Wenn **DU** zum Beispiel in einer `package.json` bist und dann über ein **Run-Script** hovert und anschließend auf **Debug-Script** klickst.

Vollständige **Informationen** können hierzu gefunden werden.

Das ist das einzige, was anscheinend nicht funktioniert.

- docs\applications\IDE\vscode\terminal\debug\package-json\debug-scripts-not-working.md









<br><br>
<br><br>







# Method #2 - Not Fully isolated


## Nott working
- Generelle Optionen -> Restriktion -> "Verhin dere die Beeinträchtigung der benutzeroberfläche"
  - Nicht aktivieren

- Sicherheitsoptionen -> Sicherheitsisolation -> "Deaktiviere Sicherheitsisolation"
 - Nicht aktivieren

Ich habe sehr vieles ausprobiert, aber es scheint generell mit der Sicherheitsisolation nicht zu funktionieren.













<br><br>

---

<br><br>


# Terminal

## Loging
- https://github.com/CyberT33N/sandboxie-cheat-sheet/tree/main/docs/troubleshooting/terminal

## ReadConsoleOutput “Access denied (0x5)” / broken STDOUT
- https://github.com/CyberT33N/sandboxie-cheat-sheet/blob/main/docs/troubleshooting/operating-systems/windows/terminal/powershell.md
