# VS Code Runtime And Catalog

## Scope

This document describes the canonical shared VS Code runtime and catalog surfaces used by the boxed-owned-toolchain method.

## Canonical shared VS Code surfaces

The following remain canonical under:

```text
C:\shared\sandbox-toolchains\ide\vscode\
```

### Runtime

```text
ide\vscode\runtime\1.121.0\
```

This is the canonical VS Code runtime source.

It is read-only from the point of view of normal project consumption.

### User catalog

```text
ide\vscode\catalog\vscode-user\
```

This contains the canonical:

- `settings.json`
- `keybindings.json`
- `snippets\`

These files are copied or mirrored into the box-local `user-data` runtime paths during bootstrap.

### Maintenance user-data

```text
ide\vscode\maintenance\user-data\
```

This is the maintenance control-plane user-data context used for shared IDE maintenance operations.

## Canonical settings role

The canonical `settings.json` contains:

- editor and terminal UX defaults
- Starship integration
- shell profile defaults

It does **not** contain project-specific absolute toolchain paths.

Those belong in bootstrap and project-adapter logic.

## Canonical settings content

```json
{
  "terminal.integrated.automationProfile.windows": {
    "path": "C:\\Windows\\System32\\cmd.exe"
  },
  "terminal.integrated.defaultProfile.windows": "Boxed PowerShell",
  "terminal.integrated.profiles.windows": {
    "Boxed PowerShell": {
      "path": "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
      "args": [
        "-NoExit",
        "-ExecutionPolicy",
        "Bypass",
        "-Command",
        "& 'C:\\Program Files\\starship\\bin\\starship.exe' init powershell --print-full-init | Out-String | Invoke-Expression"
      ],
      "env": {
        "PATH": "C:\\Program Files\\starship\\bin;${env:PATH}",
        "STARSHIP_CONFIG": "C:\\Users\\denni\\.config\\starship.toml"
      }
    },
    "Boxed CMD": {
      "path": "C:\\Windows\\System32\\cmd.exe"
    }
  },
  "terminal.integrated.inheritEnv": true
}
```

## Why not workspace settings for toolchain selection

Workspace settings are not the canonical place for:

- machine-local absolute binary paths
- shared runtime paths
- bootstrap-selected command surfaces

Those belong in the bootstrap layer.

## Why not box-specific shell copies

The final method does not rely on project-specific `cmd.exe` / `powershell.exe` copies as an architecture principle.

That was an older workaround pattern.

The final design prefers:

- normal Windows shell binaries
- canonical VS Code terminal settings
- explicit bootstrap-provided environment

## Host prompt exception

`Starship` can remain on the host because it is shell/prompt infrastructure, not project toolchain governance.

The box consumes:

- `C:\Program Files\starship\bin\starship.exe`
- `C:\Users\denni\.config\starship.toml`

## Related

- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\bootstrap\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\state\extensions-seeds-and-local-state.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\provisioning\shared-artifacts.md`
