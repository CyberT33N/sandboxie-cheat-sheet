

## Install

### Install dependency for specific version

```shell
C:\Users\denni\.pyenv\pyenv-win\versions\3.12.9\python.exe -m pip install docling
```
- Hier muss man jetzt natürlich differenzieren:

- Wenn die Python-Exe noch nicht in der Sandbox läuft, würde man sie entsprechend auf dem Host-System installieren.
- Wenn man Python dann in der Sandbox startet, würde es virtualisiert werden und die Sachen würden rübergezogen werden. Dadurch funktioniert es dann.
- Wenn Python aber in der Sandbox schon installiert ist und man dann diesen Befehl oben ausführt, wird er entsprechend natürlich in der Sandbox ausgeführt.

Das heißt in Bezug auf Versioning: Wenn man jetzt natürlich eine globale Version installiert hätte, sagen wir 3.9, und man würde es dann einfach ohne den direkten absoluten Binary-Pfad aufrufen, würde man es über den aufgelösten Pfad dementsprechend in der aktuell gesetzten globalen Version installieren, vor allem wenn man Package Manager wie ypynv benutzt.
- docs\applications\programming-languages\python\python-manager\pyenv\general.md
- docs\applications\programming-languages\python\versioning.md