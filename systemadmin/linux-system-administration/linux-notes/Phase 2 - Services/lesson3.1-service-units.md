# Lesson 3 – Understanding and Inspecting systemd Services

> **Phase 2 – Services**  
> Linux System Administration Roadmap

---

# Introduction

After learning what **systemd** is and understanding that it is the first userspace process (PID 1) responsible for managing the operating system, the next step is learning how to inspect the services it manages.

Before starting, stopping, or troubleshooting services, a Linux System Administrator should first know how to identify a service, understand its current state, and gather information about it without making any changes.

This lesson focuses on **observation before modification**, which is an important habit in production environments.

---

# Learning Objectives

After completing this lesson, you should be able to:

- Understand what a systemd service is.
- List services managed by systemd.
- Interpret the `LOAD`, `ACTIVE`, and `SUB` columns.
- Understand the difference between `active (running)` and `active (exited)`.
- Inspect a service using `systemctl status`.
- Understand the difference between **enabled** and **static** services.
- Read the basic information inside a service unit file.

---

# Why Services Exist

Many tasks in Linux need to run in the background without direct user interaction.

Examples include:

- SSH remote access
- Network time synchronization
- Logging
- Firewall management
- Network management

Instead of manually starting these programs every time Linux boots, **systemd** manages them as services.

---

# Listing Services

To display loaded services:

```bash
systemctl list-units --type=service
```

Example:

```text
UNIT                     LOAD    ACTIVE   SUB       DESCRIPTION
chronyd.service          loaded  active   running   NTP client/server
sshd.service             loaded  active   running   OpenSSH server daemon
systemd-sysctl.service   loaded  active   exited    Apply Kernel Variables
```

---

# Understanding LOAD, ACTIVE, and SUB

## LOAD

Indicates whether systemd successfully loaded the unit file.

Example:

```text
loaded
```

Meaning:

- The unit file exists.
- systemd successfully parsed it.
- The service is available for management.

---

## ACTIVE

Represents the high-level state of the service.

Examples:

- active
- inactive
- failed

Think of it as answering:

> "Is this service currently functioning as expected?"

---

## SUB

Provides the detailed runtime state.

Examples:

- running
- exited
- dead
- listening
- waiting

The SUB state provides more specific information than ACTIVE.

---

# active (running)

Example:

```text
chronyd.service

ACTIVE: active

SUB: running
```

Meaning:

- The service started successfully.
- The main process is still running.
- It continues performing work in the background.

Examples include:

- SSH
- NetworkManager
- Chrony
- Apache
- MariaDB

---

# active (exited)

Example:

```text
systemd-sysctl.service

ACTIVE: active

SUB: exited
```

This does **not** indicate a failure.

Instead, it means:

- The service started successfully.
- It completed its task.
- The process exited because there was nothing left to do.

Some services only perform a specific task and then terminate normally.

---

# Inspecting a Service

To inspect a service:

```bash
systemctl status <service>
```

Example:

```bash
systemctl status chronyd.service
```

Important information shown includes:

## Description

A short explanation of what the service does.

---

## Loaded

Shows:

- The location of the unit file.
- Whether the service is enabled or static.

---

## Active

Shows the current operational state of the service.

Examples:

```text
active (running)

active (exited)

failed
```

---

## Main PID

Displays the primary process ID managed by systemd.

Example:

```text
Main PID: 104
```

The process can be verified using:

```bash
ps -p <PID>
```

---

## Documentation

Some unit files include recommended documentation.

Example:

```text
Docs: man:chronyd(8)
```

This tells administrators where to find more information about the service.

---

## Recent Logs

The bottom of the status output displays the most recent log entries related to the service.

These logs often provide immediate clues during troubleshooting.

---

# Enabled vs Static

One of the most important concepts when working with systemd.

## Enabled

An enabled service:

- Contains an `[Install]` section.
- Can be enabled using:

```bash
systemctl enable <service>
```

Enabling a service does **not** start it immediately.

Instead, it configures the service so that systemd can start it automatically during future boots.

---

## Static

A static service:

- Does not contain an `[Install]` section.
- Cannot be enabled.
- Is started only when another unit requires or requests it.

These services usually perform a specific task and then exit once the task is complete.

---

# Reading a Unit File

To inspect a service unit file:

```bash
systemctl cat <service>
```

Example:

```bash
systemctl cat chronyd.service
```

Important fields include:

```ini
Description=
Documentation=
ExecStart=
Type=

[Install]
```

These fields help determine:

- What the service does.
- Which executable is started.
- How systemd starts the service.
- Whether the service can be enabled.

---

# Observation Before Modification

A professional Linux administrator should always investigate a service before making changes.

Recommended workflow:

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

Implement

↓

Verify Solution
```

Commands that are safe for investigation include:

```bash
systemctl status <service>

systemctl cat <service>

systemctl list-units --type=service
```

These commands only gather information and do not modify the system.

---

# Common Beginner Mistakes

## Restarting a service before checking its status

Always inspect the service first.

---

## Assuming "active (exited)" means failure

Many services intentionally complete their task and exit successfully.

---

## Confusing "enabled" with "running"

A service can be:

- Enabled but not currently running.
- Disabled but currently running.

Enabled refers to future startup behavior, while running describes the current state.

---

# Commands Learned

```bash
systemctl list-units --type=service

systemctl status <service>

systemctl cat <service>

ps -p <PID>

uname -a

hostname

uptime

cat /etc/os-release
```

---

# Production Scenario

A user reports:

> "SSH isn't working."

Instead of immediately restarting the service:

```bash
systemctl restart sshd
```

A professional administrator first gathers information:

```bash
systemctl status sshd.service
```

Only after understanding the current state should corrective actions be taken.

---

# Interview Questions

1. What is the purpose of a systemd service?

2. Explain the difference between LOAD, ACTIVE, and SUB.

3. What is the difference between `active (running)` and `active (exited)`?

4. What is the difference between an enabled service and a static service?

5. What information can you obtain from `systemctl status`?

6. Why should an administrator inspect a service before restarting it?

---

# Lesson Summary

In this lesson, you learned how to inspect services managed by systemd without modifying them.

You learned how to:

- List services.
- Interpret service states.
- Read service status information.
- Understand the difference between `active (running)` and `active (exited)`.
- Distinguish between enabled and static services.
- Read basic information from a service unit file.
- Develop the habit of observing before modifying a production system.

These skills provide the foundation for safely managing and troubleshooting services in future lessons.
