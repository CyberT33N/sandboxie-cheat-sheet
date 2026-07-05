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
- if the UI wording shows `Managed` / a managed credential option instead of the literal word `manager`, choose that managed option
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

Important current operational nuance:

- for a **fresh box / fresh private repo** the most reliable way to surface and persist that login flow is the first **direct shared `git.exe clone`** call with a user-qualified HTTPS URL
- if the credential choice and login were already stored earlier, that command should normally proceed without asking again
- if they were **not** stored yet, the same command should trigger the helper-selection / device-login flow once and then be retried if needed

## Step 6 - interpret post-login errors carefully

After the device flow completes, you may still see an error dialog from `git-credential-manager.exe`.

That dialog does **not** by itself prove that the login failed.

The correct verification step is always the Git command result:

```powershell
$GitHubUser = "yourgithubuser"

git ls-remote "https://$GitHubUser@github.com/yourorg/yourrepo.git"
```

If refs are returned, access is working.

If refs do **not** return after the browser/device flow:

- treat that as an auth / repo-visibility / SSO problem
- do **not** treat it as a VS Code / Cursor / project-bootstrap problem
- re-run the same private `ls-remote` or direct clone command only after confirming that the correct GitHub account and organization access were actually used

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

If you want one single host command for a fresh boxed clone, the current repository still documents this historical special-case shape.

This command should be treated as the **one-time pre-bootstrap auth + clone entrypoint** for a fresh private repository in a new project box:

- run it first
- let it surface the managed credential flow
- complete the login if prompted
- if the login was newly completed, retry the same command once
- for very long repository paths, set boxed `LongPathsEnabled=1` before the clone and run the direct shared `git.exe` clone with `-c core.longpaths=true`
- after that, the normal project bootstrap can assume that the repo exists

Sanitized example:

```powershell
& "C:\Program Files\Sandboxie-Plus\Start.exe" `
  /box:YOUR_PROJECT_BOX `
  "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
  -NoLogo `
  -NoExit `
  -ExecutionPolicy Bypass `
  -Command "& '$env:SystemRoot\System32\reg.exe' add 'HKLM\SYSTEM\CurrentControlSet\Control\FileSystem' /v LongPathsEnabled /t REG_DWORD /d 1 /f | Out-Null; Set-Location 'C:\'; `$env:HOME = `$env:USERPROFILE; `$env:GIT_CEILING_DIRECTORIES = 'C:/Users/example-user/source'; New-Item -ItemType Directory -Force -Path 'C:\Users\example-user\source' | Out-Null; & 'C:\shared\sandbox-toolchains\dev\git\2.54.0\cmd\git.exe' -c core.longpaths=true clone 'https://example-user@github.com/example-org/example-monorepo.git' 'C:\Users\example-user\source\example-monorepo'"
```

Important nuance:

- this is a pre-bootstrap special case
- it does **not** redefine the normal post-bootstrap Git runtime contract
- the normal project terminal should prefer the bootstrap-published `git` wrapper once that runtime exists
- the one-time direct clone call is also the place where the managed credential flow is expected to become stored for later reuse

## Recovery from a partially checked-out long-path clone

If the first direct clone already created a partial working tree and then failed with messages such as:

```text
error: unable to create file <path>: Filename too long
```

the current boxed-owned-toolchain recovery is:

1. delete the partial boxed repo tree
2. ensure boxed `LongPathsEnabled=1`
3. retry the same direct shared `git.exe` clone with:
   - `-c core.longpaths=true`

Why:

- the first private clone is still a pre-bootstrap special case
- so the normal project bootstrap has not yet had a chance to publish the boxed `git` wrapper or set the boxed Windows long-path flag for that project box
- that means the recovery must explicitly apply both long-path layers before retrying the clone

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
