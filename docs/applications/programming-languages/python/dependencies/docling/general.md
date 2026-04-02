# Docling


## settings.ini
docs\applications\programming-languages\python\box-presets\strict-unprotected-host.md


v5
```shell
$toolRoot = "C:\shared\sandbox-toolchains\python-general"
$pythonExe = "$toolRoot\python\3.12.9\python.exe"
$env:PYTHONUSERBASE = "$toolRoot\userbase"
$env:PIP_CACHE_DIR = "$toolRoot\cache\pip"
$env:HF_HOME = "$toolRoot\cache\huggingface"
$env:TORCH_HOME = "$toolRoot\cache\torch"
$env:PATH = "$toolRoot\userbase\Python312\Scripts;" + $env:PATH

& "$toolRoot\userbase\Python312\Scripts\docling.exe" "$toolRoot\work\input" --from pdf --to md --image-export-mode referenced --output "$toolRoot\work\output"
```
- https://github.com/CyberT33N/docling-cheat-sheet/blob/main/docs/troubleshooting/general.md
  - Geht aktuell nicht wegen Bug. Deswegen nutze ich v4


v4
```shell
$toolRoot = "C:\shared\sandbox-toolchains\python-general"
$pythonExe = "$toolRoot\python\3.12.9\python.exe"
$env:PYTHONUSERBASE = "$toolRoot\userbase"
$env:PIP_CACHE_DIR = "$toolRoot\cache\pip"
$env:HF_HOME = "$toolRoot\cache\huggingface"
$env:TORCH_HOME = "$toolRoot\cache\torch"
$env:PATH = "$toolRoot\userbase\Python312\Scripts;" + $env:PATH

Get-ChildItem "$toolRoot\work\input" -Filter *.pdf | ForEach-Object {
    & "$toolRoot\userbase\Python312\Scripts\docling.exe" $_.FullName --from pdf --to md --image-export-mode referenced --output "$toolRoot\work\output"
}
```
