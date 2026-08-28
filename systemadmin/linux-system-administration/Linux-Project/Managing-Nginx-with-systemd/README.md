# Managing Nginx with systemd and systemctl

> **Project Type:** Linux System Administration Hands-on Lab
>
> **Operating System:** CentOS Stream 9
>
> **Difficulty:** Beginner → Intermediate
>
> **Topics Covered:**
>
> - systemd
> - systemctl
> - Service Management
> - Nginx
> - firewalld
> - HTTPS
> - SSL/TLS
> - Troubleshooting Methodology

---

# Project Overview

This project simulates the daily responsibilities of a Linux System Administrator responsible for deploying, managing, troubleshooting, and securing a production web server.

Instead of simply installing Nginx, this project follows a realistic workflow where the web server is deployed, configured, verified, troubleshot, and secured using HTTPS.

Throughout the project, Linux service management is performed entirely through **systemd** and **systemctl**, reinforcing enterprise Linux administration practices.

---

# Learning Objectives

After completing this project, I should be able to:

- Install and verify an Nginx web server.
- Understand how systemd manages services.
- Read and interpret a systemd service unit.
- Start, stop, restart and reload services appropriately.
- Enable services to start automatically during boot.
- Validate Nginx configuration before applying changes.
- Troubleshoot service availability using a structured workflow.
- Configure HTTPS using a self-signed SSL certificate.
- Manage firewall rules using firewalld.
- Understand the difference between runtime and permanent firewall configurations.

---

# Lab Environment

| Component | Value |
|-----------|-------|
| Operating System | CentOS Stream 9 |
| Web Server | Nginx |
| Service Manager | systemd |
| Firewall | firewalld |
| SSL | OpenSSL (Self-Signed Certificate) |
| Browser | Microsoft Edge / Google Chrome |
| Client Machine | Windows 11 |

---

# Project Scenario

ABC Solutions has provisioned a brand-new CentOS Stream 9 virtual machine to host an internal company website.

As the Junior Linux System Administrator, my responsibilities include:

- Installing Nginx
- Managing the service using systemd
- Deploying a static website
- Verifying service availability
- Troubleshooting connectivity issues
- Configuring HTTPS
- Verifying secure communication

The project is divided into four phases, each representing a realistic administrative task commonly performed in production environments.

---

# Project Workflow

```text
Install
    ↓
Configure
    ↓
Start Service
    ↓
Verify
    ↓
Troubleshoot
    ↓
Secure
    ↓
Document
```

---

# Troubleshooting Workflow

Throughout this project, every incident follows the same structured troubleshooting methodology.

```text
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

This workflow encourages evidence-based troubleshooting rather than making assumptions or randomly changing configurations.

---

# Project Phases

## Phase 1 — Installing Nginx

Install the Nginx package on a fresh CentOS Stream 9 server and verify that the service has been successfully installed.

Topics covered:

- Package installation
- Service discovery
- Understanding the initial service state

---

## Phase 2 — Managing Nginx with systemd

Configure the web server, start the service, enable automatic startup during boot, and verify that the website is successfully being served.

Topics covered:

- systemctl
- systemd service units
- ExecStart
- Active vs Inactive service states
- Enabled vs Disabled services

---

## Phase 3 — Troubleshooting Service Availability

Simulate a production incident where users are unable to access the website.

Using the Linux troubleshooting workflow, determine the root cause and restore service availability.

Topics covered:

- Service verification
- Nginx logs
- HTTP testing
- firewalld
- Root cause analysis

---

## Phase 4 — Securing the Website with HTTPS

Generate a self-signed SSL certificate, configure Nginx to support HTTPS, validate the configuration, and verify encrypted communication.

Topics covered:

- OpenSSL
- SSL/TLS
- HTTPS
- firewalld runtime vs permanent configuration
- Nginx configuration validation
- Reloading services

---

# Skills Demonstrated

- Linux System Administration
- systemd Service Management
- Nginx Administration
- HTTP/HTTPS Configuration
- SSL Certificate Generation
- Firewall Administration
- Configuration Validation
- Production Troubleshooting
- Root Cause Analysis
- Documentation

---

# Technologies Used

- CentOS Stream 9
- systemd
- systemctl
- Nginx
- OpenSSL
- firewalld
- curl
- ss
- journalctl

---

# Repository Structure

```text
Managing-Nginx-with-systemd/

├── README.md
│
├── docs/
│   ├── phase1-installation.md
│   ├── phase2-service-management.md
│   ├── phase3-troubleshooting.md
│   ├── phase4-https.md
│   └── cheatsheet.md
│
├── screenshots/
│
└── configs/
    └── ssl.conf
```

---

# Key Takeaways

This project demonstrates that Linux System Administration extends beyond memorizing commands. Successful administration requires understanding how services operate, validating changes before implementation, troubleshooting systematically, and documenting every significant change.

By completing this project, I practiced the complete lifecycle of deploying, managing, troubleshooting, and securing an Nginx web server using industry-standard Linux administration practices.
