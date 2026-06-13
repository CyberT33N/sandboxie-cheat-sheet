# Git Provisioning

## Scope

This document owns the **Git provisioning contract** for the boxed-owned-toolchain architecture.

It covers:

- the governed shared Git root
- the expected executable surfaces
- the current PortableGit provisioning flow

It does **not** own:

- boxed runtime projection
- Git Credential Manager execution inside the box
- authentication and clone workflows
- long-path troubleshooting

Those concerns live in their own Git-domain documents.

## Governed shared root

The current governed shared Git runtime is:

```text
C:\shared\sandbox-toolchains\dev\git\2.54.0\
```

Expected executable surfaces:

```text
C:\shared\sandbox-toolchains\dev\git\2.54.0\cmd\git.exe
C:\shared\sandbox-toolchains\dev\git\2.54.0\mingw64\bin\git-credential-manager.exe
```

## Why PortableGit is the selected form

The boxed-owned-toolchain architecture treats Git as a governed shared runtime:

- not as a host-default dependency
- not as a random workstation installation
- not as a repo-local vendored binary

PortableGit is the selected form because it gives:

- a versioned shared runtime root
- reproducible bootstrap projection into the box
- a stable path contract for later wrapper publication

## Current provisioning flow

The current provisioning path is:

1. download the 7-Zip helper
2. download the PortableGit self-extractor
3. extract the runtime into the governed shared Git root
4. verify `git.exe`

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

## Current shared Git config nuance

The current governed shared Git runtime still contains:

```ini
[credential]
    helper = helper-selector
```

inside:

```text
C:\shared\sandbox-toolchains\dev\git\2.54.0\etc\gitconfig
```

That shared config is part of the provisioned runtime, but it is **not** the final boxed execution contract by itself.

The boxed-owned-toolchain runtime contract later adds:

- a bootstrap-published boxed `git` wrapper
- a bootstrap-published boxed `git-credential-manager-boxed` helper command
- automatic `core.longpaths=true`

Those runtime concerns live here:

- `docs\applications\git\architectures\boxed-owned-toolchain\runtime-contract.md`

## Related

- `docs\applications\git\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\git\architectures\boxed-owned-toolchain\runtime-contract.md`
- `docs\applications\git\architectures\boxed-owned-toolchain\authentication-and-clone.md`
- `docs\applications\git\architectures\boxed-owned-toolchain\troubleshooting\long-paths.md`
