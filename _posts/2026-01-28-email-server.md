---
layout: post
title: "Email Server"
date: 2026-01-28 09:00:00 -0500
categories: homelab
tages: email server
image:
    path: /assets/img/headers/emailServer.png
---

# Building email server

This documentation describes email server and its core functionalities. It includes information on configuration, maintenance, and security settings. The goal is to provide a clear and practical reference for system administrators and technical staff who use or maintain the server. The document serves as a guide for efficient management and reliable email communication.

Before a single message can be sent, a choice must be made: pick your contender, define your tools, and decide what will power the heart of your mail server. This document begins where every reliable system starts – with the right foundation.

My choice:

Hetzner is a German hosting provider offering dedicated servers, cloud infrastructure, and data center services, known for reliable performance and strong price-to-to-performance ratio

![hatzner](/assets/lib/hatzner.png)

Mailcow is an open-source, Docker-based mail server stack that simplifies deploying and managing a secure, full-featured email system.

![mailcow](/assets/lib/mailCow.png)


### The first step is setting up the firewall:

|       Service       | Protocol | Port |    Container    |            Variable            |
|:--------------------|:--------:|-----:|:---------------:|:------------------------------:|
| Postfix SMTP        | TCP      |    25| postfix-mailcow |   ${SMTP_PORT}                 |
| Postfix SMTPS       | TCP      |   465| postfix-mailcow |   ${SMTPS_PORT}                |
| Postfix Submission  | TCP      |   587| postfix-mailcow |   ${SUBMISSION_PORT}           |
| Dovecot IMAP        | TCP      |   143| dovecot-mailcow |   ${IMAP_PORT}                 |
| Dovecot IMAPS       | TCP      |   993| dovecot-mailcow |   ${IMAPS_PORT}                |
| Dovecot POP3        | TCP      |   110| dovecot-mailcow |   ${POP_PORT}                  |
| Dovecot POP3S       | TCP      |   995| dovecot-mailcow |   ${POPS_PORT}                 |
| Dovecot ManageSieve | TCP      |  4190| dovecot-mailcow |   ${SIEVE_PORT}                |
| HTTP(S)             | TCP      |80/443|   nginx-mailcow |   ${HTTP_PORT}/${HTTPS_PORT}   |



### Minimal DNS configuration:

- Go on Networking.
- A table with the primary IP address will be displayed. The reverse DNS IP address need be changed to ***mail.domain.com***.
- It’s essential to configure specific settings on your domain’s name service, including the destination, to ensure the domain can correctly locate and communicate with the server.(***Often, the configuration or setup process depends on the provider.***)

Change the host nama

```bash
    nano /etc/hostname
```

![Hostname](/assets/lib/change_name1.png)


```bash
    nano /etc/hosts
```

![Hostname](/assets/lib/chanege_name2.png)


- Change time and reboot

```bash
    timedatectl set-timezone Europe/Berlin
    apt-get update
    apt-get upgrade -y
```

![Time](/assets/lib/change_time.png)


## Installation ***mailcow***

```bash
     curl -sSL https://get.docker.com/ | CHANNEL=stable sh
     systemctl enable --now docker
```

- Cheking if Docker is intsalled and ready for work

```bash
     docker run hello-world
```

And this is how you know

![Docker](/assets/lib/docker.png)


Need more one reboot

```bash
    reboot
```

Installing docker-compose

```bash
    LATEST=$(curl -Ls -w %{url_effective} -o /dev/null https://github.com/docker/compose/releases/latest) && \
    LATEST=${LATEST##*/} && \
    curl -L https://github.com/docker/compose/releases/download/$LATEST/docker-compose-$(uname -s)-$(uname -m) > /usr/local/bin/docker-compose
    chmod +x /usr/local/bin/docker-compose
```

- It is necessary to have 

```bash
    apt-get install git
```

1. Clone the repository:

```bash
    su
    umask 0022
    cd /opt
    git clone https://github.com/mailcow/mailcow-dockerized
    cd mailcow-dockerized
```

2. Generate the configuration file:
- A very important point, read carefully

```bash
    ./generate_config.sh
```

3. Adjust the configuration if necessary:

```bash
    nano mailcow.conf
```