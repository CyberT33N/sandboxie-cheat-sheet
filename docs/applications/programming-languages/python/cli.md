
# Python CLI workflow

This document describes the recommended CLI workflow for the dedicated shared toolchain architecture.

It is intentionally not based on host-side `pyenv` shims.

## Scope

Use this workflow when:

- the sandbox uses the dedicated root `C:\shared\sandbox-toolchains\python-general\`
- the package should be installed from inside the sandbox
- the CLI should run from the sandbox-owned toolchain instead of from a host installation

## Session bootstrap

Start PowerShell inside the sandbox and set the toolchain variables:

```powershell
$toolRoot = "C:\shared\sandbox-toolchains\python-general"
$pythonExe = "$toolRoot\python\3.12.9\python.exe"
$env:PYTHONUSERBASE = "$toolRoot\userbase"
$env:PIP_CACHE_DIR = "$toolRoot\cache\pip"
$env:HF_HOME = "$toolRoot\cache\huggingface"
$env:TORCH_HOME = "$toolRoot\cache\torch"
$env:PATH = "$toolRoot\userbase\Python312\Scripts;" + $env:PATH
```

## Verify the runtime and generated launchers

```powershell
& $pythonExe --version
Get-ChildItem "$toolRoot\userbase\Python312\Scripts"
```

## Generic CLI execution pattern

Preferred launcher pattern:

```powershell
& "$toolRoot\userbase\Python312\Scripts\<tool>.exe" <arguments>
```

If the package exposes a supported CLI name through the generated launcher path, you can also call it after the script directory was added to `PATH`:

```powershell
<tool> <arguments>
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

& "$toolRoot\userbase\Python312\Scripts\docling.exe" "$toolRoot\work\input" --from pdf --to md --image-export-mode referenced --output "$toolRoot\work\output"
```

## Generic script execution pattern

When the workload is a script instead of a generated CLI launcher, execute it with the dedicated interpreter from the toolchain root:

```powershell
& $pythonExe "C:\path\to\script.py"
```

## Operational rules

- Do not use host-side `pyenv` shims as the primary CLI entry point.
- Do not treat host installation plus virtualization as the recommended model.
- Always bootstrap the session variables before installing or executing tools.
- Reinstall the package inside the selected Python version when the runtime version changes.

## Related documents

- docs\applications\programming-languages\python\general.md
- docs\applications\programming-languages\python\dependencies.md
- docs\applications\programming-languages\python\versioning.md
- docs\applications\programming-languages\python\python-manager\pyenv\general.md
