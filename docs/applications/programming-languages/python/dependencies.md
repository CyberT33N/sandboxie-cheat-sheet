# Python dependencies

## Architectural rule

Dependencies must be installed into the dedicated toolchain root for the Python version that will execute them.

The installation must use the `--user` flag so that the package is written into the dedicated toolchain `userbase` and not into the base interpreter directories.

The recommended model is:

- runtime under `C:\shared\sandbox-toolchains\python-general\python\<version>\`
- packages under `C:\shared\sandbox-toolchains\python-general\userbase\`
- caches under `C:\shared\sandbox-toolchains\python-general\cache\`

Host installation is a legacy pattern and is not the recommended default.

## Session bootstrap before dependency installation

```powershell
$toolRoot = "C:\shared\sandbox-toolchains\python-general"
$pythonExe = "$toolRoot\python\3.12.9\python.exe"
$env:PYTHONUSERBASE = "$toolRoot\userbase"
$env:PIP_CACHE_DIR = "$toolRoot\cache\pip"
$env:HF_HOME = "$toolRoot\cache\huggingface"
$env:TORCH_HOME = "$toolRoot\cache\torch"
$env:PATH = "$toolRoot\userbase\Python312\Scripts;" + $env:PATH
```

## Install a package for the active Python version

```powershell
& $pythonExe -m pip install --user <package-name>
```

This `--user` flag is mandatory for the documented toolchain layout.

Without `--user`, `pip` may install packages and launchers under:

- `python\<version>\Lib\site-packages`
- `python\<version>\Scripts`

which causes path drift relative to the documented `userbase\Python312\Scripts` launcher path.

## Upgrade a package for the active Python version

```powershell
& $pythonExe -m pip install --user --upgrade <package-name>
```

## Force a clean reinstall for the active Python version

```powershell
& $pythonExe -m pip install --user --upgrade --force-reinstall <package-name>
```

The same rule applies to version changes and downgrades: use `--user` consistently.

Example downgrade pattern:

```powershell
& $pythonExe -m pip install --user --upgrade --force-reinstall "<package-name>==<version>"
```

## `docling` example

```powershell
$toolRoot = "C:\shared\sandbox-toolchains\python-general"
$pythonExe = "$toolRoot\python\3.12.9\python.exe"
$env:PYTHONUSERBASE = "$toolRoot\userbase"
$env:PIP_CACHE_DIR = "$toolRoot\cache\pip"
$env:HF_HOME = "$toolRoot\cache\huggingface"
$env:TORCH_HOME = "$toolRoot\cache\torch"
$env:PATH = "$toolRoot\userbase\Python312\Scripts;" + $env:PATH

& $pythonExe -m pip install --user --upgrade --force-reinstall docling
```

## Dependency selection by sandbox profile

### Compatibility profile

Use the compatibility profile when the dependency tree includes native modules or Windows-facing runtime behavior.

Examples:

- `docling`
- Pillow-backed image stacks
- torch-backed packages
- OCR packages

### Strict profile

Use the strict profile only when the dependency set is simple enough to work with the stricter DLL-loading policy.

Examples:

- pure-Python calculation packages
- lightweight internal utilities

## Anti-pattern to avoid

The following model is intentionally not the recommended baseline:

- install the package on the host
- rely on host virtualization behavior later
- treat host-side `pyenv` as the primary package manager boundary

This pattern remains documented for archive purposes only.

## Related documents

- docs\applications\programming-languages\python\general.md
- docs\applications\programming-languages\python\cli.md
- docs\applications\programming-languages\python\versioning.md
- docs\applications\programming-languages\python\python-manager\pyenv\general.md
