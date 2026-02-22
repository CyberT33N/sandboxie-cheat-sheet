### Wie Sandboxie‑Debugging hier “wirklich” funktioniert (Kurzmodell)

Du findest die Ursache fast nie über **“SBIE‑Nachrichten”** allein, weil dort nur **bestimmte** Block-Events als Message landen (z. B. Start/Run‑Restriction → `SBIE1308`).  
Für “**Privacy/Security Mode blockiert TSX**” ist das richtige Vorgehen: **Reproduzieren → Trace Log auf “verweigert” filtern → genau diese Deny‑Einträge in passende INI‑Regeln übersetzen**.

### Welche Ausgabe du mir geben musst (minimal, zielgenau)

Ich brauche nur:

- **Deny‑Liste (X / verweigert)** aus dem Tab **`Rückverfolgungsprotokoll`** für den **PID von `node.exe`**, der `tsx watch app.ts` ausführt.
- **Welche Prozesse laufen**, während es “hängt” (mindestens: `node.exe`, evtl. `esbuild.exe`, evtl. weitere `node.exe` Kinder) – ein Screenshot der Prozessliste oder einfach die PIDs/Namen reicht.

Wenn du mir genau das gibst, kann ich dir danach **die 1–3 minimalen `NormalFilePath`/`OpenPipePath`/`OpenIpcPath` Regeln** nennen, die wirklich fehlen.

### Schritt‑für‑Schritt in der UI: so erzeugst du einen kleinen “nur verbotenes” Trace

1. **Noise runterfahren (sonst explodiert die Loggröße)**  
   In `Sandbox Optionen → Erweiterte Optionen → Rückverfolgung`:
   - **Anlassen**: `Datei‑Rückverfolgung`, `Pipe‑Rückverfolgung`, `IPC‑Rückverfolgung`
   - **Ausschalten** (für diesen Debug‑Run): `Systemaufruf…`, `API‑Aufruf…`, `Debug‑Ausgabe…`, `alle SetError…`, `Hooks…` (die machen die 10–20 MB)
   - **Wichtig**: `Ressourcenzugriffsmonitor deaktivieren` **nicht** aktivieren (sonst siehst du im Trace‑Tab nichts)

2. **Trace‑Tab öffnen und leeren**  
   In SandMan: `Ansicht → Rückverfolgungsprotokoll / Trace Logging` (du hast den Tab ja schon).  
   Dann **Clear/Leeren** (Eraser/Trash‑Icon oder Kontextmenü), damit nur die neue Repro drin ist.

3. **Repro so klein wie möglich**  
   - **Nur Backend starten**, nicht das ganze FE/Browser‑Polling, damit kein Edge‑Noise reinkommt.
   - In der Sandbox‑Konsole im Backend‑Ordner: `npm run dev` (oder genau der Befehl, der hängen bleibt).
   - **Sobald** du siehst “hängt / keine weitere Ausgabe”: direkt zum Trace‑Tab.

4. **Im `Rückverfolgungsprotokoll` filtern (damit du nur “verboten” siehst)**  
   Oben in der Filterleiste:
   - **PID**: den `node.exe (PID)` auswählen, der zu TSX gehört  
     (wenn du noch `msedge.exe` ausgewählt hast, siehst du natürlich kein Node‑Problem)
   - **Status**: auf **verweigert** stellen (in deinen Logs ist das typischerweise `X`, z. B. `File X`, `Ipc X`)
   - Optional: **Typ** auf `File`/`Pipe`/`Ipc` begrenzen, falls es noch zu viel ist

5. **Kopieren statt Speichern**  
   Im Trace‑Tab **nur die gefilterte Ansicht** kopieren:
   - Entweder **Zeilen markieren → Copy**, oder Rechtsklick → **`Copy Panel`** (danach direkt hier einfügen).  
   Wenn es trotzdem zu groß wird: nur die **obersten ~50 Deny‑Zeilen** schicken.

### Wie du Deny‑Zeilen in “was muss ich erlauben?” übersetzt

- **`File X … C:\…`**: Datei/Ordnerzugriff blockiert → in Privacy‑Boxen fast immer **`NormalFilePath=<exe>,<pfad>\`** (nicht `OpenFilePath`, wenn du Least‑Privilege willst)
- **`Pipe X … \Device\NamedPipe\…`**: Named Pipe blockiert → **`OpenPipePath=<exe>,\Device\NamedPipe\…`**
- **`Ipc X …`**: IPC‑Objekt blockiert → **`OpenIpcPath=…`**
- **SBIE‑Message `SBIE1308`** (falls sie doch auftaucht): Start/Run‑Restriction aktiv → dann ist es **kein File‑Problem**, sondern Start/Run‑Policy
