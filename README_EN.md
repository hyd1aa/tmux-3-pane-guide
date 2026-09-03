# tmux 3-Pane Beginner Guide

**English** | [中文](./README.md)

A from-scratch guide to tmux on Linux, SSH, and VPS servers, ending with one large pane on the left and two stacked panes on the right.

![tmux 3-pane layout](https://img.cdn1.vip/i/6a99239a778b0_1788421018.webp)

## Contents

- [What is tmux?](#what-is-tmux)
- [Install tmux](#install-tmux)
- [Session, Window, and Pane](#session-window-and-pane)
- [Build the three-pane layout](#build-the-three-pane-layout)
- [Navigate and resize panes](#navigate-and-resize-panes)
- [Enable mouse support](#enable-mouse-support)
- [Detach and reconnect](#detach-and-reconnect)
- [Close a Pane, Window, or Session](#close-a-pane-window-or-session)
- [Useful commands](#useful-tmux-commands)
- [Keyboard cheat sheet](#tmux-keyboard-shortcuts--cheat-sheet)
- [Three panes in one minute](#three-panes-in-one-minute)
- [FAQ](#frequently-asked-questions)

## What is tmux?

tmux is a terminal multiplexer. It lets you create multiple pages and split panes inside one terminal. More importantly, programs inside tmux keep running when your SSH connection drops. Reconnect to the server and attach to the Session to continue where you left off.

Common uses include:

- Editor on the left, logs at the top right, and a shell at the bottom right
- Long downloads, builds, and services that survive SSH disconnects
- Several terminal tasks inside one SSH connection

## Install tmux

### Ubuntu / Debian

```bash
sudo apt update
sudo apt install tmux -y
```

### Rocky Linux / AlmaLinux / Fedora

```bash
sudo dnf install tmux -y
```

### CentOS 7 or systems that still use yum

```bash
sudo yum install tmux -y
```

### Arch Linux

```bash
sudo pacman -S tmux
```

Verify the installation:

```bash
tmux -V
```

An output such as `tmux 3.4` means tmux is installed.

## Session, Window, and Pane

```text
tmux server
└── Session: a persistent tmux workspace
    ├── Window 1: one full page in the Session
    │   ├── Pane 1: one terminal region
    │   └── Pane 2: another terminal region
    └── Window 2: another full page
        └── Pane 1
```

- **Session:** a persistent collection of tmux workspaces.
- **Window:** one full page inside a Session, similar to a browser tab.
- **Pane:** a split terminal region inside a Window.

## How to press the Prefix

The default tmux Prefix is `Ctrl + b`. Most shortcuts start with the Prefix, followed by another key.

> `Ctrl + b → %` does not mean pressing all three keys together. Hold `Ctrl`, press `b`, release both keys, and then press `%` (usually `Shift + 5`).

This guide uses `Prefix → x` to mean “press `Ctrl + b`, release it, then press `x`.”

## Build the three-pane layout

Target layout:

```text
┌──────────────────────┬──────────────────────┐
│                      │                      │
│                      │    Top-right Pane    │
│                      │                      │
│      Left Pane       ├──────────────────────┤
│                      │                      │
│                      │   Bottom-right Pane  │
│                      │                      │
└──────────────────────┴──────────────────────┘
```

### 1. Create a named Session

```bash
tmux new -s work
```

### 2. Split left and right

```text
Ctrl + b → %
```

tmux splits the current Pane vertically and normally places the cursor in the new right-hand Pane.

### 3. Split the right side top and bottom

Keep the cursor in the right-hand Pane, then press:

```text
Ctrl + b → "
```

On most keyboards, `"` requires `Shift + '`. If the cursor is not on the right, move there first with `Ctrl + b → Arrow key`.

## Navigate and resize panes

Move between panes:

```text
Prefix → ← / → / ↑ / ↓
```

You can also use `Prefix → o` to cycle, or press `Prefix → q` and select the displayed Pane number.

Temporarily zoom the active Pane, then restore it with the same shortcut:

```text
Prefix → z
```

Resize with the default key bindings:

```text
Prefix → Ctrl + ←
Prefix → Ctrl + →
Prefix → Ctrl + ↑
Prefix → Ctrl + ↓
```

Some terminals or desktop environments intercept these combinations. For beginners, enabling the mouse and dragging a Pane border is often easier.

## Enable mouse support

Open the configuration file:

```bash
nano ~/.tmux.conf
```

Add this beginner-friendly configuration:

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

In nano, press `Ctrl + O`, `Enter` to save, and `Ctrl + X` to exit. Reload the file in the running tmux server:

```bash
tmux source-file ~/.tmux.conf
```

You can now click panes, drag their borders, and use the wheel to inspect scrollback.

## Detach and reconnect

### Detach without stopping programs

```text
Ctrl + b → d
```

This leaves tmux but keeps the Session and its programs running. Use it before closing an SSH connection.

### Find and reattach

```bash
tmux ls
tmux attach -t work
```

Short form:

```bash
tmux a -t work
```

> `Ctrl + b → d` and `exit` are not the same. Detach preserves the entire Session. `exit` closes the current shell and may therefore close its Pane.

## Close a Pane, Window, or Session

- Current Pane: run `exit`, or press `Prefix → x` and confirm.
- Current Window: press `Prefix → &` and confirm.
- Leave while keeping everything alive: press `Prefix → d`.
- Named Session: run `tmux kill-session -t work` from a regular shell.
- Every tmux Session: run `tmux kill-server`. This also terminates programs in those Sessions, so use it carefully.

## Useful tmux commands

| Command | Purpose |
| --- | --- |
| `tmux` | Create a default Session |
| `tmux -V` | Print the tmux version |
| `tmux new -s work` | Create and enter a Session named `work` |
| `tmux new -d -s work` | Create `work` in the background |
| `tmux ls` | List all Sessions |
| `tmux list-sessions` | Full form of `tmux ls` |
| `tmux attach` | Attach to an available Session |
| `tmux attach -t work` | Attach to `work` |
| `tmux a -t work` | Short form of the previous command |
| `tmux attach -d -t work` | Detach other clients from `work`, then attach |
| `tmux rename-session -t work newname` | Rename a Session |
| `tmux kill-session -t work` | Close `work` |
| `tmux kill-session -a -t work` | Close every Session except `work` |
| `tmux kill-server` | Close the tmux server and every Session |
| `tmux list-windows` | List Windows in the current or selected Session |
| `tmux list-panes` | List Panes in the current or selected Window |
| `tmux source-file ~/.tmux.conf` | Reload the configuration |

## tmux Keyboard Shortcuts / Cheat Sheet

These are default tmux bindings. Unless noted otherwise, press `Ctrl + b` (the Prefix), release it, and then press the listed key.

### Pane

| Shortcut | Action |
| --- | --- |
| `Prefix → %` | Split left and right |
| `Prefix → "` | Split top and bottom |
| `Prefix → ← / → / ↑ / ↓` | Move to the Pane in that direction |
| `Prefix → o` | Move to the next Pane |
| `Prefix → ;` | Return to the previously active Pane |
| `Prefix → q` | Show Pane numbers; press a number while visible to select it |
| `Prefix → z` | Zoom or restore the current Pane |
| `Prefix → x` | Close the current Pane after confirmation |
| `Prefix → Space` | Cycle through preset Pane layouts |

### Pane resize

| Shortcut | Action |
| --- | --- |
| `Prefix → Ctrl + ←` | Resize toward the left |
| `Prefix → Ctrl + →` | Resize toward the right |
| `Prefix → Ctrl + ↑` | Resize upward |
| `Prefix → Ctrl + ↓` | Resize downward |

If your terminal intercepts these keys, drag the border with the mouse, or press `Prefix → :` and enter `resize-pane -L 5`, `-R 5`, `-U 5`, or `-D 5`.

### Window

| Shortcut | Action |
| --- | --- |
| `Prefix → c` | Create a new Window |
| `Prefix → n` | Next Window |
| `Prefix → p` | Previous Window |
| `Prefix → l` | Previously selected Window |
| `Prefix → w` | Open the Window chooser |
| `Prefix → ,` | Rename the current Window |
| `Prefix → &` | Close the current Window after confirmation |
| `Prefix → 0-9` | Select the Window with that number |

### Session

| Shortcut | Action |
| --- | --- |
| `Prefix → d` | Detach while keeping the Session running |
| `Prefix → s` | Open the Session chooser |
| `Prefix → $` | Rename the current Session |
| `Prefix → (` | Previous Session |
| `Prefix → )` | Next Session |

### Copy and history

| Shortcut | Action |
| --- | --- |
| `Prefix → [` | Enter copy mode for scrollback and selection |
| `Prefix → ]` | Paste the most recent tmux buffer |
| `Prefix → =` | Open the buffer chooser |

Usually `q` or `Esc` exits copy mode. Selection keys depend on whether tmux is configured for vi or emacs mode.

### Help and command mode

| Shortcut | Action |
| --- | --- |
| `Prefix → ?` | Open tmux's built-in key-binding help; press `q` to exit |
| `Prefix → :` | Open the tmux command prompt |

## Three panes in one minute

```bash
tmux new -s work
```

Then press, in order:

```text
Ctrl + b → %
Ctrl + b → "
```

Done. Three useful follow-up actions:

```text
Switch Pane: Ctrl + b → Arrow key
Leave temporarily: Ctrl + b → d
```

Come back with:

```bash
tmux a -t work
```

## Frequently asked questions

### 1. Why did `%` not split the Window?

Press and release `Ctrl + b`, then press `%`. On most keyboards `%` is `Shift + 5`. Do not press all three keys together.

### 2. Why does `"` do nothing?

Press and release the Prefix, then type a double quote, usually `Shift + '`. Check whether your input method or terminal intercepts it.

### 3. How exactly do I press the Prefix?

Hold `Ctrl`, press `b`, release both, then press the command key. The default Prefix is `Ctrl + b`.

### 4. Are programs still running after SSH disconnects?

Normally, yes. They continue as long as the server and tmux Session remain alive.

### 5. How do I return to tmux?

Run `tmux ls`, then `tmux attach -t NAME` with the Session name.

### 6. How do I list every Session?

Run `tmux ls` or `tmux list-sessions`.

### 7. How do I close one Pane?

Run `exit` in that Pane, or press `Prefix → x` and confirm.

### 8. How do I close an entire Session?

Run `tmux kill-session -t work`, replacing `work` with its actual name.

### 9. How do I leave without stopping programs?

Press `Prefix → d` to detach. Do not run `exit` in every Pane.

### 10. How do I enable the mouse?

Add `set -g mouse on` to `~/.tmux.conf`, then run `tmux source-file ~/.tmux.conf`.

### 11. How do I drag a split border?

Enable mouse mode, place the pointer on a Pane border, and drag it.

### 12. How do I temporarily make one Pane full-screen?

Select it and press `Prefix → z`. Press the same shortcut again to restore the layout.

### 13. What if I forget a shortcut?

Press `Prefix → ?` for tmux's built-in help, then press `q` to leave help.

### 14. What is the difference between Session, Window, and Pane?

A Session is the full persistent workspace, a Window is one page inside it, and a Pane is one split terminal region within a Window.

## License

This guide is available under the [MIT License](./LICENSE).
