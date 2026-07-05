# Debugging



## Electron

- **WICHTIG**: Der nachfolgende Teil kann optional sein.

In einem eigenständigen Electron JS-/Electron Vite-Projekt musste ich diese Stelle hier nicht hinzufügen. Als ich mich aber in einem Monorepo mit NX wiedergefunden habe, hatte ich auf einmal die folgende Problemstellung und keinen Zugriff mehr.

Wenn ich die App gebaut und die Electron-Applikation im Dev-Modus gestartet habe, habe ich im Inspektorfenster diesen Fehler bekommen. Das ließ sich dadurch lösen, dass die ElectronExe entsprechend Zugriff auf diesen Ort hat.

Das kann damit zusammenhängen, dass gegebenenfalls eine andere ElectronExe aus einem anderen Pfad verwendet wird, wenn man sich in einem Monorepo befindet.




**WICHTIG** ## Priorisierung und Ausgangslage — Cursor in Box und Host-Bereich

Von der **Priorisierung** her läuft **Cursor** in der **Box**. Die **Problemstellung** ist aktuell, dass man probiert zu debuggen und das über den **Host-Bereich** passiert, während man parallel auch in der **Box** ist. Daher ist der richtige Weg, dass **Cursor** in der **Sandbox** läuft. Im aktuellen Stand ist das aber nicht möglich.

## Bevorzugter Weg bei aktueller und zukünftiger Umsetzbarkeit

Falls es später mit einer neueren **Version** möglich sein **MUSS**, **MUSS** dieser Weg gegangen werden, dass **Cursor** in der **Sandbox** läuft. Sonst ist der präferierte Weg dieser hier: ➡️ Der explizite **Bereich** **MUSS** zum **Debuggen** freigegeben werden.
- https://github.com/sandboxie-plus/Sandboxie/issues/5235


Bis dahin ist die einzige Lösung, dass man die mindeste Zugriffsform erlaubt, in dem Fall readFilePath. Das ist die tiefste Ebene, die geht. Wenn man weiter auf Extension gehen würde, würde es nicht gehen. Das heißt, man muss den App-Ordner hier zulassen und dadurch hat man Zugriff drauf und kann debuggen.


```ini
ReadFilePath=electron.exe,C:\Users\denni\AppData\Local\Programs\cursor\resources\app\
```

```
Uncaught Error: Cannot find module 'c:/Users/denni/AppData/Local/Programs/cursor/resources/app/extensions/ms-vscode.js-debug/src/bootloader.js'
Require stack:
- internal/preload
    at Module._resolveFilename (node:internal/modules/cjs/loader:1055:15)
    at Module._load (node:internal/modules/cjs/loader:908:27)
    at c._load (node:electron/js2c/node_init:2:13672)
    at internalRequire (node:internal/modules/cjs/loader:173:19)
    at Module._preloadModules (node:internal/modules/cjs/loader:1436:5)
    at loadPreloadModules (node:internal/process/pre_execution:714:5)
    at setupUserModules (node:internal/process/pre_execution:173:3)
    at prepareExecution (node:internal/process/pre_execution:132:5)
    at prepareMainThreadExecution (node:internal/process/pre_execution:55:3)
    at node:internal/main/run_main_module:10:1
```
