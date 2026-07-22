# Deploying File Browser as a systemd Service with SELinux Troubleshooting

**Project Type:** Linux System Administration Hands-on Lab

**Operating System:** CentOS Stream 9

**Difficulty:** Intermediate

---

# Topics Covered

- Third-Party Application Deployment
- Linux Filesystem Hierarchy
- `/tmp`
- `/opt`
- systemd
- Custom Service Units
- Service Management
- Process Management
- Linux Process Lifecycle
- Service Enablement
- ExecStart
- WorkingDirectory
- journalctl
- systemctl
- SELinux
- Security Contexts
- restorecon
- Root Cause Analysis
- Production Troubleshooting Methodology

---

# Project Overview

This project simulates the responsibilities of a Linux System Administrator deploying a third-party application that is not available through the operating system package manager.

Instead of installing software using `dnf`, the application is manually downloaded, extracted, deployed into the appropriate Linux filesystem hierarchy, and integrated into the operating system using a custom **systemd Service Unit**.

The project focuses on understanding how manually deployed applications become production services managed by systemd while following Linux filesystem standards and enterprise administration practices.

During deployment, a real production issue involving **SELinux security contexts** prevents the service from starting even though Linux file permissions appear correct. The issue is investigated and resolved using structured troubleshooting instead of disabling security mechanisms.

---

# Learning Objectives

After completing this project, I should be able to:

- Understand where manually installed applications belong in Linux.
- Understand the purpose of `/tmp` during software deployment.
- Understand why `/opt` is used for third-party software.
- Deploy applications without using the package manager.
- Create custom systemd Service Units.
- Understand the purpose of:
  - `[Unit]`
  - `[Service]`
  - `[Install]`
- Validate Service Units before loading them.
- Reload systemd after creating new services.
- Understand the difference between:
  - Starting a service
  - Enabling a service
- Understand how systemd launches Linux processes.
- Understand how the Linux kernel creates Process IDs.
- Troubleshoot service startup failures.
- Read service logs using `journalctl`.
- Understand the difference between:
  - Linux DAC permissions
  - SELinux MAC policies
- Restore incorrect SELinux security contexts.
- Apply production troubleshooting methodology.

---

# Lab Environment

| Component | Value |
|-----------|------|
| Operating System | CentOS Stream 9 |
| Application | File Browser |
| Service Manager | systemd |
| Security Module | SELinux |
| Existing Web Server | Nginx |
| Client Machine | Windows 11 |
| Browser | Microsoft Edge / Google Chrome |

---

# Project Scenario

ABC Solutions plans to deploy an internal web-based File Browser for employees to browse and download company documents.

The application is distributed as a standalone binary instead of an RPM package.

As the assigned Junior Linux System Administrator, my responsibilities include:

- Downloading the application.
- Deploying it following Linux filesystem standards.
- Integrating it with systemd.
- Managing it as a Linux service.
- Troubleshooting startup failures.
- Resolving SELinux restrictions.
- Preparing the service for future production deployment behind Nginx.

---

# Project Workflow

```
Download Application
        ↓
Extract Files
        ↓
Deploy into /opt
        ↓
Create systemd Service
        ↓
Validate Service Unit
        ↓
Load into systemd
        ↓
Start Service
        ↓
Troubleshoot Failure
        ↓
Resolve SELinux Issue
        ↓
Verify Service
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

The File Browser application is deployed manually and managed entirely by systemd.

```
            Linux Administrator
                    |
                    |
           Deploy File Browser
                    |
                    ↓
          /opt/filebrowser/
                    |
                    ↓
          systemd Service Unit
                    |
                    ↓
             Linux Kernel
                    |
                    ↓
             File Browser Process
```

The application initially listens only on:

```
127.0.0.1:8080
```

A future phase of this project will publish the application through **Nginx Reverse Proxy** instead of exposing port `8080` directly.

---

# Project Phases

## Phase 1 — Deploying File Browser

Deploy a third-party application manually without using the Linux package manager.

The goal of this phase is understanding the proper Linux filesystem locations used during manual software deployment.

Topics covered:

- `/tmp`
- `/opt`
- Binary extraction
- Linux filesystem hierarchy
- File ownership
- Initial application execution

Deployment structure:

```
/opt/

└── filebrowser
    ├── filebrowser
    └── filebrowser.db
```

---

## Phase 2 — Managing File Browser with systemd

Create a custom Service Unit allowing systemd to manage File Browser as a production Linux service.

Topics covered:

- Custom Service Units
- ExecStart
- WorkingDirectory
- systemctl
- daemon-reload
- systemd-analyze verify
- Enable vs Start
- Linux Process Management
- Main PID

---

## Phase 3 — Troubleshooting SELinux Service Startup Failure

Simulate a production incident where File Browser executes successfully from the shell but fails when started by systemd.

Investigation:

```
systemctl status
        ↓
journalctl
        ↓
Linux Permissions
        ↓
Ownership
        ↓
ExecStart Validation
        ↓
SELinux Context
```

Root Cause:

The File Browser executable retained the SELinux context:

```
user_tmp_t
```

because it originated from `/tmp`.

systemd refused to execute a temporary user file.

Solution:

```
restorecon -v /opt/filebrowser/filebrowser
```

Topics covered:

- SELinux Security Contexts
- DAC vs MAC
- `ls -Z`
- `restorecon`
- Production Root Cause Analysis

---

## Phase 4 — Publishing File Browser through Nginx *(Upcoming)*

Configure Nginx as a Reverse Proxy so users access File Browser through the existing web server instead of exposing port `8080`.

Topics planned:

- Reverse Proxy
- Backend Applications
- Loopback Interface
- Production Web Architecture

---

# Skills Demonstrated

- Linux System Administration
- Third-Party Software Deployment
- Linux Filesystem Management
- systemd Administration
- Service Management
- Linux Process Management
- SELinux Troubleshooting
- Root Cause Analysis
- Production Troubleshooting
- Technical Documentation

---

# Technologies Used

- CentOS Stream 9
- systemd
- systemctl
- File Browser
- SELinux
- restorecon
- journalctl
- wget
- tar
- curl
- ss
- Nginx

---

# Repository Structure

```
Deploying-FileBrowser-as-a-systemd-Service/

├── README.md
│
├── docs/
│   ├── phase1-deployment.md
│   ├── phase2-systemd-service.md
│   ├── phase3-selinux-troubleshooting.md
│   ├── phase4-nginx-reverse-proxy.md
│   └── cheatsheet.md
│
├── screenshots/
│
└── configs/
    ├── filebrowser.service
    └── nginx-filebrowser.conf (Upcoming)
```

---

# Key Takeaways

This project demonstrates that deploying Linux applications involves much more than downloading and executing binaries.

A professional Linux System Administrator must understand:

- Where manually installed software belongs.
- How systemd manages production services.
- How Linux creates and supervises service processes.
- How SELinux security policies interact with services.
- How to investigate failures systematically instead of weakening security controls.

By completing this project, I practiced the complete lifecycle of deploying a manually installed Linux application:

```
Deploy
   ↓
Integrate
   ↓
Manage
   ↓
Troubleshoot
   ↓
Restore
   ↓
Verify
   ↓
Document
```
