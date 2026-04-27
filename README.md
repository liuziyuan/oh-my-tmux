# oh-my-tmux

一套由 AI 自动写入的 tmux 配置，涵盖快捷键、Pane 边框、状态栏配色，开箱即用。

## 前提条件

- tmux >= 3.1

## AI 安装

将以下提示词发给任意 AI（Claude、GPT 等），AI 会自动读取文档并逐步完成配置写入：

```text
阅读文档，并按照文档指引执行：https://github.com/liuziyuan/oh-my-tmux/blob/main/AI_INSTALL.md
```

安装过程分为以下步骤：

1. **备份**：自动备份现有 `~/.tmux.conf`
2. **写入配置**：基础设置、Pane 边框、快捷键、状态栏配色（AI 会询问偏好）
3. **插件管理**：自动安装 TPM 与会话持久化插件（tmux-resurrect / tmux-continuum）
4. **终端适配**：自动检测 Ghostty，按需写入兼容配置
5. **验证**：检查配置语法，列出已绑定的快捷键和已安装插件

---

## 配置内容

### 快捷键

Prefix 为 `Ctrl+a`（按下后松开，再按对应键）。

| 快捷键 | 功能 |
|--------|------|
| `prefix c` | 创建新 window |
| `prefix n` | 下一个 window |
| `prefix p` | 上一个 window |
| `prefix X` | 关闭当前 window |
| `prefix d` | detach 当前 session |
| `prefix \|` | 左右分屏 |
| `prefix -` | 上下分屏 |
| `prefix x` | 关闭当前 pane |
| `prefix h/j/k/l` | 切换 pane（vim 风格） |

### Pane 边框

开启边框状态栏（显示在顶部），AI 安装时会询问显示内容：

| 选项 | 效果示例 |
|------|---------|
| 编号 | ` 1 ` |
| 编号 + 标题 | ` 1: zsh ` |
| 编号 + 当前命令 | ` 1: vim ` |
| 编号 + 标题 + 当前命令 | ` 1: zsh [vim] ` |

### 状态栏配色

AI 安装时会询问配色方案，内置 5 种预设：

| 方案 | 风格 |
|------|------|
| **A. Catppuccin Mocha**（默认） | 柔和马卡龙风 |
| **B. Dracula** | 暗夜吸血鬼风 |
| **C. Tokyo Night** | 东京夜色风 |
| **D. Gruvbox Dark** | 复古暖色风 |
| **E. Nord** | 北极冰蓝风 |

### 会话持久化

通过 [tmux-resurrect](https://github.com/tmux-plugins/tmux-resurrect) +
[tmux-continuum](https://github.com/tmux-plugins/tmux-continuum) 实现：

- **自动保存**：每 15 分钟保存一次（含 pane 布局、运行命令、输出内容）
- **启动恢复**：tmux 启动时自动恢复上次会话
- **手动控制**：`prefix + Ctrl-s` 保存，`prefix + Ctrl-r` 恢复

首次安装由 AI 自动完成 TPM 与插件下载；后续如手动声明新插件，按 `prefix + I` 安装。

---

## 终端适配

### Ghostty

Ghostty 默认会拦截第一次鼠标点击，导致 tmux 鼠标单击无法切换 pane。

AI 安装时会自动检测是否使用 Ghostty，并询问是否写入以下配置（`~/.config/ghostty/config`）：

```
focus-follows-mouse = true
```

写入后需重启 Ghostty 生效。
