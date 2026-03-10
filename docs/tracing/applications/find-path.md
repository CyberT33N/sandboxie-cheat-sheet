
# Find Path
- Man sollte sich generell mit dem Dokument hier unten vertraut machen, damit man weiß, wie das Debugging mit der Rückverfolgung grundsätzlich funktioniert.

Primär ist es aber so: Wenn die Rückverfolgung aktiviert ist, schließen sich die ganzen EXEs nicht. Das heißt, wenn keine Rückverfolgung an ist und man eine Applikation startet, sagen wir, eine Node.js-Applikation, in der eine Electron-EXE gestartet wird, dann wäre es so:

- Wenn es Probleme mit der Electron-EXE gibt, weil diese bei den Permissions nicht zugelassen worden ist, wäre sie nach ein, zwei Sekunden in der Sandbox-Oberfläche einfach wieder weg, weil sie nicht starten kann.

Wenn die Rückverfolgung aber an ist, bleibt sie weiterhin aktiv und beendet sich nicht. Dann kann man zum Beispiel mit der rechten Maustaste auf die EXE klicken und dann auf „Spalte kopieren“ gehen. Dadurch hat man dann den jeweiligen vollständigen Pfad.

Das Hauptproblem ist sehr oft, dass einfach nur „Electron.exe“ dasteht. Man weiß dann aber nicht, woher es kommt beziehungsweise aus welchem Pfad.

  - docs\tracing\applications\general.md