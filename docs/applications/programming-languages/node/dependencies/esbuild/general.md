# esbuild

## Architectural status

For the current recommended install-box / run-box architecture, `esbuild` is not treated as a broad host write exception.

Instead:

- the install box allows the `esbuild.exe` binary to initialize normally
- the run box allows only the smallest framework-owned cache/output surfaces that `esbuild` or Vite must write

## Why `esbuild.exe` needed an explicit rule

During boxed installs, `esbuild` runs a postinstall validation step that executes the platform-specific `esbuild.exe`.

Without an explicit visibility rule in a privacy box, the binary can fail during process initialization (`SBIE1231` / `SBIE2314`) even though the rest of the dependency tree is present.

Recommended install-box rule:

```ini
NormalFilePath=esbuild.exe,C:\git\test\test-mono\
```

The path should be scoped to the workspace that owns the `esbuild` packages.

## Vite / Electron-Vite cache note

In the run box, the relevant write surface is often not the `esbuild` package directory itself but the framework-managed Vite cache under:

```text
C:\git\test\test-mono\apps\desktop-app\node_modules\.vite\
```

That cache can stay tightly scoped:

```ini
OpenFilePath=node.exe,C:\git\test\test-mono\apps\desktop-app\node_modules\.vite\
```

Do **not** broadly open the entire package `node_modules` tree just because `esbuild` participates in the build.

## Related documents

- `docs\applications\IDE\vscode\methods\host-sync\templates\node-monorepo-materialized-dependencies.md`
- `docs\applications\programming-languages\node\dependencies\frameworks\electron\electron-vite\general.md`
- `docs\applications\programming-languages\node\dependencies\frameworks\angular\general.md`
- `docs\applications\programming-languages\node\dependencies\tsx\general.md`
