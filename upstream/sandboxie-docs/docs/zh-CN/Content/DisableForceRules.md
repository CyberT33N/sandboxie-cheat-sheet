# 禁用强制规则

`DisableForceRules` 是一个针对每个沙箱的设置（在 v1.9.1 / 5.64.1 中引入），用于禁用该沙箱内的所有“强制”规则。启用后，驱动程序将跳过处理该沙箱配置中的 `ForceFolder`、`ForceProcess` 和 `ForceChildren` 项，从而有效阻止基于这些规则的自动沙箱强制操作。

此设置主要适用于希望沙箱存在但不参与自动强制决策的场景（例如，仅用于手动运行或特殊情况操作的沙箱）。

## 语法

在 Sandboxie 配置文件的某个沙箱段落下设置此项：

```ini
[DefaultBox]

DisableForceRules=<y|n>
```

有效值：

- `y` — 启用：忽略该沙箱的强制规则。
- `n` — 禁用（默认）：在构建强制列表时评估该沙箱的强制规则。

## 行为

- 当驱动程序遍历配置段落以构建内存中的强制规则时，会调用 `Conf_IsBoxEnabled` 并检查 `DisableForceRules`。如果某个沙箱设置了 `DisableForceRules`，则该沙箱在强制规则处理时会被跳过，其 `ForceFolder`、`ForceProcess` 和 `ForceChildren` 项不会被使用[^1]

- 影响沙箱是否被考虑的其他设置还包括针对当前 SID/会话的沙箱启用情况——如果沙箱未针对当前用户/会话启用，则在创建强制列表时不会被考虑[^2]

- `DisableForceRules` 不会移除沙箱或改变其他非强制相关行为；它只阻止驱动程序将该沙箱的强制项添加到运行时强制列表中。

## 示例

为名为 `NoAutoForce` 的沙箱禁用强制规则：

```ini
[NoAutoForce]

DisableForceRules=y
```

保持强制规则启用（默认）：

```ini
[MyBox]

DisableForceRules=n
```

## 图形界面（Sandboxie Manager / SandMan）

通过 Sandboxie Manager（SandMan）界面可以在以下位置切换“禁用强制规则”：

- 上下文菜单 -> Sandbox Presets -> Disable Force Rules
	- 右键点击沙箱列表中的某个沙箱，打开“Sandbox Presets”子菜单，切换“Disable Force Rules”项
- 沙箱选项 -> Force 标签页
	- 右键点击沙箱，选择“Sandbox Options”，切换至“Program Control”下的“Force Programs”标签，勾选/取消勾选“Disable forced Process and Folder for this sandbox”复选框，然后保存

这些界面操作直接对应 `DisableForceRules` 沙箱设置，并会立即更新所选沙箱的该项配置[^4][^5]

## 实现说明（驱动程序行为）

- 驱动程序调用 `Process_CreateForceData` 为每个启用的沙箱构建内存中的 `FORCE_BOX` 列表。在创建过程中遍历配置段落，并跳过任何 `Conf_Get_Boolean(section, L"DisableForceRules", 0, FALSE)` 返回为真（true）的沙箱[^1]

- 在 `Process_CreateForceData` 中跳过沙箱意味着该沙箱的 `ForceFolder`、`ForceProcess` 和 `ForceChildren` 列表不会被添加到运行时 `boxes` 列表中，该列表被 `Process_GetForcedStartBox` 及相关检查使用。

## 相关内容

- [ForceProcess](ForceProcess.md) — 按名称或路径强制进程进入沙箱
- [ForceFolder](ForceFolder.md) — 按文件夹路径或模式强制进程进入沙箱
- [ForceChildren](ForceChildren.md) — 基于父进程的强制规则（匹配父进程的子进程将被强制沙箱化）

## 注释

[^1]: 可参考 `Process_CreateForceData` 如何遍历所有配置段落和 `if (Conf_Get_Boolean(section, L"DisableForceRules", 0, FALSE)) continue;` 跳过已启用此设置的沙箱

[^2]: `Conf_IsBoxEnabled` 用于判断目标沙箱在当前 SID/会话是否激活，然后才会为其构建相关强制列表

[^3]: 驱动程序在决定进程是否可以被强制时会检查全局/会话级标志，如 `AllowForceSystem` 及会话级强制禁用标志；`DisableForceRules` 仅影响每个沙箱是否被包含进强制列表

[^4]: 预设菜单切换功能由 SandMan 的 `SbieView.cpp` (`m_pMenuPresetsForce`) 实现，并通过 `SetBoolSafe("DisableForceRules", ...)` 修改设置

[^5]: 沙箱选项对话框读写复选框 `ui.chkDisableForced`，在保存选项时将 `DisableForceRules` 设置持久化