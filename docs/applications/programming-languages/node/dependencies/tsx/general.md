# INI Settings
```ini
# =========================
# tsx - test-forms-backend (esbuild)
# =========================
NormalFilePath=esbuild.exe,C:\git\test\test-project\
```





<br><br>

---


<br><br>

# Troubleshooting

## tsx watch
- Wir haben alles Mögliche probiert, um TSX Watch zum Laufen zu bekommen, inklusive einer normalen Dummy-Datei mit ganz einfachem Code.

Es gibt offenbar ein Problem mit dem Watch-Modus, das intern mit der Library in Kombination mit Sandboxie zusammenhängt. Wir hatten dafür eine komplette Sandbox-Box, in der alles deaktiviert war: eine Yellow Box, Sicherheitsisolation aus, etc. Trotzdem hat nichts funktioniert.

Unterm Strich: Aus welchen Gründen auch immer funktioniert der Watch-Modus nicht in einer Sandbox.
