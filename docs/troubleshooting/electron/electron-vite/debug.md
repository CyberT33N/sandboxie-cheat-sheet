# Debug

1. Create .vscode\settings.json
- docs\troubleshooting\IDE\vscode\terminal\debug.md

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