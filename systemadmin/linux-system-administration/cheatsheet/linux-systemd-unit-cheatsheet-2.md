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
