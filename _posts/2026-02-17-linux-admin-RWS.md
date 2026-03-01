---
layout: post
title: "Linux System Administrator Real-World Scenario"
date: 2026-02-17 09:00:00 -0500
categories: Real-World Scenario-RWS
tages: linux admin real world scenario
image:
    path: /assets/img/headers/RWS.jpeg
---


# Production Web Server Performance Issue
---

Users report that:

- The web application is very slow
- Sometimes returns **502 Bad Gateway**
- Occasionally becomes completely unresponsive

You SSH into the server and observe:

- CPU usage: 100%
- Load average: 25
- RAM: 100% used
- Swap: 100% used

Your task:

1. Diagnose the issue  
2. Identify the root cause  
3. Stabilize the system  
4. Provide a long-term solution  
5. Improve security if needed  

---

# Complete Solution

---

## System Diagnostics

## Check CPU Usage

```bash
top
```

Or:

```bash
htop
```

To see top CPU-consuming processes:

```bash
ps aux --sort=-%cpu | head -10
```

---

## Check Memory Usage

```bash
free -m
```

If RAM and swap are both fully utilized, the system is under memory pressure.

---

## Check Load Average

```bash
uptime
```

A load average of 25 on a 2-core system means processes are waiting for CPU time — severe overload.

---

## Check Disk Space

```bash
df -h
```

Ensure no partitions are full.

---

## Check Disk I/O

```bash
iostat -x 1
```

If `%util` is near 100%, disk I/O is saturated.

---

# Check Services

## Verify Nginx

```bash
systemctl status nginx
```

## Verify MySQL

```bash
systemctl status mysql
```

## Verify Node Application

If using systemd:

```bash
systemctl status node-app
```

If using PM2:

```bash
pm2 list
pm2 monit
```

If Nginx returns 502 errors, the backend service (Node.js) is likely not responding.

---

# Log Analysis

## Nginx Logs

```bash
tail -f /var/log/nginx/error.log
tail -f /var/log/nginx/access.log
```

Look for:
- upstream timed out
- connection refused
- worker connection limits

---

## MySQL Logs

```bash
tail -f /var/log/mysql/error.log
```

Look for:
- Too many connections
- Out of memory
- InnoDB errors

---

## System Logs

```bash
journalctl -xe
```

Look for:

```
Out of memory: Kill process XXXX (node)
```

This indicates the OOM killer is terminating processes.

---

# Root Cause Analysis

Given:

- CPU 100%
- Load average 25
- RAM 100%
- Swap 100%

## Most Likely Cause

Memory exhaustion leading to swap thrashing.

When RAM is full:
- The system swaps to disk
- Disk I/O increases
- CPU waits for memory
- Backend becomes unresponsive
- Nginx returns 502

This is a classic **memory bottleneck scenario**.

---

# Immediate Stabilization

## Restart the Failing Service

If Node is failing:

```bash
systemctl restart node-app
```

Or:

```bash
pm2 restart all
```

If MySQL is consuming excessive memory:

```bash
systemctl restart mysql
```

---

## Reduce MySQL Memory Usage

Edit configuration:

```bash
sudo nano /etc/mysql/my.cnf
```

Adjust:

```
innodb_buffer_pool_size = 256M
max_connections = 50
```

Restart MySQL:

```bash
systemctl restart mysql
```

---

## Temporarily Increase Swap

```bash
fallocate -l 1G /swapfile2
chmod 600 /swapfile2
mkswap /swapfile2
swapon /swapfile2
```

⚠ This is a temporary mitigation only.

---

# Long-Term Solution

## Upgrade Server Resources

2GB RAM is insufficient for:
- Nginx
- Node.js
- MySQL
- OS overhead

Upgrade to:
- Minimum 4GB RAM
- Preferably 8GB for production

---

## Optimize MySQL

- Enable slow query log
- Add missing indexes
- Tune buffer sizes
- Limit max_connections

---

## Optimize Node.js

Run with memory limit:

```bash
node --max-old-space-size=512 app.js
```

Use cluster mode for better CPU utilization.

---

## Architectural Improvements

- Move MySQL to a separate server
- Add Redis caching
- Implement load balancing
- Use horizontal scaling

---

# Security Improvements

Open ports detected:

- 22 (SSH)
- 80 (HTTP)
- 3306 (MySQL)

## Problem

Port 3306 should NOT be publicly accessible.

Check listening services:

```bash
ss -tulpn
```

---

## Bind MySQL to Localhost

Edit:

```bash
sudo nano /etc/mysql/my.cnf
```

Set:

```
bind-address = 127.0.0.1
```

Restart MySQL:

```bash
systemctl restart mysql
```

---

## Configure Firewall (UFW)

```bash
ufw allow 22
ufw allow 80
ufw deny 3306
ufw enable
```

Allow MySQL only from internal IP if needed.

---

# Final Conclusion

Root Cause:

> The server is under-provisioned (2GB RAM) leading to memory exhaustion and swap thrashing. This causes high load, CPU saturation, backend failure, and 502 errors from Nginx.

Immediate Fix:

- Restart affected services
- Reduce memory usage
- Increase swap (temporary)

Permanent Fix:

- Upgrade RAM
- Optimize database and application
- Improve architecture
- Secure exposed services




