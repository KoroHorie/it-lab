# Hosting Multiple Websites with Nginx Virtual Hosts, HTTPS, and SELinux

**Project Type:** Linux System Administration Hands-on Lab

**Operating System:** CentOS Stream 9

**Difficulty:** Intermediate

---

# Topics Covered

- Nginx Virtual Hosts
- Nginx Server Blocks
- Multiple Website Hosting
- Domain-Based Routing
- Document Roots
- SSL/TLS
- HTTPS
- HTTP → HTTPS Redirects
- SELinux
- Security Contexts
- firewalld
- Configuration Validation
- Troubleshooting Methodology
- Production Web Server Administration

---

# Project Overview

This project simulates the responsibilities of a Linux System Administrator managing a production web server responsible for hosting multiple internal company websites.

Instead of deploying multiple physical servers for every department website, this project demonstrates how a single CentOS Stream 9 server can host multiple websites using Nginx Virtual Hosts.

The project focuses on configuring Nginx to route incoming web requests based on domain names while maintaining separate website directories, SSL/TLS encryption, and security controls.

Throughout the project, the web server is deployed, configured, secured, troubleshot, and validated following enterprise Linux administration practices.

---

# Learning Objectives

After completing this project, I should be able to:

- Understand how Nginx hosts multiple websites using server blocks.
- Configure multiple Virtual Hosts on a single Linux server.
- Understand how Nginx selects the correct website based on:
  - Listening port
  - Host header
  - `server_name` directive
- Create separate document roots for different websites.
- Configure HTTPS using SSL certificates.
- Understand the difference between HTTP and HTTPS listeners.
- Configure HTTP traffic redirection to HTTPS.
- Troubleshoot Nginx website access issues.
- Understand how SELinux affects web server access.
- Identify the difference between:
  - Linux DAC permissions
  - SELinux MAC policies
- Validate Nginx configuration before applying changes.
- Apply production troubleshooting methodology.

---

# Lab Environment

| Component | Value |
|---|---|
| Operating System | CentOS Stream 9 |
| Web Server | Nginx |
| Service Manager | systemd |
| Firewall | firewalld |
| SSL | OpenSSL Self-Signed Certificate |
| Security Module | SELinux |
| Client Machine | Windows 11 |
| Browser | Microsoft Edge / Google Chrome |

---

# Project Scenario

ABC Solutions currently operates multiple internal department websites.

The company wants to consolidate these websites into a single Linux server instead of maintaining separate servers for every department.

The assigned Linux System Administrator is responsible for deploying a multi-site Nginx environment.

The websites required are:

```
company.local
hr.company.local
it.company.local
```

Each website must have:

- Separate website content
- Independent document roots
- HTTPS support
- Secure access
- HTTP to HTTPS redirection

As the Junior Linux System Administrator, my responsibilities include:

- Designing the Nginx multi-site architecture.
- Creating website directories.
- Configuring Virtual Hosts.
- Deploying HTTPS.
- Troubleshooting access issues.
- Resolving SELinux restrictions.
- Validating the final deployment.

---

# Project Workflow

```
Design Architecture
        ↓
Create Website Structure
        ↓
Configure Nginx Virtual Hosts
        ↓
Enable HTTPS
        ↓
Verify Website Routing
        ↓
Troubleshoot Security Restrictions
        ↓
Configure HTTP Redirect
        ↓
Document
```

---

# Troubleshooting Workflow

Throughout this project, every incident follows the same structured troubleshooting methodology.

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

This workflow prevents random configuration changes and encourages evidence-based troubleshooting.

---

# Project Architecture

A single Nginx server hosts multiple websites.

```
                    Client Browser
                          |
                          |
                    HTTP / HTTPS
                          |
                          ↓
                  Nginx Web Server
                          |
        -----------------------------------
        |                 |               |
        ↓                 ↓               ↓

 company.local     hr.company.local   it.company.local

 /var/www/company  /var/www/hr        /var/www/it
```

Each website is isolated through separate Nginx server blocks and document roots.

---

# Project Phases

# Phase 1 — Designing Multi-Site Nginx Architecture

Create the required directory structure for multiple websites.

The goal of this phase is understanding how one Linux server can provide multiple websites.

Topics covered:

- Nginx multi-site architecture
- Document roots
- Website directory organization
- Server block concept

Directory structure:

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

# Phase 2 — Configuring Nginx Virtual Hosts

Configure separate Nginx server blocks for each website.

Each Virtual Host defines:

- Domain name
- Listening port
- Website root directory
- SSL configuration

Websites configured:

| Domain | Document Root |
|---|---|
| company.local | `/var/www/company` |
| hr.company.local | `/var/www/hr` |
| it.company.local | `/var/www/it` |

Topics covered:

- Nginx server blocks
- `server_name`
- `root`
- `index`
- Configuration organization

---

# Phase 3 — Securing Websites with HTTPS

Configure SSL/TLS support for all Virtual Hosts.

HTTPS architecture:

```
Client
  |
  |
 HTTPS :443
  |
  ↓
Nginx
  |
  |
  ├── company.local
  |
  ├── hr.company.local
  |
  └── it.company.local
```

Topics covered:

- OpenSSL
- SSL certificates
- Private keys
- Port 443
- Nginx SSL configuration
- HTTPS firewall rules

---

# Phase 4 — Troubleshooting SELinux Web Access

Simulate a production incident where websites return:

```
403 Forbidden
```

even though:

- Nginx configuration is correct.
- Files exist.
- Linux permissions appear correct.

Investigation:

```
Nginx
  |
  ↓
Linux Kernel
  |
  ↓
SELinux Policy Check
  |
  ↓
File Security Context
```

Root Cause:

The website files were labeled:

```
var_t
```

instead of:

```
httpd_sys_content_t
```

Because of this, SELinux prevented the Nginx process from reading the files.

Solution:

Configure the correct SELinux context:

```bash
semanage fcontext -a -t httpd_sys_content_t "/var/www(/.*)?"

restorecon -Rv /var/www
```

Topics covered:

- SELinux security contexts
- DAC vs MAC
- SELinux file labeling
- `ls -Z`
- `semanage`
- `restorecon`

---

# Phase 5 — Implementing HTTP to HTTPS Redirect

Configure HTTP Virtual Hosts to redirect users automatically to HTTPS.

Before:

```
http://company.local

        ↓

Default Nginx Website
```

After:

```
http://company.local

        ↓

301 Redirect

        ↓

https://company.local

        ↓

Correct Virtual Host
```

Topics covered:

- HTTP status codes
- 301 redirects
- Port 80 Virtual Hosts
- Secure web deployment practices

---

# Skills Demonstrated

- Linux System Administration
- Nginx Administration
- Virtual Host Configuration
- Multi-Site Web Hosting
- HTTPS Deployment
- SSL Certificate Management
- SELinux Troubleshooting
- Firewall Administration
- Configuration Validation
- Production Troubleshooting
- Root Cause Analysis
- Technical Documentation

---

# Technologies Used

- CentOS Stream 9
- Nginx
- systemd
- systemctl
- OpenSSL
- firewalld
- SELinux
- semanage
- restorecon
- curl
- ss
- journalctl

---

# Repository Structure

```
Hosting-Multiple-Websites-with-Nginx/

├── README.md
│
├── docs/
│   ├── phase1-nginx-architecture.md
│   ├── phase2-virtual-hosts.md
│   ├── phase3-https.md
│   ├── phase4-selinux.md
│   ├── phase5-http-redirect.md
│   └── cheatsheet.md
│
├── screenshots/
│
└── configs/
    |
    ├── company.conf
    ├── hr.conf
    ├── it.conf
    ├── company-http.conf
    ├── hr-http.conf
    └── it-http.conf
```

---

# Key Takeaways

This project demonstrates that Linux System Administration is not only about installing services.

A professional administrator must understand:

- How services process requests.
- How applications are exposed to users.
- How security controls interact with services.
- How to troubleshoot failures systematically.
- How to document infrastructure changes.

By completing this project, I practiced the complete lifecycle of deploying a production-style multi-site Nginx server:

```
Design
 ↓
Configure
 ↓
Secure
 ↓
Troubleshoot
 ↓
Verify
 ↓
Document
```
