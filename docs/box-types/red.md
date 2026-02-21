
# Programme starten nicht

## Was ich in der Doku/Logs dazu gefunden habe (SBIE2314 vs. SBIE1231)
- docs\troubleshooting\error-codes\SBIE1231.md

## Warum deine Electron‑App im Privacy‑Modus wahrscheinlich nicht startet
Privacy Mode arbeitet so, dass **User‑Space standardmäßig verborgen/blockiert** ist (default block) und nur definierte Bereiche lesbar bleiben ([Privacy Mode](https://sandboxie-plus.github.io/sandboxie-docs/PlusContent/privacy-mode/)).  
Wenn deine App z. B. unter **`C:\privYou\privYou.exe`** liegt, ist das **User‑Space** (nicht `C:\Windows`/`C:\Program Files*`). Dann sieht die Box diese Datei ggf. **nicht mehr** → Prozess‑Initialisierung kann scheitern → `SBIE1231` → `SBIE2314`.

## Minimal‑Allow Fix (so wenig freigeben wie möglich)
### Schritt 1 — App‑Ordner explizit “Normal” erlauben (nicht “Open”)
In deiner Privacy‑Box:

1) **Sandbox Options → Resource Access → Files**
2) **Add File/Folder**: den Ordner, in dem deine EXE liegt (z. B. `C:\privYou\`)
3) **Access = Normal** (entspricht `NormalFilePath=...`)  
   Doku: [NormalFilePath](https://sandboxie-plus.github.io/sandboxie-docs/Content/NormalFilePath/)

**Warum “Normal” und nicht “Open”?**  
- **Normal** = Standard‑Sandboxing (lesen ok, schreiben wird in die Box virtualisiert).  
- **OpenFilePath** wäre ein echtes “Loch” (direkter Host‑Zugriff) und ist dafür nicht nötig ([OpenFilePath](https://sandboxie-plus.github.io/sandboxie-docs/Content/OpenFilePath/)).

> Hintergrund: Privacy Mode versteckt Host‑Inhalte typischerweise über “write-only/hidden”-Mechanik ([WriteFilePath](https://sandboxie-plus.github.io/sandboxie-docs/Content/WriteFilePath/)). Für dein EXE‑Verzeichnis brauchst du eine gezielte Ausnahme.

4) **Alle Prozesse in der Box beenden** → App neu starten.

Wenn die App danach startet: du hast den minimal notwendigen Allow‑Pfad schon gefunden.

---

## Wenn es danach noch nicht startet: Ja, du MUSST “debuggen” – aber über Sandboxie‑Logs
Nicht “blind freigeben”, sondern **evidence‑basiert** minimal öffnen.

### Schritt 2 — Trace Log aktivieren und genau die Denies finden
1) In SandMan: **View → Trace Logging** aktivieren ([Trace logging](https://sandboxie-plus.github.io/sandboxie-docs/PlusContent/TraceLog/))
2) Privacy‑Box starten → App starten (sie darf ruhig direkt sterben)
3) Im Trace Log nach **Denied** Einträgen suchen (CTRL+F, nach `D`, `privYou.exe`, oder nach Pfaden)
4) **Right‑click → Copy Panel** (oder Speichern) ([Trace logging](https://sandboxie-plus.github.io/sandboxie-docs/PlusContent/TraceLog/))

### Schritt 3 — Deny → minimale Regel ableiten
- Wenn es ein **Dateipfad** ist → in **Resource Access → Files** als **Normal** (oder notfalls Read Only/Closed) setzen  
  (siehe: [WriteFilePath](https://sandboxie-plus.github.io/sandboxie-docs/Content/WriteFilePath/), [ClosedFilePath](https://sandboxie-plus.github.io/sandboxie-docs/Content/ClosedFilePath/))
- Wenn es ein **IPC/NamedPipe** ist → entsprechende IPC/Pipe‑Ausnahme (Doku-Workflow erklärt das Trace‑Prinzip) ([Sandboxie Trace](https://sandboxie-plus.github.io/sandboxie-docs/Content/SandboxieTrace/))
- Nach jeder Änderung: Prozesse in der Box beenden → erneut testen.

---

### Entscheidungshilfe: “Ganz C blocken außer Windows?”  
In **Privacy Mode** ist das Konzept bereits genau so gedacht (User‑Space default block, Systempfade lesbar) ([Privacy Mode](https://sandboxie-plus.github.io/sandboxie-docs/PlusContent/privacy-mode/)).  
Du solltest nur **gezielt** deine App‑Location (und danach nur das, was der Trace wirklich zeigt) als **Normal** öffnen – nicht mehr.

---

## Sandboxie (Red/Privacy Box) – Resource Access rule fields

### What “Program: All Programs” means
- **Yes**: `All Programs` means the rule applies to **every process running inside this sandbox**.
- **How to think about it**: It’s the **scope selector** for the rule.
  - `All Programs` = global rule for the whole box
  - `some.exe` = rule applies **only** when that specific process is the one accessing the path

### Fields you see in the rule list
- **Type (Typ)**: Which resource category the rule controls.
  - Common examples: `File/Folder`, `Registry`, `IPC/Named Pipe` (depends on the sub-tab)
- **Program (Programm)**: Which sandboxed process(es) the rule applies to.
  - `All Programs` = every boxed process
  - `privYou.exe` (example) = only that executable
- **Access (Zugriff)**: What Sandboxie should do when a boxed process accesses the resource.
  - **Normal**: Default sandbox behavior (read is allowed; writes are virtualized into the sandbox).
  - **Read Only**: Host path is readable but **not writable** (`ReadFilePath` behavior).
  - **Box Only (Write Only)**: Host contents are hidden; the app can create new files in the sandbox as if the folder were empty (`WriteFilePath` behavior).
  - **Closed**: Access is fully denied (read/write blocked) (`ClosedFilePath` behavior).
  - **Open**: Direct access to the host path with **no sandboxing** for that location (a deliberate “hole”; `OpenFilePath` behavior).
- **Path (Pfad)**: The path pattern the rule applies to (often with wildcards like `C:\privYou\*`).

### Practical guidance for Red/Privacy boxes (least privilege)
- Prefer **Program-specific** rules (`privYou.exe`) over `All Programs` when possible.
- Prefer **Normal / Read Only / Box Only** over **Open**.
- Use **Open** only when you explicitly accept a host-access “hole” for compatibility.