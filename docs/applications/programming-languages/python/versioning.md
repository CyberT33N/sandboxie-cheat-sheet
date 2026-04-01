
# Versioning

- Es ist sehr wichtig zu verstehen, dass man mit verschiedenen Python-Versionen arbeitet. Das heißt, man kann unterschiedliche Versionen installieren.

Wenn ich jetzt zum Beispiel eine Default-Version gesetzt habe, etwa 3.9.1.3, und ich dann hier ein `pip install` für eine Dependency mache, dann wird sie nur für diese Version installiert.

Wenn ich jetzt diesem Guide hier folge und es entsprechend in der Sandbox starte, dann ist es verfügbar, weil es entweder vorher auf dem Host-System installiert worden ist, als es noch nicht sandboxed war, oder weil es in der Sandbox installiert ist.

Wenn ich jetzt aber zu einer anderen Version wechsle und dort eine Dependency noch nicht installiert ist, dann ist sie entsprechend noch nicht vorhanden. Das heißt, ich muss sicherstellen, dass sie natürlich auch in der anderen Version installiert ist.


```
─denni@MININT-A8821UQ ⟶  ~
╰─> pyenv versions
  3.12.9
* 3.9.13 (set by C:\Users\denni\.pyenv\pyenv-win\version)
```

---


## Install

### Install dependency for specific version
- docs\applications\programming-languages\python\dependencies.md