## Referenzbericht — Cursor/VS Code + Sandboxie (Privacy Mode) + `Debug Script` vs. Attach‑Inspect (Electron‑Vite / pnpm)

### Ziel & Rahmenbedingungen
- **Ziel**: In einer **Sandboxie Privacy-Box** (stark isoliert) aus VS Code/Cursor heraus Electron‑Vite über pnpm entwickeln und debuggen.
- **Wichtige Randbedingung**: **Cursor/VS Code bleibt im Host-Modus (unboxed)**, nur das **Dev‑Terminal / die Dev‑Prozesse** laufen boxed.
- **Beobachtung**:  
  - `Run Script` (aus `package.json`) **läuft**  
  - `Debug Script` öffnet zwar ein **boxed Debug-Terminal**, aber **der Prozess startet nicht sauber / läuft nicht weiter**

---

## 1) Warum `Run Script` funktioniert, `Debug Script` aber nicht

### 1.1 `Run Script` = Task-Ausführung
- VS Code startet das Script als **Task** (Shell/Command Runner).
- Das ist im Sandbox‑Terminal zuverlässig, weil es “nur” den Runner (`pnpm`, `npm`, `node`) startet.

### 1.2 `Debug Script` = JavaScript Debug Terminal (js-debug) + Bootloader + Named Pipe
Beim Klick auf **`Debug Script`** macht VS Code (js-debug) etwas Fundamentales zusätzlich:

- Setzt `NODE_OPTIONS` und injiziert einen Preload:
  - Beispiel aus deinem Debug-Terminal:

```text
NODE_OPTIONS=
 --require c:/Users/denni/AppData/Local/Programs/cursor/resources/app/extensions/ms-vscode.js-debug/src/bootloader.js  --inspect-publish-uid=http
```

- Setzt `VSCODE_INSPECTOR_OPTIONS` mit einer **IPC-Pipe**:
  - Beispiel (gekürzt):

```text
{"inspectorIpc":"\\.\pipe\node-cdp.37208-....sock", ...}
```

**Kernaussage**: `Debug Script` hängt davon ab, dass der sandboxed Node-Prozess
1) den Bootloader laden kann und
2) zur **Named Pipe `\\.\pipe\node-cdp.*`** verbinden kann.

---

## 2) Erkenntnisse aus den Tests (Evidence-basiert)

### 2.1 Phase A — Bootloader-Dateipfad war im Privacy Mode “nicht existent” (ENOENT)
**Symptom** beim Ausführen von `pnpm` im Debug-Terminal:

- Node brach mit einem Fehler ab, bevor `pnpm` überhaupt arbeiten konnte:
  - `ENOENT ... lstat 'c:\Users\denni\AppData\Local\Programs\cursor'`

**Erkenntnis**
- In Privacy Mode wurde der Cursor-Installationspfad für boxed Prozesse so behandelt, als wäre er **nicht vorhanden**.

**Fix, der diesen Teil gelöst hat**
- `ReadFilePath` für den Cursor‑Root (für `node.exe`/`electron.exe`), damit der Bootloader-Pfad **auflösbar** ist.

**Validierung**
- Danach lief `pnpm -v` im Debug-Terminal stabil (`10.18.2`).

---

### 2.2 Phase B — Der echte Blocker: `node-cdp.*` Pipe ist im Box-Kontext nicht sichtbar/connectbar
Wir haben danach den Pipe-Mechanismus exakt geprüft (wie js-debug intern):

- `readdirOk=true` (Node kann `\\.\pipe\` auflisten)
- aber die **konkrete** Pipe fehlt:
  - `includesPipe=false`
  - `connectOk=false`, **`ENOENT`** beim Connect

Beispiel aus deinem Pipe-Check:

```text
pipe= \\.\pipe\node-cdp.37208-ecefd63c-9.sock
readdirOk= true
includesPipe= false
connectOk= false
error= ENOENT connect ENOENT \\.\pipe\node-cdp.37208-ecefd63c-9.sock
```

**Erkenntnis**
- Die js-debug Pipe, die vom Host-VS-Code bereitgestellt wird, ist für die Box **nicht im selben NT-Objektnamespace erreichbar**.
- Das ist kein “Pattern falsch”-Problem mehr, denn wir haben u.a. getestet:
  - `OpenPipePath=node.exe,\Device\NamedPipe\node-cdp*`
  - `OpenFilePath=node.exe,\Device\NamedPipe\node-cdp*`
  - `OpenIpcPath=node.exe,\Device\NamedPipe\node-cdp*`
  - sogar extrem breit: `OpenPipePath=node.exe,\Device\NamedPipe\`
  - außerdem als Entkopplungstests:
    - `UseSecurityMode=n`
    - `HideNonSystemProcesses=n`
    - `NtNamespaceIsolation=n`
    - `Template=BlockLocalConnect` deaktiviert

**Trotzdem** blieb die Pipe im Box‑Kontext “nicht existent” (ENOENT).

---

## 3) Warum es nicht funktioniert, wenn Cursor/VS Code im Host-Modus bleibt (nicht in der Sandbox)

**Kurzfassung**
- `Debug Script` braucht eine **bidirektionale IPC‑Verbindung** zwischen:
  - **Host** (Cursor/VS Code js-debug) = Pipe-Server `node-cdp.*`
  - **Box** (Node/pnpm/electron-vite) = Pipe-Client

**Warum das in deiner Isolation scheitert**
- Sandboxie isoliert/virtualisiert den **NT Object Namespace** (dazu gehören Named Pipes).
- Selbst wenn du “Freigaben” setzt, hilft das nur für Objekte, die im Box‑Kontext überhaupt **sichtbar** sind. Bei dir ist die konkrete `node-cdp.*` Pipe aber in der Box **nicht sichtbar** → Connect ergibt **ENOENT**.

**Ergebnis**
- Solange Cursor/VS Code **unboxed** bleibt und du gleichzeitig eine **Privacy-Box mit starker Namespace-Isolation** nutzt, ist der `Debug Script`-Mechanismus von VS Code **nicht zuverlässig herstellbar**.

---

## 4) Warum `Debug Script` nicht funktioniert (technischer Kern)
`Debug Script` scheitert bei dir nicht am Projekt oder pnpm, sondern an der js-debug Architektur:

- js-debug injiziert den Bootloader via `NODE_OPTIONS=--require ...\bootloader.js`
- der Bootloader erwartet eine erreichbare `inspectorIpc`:
  - `\\.\pipe\node-cdp....sock`
- in deiner Box ist diese Pipe **nicht erreichbar** → js-debug kann nicht attachen → Debug-Lauf startet nicht korrekt bzw. bricht sehr früh ab.

---

## 5) Schlussfolgerung (für deine gewählten Constraints)
**Unter der Vorgabe** “Cursor/VS Code bleibt Host (unboxed) + nur Dev-Terminal boxed + Privacy Mode/Isolation bleibt aktiv” ist die **einzige verlässlich funktionierende** Debugging‑Methode:

- **Attach‑Inspect** (Node Inspector + Chrome DevTools Protocol Attach)

Das ist genau der Weg, der **keine** cross-boundary `node-cdp.*` Named Pipe benötigt.


---


## 6) Stabiler Weg (empfohlen): dein bestehendes “1‑Click”-Setup verwenden
- docs\applications\IDE\vscode\terminal\package.json vs new terminal.md
- docs\applications\IDE\vscode\terminal\debug\general.md

Electron Beispiel:
- docs\applications\programming-languages\node\frameworks\electron\electron-vite\debug.md


---


## 7) Executive Summary (1 Absatz)
`Run Script` funktioniert, weil es nur als Task ausgeführt wird. `Debug Script` funktioniert in deinem Setup nicht, weil VS Code/js-debug für Debug-Terminals einen Bootloader via `NODE_OPTIONS` injiziert und dann eine Named-Pipe (`\\.\pipe\node-cdp.*`) zur Debugger‑Instanz im Host benötigt. In deiner Privacy‑Box bleibt diese Pipe im Box‑Kontext unsichtbar (belegt durch `includesPipe=false` und `connect ENOENT` trotz sehr breiter Pipe-Freigaben und deaktivierter Security-Features). Daher ist unter deiner Host/Box‑Trennung die **einzige verlässliche Debugging‑Variante**: **Attach‑Inspect** (Inspector-Port + Chrome Attach).