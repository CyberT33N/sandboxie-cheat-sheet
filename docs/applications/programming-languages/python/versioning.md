
# Versioning

## Architectural rule

Dependencies are version-specific.

That means every Python version needs its own package installation boundary.

If the runtime version changes, the required packages must be installed again for that version.

## Recommended model

In the recommended shared-toolchain architecture, versioning is explicit.

Example:

```text
C:\shared\sandbox-toolchains\python-general\python\3.12.9\python.exe
```

The runtime version is part of the path itself, which makes the active execution boundary easier to reason about.

## Why this matters

Installing a dependency into one Python version does not make it available to a different Python version.

Examples:

- a package installed for `3.9.13` is not automatically available for `3.12.9`
- a package installed for `3.12.9` must be reinstalled for a future `3.13.x` runtime if that runtime becomes the selected execution version

## Recommended install pattern for a specific version

```powershell
$toolRoot = "C:\shared\sandbox-toolchains\python-general"
$pythonExe = "$toolRoot\python\3.12.9\python.exe"
$env:PYTHONUSERBASE = "$toolRoot\userbase"
$env:PIP_CACHE_DIR = "$toolRoot\cache\pip"
$env:HF_HOME = "$toolRoot\cache\huggingface"
$env:TORCH_HOME = "$toolRoot\cache\torch"
$env:PATH = "$toolRoot\userbase\Python312\Scripts;" + $env:PATH

& $pythonExe -m pip install --user <package-name>
```

## Version-switch rule

When the runtime version changes, repeat the dependency installation for the new runtime.

Do not assume that a package from an older runtime remains valid for the newer runtime.

## Legacy note

Host-side `pyenv` version switching is archived in the legacy document, but it is not the recommended baseline for the current architecture.

- docs\applications\programming-languages\python\python-manager\pyenv\general.md

## Related documents

- docs\applications\programming-languages\python\general.md
- docs\applications\programming-languages\python\dependencies.md
- docs\applications\programming-languages\python\cli.md
