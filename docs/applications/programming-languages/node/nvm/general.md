# nvm4w

## Architectural status

`C:\nvm4w\` is only the visible shim / symlink layer.
The actual Node and pnpm runtime files are resolved from the real NVM home under:

```text
C:\Users\yourusername\AppData\Local\nvm\
```

In privacy boxes, this distinction matters.

## Required Sandboxie visibility

With `UsePrivacyMode=y`, the user-space path below `AppData\Local` would otherwise stay hidden.
The shell could then see the entry point in `C:\nvm4w\` but could not reliably resolve the real versioned target behind it.

That is why both layers must be visible:

```ini
# --- Node (nvm4w shim + real NVM home) ---
NormalFilePath=node.exe,C:\nvm4w\
NormalFilePath=powershell.exe,C:\nvm4w\
NormalFilePath=cmd.exe,C:\nvm4w\
NormalFilePath=nvm.exe,C:\nvm4w\

NormalFilePath=node.exe,C:\Users\yourusername\AppData\Local\nvm\
NormalFilePath=powershell.exe,C:\Users\yourusername\AppData\Local\nvm\
NormalFilePath=cmd.exe,C:\Users\yourusername\AppData\Local\nvm\
NormalFilePath=nvm.exe,C:\Users\yourusername\AppData\Local\nvm\
```

## Important rule for the current boxed architecture

Do **not** run `nvm use` inside the install box or the run box.

Reason:

- `nvm use` rewires the boxed shim/symlink state
- under Sandboxie that state can become virtualized in a way that later confuses `pnpm`, `node`, and child-process resolution

The validated workflow uses the **fixed versioned binaries directly**, for example:

```powershell
& "C:\Users\yourusername\AppData\Local\nvm\v26.2.0\node.exe" -v
& "C:\Users\yourusername\AppData\Local\nvm\v26.2.0\pnpm.cmd" -v
```

## Why this is not a contradiction

The repository may still declare multiple runtime layers, for example:

- package-manager / tooling shell on Node 26
- delivered desktop runtime on Node 20

That does **not** mean the boxed shell must switch runtimes dynamically with `nvm use`.
The documented architecture keeps the shell on the approved tooling line and handles the runtime boundary explicitly at the package/framework layer.

## Related documents

- `docs\applications\programming-languages\node\package-manager\pnpm\general.md`
- `docs\applications\IDE\vscode\methods\host-not-isolated\templates\node-monorepo-materialized-dependencies.md`