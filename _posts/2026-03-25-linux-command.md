---
layout: post
title: "Linux CLI"
date: 2026-03-25 09:00:00 -0500
categories: linux
tages: linux command cli
image:
    path: /assets/img/headers/cli.png
---


# A section devoted to Linux commands, their often confusing meanings, and clear explanations of what they do.


tmux is a terminal multiplexer. That means it lets you open and manage multiple terminal sessions inside one terminal window. Instead of opening many separate terminal windows, you can keep everything organized inside tmux using sessions, windows, and panes. This is one of the main reasons tmux is so useful for Linux administration, server work, and long-running tasks over SSH.

```bash
# Start / Sessions
tmux                 # Start tmux.
tmux new -s mysession # Create a new named session.
tmux ls              # List all tmux sessions.
tmux attach -t mysession # Attach to a named session.
tmux kill-session -t mysession # Kill a named session.

# Prefix key
Ctrl+b               # Default tmux prefix key.

# Sessions
Ctrl+b d             # Detach from the current session.
Ctrl+b s             # Show the list of sessions.
Ctrl+b $             # Rename the current session.

# Windows
Ctrl+b c             # Create a new window.
Ctrl+b n             # Go to the next window.
Ctrl+b p             # Go to the previous window.
Ctrl+b w             # Show the list of windows.
Ctrl+b ,             # Rename the current window.
Ctrl+b &             # Kill the current window.
Ctrl+b 0             # Switch to window 0.
Ctrl+b 1             # Switch to window 1.
Ctrl+b 2             # Switch to window 2.
Ctrl+b 3             # Switch to window 3.
Ctrl+b 4             # Switch to window 4.
Ctrl+b 5             # Switch to window 5.
Ctrl+b 6             # Switch to window 6.
Ctrl+b 7             # Switch to window 7.
Ctrl+b 8             # Switch to window 8.
Ctrl+b 9             # Switch to window 9.

# Panes
Ctrl+b %             # Split the current pane vertically (left/right).
Ctrl+b "             # Split the current pane horizontally (top/bottom).
Ctrl+b o             # Move to the next pane.
Ctrl+b ;             # Switch to the last active pane.
Ctrl+b q             # Show pane numbers.
Ctrl+b x             # Close the current pane.
Ctrl+b z             # Zoom/unzoom the current pane.
Ctrl+b Up            # Move to the pane above.
Ctrl+b Down          # Move to the pane below.
Ctrl+b Left          # Move to the pane on the left.
Ctrl+b Right         # Move to the pane on the right.

# Copy / Command mode
Ctrl+b [             # Enter copy mode.
Ctrl+b ]             # Paste copied text.
Ctrl+b :             # Open the tmux command prompt.
Ctrl+b ?             # Show all key bindings.

# Handy workflow
Ctrl+b c             # Open a new window for another task.
Ctrl+b %             # Split window into left/right panes.
Ctrl+b "             # Split window into top/bottom panes.
Ctrl+b o             # Cycle through panes.
Ctrl+b d             # Leave tmux without stopping programs.
tmux attach          # Reattach later.
```

---