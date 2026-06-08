# Boxed-Owned-Toolchain Git

## Status

This is the preferred Git architecture path in this repository.

## Why this file exists

The Git domain is being normalized toward an explicit `architectures\...` split.

This file is the architecture entrypoint for the boxed-owned-toolchain variant.

Use this document when Git authentication or cloning must happen inside the boxed-owned-toolchain method.

From a domain-driven and 12-factor perspective, the correct owner for this workflow is the Git domain, not the VS Code domain.

Why:

- authentication is a Git concern
- credential-helper selection is a Git concern
- private-repo access checks are a Git concern
- the initial boxed clone happens before the project bootstrap can operate on an existing repo path

VS Code method documents should therefore reference this document instead of re-defining the Git login flow in multiple places.

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

## Preference

This is the preferred architecture path.

The host-sync Git path remains only as a secondary / legacy reference here:

- `docs\applications\git\architectures\host-sync\overview.md`

## Boxed use

For the boxed-owned-toolchain method:

- the box consumes the governed shared `PortableGit` runtime
- project bootstrap exposes it through the local mirrored toolchain contract
- the first clone happens before project bootstrap opens the repo in boxed VS Code
- authentication remains a Git-domain concern and should not be redefined in VS Code method documents

The boxed-owned-toolchain Git model is:

- host launches a boxed shell or boxed one-shot command through `Start.exe`
- the box consumes the shared `PortableGit` runtime
- the private-repo login is completed through Git Credential Manager
- the preferred sign-in path is the GitHub device flow in the host browser
- the first boxed clone happens before project bootstrap opens the repo in the project box

## Architectural rules

### No host Git as the default path

For the boxed-owned-toolchain method, host Git is not the normal execution surface.

The only host-side participation in this workflow is:

- `Start.exe` launching the boxed process
- the host browser completing the GitHub device-code sign-in

### First clone is a Git-domain action

The project bootstrap expects the target repo path to already exist.

That means:

- the initial clone must happen before `Start-<Project>VSCode.ps1 -Action OpenTerminal`
- the initial clone is therefore documented here, in the Git area

## Validated flow

### Step 1 - start from a neutral boxed working directory

For the first clone, do **not** start from a path that forces Git to walk parent directories through a sensitive `C:\Users\...` chain in the box.

Use a neutral working directory and explicitly control the target path.

Inside the boxed shell:

```powershell
Set-Location C:\
$env:HOME = $env:USERPROFILE
$env:GIT_CEILING_DIRECTORIES = "C:/Users/yourusername/source"
```

This keeps Git repository discovery from walking above the intended source root during the pre-clone state.

### Step 2 - public Git reachability smoke test

Inside the boxed shell:

```powershell
$GitExe = "C:\shared\sandbox-toolchains\dev\git\2.54.0\cmd\git.exe"

& $GitExe `
  -c credential.helper= `
  ls-remote https://github.com/git/git.git
```

If this returns refs, then:

- the boxed Git runtime is working
- network access to GitHub is working
- the remaining problem, if any, is private-repo authentication or repo access

### Step 3 - private-repo access probe

Inside the boxed shell:

```powershell
$GitExe = "C:\shared\sandbox-toolchains\dev\git\2.54.0\cmd\git.exe"
$GitHubUser = "yourgithubuser"

& $GitExe `
  -c credential.helper=manager `
  ls-remote "https://$GitHubUser@github.com/yourorg/yourrepo.git"
```

### Step 4 - choose the credential helper

If Git for Windows shows the helper-selection dialog:

- choose `manager`
- enable `Always use this from now on`

This avoids being asked again on every private GitHub access in the same user context.

### Step 5 - complete the GitHub login

If Git Credential Manager shows a browser or device-code flow, the preferred path is:

1. open `github.com/login/device` in the normal host browser
2. use the GitHub account that already has the correct GitHub / SSO access
3. if your GitHub sign-in is federated, continue through the normal GitHub flow such as Google-backed sign-in
4. enter the device code shown by the boxed prompt
5. complete any organization / SSO confirmation that GitHub requires

This is the preferred method because:

- it avoids typing credentials into the box
- it works with modern GitHub sign-in flows
- it keeps the Git command itself inside the sandbox while letting the host browser handle the OAuth/device ceremony

### Step 6 - interpret post-login errors carefully

After the device flow completes, you may still see an error dialog from `git-credential-manager.exe`, for example a Windows system error dialog.

That dialog does **not** by itself prove that the login failed.

The correct verification step is always the Git command result:

```powershell
$GitExe = "C:\shared\sandbox-toolchains\dev\git\2.54.0\cmd\git.exe"
$GitHubUser = "yourgithubuser"

& $GitExe `
  -c credential.helper=manager `
  ls-remote "https://$GitHubUser@github.com/yourorg/yourrepo.git"
```

If refs are returned, access is working, regardless of a prior helper-side dialog.

### Step 7 - clone after access is verified

Once `ls-remote` returns refs, clone the repo:

```powershell
$GitExe = "C:\shared\sandbox-toolchains\dev\git\2.54.0\cmd\git.exe"
$GitHubUser = "yourgithubuser"

New-Item -ItemType Directory -Force -Path "C:\Users\yourusername\source" | Out-Null

& $GitExe `
  -c credential.helper=manager `
  clone "https://$GitHubUser@github.com/yourorg/yourrepo.git" "C:\Users\yourusername\source\yourrepo"
```

## Host-driven one-shot clone pattern

If you want one single host command for a fresh boxed clone, use:

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:YOUR_PROJECT_BOX `
  "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
  -NoLogo `
  -NoExit `
  -ExecutionPolicy Bypass `
  -Command "Set-Location 'C:\'; `$env:HOME = `$env:USERPROFILE; `$env:GIT_CEILING_DIRECTORIES = 'C:/Users/yourusername/source'; New-Item -ItemType Directory -Force -Path 'C:\Users\yourusername\source' | Out-Null; & 'C:\shared\sandbox-toolchains\dev\git\2.54.0\cmd\git.exe' -c credential.helper=manager clone 'https://yourgithubuser@github.com/yourorg/yourrepo.git' 'C:\Users\yourusername\source\yourrepo'"
```

This keeps the clone fully boxed while still reducing the host interaction to one reproducible command.

## Failure interpretation

### `Repository not found`

Usually means one of:

- wrong repo URL
- the authenticated GitHub account cannot see that repo
- org / SSO authorization did not actually complete for the target repo

### No refs from the private `ls-remote`

Treat this as an auth or repo-visibility problem, not as a VS Code bootstrap problem.

### `Project repo not found`

This is not a Git auth failure. It only means the project bootstrap was invoked before the repo was cloned.

## Relationship to the VS Code documents

After the initial clone succeeds:

- use the project bootstrap to open the boxed terminal
- use the project bootstrap to launch boxed VS Code

Those flows remain documented in the VS Code method area and should reference this document for the Git login / initial clone part.

## Related

- `docs\applications\git\general.md`
- `docs\applications\git\architectures\host-sync\overview.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\general.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\toolchain\git.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\boilerplates\test-mono\start.md`
