# AI Install Guide — tmux 配置

> 本文档供 AI 读取并执行，将 tmux 配置写入 `~/.tmux.conf`。

## 前提条件

- 目标文件：`~/.tmux.conf`
- 需要 tmux >= 3.1

---

## Step 1：备份

更新前**必须**备份当前配置文件。

```bash
cp ~/.tmux.conf ~/.tmux.conf.backup.$(date +%Y%m%d%H%M%S)
```

- 如果 `~/.tmux.conf` 不存在，跳过备份，直接进入 Step 2。

---

## Step 2：写入配置

按照下方「配置清单」，逐项检查 `~/.tmux.conf` 中是否已存在。

### 匹配与冲突处理规则

逐项检查 `~/.tmux.conf`，按以下优先级处理：

#### 规则 1：已存在相同配置 → 跳过（不修改）

如果目标文件中**已存在完全相同的指令**（指令部分一致，忽略行末注释），则**跳过该项，不做任何修改**。

判断方法：提取行内 tmux 指令部分（去掉 `# ...` 注释后），与清单中的指令比较。

```tmux
# 已存在以下行（无论注释内容是什么）：
set -g prefix C-a     # [AI_INSTALL] prefix 改为 Ctrl+a

# 清单项也是：
set -g prefix C-a     # [AI_INSTALL] prefix 改为 Ctrl+a

# → 指令部分一致，跳过，不写入
```

#### 规则 2：已存在但内容不同 → 注释旧项，写入新项

如果目标文件中存在**功能相同但指令不同**的配置（包括无 `[AI_INSTALL]` 标记的旧项），按以下步骤处理：

1. **保留旧项，注释掉**，在注释行末尾添加备注 `# [AI] 被 AI_INSTALL 替换于 YYYY-MM-DD`
2. **在旧项下方写入新项**，并在新项行末尾添加 `# [AI_INSTALL]`

示例：

```tmux
# set -g prefix C-b  # [AI] 被 AI_INSTALL 替换于 2026-04-11
set -g prefix C-a    # [AI_INSTALL]
```

对于**功能相同但写法不同**的项（如 `bind | split-window -h` vs `bind | split-window -h -c "#{pane_current_path}"`），也视为冲突，按规则 2 处理。

#### 规则 3：不存在 → 直接写入

目标文件中无任何相关配置时，直接写入新项，行末添加 `# [AI_INSTALL]`。

---

## 配置清单

### 2.1 基础设置

```tmux
# --- 基础设置 ---

set -g prefix C-a     # [AI_INSTALL] prefix 改为 Ctrl+a
unbind C-b            # [AI_INSTALL] 解绑默认 prefix

set -g mouse on       # [AI_INSTALL] 开启鼠标支持

set -g base-index 1   # [AI_INSTALL] window 编号从 1 开始
setw -g pane-base-index 1  # [AI_INSTALL] pane 编号从 1 开始
```

### 2.2 Pane 边框

```tmux
# --- Pane 边框 ---
set -g pane-border-lines single                   # [AI_INSTALL] 边框线型（single）
set -g pane-border-status top                     # [AI_INSTALL] 在边框顶部显示状态栏
```

**AI 执行时须询问用户**选择边框标题显示内容，根据选择写入对应的 `pane-border-format`：

| 选项 | 格式 | 效果示例 |
|---|---|---|
| 编号 (index) | `' #P '` | ` 1 ` |
| 编号 + 标题 | `' #P: #T '` | ` 1: zsh ` |
| 编号 + 当前命令 | `' #P: #{pane_current_command} '` | ` 1: vim ` |
| 编号 + 标题 + 当前命令 | `' #P: #T [#{pane_current_command}] '` | ` 1: zsh [vim] ` |

默认推荐：**编号 (index)**。

```tmux
# 默认（编号）：
set -g pane-border-format ' #P '                  # [AI_INSTALL] 边框状态栏格式
```

### 2.3 高频快捷键

```tmux
# --- 高频快捷键 ---

# Window 管理
bind c new-window          # [AI_INSTALL] 创建新 window
bind n next-window         # [AI_INSTALL] 下一个 window
bind p previous-window     # [AI_INSTALL] 上一个 window
bind d detach-client        # [AI_INSTALL] detach 当前 session
bind X kill-window          # [AI_INSTALL] 关闭当前 window

# 分屏
bind | split-window -h     # [AI_INSTALL] 左右分屏
bind - split-window -v     # [AI_INSTALL] 上下分屏

# Pane 操作
bind x kill-pane           # [AI_INSTALL] 关闭当前 pane

# Pane 切换（vim 风格）
bind h select-pane -L      # [AI_INSTALL] 切换到左侧 pane
bind j select-pane -D      # [AI_INSTALL] 切换到下方 pane
bind k select-pane -U      # [AI_INSTALL] 切换到上方 pane
bind l select-pane -R      # [AI_INSTALL] 切换到右侧 pane
```

### 2.4 状态栏配色

**AI 执行时须询问用户**选择配色方案，从下方 5 种预设中选择一种，或提供自定义色值。默认推荐：**Catppuccin Mocha**。

#### 颜色变量定义

以下变量在配色模板中引用，AI 根据用户选择的方案将具体 hex 色值填入模板。

| 变量名 | 用途说明 |
|---|---|
| `@c_status_bg` | 状态栏整体背景 |
| `@c_status_fg` | 状态栏整体前景（文字） |
| `@c_left_bg` | 左侧状态栏背景 |
| `@c_left_fg` | 左侧状态栏前景 |
| `@c_right_bg` | 右侧状态栏背景 |
| `@c_right_fg` | 右侧状态栏前景 |
| `@c_win_bg` | 非当前窗口标签背景 |
| `@c_win_fg` | 非当前窗口标签前景 |
| `@c_curwin_bg` | 当前窗口标签背景 |
| `@c_curwin_fg` | 当前窗口标签前景 |
| `@c_border` | 非活动窗格边框前景 |
| `@c_active_border` | 活动窗格边框前景 |
| `@c_msg_bg` | 消息提示背景 |
| `@c_msg_fg` | 消息提示前景 |
| `@c_mode_bg` | 选择/复制模式背景 |
| `@c_mode_fg` | 选择/复制模式前景 |

#### 预设配色方案

**方案 A：Catppuccin Mocha**（柔和马卡龙风）

| 变量 | 色值 | 色名 |
|---|---|---|
| `@c_status_bg` | `#1e1e2e` | Base |
| `@c_status_fg` | `#cdd6f4` | Text |
| `@c_left_bg` | `#b4befe` | Lavender |
| `@c_left_fg` | `#1e1e2e` | Base |
| `@c_right_bg` | `#a6e3a1` | Green |
| `@c_right_fg` | `#1e1e2e` | Base |
| `@c_win_bg` | `#313244` | Surface0 |
| `@c_win_fg` | `#cdd6f4` | Text |
| `@c_curwin_bg` | `#cba6f7` | Mauve |
| `@c_curwin_fg` | `#1e1e2e` | Base |
| `@c_border` | `#89dceb` | Teal |
| `@c_active_border` | `#b4befe` | Lavender |
| `@c_msg_bg` | `#b4befe` | Lavender |
| `@c_msg_fg` | `#1e1e2e` | Base |
| `@c_mode_bg` | `#45475a` | Surface1 |
| `@c_mode_fg` | `#cdd6f4` | Text |

**方案 B：Dracula**（暗夜吸血鬼风）

| 变量 | 色值 | 色名 |
|---|---|---|
| `@c_status_bg` | `#282a36` | Background |
| `@c_status_fg` | `#f8f8f2` | Foreground |
| `@c_left_bg` | `#bd93f9` | Purple |
| `@c_left_fg` | `#282a36` | Background |
| `@c_right_bg` | `#50fa7b` | Green |
| `@c_right_fg` | `#282a36` | Background |
| `@c_win_bg` | `#44475a` | CurrentLine |
| `@c_win_fg` | `#f8f8f2` | Foreground |
| `@c_curwin_bg` | `#ff79c6` | Pink |
| `@c_curwin_fg` | `#282a36` | Background |
| `@c_border` | `#8be9fd` | Cyan |
| `@c_active_border` | `#bd93f9` | Purple |
| `@c_msg_bg` | `#bd93f9` | Purple |
| `@c_msg_fg` | `#f8f8f2` | Foreground |
| `@c_mode_bg` | `#44475a` | CurrentLine |
| `@c_mode_fg` | `#f8f8f2` | Foreground |

**方案 C：Tokyo Night**（东京夜色风）

| 变量 | 色值 | 色名 |
|---|---|---|
| `@c_status_bg` | `#1a1b26` | bg |
| `@c_status_fg` | `#c0caf5` | fg |
| `@c_left_bg` | `#7aa2f7` | Blue |
| `@c_left_fg` | `#1a1b26` | bg |
| `@c_right_bg` | `#9ece6a` | Green |
| `@c_right_fg` | `#1a1b26` | bg |
| `@c_win_bg` | `#292e42` | bg_highlight |
| `@c_win_fg` | `#c0caf5` | fg |
| `@c_curwin_bg` | `#bb9af7` | Magenta |
| `@c_curwin_fg` | `#1a1b26` | bg |
| `@c_border` | `#7dcfff` | Cyan |
| `@c_active_border` | `#7aa2f7` | Blue |
| `@c_msg_bg` | `#7aa2f7` | Blue |
| `@c_msg_fg` | `#1a1b26` | bg |
| `@c_mode_bg` | `#292e42` | bg_highlight |
| `@c_mode_fg` | `#c0caf5` | fg |

**方案 D：Gruvbox Dark**（复古暖色风）

| 变量 | 色值 | 色名 |
|---|---|---|
| `@c_status_bg` | `#282828` | bg |
| `@c_status_fg` | `#ebdbb2` | fg |
| `@c_left_bg` | `#fe8019` | Orange |
| `@c_left_fg` | `#282828` | bg |
| `@c_right_bg` | `#b8bb26` | Green |
| `@c_right_fg` | `#282828` | bg |
| `@c_win_bg` | `#3c3836` | bg1 |
| `@c_win_fg` | `#ebdbb2` | fg |
| `@c_curwin_bg` | `#fabd2f` | Yellow |
| `@c_curwin_fg` | `#282828` | bg |
| `@c_border` | `#8ec07c` | Aqua |
| `@c_active_border` | `#fe8019` | Orange |
| `@c_msg_bg` | `#fe8019` | Orange |
| `@c_msg_fg` | `#282828` | bg |
| `@c_mode_bg` | `#504945` | bg2 |
| `@c_mode_fg` | `#ebdbb2` | fg |

**方案 E：Nord**（北极冰蓝风）

| 变量 | 色值 | 色名 |
|---|---|---|
| `@c_status_bg` | `#2e3440` | Polar Night |
| `@c_status_fg` | `#d8dee9` | Snow Storm |
| `@c_left_bg` | `#81a1c1` | Frost |
| `@c_left_fg` | `#2e3440` | Polar Night |
| `@c_right_bg` | `#a3be8c` | Aurora Green |
| `@c_right_fg` | `#2e3440` | Polar Night |
| `@c_win_bg` | `#3b4252` | Polar Night + |
| `@c_win_fg` | `#d8dee9` | Snow Storm |
| `@c_curwin_bg` | `#5e81ac` | Frost Dark |
| `@c_curwin_fg` | `#eceff4` | Snow Storm + |
| `@c_border` | `#88c0d0` | Frost Light |
| `@c_active_border` | `#81a1c1` | Frost |
| `@c_msg_bg` | `#81a1c1` | Frost |
| `@c_msg_fg` | `#2e3440` | Polar Night |
| `@c_mode_bg` | `#3b4252` | Polar Night + |
| `@c_mode_fg` | `#d8dee9` | Snow Storm |

#### 配置模板

AI 根据用户选择的方案，将上表中的色值替换下方模板中的 `#XXXXXX` 占位符后写入 `~/.tmux.conf`。

```tmux
# --- 状态栏配色（{用户选择的方案名称}） ---
set -g status-style "bg=#XXXXXX,fg=#XXXXXX"                        # [AI_INSTALL] 状态栏整体
set -g status-left-style "bg=#XXXXXX,fg=#XXXXXX"                   # [AI_INSTALL] 左侧状态栏
set -g status-right-style "bg=#XXXXXX,fg=#XXXXXX"                  # [AI_INSTALL] 右侧状态栏
setw -g window-status-style "bg=#XXXXXX,fg=#XXXXXX"                # [AI_INSTALL] 非当前窗口
setw -g window-status-current-style "bg=#XXXXXX,fg=#XXXXXX"        # [AI_INSTALL] 当前窗口
set -g pane-border-style "fg=#XXXXXX"                               # [AI_INSTALL] 非活动窗格边框
set -g pane-active-border-style "fg=#XXXXXX"                        # [AI_INSTALL] 活动窗格边框
set -g message-style "bg=#XXXXXX,fg=#XXXXXX"                       # [AI_INSTALL] 消息提示
setw -g mode-style "bg=#XXXXXX,fg=#XXXXXX"                         # [AI_INSTALL] 选择/复制模式
```

---

## Step 3：终端适配（可选）

### 3.1 Ghostty 鼠标兼容

**检测方法**（任意一条满足即视为使用 Ghostty）：

```bash
# 方法 1：检测当前终端程序
echo $TERM_PROGRAM   # 输出为 "ghostty" 则命中

# 方法 2：检测配置文件是否存在
ls ~/.config/ghostty/config 2>/dev/null && echo "Ghostty 配置存在"
```

**检测到 Ghostty 时，须向用户展示以下说明并询问是否执行**：

> 检测到你正在使用 **Ghostty** 终端。  
> Ghostty 默认会拦截第一次鼠标点击，导致 tmux 鼠标单击无法切换 pane，需要双击才有效。  
> 建议在 `~/.config/ghostty/config` 中加入：
> ```
> focus-follows-mouse = true
> ```
> 是否现在自动写入？（需重启 Ghostty 后生效）

**用户确认后执行**：

1. 检查 `~/.config/ghostty/config` 是否已含 `focus-follows-mouse`：
   - **已存在** → 提示"已有该配置，跳过"，不修改
   - **不存在** → 追加以下内容：

```bash
cat >> ~/.config/ghostty/config << 'EOF'

# 允许鼠标事件透传给终端应用（tmux），避免第一次点击被 Ghostty 拦截
focus-follows-mouse = true
EOF
```

2. 提示用户：**需重启 Ghostty 后生效**（Ghostty 不支持热重载配置）。

**用户拒绝**：跳过，不做任何修改。

---

## Step 4：验证

配置写入完成后，执行以下验证：

```bash
# 检查配置语法
tmux source ~/.tmux.conf 2>&1 && echo "配置加载成功" || echo "配置加载失败"

# 验证快捷键绑定
tmux list-keys | grep -E "bind.*(split-window|kill-pane|select-pane|new-window|next-window|previous-window|detach-client)"
```

---

## 注意事项

- 不要删除用户的自定义配置项（不在本清单中的项目应原样保留）
- `bind` 指令会自动覆盖同 key 的旧绑定，但仍建议按冲突规则注释旧项以保持可追溯性
