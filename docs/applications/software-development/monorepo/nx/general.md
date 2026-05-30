# NX



#### Debugging
- docs\applications\software-development\monorepo\nx\debugging.md









<br><br>
<br><br>




## Troubleshooting

### Cache

#### Node Binary not allowed

Option 1 (Not recommended)

Using `OpenFilePath` against `%LOCALAPPDATA%\Temp\nx-native-file-cache-*` is too broad from a security perspective. It gives sandboxed `node.exe` direct host access to a volatile temp area, bypasses file virtualization for that path, and turns a general host temp location into a native code loading boundary. In practice, this means the exception is no longer limited to the intended workspace flow. Any sandboxed `node.exe` process in the same box can use that open host-backed path.

Option 2 (recommended)

Use a dedicated native cache directory and keep the exception as small as possible. The cache directory must be moved into an isolated host location that is explicitly allowed, the Nx daemon must be disabled, and only the shared cache directory should be opened for `node.exe`.

This can matter in **both** boxes:

- **Install box**: when `pnpm install` / `pnpm rebuild` touches Nx native bindings during dependency work
- **Run box**: when root-workspace Nx flows execute during daily development

### Install / rebuild

```powershell
$env:NX_NATIVE_FILE_CACHE_DIRECTORY='C:\shared\sandbox-toolchains\node-monorepo-general\cache\nx-native'
$env:NX_DAEMON='false'
pnpm install --store-dir "C:\shared\sandbox-toolchains\node-monorepo-general\cache\pnpm-store"
```

or:

```powershell
$env:NX_NATIVE_FILE_CACHE_DIRECTORY='C:\shared\sandbox-toolchains\node-monorepo-general\cache\nx-native'
$env:NX_DAEMON='false'
pnpm rebuild
```

### Run / serve

```powershell
$env:NX_NATIVE_FILE_CACHE_DIRECTORY='C:\shared\sandbox-toolchains\node-monorepo-general\cache\nx-native'
$env:NX_DAEMON='false'
pnpm run serve:test
```

### Sandboxie access rules

Use the same narrow cache exception in every box that loads Nx native code:

```ini
# Install box
OpenFilePath=node.exe,C:\shared\sandbox-toolchains\node-monorepo-general\cache\nx-native\

# Run box
OpenFilePath=node.exe,C:\shared\sandbox-toolchains\node-monorepo-general\cache\nx-native\
```

### VS Code / Cursor terminal env

If the install or run profiles are launched from VS Code / Cursor, keep the same env vars in those profiles too:

```json
{
  "env": {
    "NX_NATIVE_FILE_CACHE_DIRECTORY": "C:\\shared\\sandbox-toolchains\\node-monorepo-general\\cache\\nx-native",
    "NX_DAEMON": "false"
  }
}
```

Related:
- docs\applications\IDE\vscode\general.md













<br><br>


<br><br>

### Electron

Im Vergleich zu einem Nicht-Monorepo-Projekt, das mit pnpm verwaltet wird, liegt die Electron-Binary direkt im selben Projekt im .pnpm-Ordner.

Wenn wir uns in einem Nx-Monorepo befinden, liegt sie hingegen auf der Workspace-Ebene im node_modules-Ordner innerhalb von pnpm.
- docs\applications\programming-languages\node\dependencies\frameworks\electron\electron-vite\general.md

```ini
NormalFilePath=test.exe,C:\git\test-mono\node_modules\.pnpm\electron@29.4.6\node_modules\electron\dist\

NormalFilePath=electron.exe,C:\git\test-mono\node_modules\.pnpm\electron@29.4.6\node_modules\electron\dist\
NormalFilePath=electron.exe,C:\git\test-mono\
NormalFilePath=electron.exe,C:\git\test-mono\apps\test\
```




