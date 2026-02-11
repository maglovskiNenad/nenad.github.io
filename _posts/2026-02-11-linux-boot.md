---
layout: post
title: "Linux Boot"
date: 2026-02-11 09:00:00 -0500
categories: homelab
tages: linux boot
image:
    path: /assets/img/headers/linuxBoot.png
---


# Linux Boot Process – Overview

This document provides a step-by-step overview of the Linux boot process.  
It shows the sequence from powering on the machine, through firmware, bootloader, kernel, initramfs, and finally to the user login.  

The purpose of this table is to give a clear and linear understanding of who controls the CPU at each stage, what components are active, and how the system progresses from bare hardware to a fully operational Linux environment.

---

| Step | What Happens | Who Controls CPU | Active Components | Notes |
|------|--------------|-----------------|-----------------|-------|
| **Power On** | CPU receives power and starts executing from hardcoded address | CPU | Firmware (BIOS/UEFI) | Nothing else is active yet |
| **Firmware (BIOS/UEFI)** | Performs hardware checks (RAM, disk, CPU), finds bootable device | Firmware | CPU + Firmware | GRUB, Kernel not active yet |
| **GRUB Bootloader** | Loads kernel (`vmlinuz`) and initramfs (`initrd.img`) into RAM, passes kernel parameters (`root=…`) | GRUB | GRUB in RAM | Kernel and userspace not yet active |
| **Kernel Start** | Kernel begins execution, initializes scheduler, memory, hardware | Kernel | Kernel, basic drivers | Prepares userspace |
| **Initramfs (/init script)** | Mini Linux in RAM, mounts real root filesystem, unlocks LUKS/LVM if needed | Kernel + `/init` | Initramfs filesystem and tools | Temporary root; prepares real root |
| **Real Root Filesystem** | Real root mounted from disk | Kernel | Disk as `/` | Initramfs hands off control |
| **PID 1 – systemd** | Kernel starts `/sbin/init` (systemd), systemd starts services: network, logs, GUI, etc. | Kernel + systemd (PID 1) | systemd | First userspace process; manages other processes |
| **Services** | Starts sshd, cron, docker, network manager, display manager | Kernel continues scheduling | Running services | System operational, ready for login |
| **Login** | TTY or GUI login prompt | Kernel schedules user sessions | User shell / GUI | Users can interact with system |


---

## Notes & Key Points
- **CPU Control:** At every stage, only one layer actively controls the CPU. The kernel always remains in control beneath everything.  
- **Initramfs:** Temporary mini-Linux in RAM, only active until the real root is mounted.  
- **Systemd (PID 1):** First userspace process that starts all services, but kernel still schedules everything.  
- **Chain of Responsibility:** Each stage prepares the next; the previous stage stops being active after handing off control.  

---

## Cheat Sheet (Quick Reference)
- **Power → Firmware → GRUB → Kernel → Initramfs → Root → systemd → Services → Login**  
- Kernel = foundation + ultimate CPU controller  
- Initramfs = temporary userspace  
- PID 1 = first userspace process, manages services  
