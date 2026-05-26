# Angular

## Troubleshooting

## Architectural status

The command example below comes from an older host-shaped dependency layout and should now be treated as a **legacy troubleshooting reference**, not as the primary recommendation for the current install-box materialization architecture.

Recommended architecture:

- `docs\applications\IDE\vscode\methods\host-not-isolated\dependencies-installed-in-box.md`
- `docs\applications\IDE\vscode\methods\host-not-isolated\templates\node-monorepo-materialized-dependencies.md`

### Start with specific node.exe

```shell
cd apps/frontend
..\..\node_modules\node\node.exe .\node_modules\@angular\cli\bin\ng.js serve --proxy-config .\proxy.conf.json --host 0.0.0.0
```
- Ohne --open Wir brauchen den Browser nicht, und es gibt auch Komplikationen mit Sandboxie.