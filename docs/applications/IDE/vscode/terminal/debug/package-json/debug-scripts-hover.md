## Zusammenfassung: Warum `Run Script` läuft, aber `Debug Script` nicht (Sandboxie/Privacy Mode)

### Was VS Code dabei *wirklich* macht
- **`Run Script`**
  - Wird als **Task** ausgeführt.
  - Startet deinen Script-Runner (bei dir effektiv `pnpm`) ganz normal im integrierten Terminal (bei dir über `terminal.integrated.automationProfile.windows` → `C:\Tools\DevBoxShell\cmd.exe`).

- **`Debug Script`**
  - Startet **kein** normales Task-Run, sondern ein **JavaScript Debug Terminal** (VS Code JavaScript-Debugger).
  - Dabei werden u.a. folgende Dinge gemacht:
    - `NODE_OPTIONS` wird gesetzt (inkl. `--require <…>\bootloader.js`)
    - `VSCODE_INSPECTOR_OPTIONS` wird gesetzt
    - Kommunikation läuft über einen **Named Pipe** wie `\\.\pipe\node-cdp.*`
  - Das Verhalten ist **nicht** über deine `launch.json`-`compounds` steuerbar – der `Debug Script`-Button benutzt diese Debug-Terminal-Route.

### Warum es in deiner Sandbox typischerweise “nur Terminal öffnet”
Im **Security/Privacy Mode** blockt Sandboxie oft mindestens eins davon:
- **Dateizugriff** auf den js-debug **Bootloader** (`…\bootloader.js`)
- **IPC-Zugriff** auf den **Named Pipe** (`node-cdp.*`)

Wenn das blockiert ist, startet der Prozess kurz und **beendet sich sofort** → du landest wieder im Prompt, als wäre “nichts passiert”.

### Stabiler Weg (empfohlen): dein bestehendes “1‑Click”-Setup verwenden
- docs\applications\IDE\vscode\terminal\package.json vs new terminal.md
- docs\applications\IDE\vscode\terminal\debug\general.md

Electron Beispiel:
- docs\applications\programming-languages\node\frameworks\electron\electron-vite\debug.md











