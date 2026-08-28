# Phase 1 — Designing Multi-Site Nginx Architecture

## Objective

The objective of this phase is to design the structure required for hosting multiple websites on a single CentOS Stream 9 server.

Before configuring Nginx, the administrator must first understand how the websites will be organized, where their content will be stored, and how requests will eventually be routed.

In production environments, proper planning prevents configuration conflicts and makes future administration easier.

---

# Production Scenario

ABC Solutions requires multiple internal websites hosted on the same Linux server.

The company currently has three departments that require separate websites:

| Department | Website |
|---|---|
| Company Portal | company.local |
| Human Resources | hr.company.local |
| Information Technology | it.company.local |

Instead of deploying three separate web servers, the company requested that all websites run on one Nginx server.

The Linux System Administrator must design a structure where:

- Each website has its own content directory.
- Websites remain logically separated.
- Nginx can route requests correctly.
- Future websites can be added easily.

---

# Understanding the Problem

A common beginner approach would be:

```
/var/www/

index.html
```

One website.

However, production servers commonly host multiple applications.

Example:

```
Web Server

company.local
hr.company.local
it.company.local
```

All running from:

```
One Linux Server
One Nginx Process
Multiple Websites
```

The challenge is organizing the filesystem so Nginx can identify which content belongs to which website.

---

# Multi-Site Design

The planned architecture:

```
                    Client Request

                         |
                         |
                         ↓

                   Nginx Server

                         |
        ---------------------------------

        |               |               |

        ↓               ↓               ↓


 company.local   hr.company.local   it.company.local


 /var/www/       /var/www/          /var/www/
 company         hr                 it

```

Each website receives its own document root.

---

# Document Root Design

A document root is the directory where website files are stored.

For this project:

| Website | Document Root |
|-|-|
| company.local | `/var/www/company` |
| hr.company.local | `/var/www/hr` |
| it.company.local | `/var/www/it` |

Final structure:

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

Create the required directories:

```bash
mkdir -p /var/www/company
mkdir -p /var/www/hr
mkdir -p /var/www/it
```

Verify:

```bash
tree /var/www
```

Expected output:

```
/var/www

├── company
├── hr
└── it
```

---

# Creating Test Website Content

Before configuring Nginx, create simple test pages.

These pages allow verification later.

---

## Company Website

Create:

```bash
vi /var/www/company/index.html
```

Content:

```html
<html>
<head>
<title>Company Portal</title>
</head>

<body>

<h1>ABC Solutions</h1>

<p>Welcome to Company Portal</p>

</body>
</html>
```

---

## HR Website

Create:

```bash
vi /var/www/hr/index.html
```

Content:

```html
<html>
<head>
<title>Human Resources</title>
</head>

<body>

<h1>Human Resources Department</h1>

<p>Welcome to HR Portal</p>

</body>
</html>
```

---

## IT Website

Create:

```bash
vi /var/www/it/index.html
```

Content:

```html
<html>
<head>
<title>IT Department</title>
</head>

<body>

<h1>IT Department</h1>

<p>Welcome to IT Portal</p>

</body>
</html>
```

---

# Verify Directory Structure

Command:

```bash
tree /var/www
```

Expected result:

```
/var/www

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

# Understanding Ownership and Permissions

At this stage, the files are created by root.

Example:

```bash
ls -l /var/www/company
```

Output:

```
-rw-r--r-- root root index.html
```

This means:

Owner:

```
root
```

Group:

```
root
```

Permissions:

```
rw-r--r--
```

For a static website, Nginx only needs read access.

The important requirement is:

```
Nginx process
        |
        ↓
Must be able to read website files
```

However, access control is not only controlled by Linux permissions.

Later phases will introduce:

- SELinux security contexts
- httpd_sys_content_t
- Mandatory Access Control

---

# Verification Checklist

Before continuing:

| Check | Status |
|-|-|
| Website directories created | ✅ |
| Separate document roots created | ✅ |
| Test HTML files created | ✅ |
| Directory structure verified | ✅ |
| Content separation confirmed | ✅ |

---

# Troubleshooting

## Problem: Directory does not exist

Verify:

```bash
ls -ld /var/www
```

Create missing directories:

```bash
mkdir -p /var/www/<website>
```

---

## Problem: Wrong website content appears

Possible causes:

- Incorrect document root
- Incorrect Nginx server block
- Default server responding

Investigation:

```bash
nginx -T
```

---

# Production Considerations

In enterprise environments:

- Website files are normally separated by application.
- Naming conventions are important.
- Directory structure should support future expansion.
- Administrators avoid placing all websites inside one directory.

Example production structure:

```
/var/www/

├── company
├── hr
├── it
├── finance
├── portal
└── applications
```

A predictable filesystem layout simplifies:

- Backup operations
- Permission management
- Troubleshooting
- Application migrations

---

# Key Takeaways

This phase focused on architecture planning before configuration.

A Linux System Administrator should understand that successful deployment starts with design.

Before configuring Nginx:

1. Determine required websites.
2. Design document roots.
3. Separate application content.
4. Verify filesystem structure.

The next phase will configure Nginx Virtual Hosts so the server can route requests to these different websites.
