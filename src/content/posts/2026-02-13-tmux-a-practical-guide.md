---
title: "tmux: A Practical Guide (What It Is, Why It’s Useful, and How to Actually Use It)"
pubDate: 2026-02-13
description: "tmux keeps your terminal sessions alive, organized, and detachable—especially over SSH. Here’s a hands-on guide to the basics and the habits that make it click."
---

If you’ve ever:

- been SSH’d into a server, started a long-running command, then lost Wi‑Fi and watched it die
- wanted your terminal to feel more like a workspace (editor + logs + shell) instead of one scrolling rectangle
- had 12 tabs open and still couldn’t find the “right” one

…you’re tmux’s target audience.

This post is a hands-on guide to **what tmux is**, **why it’s useful**, and **how to use it comfortably**—without turning it into a lifestyle.

## What tmux is (in normal human terms)

**tmux** is a “terminal multiplexer.” Translation: it lets you run **multiple terminal sessions inside one terminal window**, and it gives you tools to:

- split the screen into **panes** (side-by-side or stacked)
- create **windows** (like tabs)
- **detach** from a session and leave it running in the background
- **reattach** later (even from a new SSH connection)

The detach/reattach part is the superpower.

Think of tmux as a lightweight “workspace manager” for command-line work—especially on remote machines.

## Why tmux is useful (the “okay but why should I care” section)

### 1) Your work survives disconnects

When you SSH into a server normally, your commands are tied to that network connection. If the connection drops, your terminal closes, and your processes often get SIGHUP’d (read: they die, or get weird).

With tmux:

- you start a tmux session
- you run your stuff *inside* tmux
- you can disconnect (intentionally or accidentally)
- the tmux session keeps running on the remote machine

Later you reattach and it’s like nothing happened.

### 2) It makes “watching logs + doing work” sane

A classic workflow:

- left pane: your shell
- right pane: `tail -f` for logs
- bottom pane: running tests/builds

No alt-tabbing. No losing context.

### 3) It’s great for repeatable setups

If you tend to do the same thing every time you hop into a server—open editor, start a dev server, watch logs—tmux makes that repeatable.

Later (optional): you can save layouts or script session startup.

### 4) It’s low-dependency, works basically everywhere

tmux is old (in the best way): stable, available on most Linux boxes, and doesn’t require a GUI. It’s a perfect companion for:

- VPS work
- homelabs
- Raspberry Pis
- “this server has no desktop and that’s the point” machines

## Install tmux

### macOS

```bash
brew install tmux
```

### Ubuntu/Debian

```bash
sudo apt update
sudo apt install tmux
```

### Fedora

```bash
sudo dnf install tmux
```

### Arch

```bash
sudo pacman -S tmux
```

## The tmux mental model (2 minutes that will save you 2 hours)

tmux has three main layers:

- **Session**: the top-level container (often “one per project/server/task”). You can detach/reattach to sessions.
- **Window**: like a tab within a session.
- **Pane**: a split region within a window.

You’ll spend most of your time switching between windows and panes.

## The one command you need to start

Start tmux:

```bash
tmux
```

That’s it. You’re in.

### Create a named session (recommended)

```bash
tmux new -s work
```

Naming sessions makes reattaching much nicer.

## The tmux “prefix” key (aka: why everything starts with Ctrl-b)

Most tmux shortcuts are two-step:

1. press the **prefix** (default: `Ctrl-b`)
2. press another key (like `c` or `d`)

When you see something like **“prefix + c”**, read it as:

- hold Control and press `b`, release
- then press `c`

I’ll write it as: **`Ctrl-b` then `c`**.

## Core commands you’ll use every day

### Detach and reattach (the superpower)

Detach (leave it running):

- **`Ctrl-b` then `d`**

List tmux sessions:

```bash
tmux ls
```

Reattach:

```bash
tmux attach
```

Reattach to a specific one:

```bash
tmux attach -t work
```

### Create windows (“tabs”)

New window:

- **`Ctrl-b` then `c`**

Next/previous window:

- **`Ctrl-b` then `n`** (next)
- **`Ctrl-b` then `p`** (previous)

List windows:

- **`Ctrl-b` then `w`**

Rename the current window:

- **`Ctrl-b` then `,`** (comma)

### Split into panes

Vertical split (left/right):

- **`Ctrl-b` then `%`**

Horizontal split (top/bottom):

- **`Ctrl-b` then `"`**

Move between panes:

- **`Ctrl-b` then arrow keys**

Close a pane:

- type `exit` in that pane (recommended)

(You *can* kill a pane from tmux, but exiting normally is a good habit.)

### Resize panes

Hold prefix, then resize with arrows:

- **`Ctrl-b` then hold `Alt` and press arrows** (common config)

…but defaults vary by terminal. If resizing feels annoying, I’ll give you a tiny config below that makes it easy.

### Copy/scroll mode (aka: “I need to scroll up”) 

If you try to scroll with your mouse and it’s weird, tmux is probably intercepting scrollback.

Enter copy mode:

- **`Ctrl-b` then `[`**

Then use arrow keys / PageUp / Vim keys (depending on config). To exit copy mode, press `q`.

## A simple “real life” tmux workflow (over SSH)

Let’s say you’re deploying something to a server.

1) SSH in:

```bash
ssh you@server
```

2) Start a named tmux session:

```bash
tmux new -s deploy
```

3) Split the window into two panes:

- `Ctrl-b` then `%`

4) In the left pane, run your deploy.

5) In the right pane, watch logs:

```bash
journalctl -u your-service -f
# or
kubectl logs -f deploy/your-app
# or
tail -f /var/log/whatever.log
```

6) Need to leave? Detach:

- `Ctrl-b` then `d`

7) Come back later:

```bash
tmux attach -t deploy
```

That’s the “tmux changed my life” loop, right there.

## Quality-of-life: a tiny ~/.tmux.conf starter

Out of the box, tmux is powerful but a little… 2007.

Here’s a small config that makes tmux friendlier without turning it into a whole personality. Create `~/.tmux.conf`:

```tmux
# Use a more responsive escape time (helps in some terminals)
set -sg escape-time 10

# Enable mouse support (scroll, select panes, resize)
set -g mouse on

# Vim-style keys in copy mode
setw -g mode-keys vi

# Easier pane navigation with Vim keys
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R

# Quick pane splits
bind | split-window -h
bind - split-window -v

# Reload config
bind r source-file ~/.tmux.conf \; display-message "Reloaded ~/.tmux.conf"
```

Then reload tmux config without restarting:

- `Ctrl-b` then `r`

Notes:

- **Mouse mode** is controversial in some terminal circles. I’m pro-mouse for most humans. You can always turn it off later.
- The `|` and `-` bindings are easier to remember than `%` and `"`.

## “I’m stuck in tmux” (aka: how to get out)

- To detach: `Ctrl-b` then `d`
- To close a pane/window: type `exit`
- To end a session completely: from outside tmux:

```bash
tmux kill-session -t deploy
```

Or kill the server (all sessions):

```bash
tmux kill-server
```

(Use that last one sparingly.)

## tmux vs screen vs just using terminal tabs

- **Terminal tabs** are fine locally, but they don’t survive SSH disconnects.
- **screen** is the older alternative; it’s still useful and ubiquitous, but tmux is generally nicer.
- **tmux** hits the sweet spot: modern-ish, stable, scriptable, and widely available.

If you do *any* remote work, tmux is one of those tools that pays for itself the first time your connection drops.

## Common “gotchas” (so you don’t bounce off it)

### Gotcha: “My terminal scrollback is weird now”

Use copy mode (`Ctrl-b` then `[`) or enable mouse mode in `.tmux.conf`.

### Gotcha: “Keyboard shortcuts conflict with my shell/editor”

You can change the prefix key. Some people prefer `Ctrl-a`.

Example (replace prefix):

```tmux
unbind C-b
set -g prefix C-a
bind C-a send-prefix
```

(If you do this, remember it everywhere. Consistency matters more than perfection.)

### Gotcha: “I forget the shortcuts”

Two tips:

- Use `Ctrl-b` then `?` to see the key bindings.
- Don’t memorize everything. Memorize: **split, switch panes, new window, detach, attach**.

Everything else can be learned as-needed.

## Getting started challenge (10 minutes)

If you want tmux to stick, do this once:

1. Start `tmux new -s test`
2. Split into 2 panes
3. In one pane: run `top` (or `htop`)
4. In the other pane: run `tail -f` on any log (or just `watch date`)
5. Detach
6. Reattach

When you see both panes still running, the value becomes… extremely obvious.

## Links

- tmux (GitHub): https://github.com/tmux/tmux
- tmux man page (OpenBSD): https://man.openbsd.org/tmux
- The Tao of tmux (a classic guide): https://leanpub.com/the-tao-of-tmux

---

If you tell me what terminal/editor you use most (Terminal.app, iTerm2, VS Code, SSH-heavy, etc.), I can tailor a “my personal default tmux workflow” section with recommended bindings and a slightly nicer status bar—still keeping it simple.
