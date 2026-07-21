# Phase 2 — Configuring Nginx Virtual Hosts

## Objective

The objective of this phase is to configure Nginx Virtual Hosts that allow a single Linux server to host multiple websites.

A production web server commonly hosts multiple applications or department websites on the same infrastructure.

Instead of deploying separate servers, Nginx can use server blocks to determine which website should respond to incoming requests.

In this phase, Nginx will be configured to host:

- company.local
- hr.company.local
- it.company.local

Each website will have:

- Its own domain name.
- Its own document root.
- Its own Nginx configuration.

---

# Production Scenario

ABC Solutions has completed the initial website directory design.

The Linux server currently contains:

```
/var/www/

├── company
├── hr
└── it
```

The next requirement is configuring Nginx so users can access:

```
https://company.local
https://hr.company.local
https://it.company.local
```

The administrator must configure Nginx to correctly identify each request and serve the appropriate website.

---

# Understanding Nginx Virtual Hosts

A Virtual Host allows one web server to host multiple websites.

In Nginx, this is implemented using:

```
server {}
```

blocks.

Example:

```
Nginx Server

        |
        |
        ↓

server {}

company.local

        |

server {}

hr.company.local

        |

server {}

it.company.local
```

Each server block contains rules defining:

- Where the website files exist.
- Which domain should match.
- Which port should be used.
- How requests should be handled.

---

# How Nginx Selects a Website

When a browser sends a request:

Example:

```
https://hr.company.local
```

The browser sends:

```
Destination Port: 443

Host Header:
hr.company.local
```

Nginx performs matching:

```
Incoming Request

        |
        ↓

Listening Port

        |
        ↓

server_name Match

        |
        ↓

Correct server block selected

        |
        ↓

Website content returned
```

---

# Nginx Configuration Location

CentOS Stream 9 stores additional Nginx configurations in:

```
/etc/nginx/conf.d/
```

Files inside this directory are automatically loaded because:

```
/etc/nginx/nginx.conf
```

contains:

```nginx
include /etc/nginx/conf.d/*.conf;
```

---

# Creating Virtual Host Configurations

The following configuration files will be created:

```
/etc/nginx/conf.d/

├── company.conf
├── hr.conf
└── it.conf
```

---

# Company Virtual Host

Create:

```bash
vi /etc/nginx/conf.d/company.conf
```

Configuration:

```nginx
server {

    listen 443 ssl;
    listen [::]:443 ssl;

    server_name company.local;

    root /var/www/company;
    index index.html;

    ssl_certificate /etc/nginx/ssl/nginx.crt;
    ssl_certificate_key /etc/nginx/ssl/nginx.key;

}
```

---

# HR Virtual Host

Create:

```bash
vi /etc/nginx/conf.d/hr.conf
```

Configuration:

```nginx
server {

    listen 443 ssl;
    listen [::]:443 ssl;

    server_name hr.company.local;

    root /var/www/hr;
    index index.html;

    ssl_certificate /etc/nginx/ssl/nginx.crt;
    ssl_certificate_key /etc/nginx/ssl/nginx.key;

}
```

---

# IT Virtual Host

Create:

```bash
vi /etc/nginx/conf.d/it.conf
```

Configuration:

```nginx
server {

    listen 443 ssl;
    listen [::]:443 ssl;

    server_name it.company.local;

    root /var/www/it;
    index index.html;

    ssl_certificate /etc/nginx/ssl/nginx.crt;
    ssl_certificate_key /etc/nginx/ssl/nginx.key;

}
```

---

# Configuration Breakdown

## listen directive

Example:

```nginx
listen 443 ssl;
```

Defines:

- TCP port.
- Protocol type.
- SSL usage.

Meaning:

```
Accept HTTPS connections on port 443
```

---

## server_name directive

Example:

```nginx
server_name hr.company.local;
```

Defines which hostname should match this server block.

Incoming request:

```
Host: hr.company.local
```

matches:

```
server_name hr.company.local
```

---

## root directive

Example:

```nginx
root /var/www/hr;
```

Defines where website files are stored.

Request:

```
GET /
```

becomes:

```
/var/www/hr/index.html
```

---

## index directive

Example:

```nginx
index index.html;
```

Defines the default file returned when requesting a directory.

Example:

Request:

```
https://hr.company.local/
```

Nginx searches:

```
/var/www/hr/index.html
```

---

# Testing Nginx Configuration

Before applying changes:

```bash
nginx -t
```

Expected:

```
syntax is ok
test is successful
```

This prevents broken configurations from being loaded.

---

# Reloading Nginx

Apply the new configuration:

```bash
systemctl reload nginx
```

Why reload instead of restart?

Reload:

- Keeps existing connections.
- Loads new configuration.
- Minimizes service interruption.

Restart:

- Stops and starts the service.
- Can interrupt active users.

Production administrators prefer reload whenever possible.

---

# Verification

Check listening ports:

```bash
ss -tlnp | grep nginx
```

Expected:

```
LISTEN 0 511 0.0.0.0:443 nginx
```

---

Verify loaded configuration:

```bash
nginx -T
```

Confirm:

```
server_name company.local;

server_name hr.company.local;

server_name it.company.local;
```

---

# Troubleshooting

## Problem: Wrong website appears

Example:

```
Request:

https://hr.company.local

Shows:

company.local website
```

Possible causes:

- Incorrect server_name.
- Missing DNS/hosts entry.
- Default server responding.

Investigation:

```bash
nginx -T
```

Verify:

```nginx
server_name hr.company.local;
```

---

## Problem: Nginx configuration fails

Check:

```bash
nginx -t
```

Common causes:

- Missing semicolon.
- Incorrect file path.
- Duplicate directives.

---

## Problem: Website returns 403 Forbidden

Possible causes:

- Linux permissions.
- SELinux restrictions.

Investigation:

```bash
ls -l /var/www/hr
```

and:

```bash
ls -Z /var/www/hr
```

SELinux troubleshooting is covered in Phase 4.

---

# Verification Checklist

| Check | Status |
|-|-|
| Virtual Host files created | ✅ |
| Nginx configuration validated | ✅ |
| Nginx reloaded successfully | ✅ |
| Port 443 listening | ✅ |
| Domains mapped correctly | ✅ |
| Websites separated correctly | ✅ |

---

# Production Considerations

Real enterprise environments commonly use:

```
/etc/nginx/conf.d/

company.conf
api.conf
portal.conf
application.conf
```

or:

```
/etc/nginx/sites-available/
```

and:

```
/etc/nginx/sites-enabled/
```

The goal is the same:

- Keep configurations organized.
- Separate applications.
- Simplify troubleshooting.

---

# Key Takeaways

This phase demonstrated how Nginx can host multiple websites using Virtual Hosts.

Important concepts learned:

- One Nginx process can host many websites.
- Server blocks define website behavior.
- `server_name` controls hostname matching.
- `root` controls content location.
- Configuration validation prevents production outages.

The next phase will secure these websites using HTTPS and SSL/TLS certificates.
