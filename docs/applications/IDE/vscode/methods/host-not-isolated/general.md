


# Method #1 – VS Code on Host, Toolchain in Dedicated Box, Debug via Attach-Inspect (Recommended)

* 📌 Recommended approach (with Sandboxie)
  The best setup is to run Cursor or VS Code normally on the host while executing all relevant subprocesses inside their own dedicated box. This includes, for example:

* the Electron binary inside the PNPM directory

* terminals used for PowerShell and CMD

* Node executables

This isolates the entire toolchain while keeping full functionality. Debugging via Attached Inspect continues to work. From a structural and security perspective, this is the strongest setup achievable with Sandboxie.



----

## Dependcy installing Host or inside Box
Wenn man npm- oder pnpm-Befehle zum Installieren von Dependencies verwendet, und das wird wahrscheinlich auch für andere Programmiersprachen mit Dependencies ähnlich sein, dann werden diese auf dem Host-System installiert, wenn man das Boxed Terminal nicht in der Sandbox startet.

Wenn man dann basierend auf den ganzen Guides hier die Applikation, also Electron.js und Sonstiges, startet, funktioniert es, weil über Funktionalitäten wie mit Normal Path eine virtuelle Kopie erstellt wird.

Auf der anderen Seite hätte man aber auch Option 2: Man führt die npm- oder pnpm-Install-Befehle direkt in der Box aus. Das kann jedoch, je nach Framework, zu folgenschweren Konsequenzen führen, zum Beispiel bei Electron.js, Vite und anderen Themen.

Im aktuellen Stand dieses Dokuments habe ich das bisher noch nicht vollständig durchprobiert, aber es gab diesbezüglich schon sehr viele Problemstellen, weil dann irgendwie die Node-Exe vom Host-System verwendet wird.

Man muss also Sorge dafür tragen, dass die eigenständige Node-Exe verwendet wird. Das heißt:
- Entweder kopiert man die Node-Binary in die Box.
- Oder man benutzt direkt aus dem node_modules-Ordner die Node-Binary-Exe.

Wenn man an der package.json arbeitet, ist das dort ja vorhanden und wird mit installiert, wenn man es entsprechend setzt. Darüber kann man dann die jeweiligen Szenarien starten.

Trotzdem gab es weiterhin Probleme, zum Beispiel bei electron-vite.