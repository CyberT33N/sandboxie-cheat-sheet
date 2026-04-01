# Docling


```
$toolRoot = "C:\shared\sandbox-toolchains\python-general"
$pythonExe = "$toolRoot\python\3.12.9\python.exe"
$env:PYTHONUSERBASE = "$toolRoot\userbase"
$env:PIP_CACHE_DIR = "$toolRoot\cache\pip"
$env:HF_HOME = "$toolRoot\cache\huggingface"
$env:TORCH_HOME = "$toolRoot\cache\torch"
$env:PATH = "$toolRoot\userbase\Python312\Scripts;" + $env:PATH
& "$toolRoot\userbase\Python312\Scripts\docling.exe" "$toolRoot\work\input" --from pdf --to md --page-batch-size 1 --num-threads 1 --image-export-mode referenced --output "$toolRoot\work\output"
```


settings.ini
docs\applications\programming-languages\python\box-presets\strict-unprotected-host.md