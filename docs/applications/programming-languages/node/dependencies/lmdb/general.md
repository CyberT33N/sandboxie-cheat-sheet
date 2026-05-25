# INI Settings
```ini
ReadFilePath=node.exe,C:\git\test\test-mono\.pnpm\@lmdb+lmdb-win32-x64@3.3.0\node_modules\@lmdb\lmdb-win32-x64\
```





<br><br>

---


<br><br>

# Troubleshooting

Dein Host-Pfad ist lesbar, aber zusätzlich existiert unter `C:\Sandbox\yourusername\testbox\drive\C\...` eine **virtualisierte Kopie** derselben `.node`-Datei. Diese boxed Image-Datei wird dann mit `0xC0000022` geblockt. Das passt exakt zu `ProtectHostImages=y`.

Die architekturell richtige minimale Lösung ist deshalb:

1. **Die boxed LMDB-Kopie löschen**  
   Nicht nur allgemein „irgendwie Box leeren“, sondern gezielt den boxed Pfad zu  
   `C:\Sandbox\yourusername\testbox\drive\C\git\test\test-mono\.pnpm\@lmdb+lmdb-win32-x64@3.3.0\...`

