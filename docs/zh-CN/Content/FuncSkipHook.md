# 跳过函数钩子的功能

`FuncSkipHook` 是 [沙盘配置](SandboxieIni.md) 中的一个沙箱设置，指示沙盘不要尝试钩挂特定的导出 API 函数

当对某个函数进行钩挂尝试会导致系统不稳定、与其他安全产品不兼容或产生不期望的行为时，此设置非常有用

## 语法

```ini
[DefaultBox]

FuncSkipHook=FunctionNameA
FuncSkipHook=FunctionNameB
```

注意事项：

- 每一个 `FuncSkipHook` 项目都会与被考虑钩挂的函数名开头进行比较；若匹配，则跳过该钩挂

## 用法

通常用于模板中，以避免对特定应用程序或钩子标识符进行钩挂

示例：

```ini
[DefaultBox]

FuncSkipHook=SomeVendor_BrokenCall
FuncSkipHook=PStoreCreateInstance
```

## 行为说明

- 辅助函数 `SbieDll_FuncSkipHook` 会查询配置中的 `FuncSkipHook` 项目。对于每一个配置项，它会将配置的宽字符字符串与被钩挂的 ASCII 函数名进行前缀比较。如果在匹配函数名的起始字符时，配置字符串已经耗尽，则视为匹配，跳过钩挂
- 为了提升效率，如果第一次查询时未找到任何 `FuncSkipHook` 项目，则后续调用会直接跳过此项检查

## 技术说明

- 代码通过 `SbieApi_QueryConfAsIs` 循环读取配置项，直到查询失败（或返回不同错误）为止。每一个返回的 `WCHAR` 缓冲区都会与函数名逐字符比较，两个指针同步推进；当配置的宽字符串结束时而函数名还剩字符，则判定为匹配并跳过钩挂
- 此比较实质上是区分大小写的前缀匹配（直接并行比较每个字符）。如果你依赖于不区分大小写的匹配，请注意 `FuncSkipHook` 的行为与 `SkipHook` 有所不同

## 图形界面

`FuncSkipHook` 属于高级设置，通常在 INI 文件中手动编辑。沙盘自带的模板文件可能包含示例性的 `FuncSkipHook` 配置项

## 相关设置

- [SkipHook](SkipHook.md) —— 按模块或钩子标识符跳过钩挂的设置

## 注释

[^1]：实现位于 `SbieDll_FuncSkipHook`（参见 `dllhook.c`）。该函数通过 `SbieApi_QueryConfAsIs` 查询 `FuncSkipHook` 条目，并将配置的宽字符字符串与函数名逐字符进行比较；如果配置的字符串被耗尽，则表示匹配成功并跳过挂钩。如果没有找到任何条目，函数会设置一个内部标志，以提升效率并禁止后续的检查

