## Reference Report — Cursor/VS Code + Sandboxie (Privacy Mode) + `Debug Script` vs. Attach-Inspect (Electron-Vite / pnpm)

### Objective & Constraints

* **Objective**: Develop and debug Electron-Vite via pnpm from within a **Sandboxie Privacy Box** (strongly isolated) using VS Code/Cursor.
* **Key constraint**: **Cursor/VS Code remains on the host (unboxed)**; only the **dev terminal / dev processes** run boxed.
* **Observation**:

  * `Run Script` (from `package.json`) **works**
  * `Debug Script` opens a **boxed debug terminal**, but the **process does not start cleanly / does not continue running**

---

## 1) Why `Run Script` Works but `Debug Script` Does Not

### 1.1 `Run Script` = Task Execution

* VS Code starts the script as a **task** (shell/command runner).
* This works reliably inside the sandbox terminal because it simply launches the runner (`pnpm`, `npm`, `node`).

---

### 1.2 `Debug Script` = JavaScript Debug Terminal (js-debug) + Bootloader + Named Pipe

When clicking **`Debug Script`**, VS Code (js-debug) performs additional fundamental steps:

* It sets `NODE_OPTIONS` and injects a preload:

  * Example from your debug terminal:

```text
NODE_OPTIONS=
 --require c:/Users/denni/AppData/Local/Programs/cursor/resources/app/extensions/ms-vscode.js-debug/src/bootloader.js  --inspect-publish-uid=http
```

* It sets `VSCODE_INSPECTOR_OPTIONS` with an **IPC pipe**:

  * Example (shortened):

```text
{"inspectorIpc":"\\.\pipe\node-cdp.37208-....sock", ...}
```

**Core statement**:
`Debug Script` depends on the sandboxed Node process being able to:

1. Load the bootloader
2. Connect to the **named pipe `\\.\pipe\node-cdp.*`**

---

## 2) Findings from the Tests (Evidence-Based)

### 2.1 Phase A — Bootloader Path Was “Non-Existent” in Privacy Mode (ENOENT)

**Symptom** when executing `pnpm` in the debug terminal:

* Node terminated with an error before `pnpm` could run:

  * `ENOENT ... lstat 'c:\Users\denni\AppData\Local\Programs\cursor'`

**Insight**

* In Privacy Mode, the Cursor installation path was treated as **non-existent** for boxed processes.

**Fix that resolved this part**

* Added `ReadFilePath` access for the Cursor root (for `node.exe`/`electron.exe`), making the bootloader path resolvable.

**Validation**

* After this change, `pnpm -v` ran successfully in the debug terminal (`10.18.2`).

---

### 2.2 Phase B — The Real Blocker: `node-cdp.*` Pipe Not Visible/Connectable Inside the Box

The pipe mechanism was tested directly (mirroring js-debug behavior):

* `readdirOk=true` (Node can list `\\.\pipe\`)
* but the specific pipe is missing:

  * `includesPipe=false`
  * `connectOk=false`, **`ENOENT`** on connect

Example from your pipe check:

```text
pipe= \\.\pipe\node-cdp.37208-ecefd63c-9.sock
readdirOk= true
includesPipe= false
connectOk= false
error= ENOENT connect ENOENT \\.\pipe\node-cdp.37208-ecefd63c-9.sock
```

**Insight**

* The js-debug pipe provided by host VS Code is **not reachable within the same NT object namespace** from inside the box.
* This is no longer a pattern issue. Tested configurations included:

  * `OpenPipePath=node.exe,\Device\NamedPipe\node-cdp*`
  * `OpenFilePath=node.exe,\Device\NamedPipe\node-cdp*`
  * `OpenIpcPath=node.exe,\Device\NamedPipe\node-cdp*`
  * extremely broad: `OpenPipePath=node.exe,\Device\NamedPipe\`
  * isolation relaxation tests:

    * `UseSecurityMode=n`
    * `HideNonSystemProcesses=n`
    * `NtNamespaceIsolation=n`
    * `Template=BlockLocalConnect` disabled

**Result**:
The pipe remained “non-existent” inside the box (`ENOENT`).

---

## 3) Why It Fails When Cursor/VS Code Remains on the Host

**Short version**

* `Debug Script` requires a **bidirectional IPC connection** between:

  * **Host** (Cursor/VS Code js-debug) = pipe server `node-cdp.*`
  * **Box** (Node/pnpm/electron-vite) = pipe client

**Why this fails under your isolation model**

* Sandboxie isolates/virtualizes the **NT object namespace** (including named pipes).
* Even with explicit allowances, they only work if the object is visible within the box context.
  In your case, the concrete `node-cdp.*` pipe is not visible → `ENOENT`.

**Conclusion**

* As long as Cursor/VS Code runs unboxed and you use a **Privacy Box with strong namespace isolation**, the VS Code `Debug Script` mechanism cannot be made reliably functional.

---

## 4) Technical Core of the Failure

The failure is not related to your project or pnpm, but to js-debug architecture:

* js-debug injects the bootloader via
  `NODE_OPTIONS=--require ...\bootloader.js`
* The bootloader expects a reachable `inspectorIpc`:

  * `\\.\pipe\node-cdp....sock`
* Inside your box, this pipe is not reachable → js-debug cannot attach → debug session fails early.

---

## 5) Conclusion Under Your Constraints

Given your chosen constraints:

* Cursor/VS Code remains on the host
* Only the dev terminal is boxed
* Privacy Mode / isolation stays active

The **only reliably working debugging method** is:

* **Attach-Inspect** (Node Inspector + Chrome DevTools Protocol attach)

This method does not require cross-boundary `node-cdp.*` named pipes.

---

## 6) Stable Path (Recommended): Use Your Existing “1-Click” Setup

* docs\applications\IDE\vscode\terminal\package.json vs new terminal.md
* docs\applications\IDE\vscode\terminal\debug\general.md

Electron example:

* docs\applications\programming-languages\node\frameworks\electron\electron-vite\debug.md

---

## 7) Executive Summary (Single Paragraph)

`Run Script` works because it executes as a simple task. `Debug Script` fails in your setup because VS Code/js-debug injects a bootloader via `NODE_OPTIONS` and requires a named pipe (`\\.\pipe\node-cdp.*`) to the debugger instance running on the host. Inside your Privacy Box, this pipe remains invisible in the box context (confirmed by `includesPipe=false` and `connect ENOENT`, despite broad pipe allowances and disabled security features). Therefore, under your host/box separation model, the only reliable debugging method is **Attach-Inspect** (Inspector port + Chrome attach).
