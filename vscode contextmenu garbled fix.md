# 修复 Windows 右键菜单中 VS Code 显示乱码的问题

## 来源说明

本文解决方法来源于 **CSDN 用户 Sponge_F** 的分享，本文由 **ChatGPT** 结合实际排查过程进行整理、改写与步骤化汇总。

原文：[text](https://ask.csdn.net/questions/9425906)

---

## 问题现象

在 Windows 文件资源管理器中右键文件、文件夹或空白区域时，VS Code 的右键菜单项出现乱码。

正常情况下，该菜单项应显示为：

```text
通过 Code 打开
```

或：

```text
Open with Code
```

但实际显示为一串乱码字符。

该问题一般不影响 VS Code 本体运行，只是右键菜单显示异常。

---

## 适用环境

该方法主要适用于以下情况：

* Windows 10 / Windows 11；
* VS Code 右键菜单显示乱码；
* 卸载或重装 VS Code 后问题仍然存在；
* 右键菜单中的其他软件基本正常，主要是 VS Code 菜单项异常。

---

## 已验证的解决方法

### 方法一：删除异常的 `VSCodeContextMenu` 注册表项

这是最直接、最有效的方法。

实际测试中，重装 VS Code 无法解决该问题，但删除以下注册表项后，右键菜单恢复正常：

```text
HKEY_LOCAL_MACHINE\SOFTWARE\Classes\VSCodeContextMenu
```

---

## 操作步骤

### 1. 打开注册表编辑器

按下：

```text
Win + R
```

输入：

```text
regedit
```

然后回车。

---

### 2. 定位到问题注册表项

在注册表编辑器顶部地址栏输入：

```text
计算机\HKEY_LOCAL_MACHINE\SOFTWARE\Classes\VSCodeContextMenu
```

然后回车。

需要找到的是左侧树状目录中名为：

```text
VSCodeContextMenu
```

的注册表项。

---

### 3. 建议先备份

在左侧右键点击 `VSCodeContextMenu`，选择：

```text
导出
```

可以保存到桌面，例如命名为：

```text
VSCodeContextMenu_backup.reg
```

如果后续出现异常，可以双击该 `.reg` 文件恢复。

---

### 4. 删除该项

右键点击左侧的：

```text
VSCodeContextMenu
```

选择：

```text
删除
```

注意：只删除这一项，不要删除整个 `Classes` 目录。

需要删除的完整路径为：

```text
HKEY_LOCAL_MACHINE\SOFTWARE\Classes\VSCodeContextMenu
```

---

### 5. 重启 Windows 资源管理器

按下：

```text
Ctrl + Shift + Esc
```

打开任务管理器。

找到：

```text
Windows 资源管理器
```

右键选择：

```text
重新启动
```

然后重新右键文件或文件夹，VS Code 菜单项通常就会恢复正常。

---

## 方法二：不删除注册表项，手动添加 `Title` 值

如果不想删除 `VSCodeContextMenu` 项，也可以尝试手动补全它的标题值。

### 操作步骤

进入：

```text
计算机\HKEY_LOCAL_MACHINE\SOFTWARE\Classes\VSCodeContextMenu
```

在右侧空白区域右键：

```text
新建 → 可扩充字符串值
```

名称填写：

```text
Title
```

双击该值，数据填写：

```text
通过 Code 打开
```

然后重启 Windows 资源管理器。

不过，如果该项本身是异常残留项，更推荐使用方法一删除该项，让系统重新读取正常配置。

---

## 推荐处理顺序

建议按以下顺序排查：

1. 先尝试重装 VS Code，并在安装时勾选右键菜单相关选项；
2. 如果重装无效，进入注册表检查：

```text
HKEY_LOCAL_MACHINE\SOFTWARE\Classes\VSCodeContextMenu
```

3. 如果存在该项，先导出备份；
4. 删除 `VSCodeContextMenu`；
5. 重启 Windows 资源管理器；
6. 检查右键菜单是否恢复正常。

实际测试中，重装 VS Code 无法解决时，删除该注册表项后问题恢复。

---

## 可能原因分析

该问题可能不是 VS Code 本体损坏，而是 Windows 注册表中出现了异常的右键菜单配置项。

推测原因如下：

某些卸载清理工具可能会在：

```text
HKEY_LOCAL_MACHINE\SOFTWARE\Classes
```

中创建大量空注册表项。

其中，如果出现一个空的：

```text
HKEY_LOCAL_MACHINE\SOFTWARE\Classes\VSCodeContextMenu
```

Windows 在读取 VS Code 右键菜单时，可能会优先读取该空项，而不是读取当前用户路径下正常的 VS Code 配置项。

正常的 VS Code 右键菜单配置通常可能位于：

```text
HKEY_CURRENT_USER\Software\Classes
```

下。

但在该问题中，`HKLM\SOFTWARE\Classes\VSCodeContextMenu` 的优先级似乎更高，导致系统读取到异常或空的菜单配置，从而出现乱码。

---

## 为什么重装 VS Code 可能无效？

VS Code 重装时通常会重新写入自己的右键菜单项，但如果：

```text
HKEY_LOCAL_MACHINE\SOFTWARE\Classes\VSCodeContextMenu
```

这个异常项仍然存在，系统可能依然优先读取它。

因此，即使卸载并重新安装 VS Code，右键菜单乱码仍可能继续存在。

这也是为什么清理该注册表项后，问题可以恢复。

---

## 可选：使用命令行删除

如果熟悉命令行，也可以使用管理员 PowerShell 或管理员 CMD 执行：

```powershell
reg delete "HKLM\SOFTWARE\Classes\VSCodeContextMenu" /f
```

然后重启资源管理器：

```powershell
taskkill /f /im explorer.exe
start explorer.exe
```

---

## 注意事项

1. 修改注册表前建议先备份相关项。
2. 不要删除整个 `Classes` 目录。
3. 如果只是 VS Code 右键菜单乱码，优先只处理：

```text
HKEY_LOCAL_MACHINE\SOFTWARE\Classes\VSCodeContextMenu
```

4. 不建议一开始就运行批量注册表清理脚本。
5. 如果使用 HiBit Uninstaller、Revo Uninstaller、Geek Uninstaller 等卸载清理工具，清理注册表时应谨慎，避免误删或创建异常空项。

---

## 总结

如果 Windows 右键菜单中 VS Code 显示乱码，并且重装 VS Code 无效，可以优先检查并删除：

```text
HKEY_LOCAL_MACHINE\SOFTWARE\Classes\VSCodeContextMenu
```

删除后重启 Windows 资源管理器，通常即可恢复正常。

---

## 致谢

感谢 **CSDN 用户 Sponge_F** 提供的解决思路和问题原因分析。本文由 **ChatGPT** 根据该解决方法及实际处理流程进行整理汇总，便于在 GitHub 上记录和分享。

