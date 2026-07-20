# Lesson 3 Hands-on Lab Cheat Sheet
> **Phase 2 – Services**
>
> **Objective:** Learn how to inspect systemd services without making any modifications to the system.

---

# Lab Environment

- Operating System: **CentOS Stream 10**
- Environment: Fresh Installation
- Privileges: Regular user with `sudo` access (or root)

---

# Part 1 – Identify the System

## Display kernel information

```bash
uname -a
```

## Display machine architecture

```bash
uname -m
```

## Display hostname

```bash
uname -n
```

## Display kernel release

```bash
uname -r
```

## Display kernel version

```bash
uname -v
```

## Display system uptime

```bash
uptime
```

## Display operating system information

```bash
cat /etc/os-release
```

---

# Part 2 – List Services

## Display all loaded services

```bash
systemctl list-units --type=service
```

Observe:

- UNIT
- LOAD
- ACTIVE
- SUB
- DESCRIPTION

---

# Part 3 – Inspect a Service

We'll use **Chrony** throughout this lesson.

## Display service status

```bash
systemctl status chronyd.service
```

Investigate:

- Description
- Loaded
- Active
- Main PID
- Documentation
- Recent Logs

---

# Part 4 – Verify the Main PID

Find the process shown under **Main PID**.

Example:

```text
Main PID: 104
```

Verify it:

```bash
ps -p 104
```

---

# Part 5 – Read the Unit File

Display the unit file:

```bash
systemctl cat chronyd.service
```

Look for:

```ini
Description=
Documentation=
ExecStart=
Type=

[Install]
```

---

# Part 6 – Understand Service States

Example:

```text
LOAD    ACTIVE    SUB

loaded  active    running
```

Meaning:

- Unit file loaded successfully
- Service is active
- Process is currently running

---

Example:

```text
LOAD    ACTIVE    SUB

loaded  active    exited
```

Meaning:

- Service completed successfully
- No running process remains
- This is normal for one-time task services

---

# Part 7 – Enabled vs Static

Identify whether the service is:

```text
Loaded:
...
enabled
```

or

```text
Loaded:
...
static
```

Remember:

### Enabled

- Can be enabled
- Starts automatically on future boots
- Contains an `[Install]` section

---

### Static

- Cannot be enabled
- Started only when another unit requires it
- No `[Install]` section

---

# Safe Investigation Commands

These commands **do not modify** the system.

```bash
systemctl status <service>

systemctl cat <service>

systemctl list-units --type=service

ps -p <PID>

uname -a

hostname

uptime

cat /etc/os-release
```

---

# Production Mindset

Before changing anything, always follow:

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

Never restart or stop a service until you understand:

- What it does
- Whether it is healthy
- Its current state
- Recent logs
- How it is configured

# 1. Observe

**Objective:** Understand the reported issue.

At this stage, you are simply gathering information about what has been reported. Do not make any changes yet.

### Examples

- Users cannot SSH into the server.
- The web application is unavailable.
- NTP is not synchronizing.
- The monitoring agent is not running.

**Key Question**

> What problem is being reported?

---

# 2. Verify

**Objective:** Confirm that the reported issue actually exists.

Never assume a ticket is accurate. Verify the problem yourself before proceeding.

### Common Commands

```bash
systemctl status <service>

ping

curl

ss -tulnp

journalctl
```

**Key Question**

> Can I reproduce or confirm the issue?

---

# 3. Investigate

**Objective:** Gather evidence to understand why the issue is occurring.

This stage involves collecting clues without making changes.

### Common Tasks

- Review service logs.
- Inspect configuration files.
- Check service dependencies.
- Inspect unit files.
- Verify permissions.
- Check available disk space.
- Review firewall or network settings.

**Key Question**

> Why is this happening?

---

# 4. Determine Root Cause

**Objective:** Identify the actual reason behind the problem.

Do not confuse the symptom with the root cause.

### Example

**Symptom**

> Apache will not start.

**Root Cause**

> Port 80 is already occupied by another process.

---

Another example:

**Symptom**

> SSH service failed.

**Root Cause**

> Invalid configuration inside `sshd_config`.

**Key Question**

> What is the actual cause of the issue?

---

# 5. Plan

**Objective:** Decide on the safest solution before making changes.

Before implementing a fix, consider:

- Will this affect users?
- Is there a maintenance window?
- Is approval required?
- Can the change be rolled back?
- Is there a safer alternative?

**Key Question**

> What is the safest way to resolve this issue?

---

# 6. Implement Solution

**Objective:** Apply the planned solution.

Only after understanding the problem should changes be made.

### Examples

```bash
systemctl restart <service>

systemctl start <service>

systemctl reload <service>

vi configuration.conf

dnf install <package>
```

**Key Question**

> Have I applied the planned solution correctly?

---

# 7. Verify Solution

**Objective:** Confirm that the implemented solution resolved the problem.

Never assume success simply because a command completed without errors.

### Common Verification Commands

```bash
systemctl status <service>

curl

ping

ssh

journalctl
```

Verify from both:

- The system's perspective (service status, logs)
- The user's perspective (is the service actually working?)

**Key Question**

> Did my solution actually fix the issue?

---

# 8. Prevent Future Recurrence

**Objective:** Reduce the likelihood of the same issue occurring again.

Examples include:

- Add monitoring and alerting.
- Improve documentation.
- Automate repetitive tasks.
- Improve configuration management.
- Create backups.
- Update operational procedures.
- Train administrators.

This step separates simply fixing problems from improving the overall reliability of the environment.

**Key Question**

> How can we prevent this issue from happening again?

---

# Example Scenario

## Ticket

> Users report they cannot SSH into the server.

### Observe

Users are unable to establish SSH connections.

↓

### Verify

```bash
systemctl status sshd
```

The service is reported as failed.

↓

### Investigate

```bash
journalctl -u sshd

sshd -t
```

Investigation reveals a syntax error in the SSH configuration.

↓

### Determine Root Cause

A recent configuration change introduced an invalid directive into `sshd_config`.

↓

### Plan

Correct the configuration and restart the SSH service.

↓

### Implement Solution

```bash
systemctl restart sshd
```

↓

### Verify Solution

```bash
systemctl status sshd

ssh user@server
```

The service is active, and remote SSH access is restored.

↓

### Prevent Future Recurrence

Require administrators to validate the SSH configuration using:

```bash
sshd -t
```

before restarting the service, or introduce a configuration review process.

---

# Summary

A professional Linux System Administrator should never jump directly into making changes.

This workflow minimizes unnecessary downtime, reduces human error, and promotes a consistent, professional approach to troubleshooting production Linux systems.

---

# Lesson 3 Summary

By the end of this lab, you should be able to:

- Identify your Linux system
- List systemd-managed services
- Interpret LOAD, ACTIVE, and SUB
- Distinguish between `active (running)` and `active (exited)`
- Inspect a service using `systemctl status`
- Read a unit file using `systemctl cat`
- Understand the difference between **enabled** and **static** services
- Develop the habit of **observing before modifying** a production system
