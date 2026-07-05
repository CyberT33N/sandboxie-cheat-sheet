# Cursor Login Troubleshooting In The Boxed-Owned-Toolchain

## Scope

This document records the validated troubleshooting trail for the **Cursor login area** in the `boxed-owned-toolchain` architecture.

It explains:

- how the login flow is supposed to work when it is healthy
- what failed in this investigation
- which configuration changes and workaround attempts were tried
- which external GitHub issue posts and related references were reviewed
- which workaround suggestions from that research did **not** solve the problem in this case
- which setting finally worked here

This document is intentionally specific to the **Cursor login / authentication return path**.

It is **not** a generic explanation of all `Electron`, `Sandboxie`, or browser problems.

## Healthy login flow

In a healthy `Cursor` login flow inside the boxed-owned-toolchain method, the sequence should be:

1. `Cursor` shows the login surface inside the boxed runtime.
2. Clicking `Log In` or `Sign Up` opens the system default browser.
3. The browser navigates to a `cursor.com` login URL, typically a `loginDeepControl` or `loginDeepPage` flow.
4. After successful authentication in the browser, the browser returns control to `Cursor`, typically by invoking a `cursor://...` callback or an equivalent deep-link return path.
5. The **already running** boxed `Cursor` instance receives that return signal and updates its UI into the authenticated state.

If this works correctly:

- the default browser opens normally
- the browser can complete the login
- the callback returns to the same running boxed `Cursor` instance
- no broken second `Cursor` window is created
- no `Error mutex already exists`
- no `Failed to open: Access is denied (0x5)`

## What happened here

In this environment, the login flow was only **partially** healthy:

- the boxed `Cursor` window launched correctly
- the default browser could be opened from the boxed environment
- the browser could complete the `cursor.com` login flow
- the browser displayed the success page (`All set! Feel free to return to Cursor.`)

However, the return path back into the boxed `Cursor` runtime was broken.

The important point is:

- **the login flow can work in principle**
- **but in this investigation it did not work correctly until a later configuration change**

The repeated failure shape was:

1. `Cursor` opened the browser or a manually triggered login URL worked
2. login completed in the browser
3. the running boxed `Cursor` instance did **not** return to the logged-in UI
4. manual `cursor://...` tests triggered a second boxed `Cursor` launch path
5. that path produced:
   - `Error mutex already exists`
   - `Failed to open: Access is denied (0x5)`

## Initial assumptions

During the investigation, the working assumption changed several times:

### 1. First assumption: the login button itself was blocked

At first, it looked like the `Log In` button itself was dead or unclickable inside the box.

That assumption turned out to be too shallow.

Why:

- trace output showed access to browser executables and protocol registry paths
- browser-related registry surfaces such as `MSEdgeHTM`, `ChromeHTML`, `FirefoxURL`, and `https` associations were touched

This suggested that the click itself was reaching a browser-launch path.

### 2. Second assumption: default browser resolution was blocked

The next assumption was that the boxed `Cursor` runtime could not correctly resolve or launch the default browser.

That assumption was also disproved.

Why:

- querying the boxed `http` URL association returned `MSEdgeHTM`
- `Start-Process "https://www.cursor.com"` worked from the boxed project terminal
- the default browser (`Edge`) opened correctly from the boxed environment

So the problem was **not** simply "the browser cannot be launched from the box".

### 3. Third assumption: the browser-to-Cursor callback was failing through a second-instance / deep-link handoff

This assumption matched the observed behavior much better.

Why:

- browser login could finish
- `Cursor` still stayed logged out
- manual `cursor://anysphere.cursor/test` triggered a second `Cursor` path
- that second path produced:
  - `Error mutex already exists`
  - `Failed to open: Access is denied (0x5)`

This strongly suggested a **Windows single-instance / deep-link handoff failure** rather than a generic browser-launch failure.

### 4. Final validated interpretation for this case

The decisive fix in this environment was **not** another browser association rule, another named-pipe rule, or another window-class rule.

The correct fix here was:

```ini
ProtectHostImages=n
```

After `ProtectHostImages` was disabled, the login worked.

So the final interpretation for this case is:

- the login mechanism can work
- many earlier theories were reasonable, but they were **not** the decisive root cause here
- in this environment, the reliable fix was to disable `ProtectHostImages`

## Why `ProtectHostImages` mattered here

During the login investigation, the browser side produced `SBIE1305` notifications showing that `msedge.exe` was blocked from loading sandboxed image data from browser profile-related locations such as:

- `...\AppData\Local\Microsoft\Edge\User Data\Domain Actions\...`
- `...\AppData\Local\Microsoft\Edge\User Data\Well Known Domains\...`

This matters because the login path was not just:

- click button
- open URL

It also involved:

- browser-side state
- browser-side profile / domain-action metadata
- a return path from browser to desktop app

In this case, once `ProtectHostImages` was disabled, login succeeded.

That is the validated outcome for this environment.

## What we tried and what did not work

The following attempts were explicitly tried in context and **did not** solve the problem.

### 1. Returning the box to a more default state

Tried:

- setting the box back toward the default configuration
- switching from the hardened box shape toward a more standard / less restrictive shape

Result:

- **did not work**

This must be stated clearly:

- **setting the box back to the default did not work**

### 2. Switching to Yellow / Standard isolation

Tried:

- moving from the hardened / privacy-oriented setup to a `Yellow` / standard-isolation-style setup

Result:

- **did not work**

This must also be stated clearly:

- **setting it to Yellow mode did not work**

### 3. Disabling `UsePrivacyMode`

Tried:

```ini
UsePrivacyMode=n
```

Result:

- **did not work**

This is important because it shows that the problem was not solved by only relaxing privacy-mode hiding behavior.

### 4. Browser association registry reads

Tried:

- adding `ReadKeyPath` access for default-browser association surfaces such as:
  - `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\FileExts\`
  - `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\Shell\Associations\UrlAssociations\`
- both scoped and broader variants

Result:

- **did not work**

This confirmed that simple browser-association visibility was not the decisive blocker.

### 5. Manual browser-launch validation

Tried:

- reading the boxed `http` association
- running `Start-Process "https://www.cursor.com"` from the boxed project terminal

Result:

- the default browser opened correctly
- so browser launch itself was proven healthy
- but this did **not** solve the login-return problem

### 6. Manual `loginDeepControl` browser flow

Tried:

- extracting the `loginDeepControl` URL from trace evidence
- rewriting it to `https://www.cursor.com/...`
- launching it manually in the boxed environment

Result:

- browser login could complete
- browser displayed the success page
- but the running boxed `Cursor` instance still did **not** return to the authenticated UI

So:

- the browser side could work
- the app-side return path still failed

### 7. Overriding the `cursor://` protocol handler

Tried:

- changing the user-level `cursor://` protocol command so it pointed to the boxed runtime through `Start.exe /box:...`

Result:

- **did not solve the problem**
- manual `cursor://anysphere.cursor/test` still produced a broken second-instance path

### 8. Targeted named-pipe attempts

Tried:

- tracing named pipes
- opening the observed main socket pipe surfaces
- including the boxed `main-sock` path variants

Result:

- **did not work**

This is important because it means the same workaround pattern that can fix some real named-pipe problems in other apps did **not** solve this case.

### 9. Window-class exceptions

Tried:

```ini
OpenWinClass=#
```

and later:

```ini
OpenWinClass=*
```

Result:

- **neither worked**

This matters because `OpenWinClass=#` and `OpenWinClass=*` are often the next logical guess when Chromium/Electron apps have window-handoff problems.

In this case:

- **they did not fix the login return path**

## Confirmed fix for this case

This is the one change that **did** work here:

```ini
ProtectHostImages=n
```

This must be stated clearly:

- **setting `ProtectHostImages` to `n` worked**
- after that, the `Cursor` login worked

This was the validated solution for this environment.

It is also important to state what this means semantically:

- the successful fix was **not** "switch to Yellow box"
- it was **not** "disable privacy mode"
- it was **not** "open more protocol / pipe / IPC rules"
- it was **specifically** disabling host-image protection

## External research

## GitHub issue posts reviewed

The following GitHub issue posts were found during the research and were used as context.

### 1. Sandboxie issue: Cursor in a sandbox

- Link: [sandboxie-plus/Sandboxie#5170](https://github.com/sandboxie-plus/Sandboxie/issues/5170)
- Topic:
  - running `Cursor` inside `Sandboxie`
- Workarounds described there:
  - use a `Yellow` / standard-style box first
  - log in there first
  - one participant also described abandoning Sandboxie and using a separate Windows user instead
- Outcome in this investigation:
  - the relevant "loosen the box / Yellow mode / simpler box" direction was tried here
  - **it did not work for us**

### 2. Sandboxie issue: Chrome named pipe access

- Link: [sandboxie-plus/Sandboxie#1946](https://github.com/sandboxie-plus/Sandboxie/issues/1946)
- Topic:
  - a browser failed because of a concrete blocked named pipe
- Workaround described there:
  - add a targeted `OpenPipePath=\Device\NamedPipe\...`
- Outcome in this investigation:
  - we tried the analogous "open the specific observed Cursor login pipe" approach
  - **it did not work for us**

### 3. Sandboxie issue: Edge in Sandboxie and ESET pipe problems

- Link: [sandboxie-plus/Sandboxie#2835](https://github.com/sandboxie-plus/Sandboxie/issues/2835)
- Topic:
  - `Edge` had sandbox problems; one part involved `nod_scriptmon_pipe`
- Workaround described there:
  - update to a newer Sandboxie version
  - ESET-specific suspicion was also noted
- Outcome in this investigation:
  - the failure pattern was not the same as the ESET `nod_scriptmon_pipe` issue
  - this did **not** provide the working fix for our case

### 4. Sandboxie issue: ESET ALPC / template workaround

- Link: [sandboxie-plus/Sandboxie#4589](https://github.com/sandboxie-plus/Sandboxie/issues/4589)
- Topic:
  - `SBIE2112` with ESET communication objects
- Workaround described there:
  - add:
    ```ini
    Template=NOD32
    ```
- Outcome in this investigation:
  - our environment was not the same ESET template problem
  - this workaround was not the validated fix for our case

### 5. Sandboxie issue: broad IPC troubleshooting suggestion

- Link: [sandboxie-plus/Sandboxie#1749](https://github.com/sandboxie-plus/Sandboxie/issues/1749)
- Topic:
  - programs failing because of blocked IPC / process access
- Workaround described there:
  - temporarily try a very broad IPC allowance such as:
    ```ini
    ReadIpcPath=$:*
    ```
  - then narrow it based on logs
- Outcome in this investigation:
  - our traces never yielded a clean single missing IPC exception that explained the login failure
  - this did **not** become the fix for our case

### 6. Sandboxie issue: Electron startup workaround for Signal

- Link: [sandboxie-plus/Sandboxie#2235](https://github.com/sandboxie-plus/Sandboxie/issues/2235)
- Topic:
  - `Signal` failed as an Electron app inside Sandboxie
- Workaround described there:
  - add:
    ```ini
    UseElectronWorkaround=Signal.exe,n
    ```
  - another workaround in the same issue was `--in-process-gpu`
- Outcome in this investigation:
  - this is a different Electron failure class
  - it did **not** explain or fix the Cursor login-return problem here

### 7. Sandboxie issue: Electron startup workaround for Caprine

- Link: [sandboxie-plus/Sandboxie#2201](https://github.com/sandboxie-plus/Sandboxie/issues/2201)
- Topic:
  - `Caprine` crashed in Sandboxie
- Workaround described there:
  - add:
    ```ini
    UseElectronWorkaround=Caprine.exe,n
    ```
- Outcome in this investigation:
  - this was another Electron startup incompatibility, not the same login/deep-link return failure
  - it was not the validated fix for our case

### 8. Sandboxie issue: Chromium/Electron window-class workaround

- Link: [sandboxie-plus/Sandboxie#440](https://github.com/sandboxie-plus/Sandboxie/issues/440)
- Topic:
  - Chromium-class window behavior in Sandboxie
- Workaround described there:
  - `OpenWinClass=#`
- Outcome in this investigation:
  - we tried `OpenWinClass=#`
  - we also tried the stronger `OpenWinClass=*`
  - **neither worked for us**

### 9. Electron issue: single-instance lock crash in Windows sessions

- Link: [electron/electron#33975](https://github.com/electron/electron/issues/33975)
- Topic:
  - Windows single-instance lock and named-pipe acknowledgement behavior
- Workaround described there:
  - no practical end-user configuration workaround
  - the issue was fixed upstream in `Electron 18.2.3`
- Outcome in this investigation:
  - useful as background for the failure shape
  - not a usable local configuration fix for our case

### 10. Electron issue: requestSingleInstanceLock should use message window

- Link: [electron/electron#34235](https://github.com/electron/electron/issues/34235)
- Topic:
  - discussion of replacing named-pipe acknowledgement with message-window handling
- Workaround described there:
  - no end-user workaround
  - this was an implementation-direction discussion
- Outcome in this investigation:
  - useful for understanding the Windows handoff model
  - not a direct fix for our case

### 11. Electron issue: elevated first instance breaks second-instance event

- Link: [electron/electron#35681](https://github.com/electron/electron/issues/35681)
- Topic:
  - `app.requestSingleInstanceLock()` silently failing when the first instance is elevated
- Workaround described there:
  - a code-level custom single-instance mechanism using Node `net`
  - practical lesson: same-privilege assumptions matter
- Outcome in this investigation:
  - useful as supporting background
  - not a usable boxed-owned-toolchain configuration fix for our case

## Other research references

The following non-GitHub references were also relevant during the investigation:

### Cursor forum post about second-instance handoff failure

- Link: [Cursor Community Forum: second window / mutex already exists / workspace storage error](https://forum.cursor.com/t/cursor-opens-a-second-window-with-a-workspace-storage-error-when-i-run-cursor-r-somefile-txt-from-a-terminal-while-another-cursor-window-is-already-open/156618)
- Key point:
  - on Windows, `Cursor` can open a broken second instance instead of cleanly reusing the first instance
  - logs there also include `Error mutex already exists`
- Relevance here:
  - this strongly matched the observed local behavior

### Sandboxie documentation: `OpenIpcPath`

- Link: [Sandboxie-Plus: OpenIpcPath](https://sandboxie-plus.com/sandboxie/openipcpath/)
- Relevance:
  - confirmed the semantics of `OpenIpcPath`
  - reinforced that only trace-proven IPC exceptions should be opened

### Sandboxie documentation: `OpenWinClass`

- Link: [Sandboxie-Plus: OpenWinClass](https://sandboxie-plus.com/sandboxie/openwinclass/)
- Relevance:
  - confirmed the difference between:
    - `OpenWinClass=#`
    - `OpenWinClass=*`
  - useful for understanding why those tests were reasonable

### Sandboxie documentation: `SBIE2314`

- Link: [Sandboxie-Plus: SBIE2314](https://sandboxie-plus.com/sandboxie/sbie2314/)
- Relevance:
  - documents how Sandboxie sometimes launches helper instances to receive URLs or file-open requests from outside the sandbox
  - useful background for why URL-return paths are a special failure surface

## Why the external workaround suggestions were not enough here

The main lesson from this investigation is:

- many external workaround patterns were reasonable
- several were directly tried
- some were only background hints for different failure classes
- but **the actual decisive fix here was still different**

In particular:

- `Yellow` / default / standard box direction:
  - **did not work**
- `UsePrivacyMode=n`:
  - **did not work**
- targeted browser-association registry reads:
  - **did not work**
- manual `loginDeepControl` browser flow:
  - browser login worked, app return still failed
- protocol-handler override:
  - **did not work**
- targeted named-pipe rules:
  - **did not work**
- `OpenWinClass=#`:
  - **did not work**
- `OpenWinClass=*`:
  - **did not work**

That is why the document must explicitly state:

- the workaround suggestions gathered from issue posts were useful context
- the ones that were actually tried here still **did not** solve the problem
- the confirmed working fix in this case was **disabling `ProtectHostImages`**

## Final conclusion

The login area in `Cursor` can work in the boxed-owned-toolchain architecture.

This investigation proved that at least some of the flow was healthy:

- the boxed `Cursor` runtime launched
- the default browser could be opened from the box
- the browser could complete authentication

However, in this case the overall login flow still remained broken until the final configuration change.

The required final statement is:

- **it can work**
- **but in our case the earlier workaround attempts did not work**
- **the confirmed fix here was `ProtectHostImages=n`**
- **after disabling `ProtectHostImages`, login worked**

## Related

- `docs\applications\IDE\cursor\architectures\host-sync\debugging.md`
- `docs\applications\programming-languages\node\dependencies\frameworks\electron\architectures\boxed-owned-toolchain\troubleshooting.md`
- `docs\applications\git\architectures\boxed-owned-toolchain\authentication-and-clone.md`
