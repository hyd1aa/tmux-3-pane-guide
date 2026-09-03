# tmux 三分屏小白教程

[English](./README_EN.md) | **中文**

从零开始学会在 Linux、SSH 和 VPS 中使用 tmux，并搭建“左侧大 Pane + 右侧上下两个 Pane”的三分屏工作区。

![tmux 三分屏效果图](https://img.cdn1.vip/i/6a99239a778b0_1788421018.webp)

## 目录

- [tmux 是什么](#tmux-是什么)
- [安装 tmux](#安装-tmux)
- [理解 Session、Window 和 Pane](#理解-sessionwindow-和-pane)
- [创建三分屏](#创建三分屏)
- [切换与调整 Pane](#切换与调整-pane)
- [开启鼠标](#开启鼠标)
- [离开与重新连接](#离开与重新连接)
- [关闭 Pane、Window 和 Session](#关闭-panewindow-和-session)
- [常用命令](#常用-tmux-命令)
- [快捷键帮助](#tmux-快捷键帮助--cheat-sheet)
- [一分钟完成三分屏](#一分钟完成三分屏)
- [常见问题](#常见问题-faq)

## tmux 是什么

tmux 是一个终端复用工具。它可以在一个终端里创建多个页面和分屏，而且在 SSH 断开后继续运行程序。重新连接服务器并进入原 Session，就能接着工作。

常见用途：

- 左边运行编辑器，右上查看日志，右下执行命令
- 让下载、编译或服务在 SSH 断开后继续运行
- 在一个 SSH 窗口中管理多个终端任务

## 安装 tmux

### Ubuntu / Debian

```bash
sudo apt update
sudo apt install tmux -y
```

### Rocky Linux / AlmaLinux / Fedora

```bash
sudo dnf install tmux -y
```

### CentOS 7 或仍使用 yum 的系统

```bash
sudo yum install tmux -y
```

### Arch Linux

```bash
sudo pacman -S tmux
```

确认安装成功：

```bash
tmux -V
```

如果看到类似 `tmux 3.4` 的输出，就表示安装完成。

## 理解 Session、Window 和 Pane

```text
tmux server
└── Session：一组持续运行的工作环境
    ├── Window 1：Session 里面的一整页
    │   ├── Pane 1：这一页里的一个终端分屏
    │   └── Pane 2：另一个终端分屏
    └── Window 2：另一整页
        └── Pane 1
```

- **Session**：一组 tmux 工作环境，可以 detach 后继续运行。
- **Window**：Session 里的一整页，类似浏览器标签页。
- **Pane**：Window 里的分屏，每个 Pane 都是独立终端。

## Prefix 应该怎么按

tmux 默认 Prefix（前缀键）是 `Ctrl + b`。大多数快捷键都要先按 Prefix，再按功能键。

> `Ctrl + b → %` 不是三个键同时按。正确方式是：按住 `Ctrl`，按一下 `b`，松开两个键，再按 `%`（通常是 `Shift + 5`）。

后文用 `Prefix → x` 表示“先按 `Ctrl + b`，松开，再按 `x`”。

## 创建三分屏

目标布局：

```text
┌──────────────────────┬──────────────────────┐
│                      │                      │
│                      │       右上 Pane      │
│                      │                      │
│       左侧 Pane      ├──────────────────────┤
│                      │                      │
│                      │       右下 Pane      │
│                      │                      │
└──────────────────────┴──────────────────────┘
```

### 1. 创建命名 Session

```bash
tmux new -s work
```

### 2. 左右分屏

```text
Ctrl + b → %
```

tmux 会把当前 Pane 分成左右两个，光标通常进入新建的右侧 Pane。

### 3. 把右侧上下分屏

保持光标位于右侧 Pane，然后按：

```text
Ctrl + b → "
```

双引号通常需要 `Shift + '`。如果光标不在右侧，先用 `Ctrl + b → 方向键` 切过去。

## 切换与调整 Pane

切换 Pane：

```text
Prefix → ← / → / ↑ / ↓
```

也可以按 `Prefix → o` 依次切换，或按 `Prefix → q` 显示 Pane 编号后快速选择对应数字。

临时把当前 Pane 放大为全屏，再按一次恢复：

```text
Prefix → z
```

使用默认键位微调当前 Pane：

```text
Prefix → Ctrl + ←
Prefix → Ctrl + →
Prefix → Ctrl + ↑
Prefix → Ctrl + ↓
```

部分终端或桌面环境会拦截这些组合键。对新手而言，开启鼠标后拖动 Pane 边界通常更直观。

## 开启鼠标

编辑配置：

```bash
nano ~/.tmux.conf
```

写入推荐的新手配置：

```tmux
# Enable mouse support
set -g mouse on

# Increase scrollback history
set -g history-limit 100000

# Start window numbering from 1
set -g base-index 1

# Start pane numbering from 1
setw -g pane-base-index 1

# Renumber windows automatically
set -g renumber-windows on
```

在 nano 中按 `Ctrl + O`、`Enter` 保存，再按 `Ctrl + X` 退出。让当前 tmux 立即加载配置：

```bash
tmux source-file ~/.tmux.conf
```

开启后可以单击选择 Pane、拖动边界改变大小，并用滚轮查看历史输出。

## 离开与重新连接

### Detach：离开但保持运行

```text
Ctrl + b → d
```

这只会离开 tmux。Session 和里面的程序仍在运行，适合退出 SSH 前使用。

### 查看并重新进入

```bash
tmux ls
tmux attach -t work
```

简写：

```bash
tmux a -t work
```

> `Ctrl + b → d` 与 `exit` 不一样。Detach 保留整个 Session；`exit` 会关闭当前 shell，当前 Pane 也可能随之关闭。

## 关闭 Pane、Window 和 Session

- 关闭当前 Pane：在里面运行 `exit`，或按 `Prefix → x` 后确认。
- 关闭当前 Window：按 `Prefix → &` 后确认。
- 临时离开但保留 Session：按 `Prefix → d`。
- 关闭指定 Session：在普通 shell 运行 `tmux kill-session -t work`。
- 关闭所有 tmux Session：`tmux kill-server`。这会终止所有 tmux 内的程序，请谨慎使用。

## 常用 tmux 命令

| 命令 | 用途 |
| --- | --- |
| `tmux` | 创建一个默认 Session |
| `tmux -V` | 显示 tmux 版本 |
| `tmux new -s work` | 创建并进入名为 `work` 的 Session |
| `tmux new -d -s work` | 在后台创建 `work`，暂不进入 |
| `tmux ls` | 列出所有 Session |
| `tmux list-sessions` | `tmux ls` 的完整写法 |
| `tmux attach` | 连接到可用 Session |
| `tmux attach -t work` | 连接到 `work` |
| `tmux a -t work` | 上一条命令的简写 |
| `tmux attach -d -t work` | 从其他客户端分离 `work` 后连接 |
| `tmux rename-session -t work newname` | 重命名 Session |
| `tmux kill-session -t work` | 关闭 `work` |
| `tmux kill-session -a -t work` | 关闭除 `work` 外的其他 Session |
| `tmux kill-server` | 关闭 tmux server 和全部 Session |
| `tmux list-windows` | 列出当前或指定 Session 的 Window |
| `tmux list-panes` | 列出当前或指定 Window 的 Pane |
| `tmux source-file ~/.tmux.conf` | 重新加载配置 |

## tmux 快捷键帮助 / Cheat Sheet

以下均为 tmux 默认键位，除特别说明外，都要先按 `Ctrl + b`（Prefix），松开后再按功能键。

### Pane

| 快捷键 | 功能 |
| --- | --- |
| `Prefix → %` | 左右分屏 |
| `Prefix → "` | 上下分屏 |
| `Prefix → ← / → / ↑ / ↓` | 切换到对应方向的 Pane |
| `Prefix → o` | 切换到下一个 Pane |
| `Prefix → ;` | 返回上一次使用的 Pane |
| `Prefix → q` | 显示 Pane 编号；编号出现时按数字选择 |
| `Prefix → z` | 当前 Pane 临时全屏/恢复 |
| `Prefix → x` | 关闭当前 Pane并确认 |
| `Prefix → Space` | 切换预设 Pane 布局 |

### 调整 Pane 大小

| 快捷键 | 功能 |
| --- | --- |
| `Prefix → Ctrl + ←` | 向左调整边界 |
| `Prefix → Ctrl + →` | 向右调整边界 |
| `Prefix → Ctrl + ↑` | 向上调整边界 |
| `Prefix → Ctrl + ↓` | 向下调整边界 |

如果终端拦截组合键，请使用鼠标拖动边界，或按 `Prefix → :` 后输入 `resize-pane -L 5`、`-R 5`、`-U 5`、`-D 5`。

### Window

| 快捷键 | 功能 |
| --- | --- |
| `Prefix → c` | 创建新 Window |
| `Prefix → n` | 下一个 Window |
| `Prefix → p` | 上一个 Window |
| `Prefix → l` | 上一次使用的 Window |
| `Prefix → w` | 打开 Window 选择列表 |
| `Prefix → ,` | 重命名当前 Window |
| `Prefix → &` | 关闭当前 Window并确认 |
| `Prefix → 0-9` | 切换到对应编号的 Window |

### Session

| 快捷键 | 功能 |
| --- | --- |
| `Prefix → d` | detach，离开但保持 Session 运行 |
| `Prefix → s` | 打开 Session 选择列表 |
| `Prefix → $` | 重命名当前 Session |
| `Prefix → (` | 切换到上一个 Session |
| `Prefix → )` | 切换到下一个 Session |

### 复制与历史

| 快捷键 | 功能 |
| --- | --- |
| `Prefix → [` | 进入 copy mode，查看/复制历史 |
| `Prefix → ]` | 粘贴最近一次 tmux buffer |
| `Prefix → =` | 打开可选择的 buffer 列表 |

在 copy mode 中通常按 `q` 或 `Esc` 退出。复制按键会受 tmux 的 vi/emacs 模式配置影响。

### 帮助与命令模式

| 快捷键 | 功能 |
| --- | --- |
| `Prefix → ?` | 打开 tmux 自带的完整快捷键帮助；按 `q` 退出 |
| `Prefix → :` | 打开 tmux 命令输入行 |

## 一分钟完成三分屏

```bash
tmux new -s work
```

然后依次按：

```text
Ctrl + b → %
Ctrl + b → "
```

完成。常用的三个后续操作：

```text
切换 Pane：Ctrl + b → 方向键
暂时离开：Ctrl + b → d
```

回来：

```bash
tmux a -t work
```

## 常见问题 FAQ

### 1. 为什么 `%` 没有分屏？

先按 `Ctrl + b` 并松开，再按 `%`。在多数键盘上 `%` 是 `Shift + 5`，不要把三个键同时按。

### 2. 为什么 `"` 没反应？

先按 Prefix 并松开，再按双引号；多数键盘需要 `Shift + '`。确认输入法和终端没有拦截该键。

### 3. Prefix 应该怎么按？

按住 `Ctrl`，按 `b`，松开，然后按功能键。默认 Prefix 是 `Ctrl + b`。

### 4. SSH 断开后 tmux 里的程序还在吗？

通常仍在。只要服务器和 tmux Session 没有被关闭，程序会继续运行。

### 5. 怎么重新进入 tmux？

运行 `tmux ls` 找到名称，再运行 `tmux attach -t 名称`。

### 6. 怎么查看所有 Session？

运行 `tmux ls` 或 `tmux list-sessions`。

### 7. 怎么关闭一个 Pane？

在目标 Pane 运行 `exit`，或按 `Prefix → x` 后确认。

### 8. 怎么关闭整个 Session？

运行 `tmux kill-session -t work`；把 `work` 换成实际名称。

### 9. 怎么退出 tmux 但保持程序运行？

按 `Prefix → d` detach，不要在每个 Pane 中运行 `exit`。

### 10. 怎么开启鼠标？

在 `~/.tmux.conf` 加入 `set -g mouse on`，然后运行 `tmux source-file ~/.tmux.conf`。

### 11. 怎么拖动分屏大小？

开启鼠标后，把鼠标放到 Pane 边界上并拖动。

### 12. 怎么让 Pane 临时全屏？

切换到目标 Pane，按 `Prefix → z`；再按一次恢复。

### 13. 忘记快捷键怎么办？

按 `Prefix → ?` 查看 tmux 自带帮助，按 `q` 退出帮助。

### 14. Session、Window、Pane 有什么区别？

Session 是整套工作环境，Window 是其中一整页，Pane 是一页内的分屏。

## License

本教程采用 [MIT License](./LICENSE)。
