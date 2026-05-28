# Install-Box Config

## Single source of truth

This page documents only the `node-gyp`-specific install-box additions for the current validated host-sync architecture.

Do **not** copy this page as a full standalone install-box config.

The generic install-box baseline remains:

- `docs\applications\IDE\vscode\methods\host-not-isolated\templates\node-monorepo-materialized-dependencies.md`

This page documents only the extra rules that were required for:

- shared Python binary access
- host-provided Microsoft Visual Studio Build Tools access
- repo-local GYP / MSBuild output materialization

## Why a `node-gyp` overlay is needed

In the validated Windows flow, `node-gyp` does **not** run as a single process:

1. `node.exe` starts `node-gyp`
2. `node-gyp` starts `python.exe`
3. `python.exe` generates `build\config.gypi`, `binding.sln`, `.vcxproj`, and related files
4. `MSBuild.exe` starts the actual compiler / linker toolchain

Sandboxie file rules are process-specific, so it was not sufficient to open the repo only for `node.exe`.

## Base prerequisites from the generic monorepo baseline

The following generic requirements remain in force and are **not** redefined here:

- fixed versioned `node.exe` / `pnpm.cmd`
- `OpenFilePath=node.exe,C:\git\test\test-mono\`
- `OpenFilePath=node.exe,C:\shared\sandbox-toolchains\node-monorepo-general\cache\pnpm-store\`
- `OpenFilePath=node.exe,C:\shared\sandbox-toolchains\node-monorepo-general\cache\nx-native\`

## `node-gyp` install-box additions

Add the following lines to the generic install-box config:

```ini
# --- Shared dev toolchain root ---
# Central shared Python build helper binary.
NormalFilePath=node.exe,C:\shared\sandbox-toolchains\dev\
NormalFilePath=powershell.exe,C:\shared\sandbox-toolchains\dev\
NormalFilePath=cmd.exe,C:\shared\sandbox-toolchains\dev\
NormalFilePath=python.exe,C:\shared\sandbox-toolchains\dev\

# --- Python-generated repo-local gyp inputs / outputs ---
# python.exe is spawned by node-gyp and must read gyp_main.py, binding.gyp,
# addon.gypi, and write build\config.gypi / *.sln / *.vcxproj to the real repo.
OpenFilePath=python.exe,C:\git\test\test-mono\

# --- Native build outputs in the monorepo ---
# The spawned Microsoft toolchain processes must also materialize their output
# to the real host-visible repo path.
OpenFilePath=MSBuild.exe,C:\git\test\test-mono\
OpenFilePath=cl.exe,C:\git\test\test-mono\
OpenFilePath=link.exe,C:\git\test\test-mono\
OpenFilePath=lib.exe,C:\git\test\test-mono\
OpenFilePath=rc.exe,C:\git\test\test-mono\
OpenFilePath=mt.exe,C:\git\test\test-mono\
OpenFilePath=cvtres.exe,C:\git\test\test-mono\
OpenFilePath=mspdbsrv.exe,C:\git\test\test-mono\
OpenFilePath=tracker.exe,C:\git\test\test-mono\

# --- Visual Studio installer detection ---
NormalFilePath=node.exe,C:\Program Files (x86)\Microsoft Visual Studio\Installer\
NormalFilePath=powershell.exe,C:\Program Files (x86)\Microsoft Visual Studio\Installer\
NormalFilePath=cmd.exe,C:\Program Files (x86)\Microsoft Visual Studio\Installer\

# --- Visual Studio Build Tools / MSBuild / VC toolset ---
NormalFilePath=node.exe,C:\Program Files\Microsoft Visual Studio\
NormalFilePath=powershell.exe,C:\Program Files\Microsoft Visual Studio\
NormalFilePath=cmd.exe,C:\Program Files\Microsoft Visual Studio\

# --- Windows SDK ---
NormalFilePath=node.exe,C:\Program Files (x86)\Windows Kits\
NormalFilePath=powershell.exe,C:\Program Files (x86)\Windows Kits\
NormalFilePath=cmd.exe,C:\Program Files (x86)\Windows Kits\
```

## Why these rules are process-specific

### Shared dev root

The shared `dev` root is consumed by:

- `powershell.exe`
- `cmd.exe`
- `node.exe`
- `python.exe`

That is why the validated baseline exposes the shared `dev` root through `NormalFilePath` for all of those processes.

### `python.exe` repo access

`node.exe` can start `python.exe`, but its repo `OpenFilePath` does not automatically cover the spawned Python process.

Without the `python.exe` repo `OpenFilePath`, validation produced failures such as:

- `python.exe: can't open file ...\gyp_main.py`
- missing host-visible `build\binding.sln`
- missing host-visible `.vcxproj` output

### MSBuild / compiler / linker repo access

Even after `python.exe` was fixed, the spawned Microsoft build tools also needed repo write access so that the final native addon outputs were materialized to the real host-visible repo path.

Without those process-specific `OpenFilePath` lines, validation produced failures around missing repo-local solution / project files or final build outputs during `msbuild` execution.

## Sanitized example project root

All snippets in this overlay use the sanitized example repo root:

```text
C:\git\test\test-mono\
```

Replace that path with your real monorepo root when applying the config.

## Related documents

- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\general.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\visual-studio-build-tools.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\commands.md`
