# 强制子进程

`ForceChildren` 是 [沙盘配置](SandboxieIni.md) 中的一个沙箱设置（在 v1.14.5 / 5.69.5 版本中引入），可强制特定父进程启动的子进程在指定沙箱内运行。当你希望由已知父进程启动的所有进程都自动在沙箱中运行时，这项功能十分有用。

## 语法

通用格式如下：

```ini
[DefaultBox]

ForceChildren=<program>
```

其中 `<program>` 可以是：

- 带有通配符的映像路径，例如 `C:\Users\*\App\*.exe` —— 通配符模式会对规范化路径进行不区分大小写的匹配
- 映像名称（不带反斜杠），例如 `dopus.exe` —— 按照驱动程序的映像名匹配规则进行名称匹配

注意事项：

- `ForceChildren` 中的路径会在匹配前进行规范化和标准化（会移除重复的反斜杠并转换重解析点）[^1]
- 当使用通配符模式时，会创建一个模式匹配器，并对转换后的路径进行不区分大小写的匹配[^2]

## 行为及匹配规则

- `ForceChildren` 项在新进程创建时进行评估。如果父进程匹配到了任何已启用沙箱的 `ForceChildren` 规则，则其子进程将强制进入该沙箱[^3]

- 匹配尝试（大致优先级顺序）：

    - 父进程的映像名称会与所有为名称或名称模式的 `ForceChildren` 项进行比较[^4]
    - 父进程规范化后的映像路径和/或工作目录，可能会与 `ForceChildren` 项中包含 `*` 的通配符路径模式进行比较[^5]

- 包含通配符的规则使用模式引擎进行匹配；不含通配符的规则使用兼容 NLS 的字符串比较及映像匹配助手函数，以实现正确的语言感知检查[^2][^4]

- 沙箱是否启用、是否允许强制规则等设置将影响 `ForceChildren` 匹配的有效性。如果某个沙箱配置了类似 `DisableForceRules` 等选项，则该沙箱的强制规则不会生效[^6]

- 某些系统进程及沙盘内部流程被排除在强制之外。沙盘的主目录（home 目录）也不会被强制处理[^7]

## 示例

1. 强制特定父进程名称的子进程在沙箱中运行：

    ```ini
    [MyBox]
    
    ForceChildren=parentapp.exe
    ```

2. 使用通配符模式匹配父进程映像路径：

    ```ini
    [MyBox]
    
    ForceChildren=C:\Users\*\Downloads\*\app.exe
    ```

## 命令行开关

参见 [StartCommandLine](StartCommandLine.md#force_children-or-fcp) 以获取详细信息

### 相关开关：

- `/force_children` —— 启用或控制命令行下的子进程强制行为
- `/fcp` —— `/force_children` 的简写版本（如有支持）

## 与提示和其他强制设置的交互

- 当驱动程序决定新启动进程应使用哪个沙箱（如果有）时，`ForceChildren` 项会与 `ForceFolder` 和 `ForceProcess` 条目一同评估

- 如果在匹配规则时启用了告警（例如存在策略限制强制规则的情况），驱动程序可能会弹出提示，而不是强制进程进入沙箱。驱动程序会跟踪“告警”状态，此状态可阻止强制，并可选择记录日志或通知用户[^8]

- `ForceChildren` 用于补充 `ForceProcess`（直接按名称强制进程，不考虑父进程）和 `ForceFolder`（按文件夹路径强制进程）。若希望强制由特定父进程创建的进程，应选用 `ForceChildren`

## 实现说明（驱动行为）

- 驱动程序在进程创建时为每个已启用沙箱构建一份内存中的强制规则列表。每个 `ForceChildren` 条目会被转换为内部 `FORCE_ENTRY` 元素，若包含通配符则存储为编译后的 PATTERN，对应规范化路径[^2]

- 映像名称匹配由辅助函数 `Process_MatchImage` 实现，支持语言敏感及映像名称专用的匹配逻辑[^4]

- 代码确保移除重复反斜杠并解析重解析点；尽可能转换为 NT 风格路径。如果无法转换，则使用原始路径进行匹配[^1][^5]

## 参见

- [ForceProcess](ForceProcess.md) —— 按名称或路径强制进程
- [ForceFolder](ForceFolder.md) —— 按文件夹路径或模式强制进程
- [DisableForceRules](DisableForceRules.md) —— 禁用某个沙箱的所有强制规则

## 图形界面（沙盘管理器 / SandMan）

### 位置：

- 打开沙盘管理器（SandMan）
- 打开沙箱的选项：右键单击某个沙箱并选择“沙箱选项”
- 在选项窗口中找到“程序控制 / 强制程序” 区域，可以看到所有已添加的强制项

### 如何添加“强制子进程”条目：

1. 在“程序控制 / 强制程序”区域，你会看到已强制的条目列表，以及 “强制程序”、“强制子进程” 和 “强制文件夹” 按钮
2. 单击 “强制子进程”，可通过选择可执行文件添加一个条目（或者点击“浏览”按钮通过对话框选择文件）
3. 添加的项目会以类型标签“Children”展现在列表中。已勾选的条目会保存到 `ForceChildren`；未勾选的保存到 `ForceChildrenDisabled`
4. 若需移除条目，选中后点击“移除”即可

### 说明与参考

- 界面中的树形组件是 `ui.treeForced`，位于 `OptionsWindow.ui`。选项窗口通过 `OptionsForce.cpp` （LoadForced/SaveForced/AddForcedEntry）加载和保存这些列表
- “为此沙箱禁用强制进程和强制文件夹”复选框，即 `ui.chkDisableForced`，映射到 `DisableForceRules` 设置

## 注释

[^1]: 驱动程序会在存储条目前移除重复的反斜杠并解析重解析点。参见 `Process_AddForceFolders` 对 `expnd` 进行标准化并调用 `File_TranslateReparsePoints`
[^2]: 通配符项通过 `Pattern_Create` 创建 PATTERN 对象，使用 `Pattern_Match` 对规范化为小写的路径进行匹配
[^3]: `Process_FcpInsert`、`Process_FcpCheck` 及相关过程实现了基于父进程的强制映射，便于在创建新子进程时将其加入以 PID 为键的映射，并在后续创建时检查
[^4]: 映像名匹配委托给 `Process_MatchImage`（当条目不含反斜杠时由驱动调用）
[^5]: 驱动通过 `Process_TranslateDosToNt`（封装自 `File_TranslateDosToNt`）将 DOS 风格路径转换为 NT 风格，若语法错误则回退使用原始路径
[^6]: 只有为当前 SID/会话已启用、且未启用 `DisableForceRules` 的沙箱才会加载强制数据列表
[^7]: 沙盘 home 目录下的路径不会被强制处理，代码会显式检测并跳过位于 `Driver_HomePathNt` 下的匹配项
[^8]: 匹配到需告警的规则时，驱动可能会设置告警状态（`IsAlert`），并延迟强制操作。告警可能产生日志记录，并可根据设置阻止进程在沙箱内启动