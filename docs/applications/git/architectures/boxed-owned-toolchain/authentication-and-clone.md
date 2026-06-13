# Authentication And Clone

## Scope

This document owns the **authentication, reachability, and first-clone flow** for Git in the boxed-owned-toolchain architecture.

It covers:

- public Git reachability
- private-repo access checks
- Git Credential Manager login behavior
- first clone workflow

It does **not** own:

- PortableGit provisioning
- bootstrap projection code
- Git-independent long-path policy

## Why this remains a Git-domain concern

From a domain-driven and 12-factor perspective, the first private clone remains a Git concern because:

- authentication is a Git concern
- credential-helper behavior is a Git concern
- private-repo access verification is a Git concern
- the initial clone happens before the normal project bootstrap can assume an existing repo root

That is why the VS Code method documents should re-reference the Git domain instead of restating login and clone behavior inline.

## Step 1 - start from a neutral boxed working directory

For the first clone, do **not** start from a path that forces Git to walk parent directories through a sensitive `C:\Users\...` chain in the box.

Inside the boxed shell:

```powershell
Set-Location C:\
$env:HOME = $env:USERPROFILE
$env:GIT_CEILING_DIRECTORIES = "C:/Users/yourusername/source"
```

This keeps Git repository discovery from walking above the intended source root during the pre-clone state.

## Step 2 - public Git reachability smoke test

Inside the boxed shell:

```powershell
git -c credential.helper= ls-remote https://github.com/git/git.git
```

If this returns refs, then:

- the boxed Git runtime is working
- network access to GitHub is working
- the remaining problem, if any, is private-repo authentication or repo access

## Step 3 - private-repo access probe

Inside the boxed shell:

```powershell
$GitHubUser = "yourgithubuser"

git ls-remote "https://$GitHubUser@github.com/yourorg/yourrepo.git"
```

## Step 4 - choose the credential helper

If Git for Windows shows the helper-selection dialog:

- choose `manager`
- enable `Always use this from now on`

The current boxed runtime then prefers the bootstrap-published helper command surface:

```text
credential.helper=manager-boxed
```

This avoids pushing a whitespace-sensitive absolute helper path directly into Git command lines.

## Step 5 - complete the GitHub login

If Git Credential Manager shows a browser or device-code flow, the preferred path is:

1. open `github.com/login/device` in the normal host browser
2. use the GitHub account that already has the correct GitHub / SSO access
3. if your GitHub sign-in is federated, continue through the normal GitHub flow such as Google-backed sign-in
4. enter the device code shown by the boxed prompt
5. complete any organization / SSO confirmation that GitHub requires

Why this is preferred:

- it avoids typing credentials into the box
- it works with modern GitHub sign-in flows
- it keeps the Git command itself inside the sandbox while letting the host browser handle the OAuth/device ceremony

## Step 6 - interpret post-login errors carefully

After the device flow completes, you may still see an error dialog from `git-credential-manager.exe`.

That dialog does **not** by itself prove that the login failed.

The correct verification step is always the Git command result:

```powershell
$GitHubUser = "yourgithubuser"

git ls-remote "https://$GitHubUser@github.com/yourorg/yourrepo.git"
```

If refs are returned, access is working.

## Step 7 - clone after access is verified

Once `ls-remote` returns refs, clone the repo:

```powershell
$GitHubUser = "yourgithubuser"

New-Item -ItemType Directory -Force -Path "C:\Users\yourusername\source" | Out-Null

git clone "https://$GitHubUser@github.com/yourorg/yourrepo.git" "C:\Users\yourusername\source\yourrepo"
```

## Host-driven one-shot clone pattern

There is still a special pre-bootstrap clone case:

- the project bootstrap expects the repo path to exist already
- therefore the very first clone is still a Git-domain pre-bootstrap action

If you want one single host command for a fresh boxed clone, the current repository still documents this historical special-case shape:

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:YOUR_PROJECT_BOX `
  "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
  -NoLogo `
  -NoExit `
  -ExecutionPolicy Bypass `
  -Command "Set-Location 'C:\'; `$env:HOME = `$env:USERPROFILE; `$env:GIT_CEILING_DIRECTORIES = 'C:/Users/yourusername/source'; New-Item -ItemType Directory -Force -Path 'C:\Users\yourusername\source' | Out-Null; & 'C:\shared\sandbox-toolchains\dev\git\2.54.0\cmd\git.exe' -c credential.helper=manager clone 'https://yourgithubuser@github.com/yourorg/yourrepo.git' 'C:\Users\yourusername\source\yourrepo'"
```

Important nuance:

- this is a pre-bootstrap special case
- it does **not** redefine the normal post-bootstrap Git runtime contract
- the normal project terminal should prefer the bootstrap-published `git` wrapper once that runtime exists

## Failure interpretation

### `Repository not found`

Usually means one of:

- wrong repo URL
- the authenticated GitHub account cannot see that repo
- organization / SSO authorization did not actually complete for the target repo

### No refs from the private `ls-remote`

Treat this as an auth or repo-visibility problem, not as a VS Code bootstrap problem.

### `Project repo not found`

This is not a Git auth failure. It only means the project bootstrap was invoked before the repo was cloned.

## Related

- `docs\applications\git\architectures\boxed-owned-toolchain\overview.md`
- `docs\applications\git\architectures\boxed-owned-toolchain\runtime-contract.md`
- `docs\applications\git\architectures\boxed-owned-toolchain\provisioning.md`
- `docs\applications\git\architectures\boxed-owned-toolchain\troubleshooting\long-paths.md`
- `docs\applications\IDE\vscode\methods\boxed-owned-toolchain\general.md`
