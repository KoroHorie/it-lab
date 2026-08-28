# Phase 2 — Creating Multiple Nginx Virtual Hosts

## Objective

The objective of this phase is to configure Nginx to host multiple websites on a single CentOS Stream 9 server.

In the previous project, Nginx was configured as a single web server hosting one website.

For this project, ABC Solutions requires multiple internal websites to run on the same server:

- Company Portal
- HR Department Portal
- IT Department Portal

Instead of deploying separate servers, Nginx will be configured with multiple server blocks to serve different website content depending on the requested domain name.

---

# Starting Point

At the beginning of this phase, the server already contains:

- Installed Nginx
- Running Nginx service
- Existing SSL configuration from Project 1
- Working HTTPS web server

Verification:

```bash
systemctl status nginx
```

Expected:

```
active (running)
```

---

# Production Scenario

ABC Solutions currently has one Linux web server.

The company wants to host three internal websites:

```
company.local

hr.company.local

it.company.local
```

The requirement:

- Use the existing CentOS server.
- Avoid deploying additional servers.
- Separate each website's content.
- Allow Nginx to route requests correctly.

The final design:

```
                Client Browser

                      |
                      |
                      ↓

                Nginx Server

                      |
        --------------------------------

        |              |               |

        ↓              ↓               ↓


 company.local   hr.company.local   it.company.local


 /var/www/       /var/www/          /var/www/
 company        hr                 it

```

---

# Creating Website Directories

The first step was creating separate directories for each website.

Command:

```bash
mkdir -p /var/www/company
mkdir -p /var/www/hr
mkdir -p /var/www/it
```

Result:

```
/var/www

├── company
├── hr
└── it
```

---

# Creating Test Website Content

Each directory received its own test page.

This allows verification that Nginx is serving the correct website.

---

## Company Website

Location:

```
/var/www/company/index.html
```

Example:

```html
<h1>ABC Solutions</h1>

<p>Welcome to Company Portal</p>
```

---

## HR Website

Location:

```
/var/www/hr/index.html
```

Example:

```html
<h1>Human Resources</h1>

<p>Welcome to HR Portal</p>
```

---

## IT Website

Location:

```
/var/www/it/index.html
```

Example:

```html
<h1>IT Department</h1>

<p>Welcome to IT Portal</p>
```

---

# Creating Nginx Server Blocks

Nginx uses:

```nginx
server {

}
```

blocks to define website behavior.

Each website requires its own configuration.

The configuration files are stored inside:

```
/etc/nginx/conf.d/
```

---

# Company Virtual Host

Created:

```
/etc/nginx/conf.d/company.conf
```

Configuration:

```nginx
server {

   listen 443 ssl;
   listen [::]:443 ssl;

   server_name company.local;

   root /var/www/company;
   index index.html;

   ssl_certificate      /etc/nginx/ssl/nginx.crt;
   ssl_certificate_key  /etc/nginx/ssl/nginx.key;

}
```

---

# HR Virtual Host

Created:

```
/etc/nginx/conf.d/hr.conf
```

Configuration:

```nginx
server {

   listen 443 ssl;
   listen [::]:443 ssl;

   server_name hr.company.local;

   root /var/www/hr;
   index index.html;

   ssl_certificate      /etc/nginx/ssl/nginx.crt;
   ssl_certificate_key  /etc/nginx/ssl/nginx.key;

}
```

---

# IT Virtual Host

Created:

```
/etc/nginx/conf.d/it.conf
```

Configuration:

```nginx
server {

   listen 443 ssl;
   listen [::]:443 ssl;

   server_name it.company.local;

   root /var/www/it;
   index index.html;

   ssl_certificate      /etc/nginx/ssl/nginx.crt;
   ssl_certificate_key  /etc/nginx/ssl/nginx.key;

}
```

---

# Understanding the Configuration

## server_name

Example:

```nginx
server_name hr.company.local;
```

Defines which hostname should match this website.

When the browser requests:

```
https://hr.company.local
```

Nginx checks the request hostname and selects the matching server block.

---

## root

Example:

```nginx
root /var/www/hr;
```

Defines where website files are located.

Request:

```
https://hr.company.local/
```

becomes:

```
/var/www/hr/index.html
```

---

## index

Example:

```nginx
index index.html;
```

Defines the default file returned when accessing a directory.

---

# Validating Configuration

Before applying changes:

```bash
nginx -t
```

Expected:

```
syntax is ok

test is successful
```

---

# Applying Changes

Reload Nginx:

```bash
systemctl reload nginx
```

Why reload?

Because the service is already running.

Reloading allows Nginx to:

- Read the new configuration.
- Keep existing connections.
- Avoid unnecessary downtime.

---

# Domain Resolution Testing

Since this is a local lab environment, DNS records were not available.

Instead, hostname resolution was temporarily configured using:

```
/etc/hosts
```

Example:

```
192.168.x.x company.local

192.168.x.x hr.company.local

192.168.x.x it.company.local
```

This allowed the client machine to resolve the internal domains.

---

# Verification

Testing:

```
https://company.local
```

Expected:

```
ABC Solutions
Welcome to Company Portal
```

---

Testing:

```
https://hr.company.local
```

Expected:

```
Human Resources Department
Welcome to HR Portal
```

---

Testing:

```
https://it.company.local
```

Expected:

```
IT Department
Welcome to IT Portal
```

---

# Troubleshooting

## Issue: Website returns 403 Forbidden

During testing, the websites returned:

```
403 Forbidden
```

Initial investigation checked:

- File existence.
- Ownership.
- Unix permissions.

Commands:

```bash
ls -ld /var/www/company

ls -l /var/www/company
```

Files existed and permissions appeared correct.

Further investigation continued into SELinux.

This issue is documented in Phase 4.

---

# Verification Checklist

| Check | Result |
|-|-|
| Website directories created | ✅ |
| HTML files created | ✅ |
| Nginx server blocks created | ✅ |
| Configuration validated | ✅ |
| Nginx reloaded | ✅ |
| Multiple websites configured | ✅ |

---

# Key Takeaways

This phase demonstrated how one Nginx server can host multiple websites without deploying additional servers.

The important concepts learned:

- Nginx uses server blocks to separate websites.
- Each website can have its own document root.
- `server_name` determines which website responds.
- Existing infrastructure can be expanded instead of replaced.

The next phase focuses on HTTPS verification and understanding why HTTP and HTTPS behaved differently during testing.
