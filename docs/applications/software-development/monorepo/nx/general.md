# NX

## Troubleshooting

### Cache

#### Node Binary not allowed

Option 1 (Not recommended)

Using `OpenFilePath` against `%LOCALAPPDATA%\Temp\nx-native-file-cache-*` is too broad from a security perspective. It gives sandboxed `node.exe` direct host access to a volatile temp area, bypasses file virtualization for that path, and turns a general host temp location into a native code loading boundary. In practice, this means the exception is no longer limited to the intended workspace flow. Any sandboxed `node.exe` process in the same box can use that open host-backed path.

Option 2

Use a dedicated native cache directory and keep the exception as small as possible. The cache directory must be moved into an isolated host location that is explicitly allowed, the Nx daemon must be disabled, the run script must be started from the workspace root, and only the shared cache directory should be opened for `node.exe`.

```powershell
$env:NX_NATIVE_FILE_CACHE_DIRECTORY='C:\shared\nx-native-cache'
$env:NX_DAEMON='false'
pnpm run serve:test
```

```ini
OpenFilePath=node.exe,C:\shared\nx-native-cache\
```

Oder direkt in der .vscode\settings.json

```json
{
  "terminal.integrated.automationProfile.windows": {
    "path": "C:\\Tools\\DevBoxShell\\cmd.exe"
  },
  "terminal.integrated.defaultProfile.windows": "DevBox PowerShell",
  "terminal.integrated.profiles.windows": {
    "DevBox PowerShell": {
      "args": [
        "-NoExit",
        "-ExecutionPolicy",
        "Bypass",
        "-Command",
        "& 'C:\\Program Files\\starship\\bin\\starship.exe' init powershell --print-full-init | Out-String | Invoke-Expression"
      ],
      "env": {
        "PATH": "C:\\Program Files\\starship\\bin;${env:PATH}",
        "STARSHIP_CONFIG": "C:\\Users\\denni\\.config\\starship.toml",
        "NX_NATIVE_FILE_CACHE_DIRECTORY": "C:\\shared\\nx-native-cache",
        "NX_DAEMON": "false"
      },
      "path": "C:\\Tools\\DevBoxShell\\powershell.exe"
    }
  }
}
   
```

Related:
- docs\applications\IDE\vscode\general.md