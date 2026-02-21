# Force Box Docs

_ForceBoxDocs_ is a global setting in [Sandboxie Ini](SandboxieIni.md) (introduced in v1.17.0 / 5.72.0). When enabled, if a program is started outside Sandboxie with a file path argument and that file is located in a sandbox, Sandboxie automatically starts the program in that same sandbox.

Usage:

```ini
   [GlobalSettings]
   ForceBoxDocs=y
```

How this setting is evaluated:

- It applies only when a process starts unsandboxed; if the process is already started sandboxed, this setting is not used.
- It checks the parsed file path/document argument from the command line.
- Sandboxie ignores leading command-line switches (options starting with `-` or `/`, such as `/n` or `-Embedding`) to correctly identify the target file path.
- The file path/document argument check is skipped when the process being evaluated is Sandboxie's own `Start.exe`.
- Only boxes enabled for the current user/session are considered.
- Boxes with [DisableForceRules](DisableForceRules.md) are skipped.

This setting is useful when users open documents directly from boxed paths and you want the associated application to automatically start in the correct sandbox.

Command-line examples:

```text
"C:\Program Files\Microsoft Office\root\Office16\WINWORD.EXE" /n /dde "C:\Sandbox\Alice\DefaultBox\drive\C\Docs\Report.docx"
```

Result: the file path points to `DefaultBox`, so the process is forced into `DefaultBox`.

```text
"C:\Windows\System32\notepad.exe" "C:\Users\Alice\Desktop\Notes.txt"
```

Result: the file path is not inside a sandbox path, so `ForceBoxDocs` does not force the process.

See also:

- [ForceFolder](ForceFolder.md)
- [ForceProcess](ForceProcess.md)
