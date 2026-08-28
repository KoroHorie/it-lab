# Phase 2 — Lesson 2
# Hands-on Lab Cheat Sheet — Exploring systemd, Processes, and Parent-Child Relationships

> **Objective**
>
> Learn how Linux processes are related, how `systemd` fits into the process hierarchy, and how to investigate processes like a Linux System Administrator.

---

# Lab Objectives

By the end of this lab, you should be able to:

- Verify that `systemd` is PID 1
- Understand what each column in `ps -f` means
- Identify the parent process of a process
- Discover how Bash is started
- Understand why `systemd --user` exists
- Follow a process tree manually
- Investigate failed systemd units
- Practice the SysAdmin troubleshooting methodology

---

# SysAdmin Troubleshooting Workflow

Always investigate in this order:

```text
Observe
    │
    ▼
Verify
    │
    ▼
Investigate
    │
    ▼
Determine Root Cause
    │
    ▼
Plan
    │
    ▼
Implement Solution
    │
    ▼
Verify Solution
    │
    ▼
Prevent Future Recurrence
```

---

# Step 1 — Verify PID 1

## Command

```bash
ps -fp 1
```

## Purpose

Display detailed information about Process ID 1.

## Expected Output

```text
UID      PID PPID C STIME TTY TIME CMD
root       1    0 0 ...   ?   ...  /usr/lib/systemd/systemd --system
```

## What We Learned

- PID 1 is `systemd`
- `systemd` is the first userspace process
- PPID = 0 because it was started by the Linux kernel
- `systemd` has no interactive terminal (TTY = ?)

---

# Understanding ps -f Columns

| Column | Meaning |
|---------|----------|
| UID | Owner of the process |
| PID | Process ID |
| PPID | Parent Process ID |
| C | CPU utilization value |
| STIME | Process start time |
| TTY | Attached terminal |
| TIME | CPU time consumed |
| CMD | Command used to start the process |

---

# Step 2 — Find Bash PID

## Command

```bash
echo $$
```

## Purpose

Display the Process ID of the current Bash shell.

## Example

```text
6778
```

---

# Step 3 — Verify Bash Process

## Command

```bash
ps -fp <bash_pid>
```

Example

```bash
ps -fp 6778
```

## Purpose

Display detailed information about the current Bash process.

## Investigation

Observe:

- PID
- PPID
- CMD

---

# Step 4 — Investigate Bash's Parent

Suppose Bash has:

```text
PPID = 6760
```

Investigate:

```bash
ps -fp 6760
```

## Purpose

Identify which process started Bash.

---

# Example Process Tree

GNOME Desktop

```text
systemd (PID 1)
        │
        ▼
systemd --user
        │
        ▼
gnome-terminal
        │
        ▼
bash
        │
        ▼
ps
```

---

# Important Concept

## systemd

Responsible for:

- Entire operating system
- System services
- Mounts
- Timers
- Sockets
- Devices
- Targets

---

## systemd --user

Responsible for:

- User session
- User services
- User timers
- User sockets

It is **NOT** the Desktop Environment.

---

# Desktop Environment Example

GNOME

```text
systemd
      │
systemd --user
      │
gnome-terminal
      │
bash
```

KDE Plasma

```text
systemd
      │
systemd --user
      │
konsole
      │
bash
```

Notice:

Only the Desktop Environment changes.

---

# Step 5 — Check Failed Units

## Command

```bash
systemctl --failed
```

Alternative

```bash
systemctl list-units --failed
```

## Purpose

Display all failed systemd units.

---

# Lab Investigation

Failed Unit

```text
mcelog.service
```

---

# Investigate Failed Service

## Command

```bash
systemctl status mcelog.service
```

## Purpose

Determine:

- What the service is
- Why it failed
- Whether it affects production

---

# What We Learned

`mcelog`

Machine Check Exception Log

Purpose:

- Monitor CPU hardware errors
- Primarily designed for Intel CPUs
- May fail on AMD processors

---

# VM Investigation

Guest VM

```text
systemctl --failed

↓

1 Failed Unit
```

Proxmox Host

```text
systemctl --failed

↓

0 Failed Units
```

---

# Root Cause

The failure exists only inside the CentOS VM.

The physical Proxmox host is healthy.

---

# Fault Domain

```text
Physical Server
        │
        ▼
Proxmox
        │
        ▼
CentOS VM
        │
        ▼
mcelog.service Failed
```

Root Cause belongs to:

CentOS VM

NOT

- Proxmox
- Physical Hardware

---

# Important Lessons

## Never assume

Wrong mindset

```text
Service Failed

↓

Restart it immediately
```

Correct mindset

```text
Service Failed

↓

What is it?

↓

Why did it fail?

↓

Does it affect production?

↓

Investigate

↓

Then decide
```

---

# Production Mindset

Senior SysAdmins do **not** fix everything marked red.

They investigate first.

Sometimes the result is:

> Expected behavior.

No action required.

---

# Useful Commands Learned

## Display Process 1

```bash
ps -fp 1
```

---

## Display Current Bash PID

```bash
echo $$
```

---

## Display Specific Process

```bash
ps -fp <PID>
```

Example

```bash
ps -fp 6778
```

---

## Display Failed Units

```bash
systemctl --failed
```

---

## Alternative

```bash
systemctl list-units --failed
```

---

## Display Service Status

```bash
systemctl status <service>
```

Example

```bash
systemctl status mcelog.service
```

---

# Final Mental Model

```text
Linux Kernel
      │
      ▼
systemd (PID 1)
      │
      ▼
systemd --user
      │
      ▼
Desktop Environment
      │
      ▼
Terminal
      │
      ▼
bash
      │
      ▼
Every command you execute
```

---

# Key Takeaways

- `systemd` is PID 1 and the first userspace process.
- The Linux kernel starts `systemd`.
- Every command executed from Bash becomes a child process of Bash.
- `systemd --user` manages your user session and is separate from the desktop environment.
- `TTY = ?` indicates a process is not attached to an interactive terminal.
- `systemctl --failed` is a starting point for investigations, not an instruction to fix everything.
- Always determine the fault domain before taking action.
- Follow evidence instead of assumptions—this is the mindset of a professional Linux System Administrator.
