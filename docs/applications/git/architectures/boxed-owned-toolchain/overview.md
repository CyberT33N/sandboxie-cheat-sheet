# Boxed-Owned-Toolchain Git

## Status

This is the preferred Git architecture path in this repository.

## Why this file exists

The Git domain is being normalized toward an explicit `architectures\...` split.

This file is the architecture entrypoint for the boxed-owned-toolchain variant.

## Runtime contract

The current governed Git runtime is:

```text
C:\shared\sandbox-toolchains\dev\git\2.54.0\
```

Expected executable:

```text
C:\shared\sandbox-toolchains\dev\git\2.54.0\cmd\git.exe
```

## Provisioning

The current boxed-owned-toolchain provisioning path is:

1. download the 7-Zip helper
2. download the `PortableGit` self-extractor
3. extract it into the governed shared toolchain root

### 7-Zip helper

```powershell
$SevenZipInstaller = Join-Path $env:TEMP '7z2601-x64.exe'
$SevenZipDir = Join-Path $env:TEMP '7zip2601'

Invoke-WebRequest `
  -Uri 'https://github.com/ip7z/7zip/releases/download/26.01/7z2601-x64.exe' `
  -OutFile $SevenZipInstaller

Remove-Item $SevenZipDir -Recurse -Force -ErrorAction SilentlyContinue

Start-Process -FilePath $SevenZipInstaller -ArgumentList '/S',('/D=' + $SevenZipDir) -Wait
```

### PortableGit download and extraction

```powershell
$SevenZipDir = Join-Path $env:TEMP '7zip2601'
$SevenZipExe = Join-Path $SevenZipDir '7z.exe'
$GitSfx = Join-Path $env:TEMP 'PortableGit-2.54.0-64-bit.7z.exe'
$GitDest = 'C:\shared\sandbox-toolchains\dev\git\2.54.0'

Invoke-WebRequest `
  -Uri 'https://github.com/git-for-windows/git/releases/download/v2.54.0.windows.1/PortableGit-2.54.0-64-bit.7z.exe' `
  -OutFile $GitSfx

Remove-Item "$GitDest\*" -Recurse -Force -ErrorAction SilentlyContinue

& $SevenZipExe x $GitSfx ('-o' + $GitDest) -y

& "$GitDest\cmd\git.exe" --version
```

Expected verification:

```text
git version 2.54.0.windows.1
```

## Boxed use

For the boxed-owned-toolchain method:

- the box consumes the governed shared `PortableGit` runtime
- project bootstrap exposes it through the local mirrored toolchain contract
- the first clone happens before project bootstrap opens the repo in boxed VS Code
- authentication remains a Git-domain concern and should not be redefined in VS Code method documents

## Additional current details

The still-detailed Git write-up remains here:

- `docs\applications\git\boxed-owned-toolchain\overview.md`

## Related

- `docs\applications\git\general.md`
- `docs\applications\git\architectures\host-sync\overview.md`
