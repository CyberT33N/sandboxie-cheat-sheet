# INI Settings (Single repo):
- docs\applications\programming-languages\node\dependencies\frameworks\electron\electron-vite\templates\single-repo.md

# INI Settings (Mono repo):
- docs\applications\programming-languages\node\dependencies\frameworks\electron\electron-vite\templates\single-repo.md
  - Es ist auf jeden Fall aufgefallen, im Direktvergleich zwischen Single-Repo und Monorepo, dass andere Dinge freigegeben werden mussten.

Zum Beispiel musste für Cursor der Read-File-Path in AppData für die Electron-EXE gesetzt werden. Außerdem mussten andere Pfade für PNPM für die Electron-EXE gesetzt werden, da im Monorepo alles anders gehandhabt wird.

In dem Beispiel, das ich hier verwendet habe, wurde allerdings auch mit PNPM Workspaces gearbeitet. Das heißt, je nach Differenzierung, also je nachdem, wie man arbeitet, sind die Pfade auch immer unterschiedlich.

Hier sollte man sich dementsprechend am sauberen Debugging orientieren, mit der Rückverfolgung über Sandboxy, damit man weiß, wo die jeweiligen EXE-Dateien liegen und alles freischalten kann, falls sich in einer anderen Umgebung etwas ändern sollte, wenn man anders vorgeht.
  - docs\tracing\applications