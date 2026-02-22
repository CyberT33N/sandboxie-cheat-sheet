
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
    ```
    # Weißer Screen

    ### Genau diese Sandboxie‑GUI‑Einstellung

    - Öffne **deine Sandbox** (z. B. `DefaultBox`) → **Sandbox Options** → **Sicherheit Options** → Job Objekt
      - Deaktiviere "„Füge sandbox-/geboxte Prozesse zu Job‑Objekten hinzu“" 

    ### Wofür ist die Option „Füge sandbox-/geboxte Prozesse zu Job‑Objekten hinzu“?

    Sie hängt **alle Prozesse in dieser Sandbox** an ein **Windows Job Object** (Kernel‑Objekt zur Prozess‑Gruppierung). Damit kann Sandboxie Prozesse **als Einheit** kontrollieren, z. B.:

    - **Lifecycle/Containment**: Kindprozesse sauber „mitnehmen“, gruppiert verwalten, beim Beenden der Box zuverlässiger terminieren.
    - **Enforcement**: Job‑basierte Limits/Restriktionen (je nach Box/Modus) überhaupt anwenden können.
    - **Operability**: Besseres „Cleanup“ (weniger Hänger durch übrig gebliebene Prozesse/Handles).

    (Windows‑Hintergrund: Job Objects verwalten Prozessgruppen und Limits zentral, siehe Microsoft Learn „Job objects“.)

    ### Was passiert, wenn du sie deaktivierst?

    - Sandboxie **packt die Prozesse nicht** in sein Job‑Objekt.
    - Chromium/Electron kann dann seine **eigenen Job‑Objekte** (Renderer/GPU/Utility‑Prozesse) ohne Konflikt nutzen – genau das behebt typischerweise „weißes Fenster“ (bekanntes Sandboxie‑Workaround‑Muster, z. B. in Issue `#1954` mit `AllowBoxedJobs=n`).

    ### Risiken / „offene Bereiche“, wenn du sie *nicht* einschaltest

    - **Weniger zuverlässiges Prozess‑Cleanup**: Es ist wahrscheinlicher, dass **Child-/Helper‑Prozesse** nach dem Schließen weiterlaufen (auch mit Netzwerkaktivität) oder die Box‑Löschung blockieren.
    - **Weniger Job‑basierte Durchsetzung**: Alles, was Sandboxie über Job‑Limits/Job‑Policies steuert, ist für diese Box **reduziert/weg**.
    - **Containment‑Edge‑Cases**: Die Steuerung „alles gehört garantiert zur gleichen Prozess‑Gruppe“ ist schwächer; das ist primär ein **Operability- und Policy‑Gap**, nicht automatisch ein „voller Sandbox‑Escape“, aber im Enterprise‑Risk‑Modell ein klarer Control‑Verlust.

    **Empfohlene Mitigations (Least‑Privilege, pro Box):**
    - Nutze eine **dedizierte Sandbox nur für diese Electron‑App** (kein globaler Ausnahme‑Drift).
    - Stelle sicher, dass beim Beenden der Box **alle Prozesse terminiert** werden, und prüfe danach, ob wirklich nichts mehr läuft.
    - Halte **Drop Admin Rights** aktiv und setze **Start/Run‑Restriktionen** (nur die benötigten EXEs) + ggf. **WFP‑Firewall‑Regeln** pro Sandbox.
    ```








<br><br>


# INI Settings
```
# --- your app ---
NormalFilePath=test.exe,C:\test\
```













<br><br>

---

<br><br>


# Terminal

## Loging
- https://github.com/CyberT33N/sandboxie-cheat-sheet/tree/main/docs/troubleshooting/terminal

## ReadConsoleOutput “Access denied (0x5)” / broken STDOUT
- https://github.com/CyberT33N/sandboxie-cheat-sheet/blob/main/docs/troubleshooting/operating-systems/windows/terminal/powershell.md
