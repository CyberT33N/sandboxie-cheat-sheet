


# ProtectHostImages=y causes SBIE1305 errors with mozglue.dll in Firefox 135.0.1
- https://github.com/sandboxie-plus/Sandboxie/issues/4530

```text
firefox.exe: SBIE1305 Blocked loading a sandboxed image - \drive\C\Program Files\Mozilla Firefox\mozglue.dll
```


#  Option 1 (recommended)

With version 1.15.4, the `FileCopy` template was updated to fix this issue, but according to the information you shared, this template is not included in your configuration. You can resolve the issue by adding the `Template=FileCopy` OR the `DontCopy=%ProgramFiles%*\Mozilla Firefox\*` setting to your configuration. After adding the necessary setting to your configuration, follow the steps below and delete the folder containing the files that cause this issue.

1. In the main interface, locate and **right-click** on the `DefaultBox` sandbox.

2. From the context menu, select **"Box Content > Explorer Content"**.
   - This will open a File Explorer window (unsandboxed) showing the virtual file system of the `DefaultBox` sandbox.

3. In the File Explorer window, go to the following directory:
     ```
     C:\Sandbox\%USERNAME%\DefaultBox\drive\C\Program Files
     ```

4. Locate the `Mozilla Firefox` folder inside the `Program Files` directory.
   - **Right-click** on the `Mozilla Firefox` folder and select **Delete**.
   - Confirm the deletion if prompted.

5. Launch Mozilla Firefox from within the `DefaultBox` sandbox to verify the issue is resolved.

---

**Additional Notes:**
- If you have multiple sandboxes, repeat these steps for each sandbox where the issue occurs.
- Ensure no instances of Firefox are running in the sandbox before deleting the folder.
- If you are using a version prior to `1.15.4`, add the following setting under **"Global Settings > Edit ini Section"**.
    ```ini
    DontCopy=%ProgramFiles%*\Mozilla Firefox\*
    ```
- If you are using version `1.15.4` or later, add the following setting under **"Sandbox Options > Edit ini Section"** OR **"Global Settings > Edit ini Section"**.
    ```ini
    Template=FileCopy
    ```



---

Option 2 (Without Template=FileCopy)
  


**Meaning:** Firefox attempted to load a *boxed* (sandboxed) copy of `mozglue.dll` from inside the sandbox. When `ProtectHostImages=y` is enabled, Sandboxie blocks host-installed programs from loading DLLs from the sandbox.


This issue is commonly triggered by running **Firefox updates inside the sandbox**. Updates (or repair operations) can create/modify boxed copies under:

- `C:\Sandbox\denni\Browser_Firefox\drive\C\Program Files\Mozilla Firefox\`

After that, Firefox may try to load those boxed binaries and hit `SBIE1305`.

### Verified — Option 1: Delete boxed Mozilla Firefox folder (quick fix)

1) Delete the boxed Mozilla Firefox folder inside the sandbox:

- `C:\Sandbox\denni\Browser__Firefox\drive\C\Program Files\Mozilla Firefox\`

2) Start Firefox in the sandbox again.

This reliably fixes the `SBIE1305` startup block.

**Why this is not the right long-term approach:** once you start touching updates while Firefox is running under Sandboxie, you can run into update-related warnings like:

```text
firefox.exe: SBIE2191 Firefox should not be updated when it is run with Sandboxie.
firefox.exe: SBIE2192 Do not update the program under the control of Sandboxie.
```

### Recommended — Option 2 (architecture-correct)

Delete the **entire** box content and then start Firefox again.

⚠️ If you rely on persistent data in this sandbox, you MUST back it up first and then copy it back to the correct sandbox paths after recreating the box.







