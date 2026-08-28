# Phase 2 — Services
# Lesson 1 — Introduction to systemd

---

# Introduction

Modern Linux systems consist of two major parts:

1. **Kernel Space**
2. **User Space**

The Linux kernel is responsible for interacting with the computer's hardware. It manages CPU scheduling, memory, storage devices, filesystems, networking, and process creation. However, the kernel alone does not provide a usable operating system.

After the Linux kernel finishes initializing during the boot process, another program must take over the responsibility of starting and managing everything that runs in user space.

On modern Linux distributions, that program is **systemd**.

systemd is the first userspace process started by the Linux kernel. It is assigned **Process ID (PID) 1**, making it one of the most important components of the operating system.

Without systemd (or another init system), Linux would boot the kernel successfully but would not have a mechanism to start essential services such as:

- SSH
- Web servers
- Database servers
- Logging services
- Network services
- Scheduled tasks

Understanding systemd is one of the most important milestones in becoming a Linux System Administrator because almost every modern enterprise Linux distribution relies on it.

---

# Why This Technology Exists

## The Problem

Imagine a Linux server without systemd.

The Linux kernel boots successfully.

Now ask yourself:

- Who starts SSH?
- Who starts Apache?
- Who starts MariaDB?
- Who mounts filesystems?
- Who activates swap?
- Who starts logging?
- Who restarts failed services?

The kernel is **not** responsible for doing these tasks.

Someone must coordinate everything that happens after the kernel has finished booting.

That responsibility belongs to an **init system**.

Modern Linux uses **systemd** as its init system.

---

## Why was systemd created?

Older Linux systems used an init system called **SysVinit**.

Although SysVinit worked for many years, enterprise Linux environments eventually encountered several limitations.

Some of those limitations included:

- Slow boot times
- Sequential service startup
- Poor dependency management
- Limited service monitoring
- Weak logging integration
- Minimal automation capabilities

As Linux evolved into an operating system capable of running enterprise servers, cloud infrastructure, containers, and high-availability systems, administrators needed something more powerful.

systemd was created to solve those problems.

---

## What problems does systemd solve?

systemd provides a centralized framework for managing the entire userspace environment.

Instead of having different tools responsible for different parts of the operating system, systemd provides one unified management system.

It can:

- Start services
- Stop services
- Restart services
- Monitor services
- Automatically restart failed services
- Manage mount points
- Activate swap
- Handle timers
- Manage sockets
- Track devices
- Coordinate boot targets

Rather than having dozens of unrelated management utilities, systemd manages them through one consistent architecture.

---

## Why should a Linux System Administrator care?

Nearly every enterprise Linux distribution uses systemd.

Examples include:

- Red Hat Enterprise Linux
- Rocky Linux
- AlmaLinux
- Fedora
- Ubuntu
- Debian
- SUSE Linux Enterprise

As a Linux System Administrator, you will interact with systemd almost every day.

Whether you are:

- Deploying applications
- Troubleshooting failed services
- Investigating boot problems
- Managing databases
- Hosting websites
- Maintaining production servers

systemd will almost always be involved.

Understanding systemd is therefore a foundational skill rather than an optional one.

---

# Learning Objectives

After completing this lesson, you should understand:

- What systemd is
- Why systemd exists
- The relationship between the Linux kernel and systemd
- The concept of userspace
- Why systemd runs as PID 1
- What responsibilities systemd has
- What Units are
- The different types of Units managed by systemd
- Why systemd manages more than just services

---

# Understanding Linux Boot (High-Level View)

One of the biggest misconceptions beginners have is assuming Linux begins with systemd.

It does not.

The Linux boot process begins with the kernel.

A simplified view looks like this:

```text
Computer Power On
        │
        ▼
Firmware (BIOS / UEFI)
        │
        ▼
Bootloader (GRUB)
        │
        ▼
Linux Kernel
        │
        ▼
systemd (PID 1)
        │
        ▼
Entire Userspace
```

Notice that **systemd is not the operating system itself.**

Instead, it is the **first userspace process** that begins after the kernel has initialized.

---

# Kernel Space vs Userspace

Understanding systemd requires understanding the distinction between two execution environments.

## Kernel Space

Kernel space is where the Linux kernel executes.

Its responsibilities include:

- Memory management
- CPU scheduling
- Process creation
- Device drivers
- Filesystems
- Networking
- Hardware communication

Applications do not run directly inside kernel space.

---

## Userspace

Userspace is where normal programs execute.

Examples include:

- SSH Server
- Apache HTTP Server
- MariaDB
- Docker
- Bash
- Python
- System utilities

systemd itself is a userspace program.

However, it is the **first** userspace program created by the kernel.

---

## Relationship Diagram

```text
+--------------------------------------+
|             User Space               |
|--------------------------------------|
| Bash                                 |
| Apache                               |
| SSH                                  |
| MariaDB                              |
| Docker                               |
| systemd (PID 1)                      |
+--------------------------------------+
                 ▲
                 │
+--------------------------------------+
|            Kernel Space              |
|--------------------------------------|
| Process Scheduler                    |
| Memory Manager                       |
| Filesystems                          |
| Networking                           |
| Device Drivers                       |
+--------------------------------------+
```

---

# Why PID 1 Matters

Every running process inside Linux has a Process ID (PID).

Examples:

```text
PID 1
PID 278
PID 840
PID 1432
```

systemd is always assigned:

```text
PID 1
```

This is significant because PID 1 becomes the parent process for the userspace environment.

Since systemd is responsible for starting and supervising many essential processes, a failure of PID 1 would leave the operating system without its central management component.

Although the Linux kernel would still be running, the userspace environment would become unstable and largely unusable.

---

# What is systemd?

systemd is:

- an Init System
- a System Manager
- a Service Manager

These three roles are closely related but represent different responsibilities.

## Init System

systemd is responsible for initializing userspace after the Linux kernel finishes booting.

It becomes the first process created by the kernel.

---

## System Manager

systemd coordinates many components that make up the operating system.

Examples include:

- Services
- Mounts
- Timers
- Swap
- Devices
- Sockets

Rather than allowing every subsystem to be managed independently, systemd provides centralized management.

---

## Service Manager

This is the role most administrators interact with.

systemd manages service lifecycles.

It can:

- Start services
- Stop services
- Restart services
- Reload services
- Automatically restart failed services
- Track service state

Examples include:

- Apache
- SSH
- MariaDB
- Docker
- PostgreSQL

---

# An Office Manager Analogy

Imagine a company office.

The employees represent services:

- Accountant
- Receptionist
- Security Guard
- IT Technician

Each employee has a specific job.

However, someone must:

- Schedule employees
- Supervise them
- Ensure they arrive at work
- Replace them if necessary
- Coordinate communication

That role belongs to the office manager.

systemd plays a similar role.

It is not the employee.

It manages the employees.

Likewise:

systemd is not Apache.

systemd manages Apache.

systemd is not SSH.

systemd manages SSH.

systemd is not MariaDB.

systemd manages MariaDB.

This distinction is fundamental.

---

# What Does systemd Actually Manage?

Many new Linux administrators believe systemd only manages services.

This is incorrect.

Services are only one category of objects managed by systemd.

systemd manages **Units**.

A Unit represents an object that systemd knows how to manage.

Examples include:

| Unit Type | Purpose |
|------------|---------|
| `.service` | Manage services and daemons |
| `.mount` | Manage mounted filesystems |
| `.swap` | Activate swap devices/files |
| `.socket` | Manage communication sockets |
| `.timer` | Schedule tasks |
| `.device` | Represent hardware devices |
| `.path` | Monitor filesystem paths |
| `.target` | Represent system operating states |
| `.slice` | Organize resource allocation using cgroups |
| `.scope` | Manage externally created process groups |

Instead of learning ten different management frameworks, Linux administrators interact with these resources through one consistent system.

---

# Understanding Path Units

One unit type that often confuses beginners is the **Path Unit**.

A Path Unit does **not** manage directories or files.

Instead, it **monitors a specific filesystem path**.

When a configured event occurs, such as a file appearing or changing, systemd can trigger another unit (usually a Service Unit).

Conceptually:

```text
Watch:

/srv/uploads

        │
        ▼

File changes

        │
        ▼

Trigger:

virus-scan.service
```

Internally, Path Units rely on Linux kernel filesystem notifications rather than continuously scanning directories.

Their purpose is to react to filesystem events.

They are specialized units and are encountered much less frequently than Service Units in day-to-day administration.

---
