# Angular

## Troubleshooting

### Start with specific node.exe

```shell
cd apps/frontend
..\..\node_modules\node\node.exe .\node_modules\@angular\cli\bin\ng.js serve --proxy-config .\proxy.conf.json --host 0.0.0.0
```
- Ohne --open Wir brauchen den Browser nicht, und es gibt auch Komplikationen mit Sandboxie.