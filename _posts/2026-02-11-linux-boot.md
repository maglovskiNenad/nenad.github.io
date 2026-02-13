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

# Real flow of events that actually happens inside the machine.

## The moment you press the Power button

When you press the power button, the CPU is essentially “dumb.” It does not know what Linux is, what Windows is, what a disk is, what a file system is. 
It knows only one thing:

When it receives power – it begins executing instructions from a predefined memory address.
That address is defined by the hardware manufacturer and points to the firmware.

## Firmware: BIOS or UEFI

Today, almost exclusively UEFI is used, while the older system was BIOS.

Firmware is a small program stored on the motherboard (on a special chip). It is the first software that the CPU executes.

Its job is:
	•	to initialize RAM
	•	to check the CPU
	•	to initialize graphics
	•	to check disks
	•	to enable the keyboard
	•	to perform POST (Power-On Self Test)

At this moment, there is still no operating system. Only hardware and firmware.
If some hardware component does not work — this is where you get those familiar beep codes.

## Finding the boot device

The firmware must now decide:

From which device will I load the operating system?

It can be:
	•	SSD
	•	HDD
	•	USB
	•	network (PXE boot)

In UEFI systems, there is a special small partition called the EFI System Partition (ESP). It is usually formatted as FAT32 and contains bootloaders.
This brings us to the next key component.

## Bootloader

On Linux systems, this is most commonly:

GRUB (GRand Unified Bootloader)

The bootloader is a program that:
	•	knows how to read a file system
	•	knows where the kernel is located
	•	knows how to load the kernel into memory
	•	allows choosing the operating system (dual boot)
	•	can pass parameters to the kernel

It is important to understand:

The firmware does not know what ext4 is, does not know what a Linux kernel is.
It only knows how to load an executable file from the EFI partition.
The bootloader is the first component that begins to understand the “operating system.”

## Loading the Kernel

When you select Linux in GRUB, the bootloader:
	1.	Loads the Linux kernel file (usually /boot/vmlinuz-...)
	2.	Loads initramfs
	3.	Passes control to the kernel

From that moment on, firmware and bootloader have finished their job.
Now the Linux kernel takes control of the machine.

## What the kernel actually does when it starts

When the kernel is loaded into RAM, it:
	•	switches the CPU from real mode to protected/long mode
	•	sets up memory management (paging)
	•	detects CPU features
	•	initializes the scheduler
	•	sets up interrupt handling
	•	initializes device drivers

Now, for the first time, the system has:
	•	control over memory
	•	control over processes
	•	control over hardware

But there is still no user space.
The kernel is the heart of the system — but without user space, the system is practically unusable.

## Initramfs – a temporary mini system

Here comes a very important part that people often skip:
initramfs
It is a small temporary file system loaded into RAM together with the kernel.

Why is it needed?
Because the kernel may not immediately know how to access the real root file system.

For example:
	•	Root is on LVM
	•	Root is encrypted
	•	Root is on RAID
	•	Root is on an NVMe disk that requires a special driver

Initramfs contains a minimal userspace and the necessary drivers to enable access to the real root file system.

It:
	•	loads modules
	•	mounts the real root
	•	performs switch_root
	•	hands control over to the real system

## Mounting the root filesystem

When the kernel successfully mounts the real root (for example /dev/sda2 or /dev/nvme0n1p2), then:
The temporary initramfs disappears from the picture.
Now the system transitions to the real file system from the disk.
And now we come to one of the most important moments in the entire boot process.

## Starting PID 1

The kernel must now start the first userspace process.

It starts the program defined as init.

In modern distributions, this is:

systemd

This is the process with PID 1.

Why is PID 1 so important?

Because:
	•	all other processes originate from it
	•	if PID 1 dies → the system crashes
	•	it controls the service system


## What systemd does

Systemd:
	•	mounts additional file systems
	•	starts udev (device manager)
	•	starts the network service
	•	starts logging
	•	starts the graphical server
	•	starts all services (ssh, cron, docker…)

It uses the concept of “unit files.”
It parallelizes the boot process.
Old init systems worked sequentially.
Systemd works in parallel, and that is why boot today is much faster.

## From CLI to GUI

If the system is a server:
	•	you will get a TTY login prompt

If it is a desktop:
Systemd will start a display manager (for example gdm, sddm).
Here the graphical stack begins:
	•	display server (Wayland/X11)
	•	login manager
	•	desktop environment

At that moment, the system is fully operational.


# Now let’s look at everything as a chain of control

It is important that you understand one essential thing:

The boot process is a process of gradually taking control.
	1.	Hardware gives control to firmware
	2.	Firmware gives control to the bootloader
	3.	Bootloader gives control to the kernel
	4.	Kernel gives control to the init system
	5.	The init system starts userspace

This is a chain of authority.

Each level knows more than the previous one.

## What is most important to deeply understand?

1.	The kernel is NOT the operating system — it is the core.
2.	The bootloader is NOT part of the kernel.
3.	Firmware is NOT part of the OS.
4.	Initramfs exists because the kernel initially does not have all the information.
5.	PID 1 is the absolute foundation of userspace.

## The key mental model

Imagine booting as turning on a city:
	•	Power arrives → the city receives energy
	•	Central control (firmware) checks the infrastructure
	•	The gate (bootloader) decides which city to activate
	•	The mayor (kernel) takes control
	•	The administration (systemd) starts the services
	•	The people (processes) begin working


