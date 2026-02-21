## Sandboxie + PowerShell (boxed): `ReadConsoleOutput` “Access denied (0x5)” / broken STDOUT

### Scope / applies to
- **Applies to**: PowerShell (Windows PowerShell 5.1 and/or PowerShell 7) and other **console apps** running **inside Sandboxie**.
- **Not Electron-specific**: Electron was only the trigger for needing a boxed terminal with visible logs.

### Symptoms
- Running a boxed PowerShell script (often via `pnpm`, `npm`, `node`, `cross-env`, etc.) fails with errors like:
  - **`Internal Win32 error "Access is denied" 0x5 when reading the console output buffer`**
  - Errors mentioning **`ReadConsoleOutput`**, **`Out-LineOutput`**, `Write-Progress`, or progress rendering.
- Typical outcome:
  - Commands abort (`ELIFECYCLE`, exit code 1), and the dev chain stops.
- Related symptom:
  - **STDOUT logs “disappear”** when starting a boxed app from a host terminal (process is not a child of that terminal / no STDIO inheritance).

### Root cause (Windows integrity boundary)
- Windows console apps talk to **Console Window Host (`conhost.exe`)**.
- In Sandboxie, `conhost.exe` for a boxed console session can run **outside the sandbox** at a **higher integrity level** than the boxed process.
- Windows blocks certain console operations (“wrong-way verbs”), including **reading the output buffer** (`ReadConsoleOutput`), when the caller has a **lower integrity level** than `conhost.exe`.
- PowerShell triggers this often (progress UI / `Out-LineOutput`), so you see it quickly.

### Fix (recommended): Drop console host integrity for boxed consoles
Enable the Sandboxie setting:

- **`DropConHostIntegrity=y`** (per sandbox/box)
- Oder per UI Sicherheitsoptionen -> Erweiterte Sicherheit -> "Verwerfe den Prozessintegriotätslevel von Con host.exe"

This makes Sandboxie lower the integrity level of the related `conhost.exe` instance(s) so boxed PowerShell can safely perform console buffer operations.

#### How to apply (SandMan GUI → config editor)
1) Open **SandMan**.
2) Go to **Configure → Edit Configuration**.
3) In your sandbox section (e.g. `[DefaultBox]`), add:

   `DropConHostIntegrity=y`

4) **Terminate all programs** in that sandbox (important).
5) Start a new boxed PowerShell / VS Code integrated terminal session and retry.

> Note: Sandboxie config changes do **not** apply to processes already running.

### Verification (evidence)
Run one of these inside the boxed PowerShell session:

- `Write-Progress -Activity Test`
- Or re-run the previously failing command chain (e.g. `pnpm run dev:...`)

**Expected result**:
- No `ReadConsoleOutput` / `Out-LineOutput` access denied errors.
- Dev chain continues and logs appear.

### Rollback
To revert, remove the line:

- `DropConHostIntegrity=y`

Terminate boxed processes and restart.

### Security impact / risk assessment (Enterprise view)
- **What changes**:
  - Sandboxie adjusts the integrity level of the **console host (`conhost.exe`)** used by boxed console sessions.
- **Trust boundary**:
  - `conhost.exe` remains an OS component; this setting mainly restores compatibility for boxed console IO operations.
- **Blast radius**:
  - **Per sandbox/box**. It affects console hosts associated with boxed console programs in that box.
- **Risks**:
  - You are intentionally changing an OS-enforced integrity relationship to allow operations that Windows would otherwise block.
  - Practical risk is usually limited to console IO behavior for that boxed session, but it is still a **control change**.
- **Mitigations**:
  - Apply **only** in the sandbox(es) used for development / boxed terminals.
  - Keep other isolation controls intact (Drop Admin Rights, start/run restrictions, network rules, etc.).

### Notes for VS Code integrated terminal + Sandboxie
- If you need logs in the **integrated terminal**, the **shell process that VS Code starts** must run in the sandbox context where your toolchain runs.
- If you run the toolchain boxed but the terminal host is not in the same context, STDOUT inheritance can break.
- A stable pattern is to use a **dedicated terminal binary** for development (a copy like `pwsh-devbox.exe`) and force **only that** into the sandbox, instead of forcing `powershell.exe` globally.

### References
- Sandboxie issue/discussion for the underlying console buffer problem and the setting:
  - GitHub: [Sandboxed console programs cannot read output buffer (#678)](https://github.com/sandboxie-plus/Sandboxie/issues/678)
- Windows background on Job Objects / console behavior:
  - Microsoft Learn: [Job objects](https://learn.microsoft.com/en-us/windows/win32/procthread/job-objects)
- Sandboxie Start.exe behavior (helpful when reasoning about STDIO inheritance):
  - Sandboxie docs: [Start Command Line](https://sandboxie-plus.github.io/sandboxie-docs/Content/StartCommandLine.html)
