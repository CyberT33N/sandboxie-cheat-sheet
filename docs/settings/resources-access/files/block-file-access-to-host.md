# Free
- ### Was bedeutet die 5‑Minuten‑Meldung?
Die Meldung heißt: **Deine Box ist so konfiguriert, dass sie eine supporter‑exklusive Funktion nutzt**, deshalb werden Prozesse aus dieser Box nach ~5 Minuten beendet, solange **kein Supporter‑Zertifikat** aktiv ist.

In deinem Fall ist der Trigger offenbar **`UseRuleSpecificity`** (bzw. ein Box‑Preset/Feature, das intern darauf basiert).

- **Supporter‑Features (Beispiele)**: *Privacy enhanced / Data Protection*, *Security enhanced*, *App Compartment Boxes* – diese sind laut Tabelle **nicht in Free** enthalten ([Feature Comparison](https://sandboxie-plus.com/feature-comparison/), [Supporter Certificate](https://sandboxie-plus.com/supporter-certificate)).

---

### Muss ich `UseRuleSpecificity` setzen, um Ressourcen/File‑Access zu konfigurieren?
- **Nein**: Du kannst auch im **Gratis‑Modus** File‑Access Regeln setzen (z. B. `ClosedFilePath`, `WriteFilePath`) und damit Host‑Pfade blockieren/verbergen ([ClosedFilePath](https://sandboxie-plus.github.io/sandboxie-docs/Content/ClosedFilePath/), [WriteFilePath](https://sandboxie-plus.github.io/sandboxie-docs/Content/WriteFilePath/)).
- **Ja, für dein ursprüngliches Ziel “alles blocken, aber Systempfade erlauben”**: Dafür brauchst du ein **Default‑Block‑Modell mit Ausnahmen**, und das funktioniert sauber erst mit **Rule Specificity + Normal‑Overrides** (Konzept: [Rule Specificity](https://sandboxie-plus.github.io/sandboxie-docs/PlusContent/RuleSpecificity/), Setting: [UseRuleSpecificity](https://sandboxie-plus.github.io/sandboxie-docs/Content/UseRuleSpecificity/), `NormalFilePath`: [NormalFilePath](https://sandboxie-plus.github.io/sandboxie-docs/Content/NormalFilePath/)).  
  Genau dieses Konstrukt scheint bei dir **supporter‑gated** zu sein → daher die 5‑Minuten‑Meldung.

---

### Heißt das: Ohne Premium kann ich „Ressourcen“ gar nicht setzen?
**Nein.** Du kannst im Free‑Modus weiterhin:
- **hart verbieten**: `ClosedFilePath=…` ([ClosedFilePath](https://sandboxie-plus.github.io/sandboxie-docs/Content/ClosedFilePath/))
- **Host‑Inhalte “unsichtbar” machen, aber in der Box neu anlegen lassen**: `WriteFilePath=…` ([WriteFilePath](https://sandboxie-plus.github.io/sandboxie-docs/Content/WriteFilePath/))

Was du ohne Premium **nicht stabil** bekommst, ist das Komfort‑/Preset‑Niveau von **Privacy Mode (Data Protection)**, also “User Space default block” ([Privacy Mode](https://sandboxie-plus.github.io/sandboxie-docs/PlusContent/privacy-mode/)).

---

### Gratis‑Modus: „Host‑Dateien blocken“ ohne `UseRuleSpecificity` (pragmatisch)
Wenn du **deine persönlichen Host‑Dateien** schützen willst, mach es im Free‑Modus so:

- **Ziel**: Blocke/verberge **User‑Space** (z. B. `C:\Users\<du>\…`) statt “ganz `C:\`”.
- **Warum**: `C:\` komplett zu blocken bricht Windows‑/Programmlibraries.

Minimaler Ansatz (Box‑Sektion), Beispiel:

```ini
[testApp]
; kein UseRuleSpecificity setzen (sonst 5-Minuten-Termination ohne Zertifikat)

; Host-Userprofil verstecken, aber in der Box nutzbar machen:
WriteFilePath=C:\Users\denni\

; zusätzliche Datenlaufwerke verbergen (falls vorhanden):
WriteFilePath=D:\
WriteFilePath=E:\
```

Optional „härter“ statt „verbergen“:
- `ClosedFilePath=C:\Users\denni\Documents\` etc. (führt eher zu “Access denied” statt “leer”) ([ClosedFilePath](https://sandboxie-plus.github.io/sandboxie-docs/Content/ClosedFilePath/)).
