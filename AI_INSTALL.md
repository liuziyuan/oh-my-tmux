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

### 冲突处理规则

遇到已存在相同功能的配置项时：

1. **保留旧项，注释掉**，在注释行末尾添加备注 `# [AI] 被 AI_INSTALL 替换于 YYYY-MM-DD`
2. **在旧项下方写入新项**，并在新项行末尾添加 `# [AI_INSTALL]`

示例：

```tmux
# set -g prefix C-b  # [AI] 被 AI_INSTALL 替换于 2026-04-11
set -g prefix C-a    # [AI_INSTALL]
```

对于**功能相同但写法不同**的项（如 `bind | split-window -h` vs `bind | split-window -h -c "#{pane_current_path}"`），也视为冲突，按上述规则处理。

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

### 2.2 高频快捷键

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

### 2.3 实用工具快捷键

```tmux
# --- 实用工具 ---
bind r source-file ~/.tmux.conf \; display "配置已重载"  # [AI_INSTALL] 重载配置
```

---

## Step 3：验证

配置写入完成后，执行以下验证：

```bash
# 检查配置语法
tmux source ~/.tmux.conf 2>&1 && echo "配置加载成功" || echo "配置加载失败"

# 验证快捷键绑定
tmux list-keys | grep -E "bind.*(split-window|kill-pane|select-pane|new-window|next-window|previous-window|detach-client|source-file)"
```

---

## 注意事项

- 不要删除用户的自定义配置项（不在本清单中的项目应原样保留）
- `bind` 指令会自动覆盖同 key 的旧绑定，但仍建议按冲突规则注释旧项以保持可追溯性
