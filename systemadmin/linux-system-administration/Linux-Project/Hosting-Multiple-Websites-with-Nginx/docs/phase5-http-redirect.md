# Phase 5 — Implementing HTTP to HTTPS Redirect

## Objective

The objective of this phase is to configure Nginx so that all HTTP requests are automatically redirected to HTTPS.

During the previous phase, an investigation revealed that HTTP and HTTPS were serving different content.

HTTP requests were reaching the default Nginx website:

```
/usr/share/nginx/html
```

while HTTPS requests were correctly reaching the configured Virtual Hosts:

```
/var/www/company
/var/www/hr
/var/www/it
```

The goal is to ensure users always access the secured HTTPS version of each website.

---

# Production Scenario

ABC Solutions has completed the HTTPS deployment.

However, users may still access websites using:

```
http://company.local
```

instead of:

```
https://company.local
```

The company requires:

- No website content should be served over HTTP.
- Users accessing HTTP should automatically move to HTTPS.
- Existing HTTPS Virtual Hosts should remain unchanged.

---

# Understanding the Problem

Before this phase, the traffic flow was:

```
HTTP Request

http://company.local

        |
        ↓

Nginx Port 80

        |
        ↓

Default Server Block

        |
        ↓

/usr/share/nginx/html
```

---

HTTPS traffic was:

```
HTTPS Request

https://company.local

        |
        ↓

Nginx Port 443

        |
        ↓

company.conf

        |
        ↓

/var/www/company
```

---

# Solution Design

Instead of creating duplicate websites for HTTP, the HTTP Virtual Hosts will only perform a redirect.

New behavior:

```
HTTP Request

http://company.local

        |
        ↓

Nginx Port 80

        |
        ↓

301 Redirect

        |
        ↓

HTTPS Request

https://company.local

        |
        ↓

Correct HTTPS Virtual Host
```

---

# Creating HTTP Redirect Configuration

The HTTP configurations are created inside:

```
/etc/nginx/conf.d/
```

Files created:

```
company-http.conf

hr-http.conf

it-http.conf
```

---

# Company HTTP Redirect

Create:

```bash
vi /etc/nginx/conf.d/company-http.conf
```

Configuration:

```nginx
server {

    listen 80;
    listen [::]:80;

    server_name company.local;

    return 301 https://$host$request_uri;

}
```

---

# HR HTTP Redirect

Create:

```bash
vi /etc/nginx/conf.d/hr-http.conf
```

Configuration:

```nginx
server {

    listen 80;
    listen [::]:80;

    server_name hr.company.local;

    return 301 https://$host$request_uri;

}
```

---

# IT HTTP Redirect

Create:

```bash
vi /etc/nginx/conf.d/it-http.conf
```

Configuration:

```nginx
server {

    listen 80;
    listen [::]:80;

    server_name it.company.local;

    return 301 https://$host$request_uri;

}
```

---

# Understanding the Redirect Directive

Configuration:

```nginx
return 301 https://$host$request_uri;
```

Meaning:

```
return
```

Immediately sends a response.

```
301
```

Permanent redirect status code.

```
$host
```

Keeps the requested domain.

Example:

```
company.local
```

```
$request_uri
```

Keeps the requested path.

Example:

```
/index.html
```

Final result:

```
https://company.local/index.html
```

---

# Validating Configuration

Before applying:

```bash
nginx -t
```

Expected:

```
syntax is ok

test is successful
```

---

# Reloading Nginx

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

# Verification

## Testing HTTP Redirect

Request:

```bash
curl -I http://company.local
```

Expected:

```
HTTP/1.1 301 Moved Permanently

Location: https://company.local/
```

---

## Browser Testing

Before:

```
http://company.local
```

Result:

```
ABC Solutions

Welcome to Linux Web Server
```

---

After:

```
http://company.local
```

Browser automatically changes to:

```
https://company.local
```

Result:

```
ABC Solutions

Welcome to Company Portal
```

---

# Incident Resolution

The original issue:

```
HTTP showed wrong website
```

was resolved.

Final architecture:

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

        |            |              |

        ↓            ↓              ↓

   company       hr.company      it.company
```

---

# Verification Checklist

| Check | Result |
|-|-|
| HTTP Virtual Hosts created | ✅ |
| Redirect configuration added | ✅ |
| Nginx configuration tested | ✅ |
| Nginx reloaded successfully | ✅ |
| HTTP redirects to HTTPS | ✅ |
| HTTPS websites remain accessible | ✅ |

---

# Production Considerations

HTTP to HTTPS redirection is commonly used in production environments because it:

- Forces encrypted communication.
- Prevents accidental insecure access.
- Provides consistent user experience.

Additional enterprise improvements may include:

- HSTS configuration.
- Certificate automation.
- Monitoring certificate expiration.
- TLS security hardening.

---

# Key Takeaways

This phase completed the secure multi-site web server deployment.

Important concepts learned:

- HTTP and HTTPS require separate configurations.
- Port 80 can be used only for redirecting users.
- 301 redirects permanently move users to HTTPS.
- Production systems should avoid exposing duplicate HTTP content.

The final deployment now follows:

```
HTTP Request

        ↓

HTTPS Redirect

        ↓

Secure Virtual Host

        ↓

Correct Website Content
```

Project 2 is now complete.
