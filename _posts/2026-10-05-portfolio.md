---
layout: post
title: "Email Server"
date: 2026-10-05 09:00:00 -0500
categories: linux
tages: portfolio
image:
    path: /assets/img/headers/hero-linux-workstation.jpeg
---


# Portfolio Deployment

## Secure Docker Deployment on Ubuntu VPS

This document describes the deployment of a static portfolio website on an Ubuntu VPS using Docker, Nginx, SSH key authentication, firewall filtering, DNS, and TLS encryption.

The primary goal of the deployment is not only to make the application publicly accessible, but to keep the architecture simple, reproducible, and reasonably hardened.

---

# 1. Architecture Overview

The final architecture is:

```text
                    Internet
                       |
                       v
                 DNS Provider
                       |
                       v
              VPS Public IPv4
                       |
                       v
             Cloud Firewall
              22 / 80 / 443
                       |
                       v
                     UFW
              22 / 80 / 443
                       |
                       v
              Host Nginx
               80 / 443
                       |
              Reverse Proxy
                       |
                       v
              127.0.0.1:8080
                       |
                       v
                Docker Engine
                       |
                       v
            Portfolio Container
                       |
                       v
          Unprivileged Nginx
                       |
                       v
                Static Website
```

The Docker container is **not directly exposed to the Internet**.

The application is bound only to:

```text
127.0.0.1:8080
```

All public HTTP and HTTPS traffic is handled by the host Nginx reverse proxy.

---

# 2. Environment

The deployment uses:

- Ubuntu Server
- Hetzner Cloud VPS
- SSH public-key authentication
- Hetzner Cloud Firewall
- UFW
- Docker Engine
- Docker Compose
- Nginx reverse proxy
- Let's Encrypt TLS certificate
- Certbot
- Private GitHub repository
- GitHub Deploy Key
- External DNS provider

Example placeholders used throughout this document:

```text
SERVER_PUBLIC_IP        Public IPv4 address of the VPS
ADMIN_USER              Non-root administrative account
portfolio.example.com   Public domain
GITHUB_USERNAME         GitHub account
REPOSITORY              Private repository
```

Real server IP addresses, private SSH keys, email addresses, and other sensitive information are intentionally excluded.

---

# 3. Initial Server Access

The server was initially accessed as `root` using SSH public-key authentication.

Example:

```bash
ssh root@SERVER_PUBLIC_IP
```

After connecting, the operating system was updated:

```bash
apt update
apt upgrade
```

If required after package or kernel updates:

```bash
reboot
```

After reboot:

```bash
ssh root@SERVER_PUBLIC_IP
```

---

# 4. Administrative User

Daily administration should not be performed directly through the root account.

A dedicated administrative user was created:

```bash
adduser ADMIN_USER
```

The account was added to the `sudo` group:

```bash
usermod -aG sudo ADMIN_USER
```

Membership can be verified with:

```bash
groups ADMIN_USER
```

---

# 5. SSH Public Key for the Administrative User

The existing authorized SSH key was copied to the administrative account.

```bash
mkdir -p /home/ADMIN_USER/.ssh
cp /root/.ssh/authorized_keys /home/ADMIN_USER/.ssh/authorized_keys
```

Correct ownership was applied:

```bash
chown -R ADMIN_USER:ADMIN_USER /home/ADMIN_USER/.ssh
```

SSH directory permissions:

```bash
chmod 700 /home/ADMIN_USER/.ssh
chmod 600 /home/ADMIN_USER/.ssh/authorized_keys
```

Before modifying SSH security settings, login using the new account was tested in a **second terminal session**:

```bash
ssh ADMIN_USER@SERVER_PUBLIC_IP
```

Sudo access was verified:

```bash
sudo whoami
```

Expected output:

```text
root
```

The original SSH session should remain open until the new authentication path has been successfully tested.

---

# 6. SSH Hardening

A separate SSH configuration file was created:

```bash
sudo nano /etc/ssh/sshd_config.d/00-hardening.conf
```

Configuration:

```text
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
```

This configuration:

- disables remote root login;
- enables public-key authentication;
- disables SSH password authentication;
- disables keyboard-interactive authentication.

The SSH configuration was validated before applying it:

```bash
sudo sshd -t
```

No output indicates that the syntax is valid.

Effective configuration was inspected with:

```bash
sudo sshd -T | grep -E \
'permitrootlogin|pubkeyauthentication|passwordauthentication|kbdinteractiveauthentication'
```

Expected values:

```text
permitrootlogin no
pubkeyauthentication yes
passwordauthentication no
kbdinteractiveauthentication no
```

On this system, SSH uses systemd socket activation, therefore the SSH socket configuration was left intact instead of blindly enabling another service configuration.

A new SSH session was tested before closing the existing session.

---

# 7. Cloud Firewall

A firewall was configured at the VPS provider level.

Inbound traffic is restricted to:

```text
TCP 22    SSH
TCP 80    HTTP
TCP 443   HTTPS
```

All unnecessary inbound ports remain closed.

The application port `8080` is deliberately **not exposed through the cloud firewall**.

This provides an external filtering layer before traffic reaches the operating system.

---

# 8. UFW Host Firewall

UFW provides an additional firewall layer directly on the Ubuntu server.

Default policies:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

SSH was allowed before enabling UFW:

```bash
sudo ufw allow 22/tcp
```

UFW was then enabled:

```bash
sudo ufw enable
```

After Nginx was configured, HTTP and HTTPS were allowed:

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

Status:

```bash
sudo ufw status
```

Expected relevant rules:

```text
22/tcp     ALLOW
80/tcp     ALLOW
443/tcp    ALLOW
```

Port `8080` is not opened in UFW.

---

# 9. Docker Installation

Docker was installed using Docker's official Ubuntu repository rather than an unofficial package source.

Required packages:

```bash
sudo apt update
sudo apt install ca-certificates curl
```

Docker's repository signing key was installed under:

```text
/etc/apt/keyrings/
```

After adding the official Docker repository, the following components were installed:

```bash
sudo apt install \
docker-ce \
docker-ce-cli \
containerd.io \
docker-buildx-plugin \
docker-compose-plugin
```

Docker service status:

```bash
sudo systemctl status docker
```

Docker installation was verified:

```bash
sudo docker version
```

A test container was executed:

```bash
sudo docker run --rm hello-world
```

---

# 10. Docker Privilege Decision

The administrative account was intentionally **not added to the `docker` group**.

Instead, Docker commands are executed using:

```bash
sudo docker ...
```

or:

```bash
sudo docker compose ...
```

Membership in the Docker group effectively provides root-equivalent control over the host in many configurations.

For this small deployment, requiring `sudo` provides a clearer privilege boundary.

---

# 11. Application Directory

The application is stored under:

```text
/opt/portfolio
```

The directory was created:

```bash
sudo mkdir -p /opt/portfolio
```

Ownership was assigned to the administrative account:

```bash
sudo chown ADMIN_USER:ADMIN_USER /opt/portfolio
```

---

# 12. Private GitHub Repository Access

The application repository is private.

A dedicated SSH Deploy Key was created on the server instead of copying a personal SSH private key to the VPS.

Example:

```bash
ssh-keygen \
-t ed25519 \
-C "portfolio-deploy" \
-f ~/.ssh/portfolio_deploy
```

Private key:

```text
~/.ssh/portfolio_deploy
```

Public key:

```text
~/.ssh/portfolio_deploy.pub
```

Permissions:

```bash
chmod 600 ~/.ssh/portfolio_deploy
chmod 644 ~/.ssh/portfolio_deploy.pub
```

The **public key only** was added to:

```text
GitHub Repository
→ Settings
→ Deploy Keys
```

Write access was not enabled because the production server only needs to pull code.

The private key must never be uploaded to GitHub.

---

# 13. SSH Configuration for GitHub

A dedicated SSH host alias was configured:

```bash
nano ~/.ssh/config
```

Configuration:

```text
Host github-portfolio
    HostName github.com
    User git
    IdentityFile ~/.ssh/portfolio_deploy
    IdentitiesOnly yes
```

Permissions:

```bash
chmod 600 ~/.ssh/config
```

Authentication was tested:

```bash
ssh -T git@github-portfolio
```

The repository can then be cloned using the alias:

```bash
cd /opt/portfolio

git clone \
git@github-portfolio:GITHUB_USERNAME/REPOSITORY.git .
```

The alias is important because it ensures Git uses the dedicated deployment key rather than another SSH identity.

---

# 14. Application Container

The static website runs inside an unprivileged Nginx container.

Example Dockerfile:

```dockerfile
FROM nginxinc/nginx-unprivileged:alpine

COPY nginx.conf /etc/nginx/conf.d/default.conf

COPY --chown=nginx:nginx index.html /usr/share/nginx/html/index.html
COPY --chown=nginx:nginx css /usr/share/nginx/html/css
COPY --chown=nginx:nginx js /usr/share/nginx/html/js
COPY --chown=nginx:nginx assets /usr/share/nginx/html/assets

EXPOSE 8080

HEALTHCHECK --interval=30s \
            --timeout=3s \
            --start-period=5s \
            --retries=3 \
    CMD wget -q -O /dev/null http://127.0.0.1:8080/ || exit 1
```

Using `nginx-unprivileged` avoids running the web server process inside the container as root.

---

# 15. Container Nginx Configuration

Example:

```nginx
server {
    listen 8080;
    server_name _;

    root /usr/share/nginx/html;
    index index.html;

    server_tokens off;
    charset utf-8;

    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "DENY" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Permissions-Policy "camera=(), microphone=(), geolocation=()" always;

    add_header Content-Security-Policy
        "default-src 'self';
         script-src 'self';
         style-src 'self';
         img-src 'self' data:;
         font-src 'self';
         connect-src 'self';
         object-src 'none';
         frame-ancestors 'none';
         base-uri 'self';
         form-action 'self'"
        always;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location ~* \.(?:css|js|svg|jpg|jpeg|png|webp)$ {
        expires 7d;
        add_header Cache-Control "public, max-age=604800, immutable";
        try_files $uri =404;
    }

    location ~ /\. {
        deny all;
        return 404;
    }
}
```

The configuration disables server version disclosure and adds several HTTP security headers.

---

# 16. Docker Compose

Example `docker-compose.yml`:

```yaml
services:
  portfolio:
    build: .
    container_name: portfolio
    restart: unless-stopped

    ports:
      - "127.0.0.1:8080:8080"

    read_only: true

    tmpfs:
      - /tmp:size=16m,noexec,nosuid,nodev
      - /var/cache/nginx:size=16m,noexec,nosuid,nodev
      - /var/run:size=4m,noexec,nosuid,nodev

    security_opt:
      - no-new-privileges:true

    cap_drop:
      - ALL
```

The most important networking decision is:

```yaml
ports:
  - "127.0.0.1:8080:8080"
```

and **not**:

```yaml
ports:
  - "8080:8080"
```

Binding the published port to `127.0.0.1` prevents the application container from being directly reachable through the server's public network interface.

---

# 17. Building the Application

From the project directory:

```bash
cd /opt/portfolio
```

Build:

```bash
sudo docker compose build
```

Start:

```bash
sudo docker compose up -d
```

Verify:

```bash
sudo docker compose ps
```

or:

```bash
sudo docker ps
```

The container should eventually report:

```text
healthy
```

---

# 18. Local Container Test

Before exposing anything publicly, the application was tested locally from the server:

```bash
curl -I http://127.0.0.1:8080
```

Expected:

```text
HTTP/1.1 200 OK
```

The listening socket can be inspected with:

```bash
sudo ss -tulpn | grep 8080
```

The application should be bound to:

```text
127.0.0.1:8080
```

It should not be publicly bound to:

```text
0.0.0.0:8080
```

---

# 19. Host Nginx

Nginx was installed directly on the Ubuntu host:

```bash
sudo apt update
sudo apt install nginx
```

Status:

```bash
sudo systemctl status nginx
```

Listening ports can be inspected with:

```bash
sudo ss -tulpn | grep -E ':80|:8080'
```

The expected architecture is:

```text
0.0.0.0:80          Host Nginx
127.0.0.1:8080      Docker application
```

---

# 20. Reverse Proxy

A dedicated Nginx virtual host was created:

```bash
sudo nano /etc/nginx/sites-available/portfolio
```

Initial configuration:

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name portfolio.example.com;

    location / {
        proxy_pass http://127.0.0.1:8080;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

The default Nginx site was disabled:

```bash
sudo rm /etc/nginx/sites-enabled/default
```

The portfolio configuration was enabled:

```bash
sudo ln -s \
/etc/nginx/sites-available/portfolio \
/etc/nginx/sites-enabled/portfolio
```

Configuration validation:

```bash
sudo nginx -t
```

Expected:

```text
syntax is ok
test is successful
```

Nginx was reloaded:

```bash
sudo systemctl reload nginx
```

---

# 21. Reverse Proxy Verification

Before relying on external DNS, the reverse proxy was tested locally:

```bash
curl -I http://127.0.0.1
```

Expected:

```text
HTTP/1.1 200 OK
```

At this stage the request path is:

```text
curl
  |
  v
Host Nginx :80
  |
  | proxy_pass
  v
127.0.0.1:8080
  |
  v
Docker Container
  |
  v
Portfolio
```

---

# 22. DNS Configuration

The DNS provider originally contained parking/default records.

Default web-hosting records that conflicted with the VPS deployment were removed.

An `A` record was created:

```text
Type:      A
Host:      root domain
Value:     SERVER_PUBLIC_IP
TTL:       provider default
```

Conceptually:

```text
portfolio.example.com
        |
        | A record
        v
SERVER_PUBLIC_IP
```

Mail-related MX and SPF records were not removed unnecessarily.

---

# 23. DNS Verification

DNS resolution was checked from a separate machine:

```bash
dig portfolio.example.com
```

A cleaner check:

```bash
dig +short portfolio.example.com
```

Expected result:

```text
SERVER_PUBLIC_IP
```

This confirms that the domain resolves to the VPS rather than the previous/default hosting service.

---

# 24. Public HTTP Test

After allowing TCP port `80` through both firewall layers:

```bash
curl -I http://portfolio.example.com
```

The site was also tested from a browser.

At this point the complete HTTP path was:

```text
Client
  |
  v
DNS
  |
  v
Cloud Firewall
  |
  v
UFW
  |
  v
Host Nginx :80
  |
  v
127.0.0.1:8080
  |
  v
Docker
  |
  v
Portfolio
```

---

# 25. HTTPS Firewall Rule

Before enabling HTTPS, TCP `443` was allowed through the cloud firewall.

UFW:

```bash
sudo ufw allow 443/tcp
```

Verification:

```bash
sudo ufw status
```

Publicly allowed services are therefore limited to:

```text
22/tcp
80/tcp
443/tcp
```

---

# 26. Certbot

Certbot was installed using Snap:

```bash
sudo snap install --classic certbot
```

A command path was created if required:

```bash
sudo ln -s /snap/bin/certbot /usr/local/bin/certbot
```

Verification:

```bash
certbot --version
```

---

# 27. TLS Certificate

A Let's Encrypt certificate was requested using the Nginx plugin:

```bash
sudo certbot --nginx -d portfolio.example.com
```

Certbot:

1. verified control of the domain;
2. requested a certificate from Let's Encrypt;
3. configured Nginx for TLS;
4. enabled HTTPS;
5. configured HTTP-to-HTTPS redirection.

No private certificate material is stored in the repository.

---

# 28. HTTPS Verification

HTTPS:

```bash
curl -I https://portfolio.example.com
```

Expected:

```text
HTTP/2 200
```

or:

```text
HTTP/1.1 200 OK
```

HTTP redirect:

```bash
curl -I http://portfolio.example.com
```

Expected:

```text
HTTP/1.1 301 Moved Permanently
Location: https://portfolio.example.com/
```

This ensures that clients using plain HTTP are redirected to the encrypted endpoint.

---

# 29. Certificate Renewal

Automatic renewal was verified.

Timer inspection:

```bash
systemctl list-timers | grep certbot
```

A simulated renewal was performed:

```bash
sudo certbot renew --dry-run
```

Expected result:

```text
Congratulations, all simulated renewals succeeded
```

This verifies that future certificate renewal should work without manual intervention.

---

# 30. Final Network Architecture

The final request path is:

```text
User Browser
     |
     | HTTPS :443
     v
DNS Resolution
     |
     v
SERVER_PUBLIC_IP
     |
     v
Cloud Firewall
     |
     | 22 / 80 / 443 only
     v
UFW
     |
     | 22 / 80 / 443 only
     v
Host Nginx
     |
     | TLS termination
     | Reverse proxy
     v
127.0.0.1:8080
     |
     v
Docker Engine
     |
     v
Unprivileged Nginx Container
     |
     v
Static Portfolio
```

---

# 31. Security Decisions

Several security decisions were deliberately made during the deployment.

### SSH keys instead of passwords

Remote SSH password authentication is disabled.

Authentication requires possession of an authorized private key.

### Root SSH login disabled

Remote administration is performed through a dedicated account with controlled `sudo` escalation.

### Two firewall layers

Both the cloud provider firewall and the Ubuntu host firewall restrict inbound traffic.

### Minimal exposed ports

Only:

```text
22   SSH
80   HTTP
443  HTTPS
```

are publicly required.

### Docker application not publicly exposed

The container is bound to:

```text
127.0.0.1:8080
```

instead of:

```text
0.0.0.0:8080
```

Therefore external clients cannot directly bypass the reverse proxy and connect to the application port.

### Reverse proxy as the public entry point

Nginx is responsible for public HTTP/HTTPS traffic and TLS termination.

### Dedicated GitHub Deploy Key

The production server does not contain the developer's normal GitHub private key.

The deploy key is scoped specifically to the application repository.

### Read-only container filesystem

The application container uses:

```yaml
read_only: true
```

Temporary writable locations are provided through restricted `tmpfs` mounts.

### Linux capabilities dropped

The container configuration contains:

```yaml
cap_drop:
  - ALL
```

reducing unnecessary container privileges.

### No new privileges

The following option is enabled:

```yaml
security_opt:
  - no-new-privileges:true
```

to prevent processes from gaining additional privileges through mechanisms such as setuid binaries.

### Unprivileged container

The website is served using an unprivileged Nginx image rather than running the web server process as root.

### Docker group avoided

The administrative account is not placed in the `docker` group.

Docker administration requires `sudo`.

### HTTPS

Public web traffic is encrypted using a Let's Encrypt TLS certificate.

### Automatic certificate renewal

Certbot renewal was tested using:

```bash
sudo certbot renew --dry-run
```

---

# 32. Deployment Updates

For future application updates:

```bash
cd /opt/portfolio
```

Pull the latest version:

```bash
git pull
```

Rebuild:

```bash
sudo docker compose build
```

Recreate the application:

```bash
sudo docker compose up -d
```

Verify:

```bash
sudo docker compose ps
```

Test locally:

```bash
curl -I http://127.0.0.1:8080
```

Test through the production endpoint:

```bash
curl -I https://portfolio.example.com
```

This keeps deployment verification explicit instead of assuming that a successful build means the application is healthy.

---

# 33. Useful Troubleshooting Commands

Container status:

```bash
sudo docker ps
```

Compose status:

```bash
sudo docker compose ps
```

Container logs:

```bash
sudo docker logs portfolio
```

Follow logs:

```bash
sudo docker logs -f portfolio
```

Nginx configuration test:

```bash
sudo nginx -t
```

Nginx status:

```bash
sudo systemctl status nginx
```

Nginx logs:

```bash
sudo journalctl -u nginx
```

Recent Nginx events:

```bash
sudo journalctl -u nginx --since "30 minutes ago"
```

Listening TCP/UDP sockets:

```bash
sudo ss -tulpn
```

Relevant web ports:

```bash
sudo ss -tulpn | grep -E ':80|:443|:8080'
```

Firewall:

```bash
sudo ufw status verbose
```

DNS:

```bash
dig +short portfolio.example.com
```

Local application:

```bash
curl -I http://127.0.0.1:8080
```

Reverse proxy:

```bash
curl -I http://127.0.0.1
```

Production endpoint:

```bash
curl -I https://portfolio.example.com
```

Certificate information:

```bash
sudo certbot certificates
```

Renewal test:

```bash
sudo certbot renew --dry-run
```

---

# 34. Troubleshooting Methodology

The infrastructure should be debugged layer by layer rather than randomly changing configuration.

For example, if the website becomes unavailable:

```text
1. DNS
   |
   v
2. Cloud Firewall
   |
   v
3. UFW
   |
   v
4. Host Nginx
   |
   v
5. localhost:8080
   |
   v
6. Docker container
   |
   v
7. Application
```

Examples:

Check DNS:

```bash
dig +short portfolio.example.com
```

Check public Nginx:

```bash
curl -I https://portfolio.example.com
```

Check local reverse proxy:

```bash
curl -I http://127.0.0.1
```

Check application directly:

```bash
curl -I http://127.0.0.1:8080
```

Check container:

```bash
sudo docker compose ps
```

Check logs:

```bash
sudo docker compose logs
```

This approach helps identify which layer is failing before configuration changes are made.

---

# 35. Example Failure Scenarios

## Domain does not resolve

Check:

```bash
dig +short portfolio.example.com
```

If the expected server IP is not returned, investigate DNS before changing Docker or Nginx.

---

## Nginx returns 502 Bad Gateway

Check whether the application is available:

```bash
curl -I http://127.0.0.1:8080
```

Then:

```bash
sudo docker compose ps
```

And:

```bash
sudo docker compose logs
```

A `502` from host Nginx commonly means the reverse proxy is reachable but its upstream application is not.

---

## Container is restarting

Inspect:

```bash
sudo docker ps
```

Then:

```bash
sudo docker logs portfolio
```

Check the container configuration, filesystem permissions, healthcheck, and application startup.

---

## HTTPS fails but HTTP works

Check:

```bash
sudo nginx -t
```

Then:

```bash
sudo certbot certificates
```

And:

```bash
sudo ss -tulpn | grep ':443'
```

Also verify TCP `443` in both firewall layers.

---

## SSH stops working

Do not immediately assume the SSH daemon is broken.

Check the complete path:

```text
Client
  ↓
Network
  ↓
Cloud Firewall :22
  ↓
UFW :22
  ↓
SSH socket/service
  ↓
Authentication
```

This is also why an existing SSH session should remain open while SSH or firewall configuration is being changed.

---

# 36. Secrets and Repository Hygiene

The following information must never be committed to the repository:

```text
SSH private keys
Deploy private keys
Passwords
API tokens
Cloud credentials
Private TLS keys
Personal email addresses
Unnecessary infrastructure identifiers
```

Before pushing documentation:

```bash
git status
```

Inspect changes:

```bash
git diff
```

Search the repository for accidentally committed secrets when appropriate.

Only public configuration examples and sanitized infrastructure information should be included in documentation.

---

# 37. Future Improvements

Possible future improvements include:

- automated deployment through CI/CD;
- centralized logging;
- Docker image vulnerability scanning;
- unattended security updates;
- Fail2ban or equivalent SSH abuse protection where appropriate;
- monitoring and alerting;
- Docker resource limits;
- backup and restore procedures;
- deployment rollback strategy;
- HTTP security header review;
- automated infrastructure provisioning;
- separate production and staging environments.

These are intentionally outside the initial deployment scope.

The current goal is to maintain a small architecture that can be understood, operated, and troubleshot manually before adding further automation.

---

# 38. Result

The portfolio is deployed using a layered architecture:

```text
DNS
 ↓
Cloud Firewall
 ↓
Host Firewall
 ↓
Nginx + TLS
 ↓
Loopback-only Docker Port
 ↓
Unprivileged Container
 ↓
Application
```

The deployment demonstrates practical Linux system administration skills including:

- Linux server provisioning;
- user and privilege management;
- SSH hardening;
- public-key authentication;
- firewall configuration;
- Git SSH authentication;
- private repository deployment;
- Docker containerization;
- container hardening;
- Nginx administration;
- reverse proxy configuration;
- DNS administration;
- TLS certificate deployment;
- certificate renewal;
- service verification;
- log analysis;
- network troubleshooting.

The application is publicly available through HTTPS while the application container itself remains isolated from direct Internet access.