# Phase 3 — Expanding HTTPS Support for Multiple Websites

## Objective

The objective of this phase is to extend the existing HTTPS configuration from Project 1 and apply it to multiple Nginx Virtual Hosts.

Instead of creating a new web server or new SSL infrastructure, the existing Nginx HTTPS configuration is reused.

The goal is to verify that multiple websites can securely run through HTTPS on the same Linux server.

---

# Starting Point

Before this phase:

- Nginx was already running.
- HTTPS was already configured from Project 1.
- SSL certificate files already existed.

Existing certificate files:

```
/etc/nginx/ssl/

├── nginx.crt
└── nginx.key
```

The project continues by reusing the existing SSL configuration.

---

# Production Scenario

ABC Solutions successfully deployed multiple websites using Nginx Virtual Hosts.

The next requirement is ensuring every website is accessible securely through HTTPS.

Required websites:

```
https://company.local

https://hr.company.local

https://it.company.local
```

The administrator must verify:

- HTTPS connectivity.
- Correct Virtual Host selection.
- Correct website content.
- Secure communication.

---

# HTTPS Virtual Host Design

Each website uses the same HTTPS listener:

```
Nginx

Port 443 HTTPS

        |
        |
        ↓

server_name company.local

        |
        ↓

/var/www/company


server_name hr.company.local

        |
        ↓

/var/www/hr


server_name it.company.local

        |
        ↓

/var/www/it
```

Although all websites use the same Nginx process and port, Nginx separates them using the requested hostname.

---

# Reusing Existing SSL Configuration

The existing SSL directives from Project 1 are included in each Virtual Host.

Example:

```nginx
ssl_certificate /etc/nginx/ssl/nginx.crt;

ssl_certificate_key /etc/nginx/ssl/nginx.key;
```

The certificate and key are shared by the websites in this lab environment.

---

# Applying HTTPS Configuration

After creating the Virtual Host configurations:

Validate:

```bash
nginx -t
```

Expected:

```
syntax is ok

test is successful
```

Apply changes:

```bash
systemctl reload nginx
```

Verify:

```bash
systemctl status nginx
```

Expected:

```
active (running)
```

---

# Verifying HTTPS Websites

Each website was tested using HTTPS.

---

## Company Portal

Request:

```
https://company.local
```

Expected:

```
ABC Solutions

Welcome to Company Portal
```

---

## HR Portal

Request:

```
https://hr.company.local
```

Expected:

```
Human Resources Department

Welcome to HR Portal
```

---

## IT Portal

Request:

```
https://it.company.local
```

Expected:

```
IT Department

Welcome to IT Portal
```

---

# Incident Investigation — HTTP and HTTPS Returned Different Content

During testing, an unexpected behavior was discovered.

Accessing:

```
http://company.local
```

returned:

```
ABC Solutions

Welcome to Linux Web Server
```

However:

```
https://company.local
```

returned:

```
Correct Company Website
```

---

# Initial Question

The question was:

> Are HTTP and HTTPS using different applications?

The answer:

No.

They are both handled by the same Nginx service.

However, they are different listeners with different configurations.

---

# Investigation

The loaded Nginx configuration was checked:

```bash
nginx -T | grep -n "listen"
```

Result:

```
listen 80;

listen 443 ssl;
```

This showed that Nginx had:

HTTP:

```
Port 80
```

and HTTPS:

```
Port 443
```

configured separately.

---

# Finding the HTTP Configuration

The default HTTP server block was identified:

```nginx
server {

    listen 80;

    server_name _;

    root /usr/share/nginx/html;

}
```

This was the reason HTTP displayed:

```
ABC Solutions

Welcome to Linux Web Server
```

The content came from:

```
/usr/share/nginx/html
```

not:

```
/var/www/company
```

---

# Root Cause Analysis

The issue was caused by having different configurations for HTTP and HTTPS.

Current behavior:

```
HTTP Request

company.local

        |
        ↓

Port 80

        |
        ↓

Default Nginx Server

        |
        ↓

/usr/share/nginx/html
```

---

```
HTTPS Request

company.local

        |
        ↓

Port 443

        |
        ↓

company.conf

        |
        ↓

/var/www/company
```

---

# Important Lesson

HTTP and HTTPS are not automatically connected.

Nginx does not assume:

```
HTTP website
        |
        ↓
HTTPS website
```

They must be configured separately.

If HTTP should always use HTTPS, an administrator must explicitly configure a redirect.

---

# Verification Checklist

| Check | Result |
|-|-|
| Existing SSL certificate reused | ✅ |
| HTTPS Virtual Hosts working | ✅ |
| Company HTTPS verified | ✅ |
| HR HTTPS verified | ✅ |
| IT HTTPS verified | ✅ |
| HTTP/HTTPS behavior investigated | ✅ |
| Root cause identified | ✅ |

---

# Production Considerations

In production environments:

HTTP traffic is commonly redirected:

```
http://website.com

        ↓

301 Redirect

        ↓

https://website.com
```

Reasons:

- Enforce encryption.
- Prevent users from accessing insecure endpoints.
- Improve security consistency.

The next phase will implement HTTP → HTTPS redirection.

---

# Key Takeaways

This phase demonstrated how existing infrastructure can be expanded.

Important concepts learned:

- Multiple HTTPS websites can share one Nginx server.
- HTTPS Virtual Hosts are selected using hostname matching.
- HTTP and HTTPS are separate configurations.
- Troubleshooting requires checking the actual loaded configuration.

The next phase will configure HTTP redirection so all users are automatically moved from HTTP to HTTPS.
