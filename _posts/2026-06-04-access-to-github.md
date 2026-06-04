---
layout: post
title: "Allowing a Server to Access a Private GitHub Repository"
date: 2026-06-04 09:00:00 -0500
categories: linux
tages: access server private github repository
image:
    path: /assets/img/headers/github_key.png
---




# Allowing a Server to Access a Private GitHub Repository

This guide explains how to allow a Linux server to access a private GitHub repository using an **SSH Deploy Key**.

This setup is useful when a server needs to clone or pull code from a private GitHub repository during deployment.

---

## Why Use a Deploy Key?

A **Deploy Key** is an SSH key that gives a server access to one specific GitHub repository.

This is better than using a personal GitHub account on the server because:

- the server only gets access to one repository
- the key can be read-only
- personal GitHub credentials are not exposed
- it is simple and secure for deployment

By default, deploy keys are read-only unless write access is explicitly enabled.

---

## Create an SSH Key on the Server

Log in to the server and create a new SSH key:

```bash
ssh-keygen -t ed25519 -C "deploy-key-PROJECT_NAME"
```

When asked where to save the key, use a custom filename:

```bash
/home/USERNAME/.ssh/PROJECT_NAME_deploy_key
```

Example format:

```bash
/home/USERNAME/.ssh/PROJECT_NAME_deploy_key
```

When asked for a passphrase, press **Enter** to leave it empty.

This is useful if the server needs to run `git pull` automatically without asking for a password.

---

## Display the Public Key

Show the public key with:

```bash
cat ~/.ssh/PROJECT_NAME_deploy_key.pub
```

The output will look similar to this:

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI... deploy-key-PROJECT_NAME
```

Copy the full output.

Important:

- the `.pub` file is the public key
- the file without `.pub` is the private key
- only the public key should be added to GitHub
- never share the private key

---

## Add the Public Key to GitHub

Open the private GitHub repository in the browser.

Go to:

```text
Repository → Settings → Deploy keys → Add deploy key
```

Fill in the fields:

```text
Title: server-deploy-key
Key: paste the public key here
```

If the server only needs to clone or pull the repository, do **not** enable:

```text
Allow write access
```

Then click:

```text
Add key
```

---

## Create an SSH Config Entry

On the server, open the SSH config file:

```bash
nano ~/.ssh/config
```

Add the following configuration:

```sshconfig
Host github-PROJECT_NAME
    HostName github.com
    User git
    IdentityFile ~/.ssh/PROJECT_NAME_deploy_key
    IdentitiesOnly yes
```

Explanation:

```text
Host github-PROJECT_NAME
```

This is a custom shortcut name. It will be used instead of `github.com`.

```text
HostName github.com
```

This tells SSH that the real server is GitHub.

```text
User git
```

GitHub SSH connections always use the user `git`.

```text
IdentityFile ~/.ssh/PROJECT_NAME_deploy_key
```

This tells SSH which private key to use.

```text
IdentitiesOnly yes
```

This forces SSH to use only this key for this connection.

---

## Set Correct File Permissions

Run the following commands:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/config
chmod 600 ~/.ssh/PROJECT_NAME_deploy_key
chmod 644 ~/.ssh/PROJECT_NAME_deploy_key.pub
```

These permissions are important because SSH may refuse to use keys if the files are too open.

---

## Test the SSH Connection

Test the connection with:

```bash
ssh -T github-PROJECT_NAME
```

If everything is configured correctly, GitHub should recognize the key.

You may see a message similar to:

```text
Hi USERNAME/REPOSITORY_NAME! You've successfully authenticated...
```

This means the server can authenticate with GitHub using the deploy key.

---

## Clone the Private Repository

Use the custom SSH host from the SSH config.

Instead of this:

```bash
git clone git@github.com:USERNAME/REPOSITORY_NAME.git
```

use this:

```bash
git clone git@github-PROJECT_NAME:USERNAME/REPOSITORY_NAME.git
```

---

## If the Repository Already Exists on the Server

If the repository already exists on the server, go into the project directory:

```bash
cd /path/to/project
```

Change the remote URL:

```bash
git remote set-url origin git@github-PROJECT_NAME:USERNAME/REPOSITORY_NAME.git
```

Check the current remote URL:

```bash
git remote -v
```

Then test pulling the code:

```bash
git pull
```

---

## Common Problems

### Permission Denied

If you see this error:

```text
Permission denied (publickey)
```

check the following:

- the public key was added to the correct GitHub repository
- the private key exists on the server
- the `IdentityFile` path in `~/.ssh/config` is correct
- file permissions are correct
- the custom host is being used, for example `github-PROJECT_NAME`
- the remote URL is not still using `github.com`

---

### Wrong Remote URL

If the remote still looks like this:

```text
git@github.com:USERNAME/REPOSITORY_NAME.git
```

SSH may not use the custom deploy key.

Change it to:

```text
git@github-PROJECT_NAME:USERNAME/REPOSITORY_NAME.git
```

Use:

```bash
git remote set-url origin git@github-PROJECT_NAME:USERNAME/REPOSITORY_NAME.git
```

---

### Do Not Copy the Private Key to GitHub

Only this file goes to GitHub:

```text
PROJECT_NAME_deploy_key.pub
```

This file must stay only on the server:

```text
PROJECT_NAME_deploy_key
```

The private key is like a password. Do not share it.

---

After that, the server can run:

```bash
git clone
```

or:

```bash
git pull
```

without needing a personal GitHub login.