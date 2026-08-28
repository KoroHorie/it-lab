# Nginx Multi-Website Hosting — Cheatsheet

## Project Overview

Project 2 expanded a single Nginx web server into a multi-site web hosting environment.

The server was configured to host:

```
company.local

hr.company.local

it.company.local
```

using:

- Nginx Virtual Hosts
- HTTPS
- SELinux Contexts
- HTTP → HTTPS Redirects

---

# Nginx Service Management

## Check Nginx Status

```bash
systemctl status nginx
```

Purpose:

Verify whether the Nginx service is running.

---

## Start Nginx

```bash
systemctl start nginx
```

---

## Stop Nginx

```bash
systemctl stop nginx
```

---

## Restart Nginx

```bash
systemctl restart nginx
```

Use when:

- Service state must be completely restarted.
- Reload is not enough.

---

## Reload Nginx

```bash
systemctl reload nginx
```

Preferred when:

- Configuration changes are made.
- Service interruption should be avoided.

---

# Configuration Validation

Before applying changes:

```bash
nginx -t
```

Checks:

- Syntax errors.
- Missing files.
- Invalid directives.

Expected:

```
syntax is ok
test is successful
```

---

# Viewing Loaded Configuration

Show complete active configuration:

```bash
nginx -T
```

Useful for:

- Finding active server blocks.
- Checking included files.
- Troubleshooting wrong websites.

---

# Checking Listening Ports

```bash
ss -tlnp | grep nginx
```

Example output:

```
*:80
*:443
```

Meaning:

Nginx is listening for:

```
HTTP  → Port 80

HTTPS → Port 443
```

---

# Website Directory Structure

Project layout:

```
/var/www/

├── company
│   └── index.html
│
├── hr
│   └── index.html
│
└── it
    └── index.html
```

---

# Creating Website Directories

Example:

```bash
mkdir -p /var/www/company
mkdir -p /var/www/hr
mkdir -p /var/www/it
```

---

# Checking Website Files

```bash
tree /var/www
```

or:

```bash
ls -lR /var/www
```

---

# Nginx Virtual Host Location

CentOS loads additional configurations from:

```
/etc/nginx/conf.d/
```

Because:

```nginx
include /etc/nginx/conf.d/*.conf;
```

exists inside:

```
/etc/nginx/nginx.conf
```

---

# Virtual Host Important Directives

## listen

Example:

```nginx
listen 443 ssl;
```

Defines:

- Listening port.
- Protocol.
- SSL usage.

---

## server_name

Example:

```nginx
server_name hr.company.local;
```

Defines:

Which hostname should match this server block.

---

## root

Example:

```nginx
root /var/www/hr;
```

Defines:

Where website files are stored.

---

## index

Example:

```nginx
index index.html;
```

Defines:

Default file returned.

---

# Testing Websites

Using curl:

## HTTPS

```bash
curl -k https://company.local
```

`-k` allows testing self-signed certificates.

---

## Checking HTTP Redirect

```bash
curl -I http://company.local
```

Expected:

```
HTTP/1.1 301 Moved Permanently

Location: https://company.local/
```

---

# HTTP vs HTTPS Troubleshooting

## Check Listening Ports

```bash
nginx -T | grep listen
```

Example:

```
listen 80;

listen 443 ssl;
```

Remember:

Port 80 and 443 are separate configurations.

---

## Finding Default HTTP Website

Check:

```bash
nginx -T
```

Look for:

```nginx
server {

listen 80;

root /usr/share/nginx/html;

}
```

This means HTTP is serving:

```
/usr/share/nginx/html
```

---

# SELinux Troubleshooting

## Check SELinux Context

Example:

```bash
ls -Zd /var/www/company
```

Problem:

```
var_t
```

Expected:

```
httpd_sys_content_t
```

---

# Apply Correct SELinux Context

Define rule:

```bash
semanage fcontext \
-a \
-t httpd_sys_content_t \
"/var/www(/.*)?"
```

Apply labels:

```bash
restorecon -Rv /var/www
```

---

# Verify SELinux Label

```bash
ls -Zd /var/www/company
```

Expected:

```
httpd_sys_content_t
```

---

# Troubleshooting Workflow

Always follow:

```
Observe

↓

Verify

↓

Investigate

↓

Determine Root Cause

↓

Plan

↓

Implement Solution

↓

Verify Solution

↓

Prevent Future Recurrence
```

---

# Common Problems

## Problem

```
403 Forbidden
```

Check:

1. Nginx status

```bash
systemctl status nginx
```

2. Files exist

```bash
ls -l /var/www
```

3. Permissions

```bash
ls -ld /var/www/company
```

4. SELinux context

```bash
ls -Zd /var/www/company
```

---

## Problem

Wrong website appears.

Check:

```bash
nginx -T
```

Verify:

```nginx
server_name
```

and:

```nginx
root
```

---

## Problem

HTTP shows old/default website.

Check:

```bash
nginx -T
```

Look for:

```nginx
listen 80;
```

and:

```nginx
root /usr/share/nginx/html;
```

---

# Final Architecture

```
                 Client

                   |
                   |

              HTTP :80

                   |
                   ↓

              301 Redirect

                   |
                   ↓

             HTTPS :443

                   |
                   ↓

             Nginx Virtual Hosts


        -----------------------------

        |             |             |

        ↓             ↓             ↓

   company.local  hr.company.local  it.company.local


        |             |             |

        ↓             ↓             ↓


 /var/www/company /var/www/hr /var/www/it
```

---

# Project 2 Skills Practiced

- Nginx Virtual Hosts
- Server Blocks
- HTTPS Configuration
- HTTP Redirects
- Nginx Troubleshooting
- SELinux Context Management
- Linux Access Control Investigation
- Production Troubleshooting Workflow
