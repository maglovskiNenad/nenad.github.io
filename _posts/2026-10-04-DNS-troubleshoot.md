---
layout: post
title: "DNS"
date: 2026-10-04 09:00:00 -0500
categories: linux
tages: dns
image:
    path: /assets/img/headers/DNS.png
---

# DNS Troubleshooting – `www` Subdomain Not Resolving

## 1. Problem Description

The main domain was accessible:

```text
https://example-domain.com
```

but the `www` subdomain was not:

```text
https://www.example-domain.com
```

The goal was to determine whether the issue originated from:

- DNS configuration
- Web server configuration
- TLS/SSL configuration
- Firewall or network connectivity
- Application/container configuration

---

## 2. Initial DNS Verification

First, verify whether both hostnames resolve correctly.

```bash
dig example-domain.com
```

```bash
dig www.example-domain.com
```

Alternative:

```bash
host example-domain.com
host www.example-domain.com
```

Expected result:

```text
example-domain.com      -> SERVER_PUBLIC_IP
www.example-domain.com  -> example-domain.com
```

The real public server IP should not be included in public documentation. Use placeholders such as:

```text
SERVER_PUBLIC_IP
```

instead.

---

## 3. DNS Configuration

The DNS provider must contain a record for the `www` hostname.

Example configuration:

```text
Type:   CNAME
Host:   www
Value:  example-domain.com
TTL:    Default
```

This creates the following relationship:

```text
www.example-domain.com
        |
        v
example-domain.com
        |
        v
SERVER_PUBLIC_IP
```

The root domain usually uses an `A` record:

```text
Type:   A
Host:   @
Value:  SERVER_PUBLIC_IP
```

## 4. Check DNS Resolution From the Server

After updating DNS records:

```bash
dig +short example-domain.com
```

```bash
dig +short www.example-domain.com
```

A correct result could look conceptually like:

```text
SERVER_PUBLIC_IP
```

or:

```text
example-domain.com.
SERVER_PUBLIC_IP
```

If `www.example-domain.com` returns no result, the DNS configuration is still incorrect or DNS propagation has not completed.

---

## 5. Verify Web Server Configuration

After DNS resolves correctly, verify that Nginx accepts requests for both hostnames.

Example:

```nginx
server {
    listen 80;
    server_name example-domain.com www.example-domain.com;

    location / {
        proxy_pass http://APP_CONTAINER:APP_PORT;
    }
}
```

Before applying changes, always validate the configuration:

```bash
sudo nginx -t
```

Expected output:

```text
syntax is ok
test is successful
```

Only after successful validation should Nginx be reloaded:

```bash
sudo systemctl reload nginx
```

Using `reload` is preferable to a full restart when only configuration has changed because existing connections are not unnecessarily interrupted.

---

## 6. Recommended `www` Redirect

If the preferred address is:

```text
https://example-domain.com
```

then `www` can be redirected permanently to the canonical domain.

Example:

```nginx
server {
    listen 80;
    server_name www.example-domain.com;

    return 301 https://example-domain.com$request_uri;
}
```

The request flow becomes:

```text
www.example-domain.com
        |
        | HTTP 301
        v
https://example-domain.com
```

This avoids having two different URLs serving identical content.

---

## 7. SSL/TLS Verification

The TLS certificate should include both:

```text
example-domain.com
www.example-domain.com
```

If Certbot is used:

```bash
sudo certbot certificates
```

The certificate should list both DNS names.

Example sanitized output:

```text
Domains:
  example-domain.com
  www.example-domain.com
```

Never include private keys or the content of files such as:

```text
/etc/letsencrypt/live/example-domain.com/privkey.pem
```

Private keys must never be copied into documentation, Git repositories, tickets, or chat logs.

---

## 8. Test HTTPS

Test the primary domain:

```bash
curl -I https://example-domain.com
```

Then test the `www` hostname:

```bash
curl -I https://www.example-domain.com
```

If `www` redirects to the primary domain, the expected response is similar to:

```text
HTTP/2 301
location: https://example-domain.com/
```

The final domain should return a successful HTTP status such as:

```text
HTTP/2 200
```

---

## 9. Verify Incoming Network Traffic

If DNS resolves correctly but the website still cannot be reached, verify whether packets are actually reaching the server.

For HTTP and HTTPS traffic:

```bash
sudo tcpdump -nn -i any 'port 80 or port 443'
```

The `-nn` option prevents hostname and service-name resolution and makes the output easier to troubleshoot.

Sanitized example:

```text
CLIENT_PUBLIC_IP.CLIENT_PORT > SERVER_PUBLIC_IP.443
```

Do not publish real client IP addresses in screenshots or public documentation.

Replace them with placeholders such as:

```text
CLIENT_PUBLIC_IP
SERVER_PUBLIC_IP
```

---

## 10. Check Nginx Logs

Monitor access logs:

```bash
sudo tail -f /var/log/nginx/access.log
```

Monitor error logs:

```bash
sudo tail -f /var/log/nginx/error.log
```

Example sanitized log entry:

```text
CLIENT_PUBLIC_IP - - [DATE] "GET / HTTP/1.1" 200
```

When copying logs into documentation, remove or replace:

- public client IP addresses
- session IDs
- cookies
- authentication headers
- tokens
- API keys
- internal hostnames

---

## 11. Docker Considerations

Changing DNS or Nginx configuration does **not normally require rebuilding the Docker image**.

A Docker rebuild is generally required only if:

- application source code changed
- Dockerfile changed
- build-time configuration changed
- application dependencies changed

For DNS or reverse-proxy changes, the application container can usually continue running.

Check running containers:

```bash
docker ps
```

Example sanitized output:

```text
CONTAINER_ID   IMAGE_NAME   STATUS      PORTS
xxxxxxxxxxxx   app-image    Up          APP_PORT
```

Avoid publishing actual container IDs if the documentation is intended for public repositories.

---

## 12. Troubleshooting Flow

The recommended investigation order is:

```text
Client
  |
  v
DNS resolution
  |
  v
Public server IP
  |
  v
Firewall
  |
  v
Ports 80 / 443
  |
  v
Nginx
  |
  v
TLS certificate
  |
  v
Reverse proxy
  |
  v
Docker container
  |
  v
Application
```

This order prevents unnecessary application troubleshooting when the request never reaches the server.

---

## 13. Commands Used

```bash
dig example-domain.com
dig www.example-domain.com

dig +short example-domain.com
dig +short www.example-domain.com

sudo nginx -t
sudo systemctl reload nginx

sudo certbot certificates

curl -I https://example-domain.com
curl -I https://www.example-domain.com

sudo tcpdump -nn -i any 'port 80 or port 443'

sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log

docker ps
```

---