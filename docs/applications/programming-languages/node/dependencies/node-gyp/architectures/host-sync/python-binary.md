# Python Binary

## Purpose

In the current validated host-sync architecture, `node-gyp` does **not** rely on a random host `python.exe` discovery chain.

Instead, the install box consumes a central shared Python build helper binary from:

```text
C:\shared\sandbox-toolchains\dev\python\
```

This is intentionally a **build helper** boundary for native Node addon compilation, not a full Python application runtime architecture.

## Recommended folder structure

```text
C:\shared\sandbox-toolchains\dev\
  python\
    3.14.5\
      python.exe
    current.txt
```

### Folder responsibilities

- `python\<version>\` stores the official extracted Python binary payload for the selected version.
- `current.txt` stores the version string that should currently be used by the install-box bootstrap commands.

## Why the shared `dev` root is used here

For this validated `node-gyp` workflow:

- Python is not project-local
- Python is not installed inside the box
- Python is reused across multiple Node build flows
- the install box binds `node-gyp` explicitly to the shared binary path through environment variables

## Download the latest official Python binary

The following PowerShell script downloads the latest official Windows embeddable x64 Python package from `python.org`, extracts it into the shared `dev` toolchain root, and writes `current.txt`.

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12

$toolRoot   = "C:\shared\sandbox-toolchains\dev"
$pythonRoot = Join-Path $toolRoot "python"
$pageUrl    = "https://www.python.org/downloads/windows/"

New-Item -ItemType Directory -Force -Path $pythonRoot | Out-Null

$page = (Invoke-WebRequest -UseBasicParsing -Uri $pageUrl).Content
$pattern = '/ftp/python/(?<version>\d+\.\d+\.\d+)/(?<file>python-\d+\.\d+\.\d+-(?:embed|embeddable)-amd64\.zip)'

$releases =
    [regex]::Matches($page, $pattern) |
    ForEach-Object {
        [pscustomobject]@{
            Version  = [version]$_.Groups["version"].Value
            FileName = $_.Groups["file"].Value
            Url      = "https://www.python.org/ftp/python/$($_.Groups["version"].Value)/$($_.Groups["file"].Value)"
        }
    } |
    Sort-Object Version -Descending

$latest = $releases | Select-Object -First 1

if (-not $latest) {
    throw "No official latest Python Windows embeddable package found on python.org."
}

$version   = $latest.Version.ToString()
$destDir   = Join-Path $pythonRoot $version
$zipPath   = Join-Path $env:TEMP $latest.FileName
$currentTx = Join-Path $pythonRoot "current.txt"
$pythonExe = Join-Path $destDir "python.exe"

Write-Host "Latest Python version: $version"
Write-Host "Download URL: $($latest.Url)"
Write-Host "Destination:  $destDir"

if (-not (Test-Path $destDir)) {
    Invoke-WebRequest -UseBasicParsing -Uri $latest.Url -OutFile $zipPath

    New-Item -ItemType Directory -Force -Path $destDir | Out-Null
    Expand-Archive -LiteralPath $zipPath -DestinationPath $destDir -Force
}

if (-not (Test-Path $pythonExe)) {
    throw "python.exe was not found after extraction: $pythonExe"
}

Set-Content -Path $currentTx -Value $version -NoNewline

& $pythonExe --version
```

## Validate the shared Python binary

Run this inside the install box after the shared Python binary has been created:

```powershell
$toolRoot      = "C:\shared\sandbox-toolchains\dev"
$pythonVersion = (Get-Content (Join-Path $toolRoot "python\current.txt") -Raw).Trim()
$pythonExe     = Join-Path $toolRoot "python\$pythonVersion\python.exe"

& $pythonExe --version
& $pythonExe -c "import sys; print(sys.executable)"
```

Expected result:

- `python.exe` is found under the shared `dev` root
- the reported executable path matches `C:\shared\sandbox-toolchains\dev\python\<version>\python.exe`

## Related documents

- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\install-box-config.md`
- `docs\applications\programming-languages\node\dependencies\node-gyp\architectures\host-sync\commands.md`
- `docs\applications\programming-languages\python\general.md`
