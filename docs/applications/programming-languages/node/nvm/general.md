# INI Settings

- `C:\nvm4w\` was only the visible shim and symlink layer.
- The actual Node and pnpm runtime files were resolved from the real NVM home under `C:\Users\denni\AppData\Local\nvm\...`.
- With `UsePrivacyMode=y`, the user-space path below `AppData\Local` was still hidden, so the shell could see the entry point in `C:\nvm4w\` but could not reliably resolve the real versioned target behind it.
- Adding `NormalFilePath` for `C:\Users\denni\AppData\Local\nvm\` fixed the architectural gap: the shell and Node can now read both the shim layer and the actual version storage, while everything still remains normally sandboxed instead of being opened directly on the host.

```ini
# --- Node (nvm4w) ---
NormalFilePath=node.exe,C:\nvm4w\
NormalFilePath=powershell.exe,C:\nvm4w\
NormalFilePath=cmd.exe,C:\nvm4w\

NormalFilePath=node.exe,C:\Users\denni\AppData\Local\nvm\
NormalFilePath=powershell.exe,C:\Users\denni\AppData\Local\nvm\
NormalFilePath=cmd.exe,C:\Users\denni\AppData\Local\nvm\
```