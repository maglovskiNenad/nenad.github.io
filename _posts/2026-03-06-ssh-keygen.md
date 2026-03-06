---
layout: post
title: "Copying an SSH Key from Windows to a Linux Server"
date: 2026-03-05 09:00:00 -0500
categories: linux
tages: ssh-keygen server connection Windows Linux
image:
    path: /assets/img/headers/ssh_keygen.png
---

# Copying an SSH Key from Windows to a Linux Server

## Problem

When you try to run the following command on Windows:

```bash
    ssh-keygen
    ssh-copy-id -i <public-key> user@cloud_server
```

you may get an error like this:

```bash
    ssh-copy-id : The term 'ssh-copy-id' is not recognized as the name of a cmdlet, function, script file, or operable program.
```

## Why this happens

This is normal on Windows.
ssh-copy-id is a standard Linux/Unix utility and is usually not available in PowerShell or CMD. The problem is not with your SSH key. The issue is that Windows does not recognize the ssh-copy-id command

## Solution

Check whether you already have SSH keys

```bash
    dir $HOME\.ssh
```

Look for files such as:
- id_rsa and id_rsa.pub
- id_ed25519 and id_ed25519.pub

If the key does not exist create a new key with:

```bash
    ssh-keygen -t ed25519
```

You can just press Enter through the prompts if you do not want custom settings.

## Manually copy the public key to the Linux server
Since ssh-copy-id is not available, you can manually add the public key to the server. If you are using an ed25519 key

```bash
    type $HOME\.ssh\id_ed25519.pub | ssh user@cloud_server "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys"
```

If you are using an rsa key:

```bash
    type $HOME\.ssh\id_rsa.pub | ssh user@cloud_server "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys"
```

### What this command does

- reads your public key from the Windows machine
- sends it to the Linux server
- appends it to the ~/.ssh/authorized_keys file
- sets the correct permissions

## Test passwordless SSH login

```bash
    ssh user@cloud_server
```

If everything is configured correctly, you should be able to log in without entering the server password
(unless your private key is protected with a passphrase).

## If type ... | ssh ... does not work in PowerShell
Try this PowerShell alternative:

```bash
    Get-Content $HOME\.ssh\id_ed25519.pub | ssh user@cloud_server "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys"
```

```bash
    Get-Content $HOME\.ssh\id_rsa.pub | ssh user@cloud_server "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys"
```

On Windows, it is normal for ssh-copy-id to be unavailable in PowerShell.
The simplest workaround is to manually send the public key to the Linux server and append it to authorized_keys.