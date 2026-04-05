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

## tmux
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

## nice

The nice command is used to start a process with a specific priority.

In Linux, processes compete for CPU time.
nice allows you to make a process less important or more important to the scheduler.

Important idea:

A higher nice value means lower priority.
A lower nice value means higher priority.

```bash
# Show the current shell nice value
nice                 # Show the current nice value.

# Start a command with a specific nice value
nice -n 10 command   # Start command with lower priority.
nice -n 5 command    # Start command with slightly lower priority.
nice -n 19 command   # Start command with very low priority.

# Start a command with higher priority
nice -n -5 command   # Start command with higher priority (usually requires root).
nice -n -10 command  # Start command with much higher priority (usually requires root).
nice -n -20 command  # Start command with highest priority (root only).

# Default behavior
command              # Start command with default nice value 0.
nice -n 0 command    # Start command with normal priority.

# Change priority of an already running process
renice 10 -p 1234    # Change process 1234 to nice value 10.
renice 5 -p 1234     # Change process 1234 to nice value 5.
renice 19 -p 1234    # Change process 1234 to very low priority.

# Set higher priority for a running process
renice -5 -p 1234    # Raise priority of process 1234 (usually requires root).
renice -10 -p 1234   # Raise priority more (usually requires root).

# Change priority by user
renice 10 -u username # Change priority for all processes of a user.

# Change priority by process group
renice 10 -g 5678    # Change priority for a process group.

# Useful monitoring commands
ps -o pid,ni,comm -p 1234 # Show PID, nice value, and command.
ps -eo pid,ni,comm | grep command # Find nice values for matching processes.
top                  # View running processes and their priorities.
htop                 # Interactive process viewer, if installed.

# Nice value range
# -20 = highest priority
#  0  = default priority
# 19  = lowest priority
```

---

## Bound

- . (dot) Atom matches with any character.
- ^ (caret) Atom matches with the beginning of a line.
- $ (dollar sign) Atom matches with the end of a line.

## Bracket Expression

Bracket expression lists also accept classes instead of just single characters and ranges. Traditional character classes are:

- [:alnum:] Represents an alphanumeric character.
- [:alpha:] Represents an alphabetic character.
- [:ascii:] Represents a character that fits into the ASCII character set.
- [:blank:] Represents a blank character, that is, a space or a tab.
- [:cntrl:] Represents a control character.
- [:digit:] Represents a digit (0 through 9).
- [:graph:] Represents any printable character except space.
- [:lower:] Represents a lowercase character.
- [:print:] Represents any printable character including space.
- [:punct:] Represents any printable character which is not a space or an alphanumeric character.
- [:space:] Represents white-space characters: space, form-feed (\f), newline (\n), carriage return (\r),horizontal tab (\t), and vertical tab (\v).
- [:upper:] Represents an uppercase letter.
- [:xdigit:] Represents hexadecimal digits (0 through F).

## Quantifiers

The reach of an atom, either a single character atom or a bracket atom, can be adjusted using an
atom quantifier.

```bash
*    # 0 or more
+    # 1 or more
?    # 0 or 1
{3}  # exactly 3
{2,5} # from 2 to 5
{2,} # 2 or more
```

## Bounds 

A bound is an atom quantifier that, as the name implies, allows a user to specify precise quantity
boundaries for an atom. In extended regular expressions, a bound may appear in three forms:

- {i} The atom must appear exactly i times (i an integer number). For example, [[:blank:]]{2}matches with exactly two blank characters.
- {i,} The atom must appear at least i times (i an integer number). For example, [[:blank:]]{2,} matches with any sequence of two or more blank characters.
- {i,j} The atom must appear at least i times and at most j times (i and j integer numbers, j greater then i). For example, xyz{2,4} matches the xy string followed by two to four of the z character.

---

## grep, egrep, fgrep

One of the most common uses of grep is to facilitate the inspecion of long files,using the regular expression as a filter applied to each line.

```bash
$ grep '^options' /etc/modprobe.d/alsa-base.conf

options snd-pcsp index=-2
options snd-usb-audio index=-2
options bt87x index=-2
options cx88_alsa index=-2
options snd-atiixp-modem index=-2
options snd-intel8x0m index=-2
options snd-via82xx-modem index=-2
```

The option can be placed before or after the regular expression. Other important grep options
are:

-c or --count Instead of displaying the search results, only display the total count for how many times a match occurs in any given file.

-i or --ignore-case Turn the search case-insensitive.

-f FILE or --file=FILE Indicate a file containing the regular expression to use.

-n or --line-number Show the number of the line.

-v or --invert-match Select every line, except those containing matches.

-H or --with-filename Print also the name of the file containing the line.

-z or --null-data Rather than have grep treat input and output data streams as separate lines (using the newline
by default) instead take the input or output as a sequence of lines. When combining output
from the find command using its -print0 option with the grep command, the -z or --null
-data option should be used to process the stream in the same manner.

```bash
$ find /usr/share/doc -type f -exec grep -i '3d modeling' "{}" \; | cut -c -100
artistic aspects of 3D modeling. Thus this might be the application you are
This major approach of 3D modeling has not been supported
oce is a C++ 3D modeling library. It can be used to develop CAD/CAM softwares, for instance
[FreeCad
```

---

## sed - stream editor

The purpose of the sed program is to modify text-based data in a non-interactive way. It means
that all the editing is made by predefined instructions, not by arbitrarily typing directly into a text
displayed on the screen.

```bash 
sed 's/old/new/'


sed 's/old/new/'      # replace first match in a line
sed 's/old/new/g'     # replace all matches in a line
sed '2d' file.txt     # delete line 2
sed -n '3p' file.txt  # print only line 3
```

---

## fdisk

### Understanding MBR and GPT
There are two main ways of storing partition information on hard disks. The first one is MBR
(Master Boot Record), and the second one is GPT (GUID Partition Table).

- MBR This is a remnant from the early days of MS-DOS (more specifically, PC-DOS 2.0 from 1983) and for decades was the standard partitioning scheme on PCs. The partition table is stored on the first sector of a disk, called the Boot Sector, along with a boot loader, which on Linux systems is usually the GRUB bootloader. But MBR has a series of limitations that hinder its use on modern systems, like the inability to address disks of more than 2 TB in size, and the limit of only 4 primary partitions per disk.

- GUID A partitioning system that addresses many of the limitations of MBR. There is no practical limit on disk size, and the maximum number of partitions are limited only by the operating system itself. It is more commonly found on more modern machines that use UEFI instead of the old PC BIOS.

### Managing MBR Partitions with FDISK

```bash
fdisk /dev/sda

Welcome to fdisk (util-linux 2.33.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.
Command (m for help):
```

### Printing the Current Partition Table

```bash
Command (m for help): p
Disk /dev/sda: 111.8 GiB, 120034123776 bytes, 234441648 sectors
Disk model: CT120BX500SSD1
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x97f8fef5
Device    Boot      Start     End       Sectors    Size    Id  Type
/dev/sda1           4096      226048942 226044847  107.8G  83    Linux
/dev/sda2           226048944 234437550 8388607    4G      82 Linux swap / Solaris
```

- Device => The device assigned to the partition.
- Boot => Shows whether the partition is “bootable” or not.
- Start => The sector where the partition starts.
- End => The sector where the partition ends.
- Sectors => The total number of sectors in the partition. Multiply it by the sector size to get the partition size in bytes.
- Size => The size of the partition in “human readable” format. In the example above, values are in gigabytes.
- Id => The numerical value representing the partition type.
- Type => The description for the partition type.

# fdisk Command Reference

| Command | Meaning |
|---|---|
| `m` | Display the help menu / list available commands |
| `p` | Print the current partition table |
| `n` | Create a new partition |
| `d` | Delete a partition |
| `t` | Change a partition type |
| `w` | Write changes to disk and exit |
| `q` | Quit without saving changes |
| `g` | Create a new empty GPT partition table |
| `o` | Create a new empty DOS/MBR partition table |
| `l` | List known partition types |
| `i` | Show information about a partition |
| `a` | Toggle the bootable flag |
| `F` | Display unpartitioned free space |
| `v` | Verify the partition table |
| `u` | Change display units |

## Most Important Commands

| Task | Command |
|---|---|
| View current partitions | `p` |
| Create a new partition | `n` |
| Delete a partition | `d` |
| Change partition type | `t` |
| Save changes | `w` |
| Exit without saving | `q` |

## Typical Workflow

```bash
sudo fdisk /dev/sdX
p
n
w
```