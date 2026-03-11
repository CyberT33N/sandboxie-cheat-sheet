# Debug


Settings INI:

- Man muss auf jeden Fall sicherstellen, dass die gebauten Artefakte auf dem Hostsystem verfügbar sind, weil der Attach-Main-Prozess, der unten beschrieben wird, die gebauten Artefakte im `out/main`-Ordner erwartet. Das ist so gesehen der Default-Ordner, der von Electron Vite bereitgestellt wird. Wenn die Applikation mit Electron Vite ausgeführt wird, wird sie gebaut und die Artefakte werden dort abgelegt.

Das heißt: Wenn wir in einem Boxed-Terminal Electron Vite ausführen, wie unten im Guide beschrieben, würden die Artefakte dementsprechend in der Sandbox landen. Das kann nicht funktionieren, weil Cursor oder VS Code nicht vollständig in der Sandbox enthalten sind.

Wenn also VS Code oder Cursor auf dem Hostsystem läuft und dann die `launch.json` gestartet wird, setzt das voraus, dass die Artefakte ebenfalls auf dem Hostsystem verfügbar sind. Um das sicherzustellen, müssen wir dafür Sorge tragen, dass sie dort verfügbar sind.

Die einfachste Möglichkeit ist, es mit Open File Path freizugeben. Wenn wir im Boxed-Terminal den Electron-Vite-Befehl zum Bauen ausführen, wären die Artefakte dementsprechend auch auf dem Hostsystem, und dadurch kann VS Code oder Cursor sie dann lesen.

Natürlich wäre der präferierte Weg, dass Cursor oder VS Code ebenfalls in der Box laufen. Das ist aber im aktuellen Stand nicht möglich. Related:
- docs\applications\IDE\vscode\general.md
```ini
OpenFilePath=node.exe,C:\git\test\test-mono\apps\privyou\out\
OpenFilePath=electron.exe,C:\git\test\test-mono\apps\privyou\out\
```


**WICHTIG**
- Wenn man in einem existierenden Projekt zu einer Sandbox wechselt, ist die oben beschriebene Problemstellung sehr wichtig zu verstehen.

Wenn auf dem Hostsystem noch gebaute Artefakte liegen und man mit Cursor oder VS Code noch auf dem Hostsystem arbeitet, also nicht in der Sandbox, und man dann dieses Debugging-Verfahren aus dem Dokument hier abarbeitet, kann man gegebenenfalls in eine Altlast laufen.

Das heißt:
- Man hätte die gebauten Artefakte noch auf dem Hostsystem.
- Das Debugging über die Map-Dateien würde auch funktionieren.
- Man würde denken, dass alles funktioniert, auch wenn man oben diesen Open-File-Path nicht gesetzt hat, weil man noch mit alten Relikten arbeitet.

Sobald man den Out-Ordner aber löschen würde, würde man merken, dass es nicht mehr geht. Dann würde man in der Debug-Konsole von VS Code beziehungsweise Cursor sehen, dass die Chunk-Dateien nicht vorhanden sind, weil Cursor oder VS Code keinen Zugriff darauf hat.

Zusätzlich muss man der Electron Exe auch noch per Open-File-Path Zugriff auf den Out-Ordner auf dem Host-System geben.

Der Grund ist, dass die Electron Exe in der Sandbox ausgeführt wird. Daher hat sie keinen Zugriff auf die gebauten Artefakte auf dem Host-System, und dadurch können wir nicht mehr debuggen.

Das heißt, wir müssen der Electron Exe entsprechend auch erlauben, auf diesen Ort zuzugreifen.





<br><br>
<br><br>


1. Create .vscode\settings.json
- docs\applications\IDE\vscode\terminal\debug\general.md

2. .vscode\launch.json
```json
{
  "compounds": [
    {
      "configurations": ["Debug Main Process", "Debug Renderer Process"],
      "name": "Debug All",
      "presentation": {
        "order": 1
      }
    }
  ],
  "configurations": [
    {
      "args": [
        "run",
        "--inspect-brk",
        "--no-file-parallelism",
        "test\\unit\\services\\evident\\EvidentDatabaseIsolation.test.ts"
      ],
      "autoAttachChildProcesses": true,
      "console": "integratedTerminal",
      "name": "Debug Vitest",
      "program": "${workspaceFolder}/node_modules/vitest/vitest.mjs",
      "request": "launch",
      "skipFiles": ["<node_internals>/**"],
      "smartStep": true,
      "type": "node"
    },
    {
      "address": "127.0.0.1",
      "name": "Attach Main/Backend (9229)",
      "port": 9229,
      "request": "attach",
      "skipFiles": ["<node_internals>/**"],
      "timeout": 60000,
      "type": "node"
    },
    {
      "address": "127.0.0.1",
      "name": "Attach Main Process (5858)",
      "port": 5858,
      "request": "attach",
      "skipFiles": ["<node_internals>/**"],
      "timeout": 60000,
      "type": "node"
    },
    {
      "cwd": "${workspaceRoot}",
      "env": {
        "REMOTE_DEBUGGING_PORT": "9222"
      },
      "name": "Debug Main Process",
      "request": "launch",
      "runtimeArgs": ["--sourcemap"],
      "runtimeExecutable": "${workspaceRoot}/node_modules/.bin/electron-vite",
      "type": "node",
      "windows": {
        "runtimeExecutable": "${workspaceRoot}/node_modules/.bin/electron-vite.cmd"
      }
    },
    {
      "name": "Debug Renderer Process",
      "port": 9222,
      "presentation": {
        "hidden": true
      },
      "request": "attach",
      "timeout": 60000,
      "type": "chrome",
      "webRoot": "${workspaceFolder}/src/renderer"
    }
  ],
  "version": "0.2.0"
}
```

### Jedes Mal (Debug-Session starten)
1) Öffne dein **boxed Terminal** (das, wo Logs funktionieren – z. B. dein „DevBox PowerShell“).
  - Wenn man die settings.json gemäß der Anleitung oben gesetzt hat, ist das die Default-Shell.

Unabhängig davon kann man beim Öffnen eines neuen Terminals über das Pluszeichen (+) die verschiedenen Terminals auswählen. In dem Fall müsste man dann das Custom-Terminal nehmen, das in der Sandbox läuft.

2) Setze den Renderer‑Port und starte Electron‑Vite **mit festem Main‑Debug‑Port**:

```powershell
$env:REMOTE_DEBUGGING_PORT="9222"
pnpm exec electron-vite --inspect-brk=5858 --sourcemap
```
- Bitte immer ein neues Fenster öffnen. Wenn man einmal in Deluxe reindebuggt, scheint es dort weiterzulaufen; dann kann es manchmal hängen und sich anschließend nicht wieder öffnen.

3) Warte, bis du das siehst:
- `Debugger listening on ws://127.0.0.1:5858/...`
- `dev server running ...`
- `start electron app...`

4) Jetzt in VS Code/Cursor: **Run & Debug** öffnen → **Attach Main Process (5858)“** auswählen → **F5**.

5) Weil du `--inspect-brk` nutzt, ist der Main‑Process am Anfang angehalten → einmal **Continue (F5)** drücken.  
   Danach laufen Breakpoints wie gewohnt (wenn deine Stelle erreicht wird).










<br><br>
<br><br>


## Troubleshooting
- In Bezug auf dieses Dokument können gegebenenfalls Probleme auftreten, wenn die Electron-Applikation startet und im Inspektor Fehler angezeigt werden, dass eine bestimmte Extension für das Debugging nicht gefunden werden kann. Wenn das der Fall ist, bitte hier schauen.
  - docs\applications\IDE\cursor\debugging.md