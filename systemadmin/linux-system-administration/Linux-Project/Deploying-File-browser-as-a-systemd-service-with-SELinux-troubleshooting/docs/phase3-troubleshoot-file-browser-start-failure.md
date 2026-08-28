# Phase 3 — Troubleshooting File Browser Service Startup Failure

---

# Objective

The objective of this phase is to investigate why File Browser successfully executes manually from the shell but fails when started by **systemd**.

Instead of making random configuration changes, this phase follows a structured production troubleshooting methodology to determine the actual root cause.

---

# Production Incident

After creating the Service Unit, File Browser failed to start.

Attempting to start the service:

```bash
systemctl start filebrowser.service
```

returned:

```text
Job for filebrowser.service failed because the control process exited with error code.
```

At this point, no assumptions should be made.

A professional Linux System Administrator gathers evidence before attempting any solution.

---

# Step 1 — Observe

The first observation was checking the service status.

```bash
systemctl status filebrowser.service
```

Result:

```text
Active: failed (Result: exit-code)
```

This confirms:

- systemd successfully loaded the Service Unit.
- systemd attempted to execute the service.
- the service immediately failed.

The Service Unit itself is not necessarily incorrect.

---

# Step 2 — Investigate the Logs

The next step was examining the service logs.

```bash
journalctl -xeu filebrowser.service
```

The important log entries were:

```text
Failed to locate executable
Permission denied

Failed at step EXEC spawning

status=203/EXEC
```

Additional entries showed:

```text
Scheduled restart job
Restart counter is at 5
```

Eventually:

```text
Start request repeated too quickly.
```

systemd stopped attempting to restart the service.

---

# Understanding the Error

Initially, the message appeared confusing.

It reported:

```text
Failed to locate executable
```

while simultaneously reporting:

```text
Permission denied
```

Since the executable clearly existed, the investigation shifted toward determining **why systemd could not execute it**.

---

# Step 3 — Verify Linux Permissions

The executable permissions were inspected.

```bash
ls -ld /opt

ls -ld /opt/filebrowser

ls -l /opt/filebrowser/filebrowser
```

Results:

```text
drwxr-xr-x

drwxr-xr-x

-rwxr-xr-x
```

Observations:

- Directories are accessible.
- Executable permission exists.
- Root owns the executable.

Traditional Linux permissions appeared correct.

---

# Step 4 — Verify Ownership

Ownership had already been corrected during deployment.

```text
root
root
```

This eliminated ownership as the cause.

---

# Step 5 — Verify the Executable Path

The Service Unit contained:

```ini
ExecStart=/opt/filebrowser/filebrowser
```

The executable existed exactly at that location.

The path itself was not the problem.

---

# Step 6 — Consider SELinux

At this stage:

- File exists.
- Permissions are correct.
- Ownership is correct.
- ExecStart is correct.

Yet systemd still reported:

```text
Permission denied
```

This strongly suggested another security layer beyond traditional Linux permissions.

The investigation shifted toward **SELinux**.

---

# Checking SELinux Status

The current SELinux mode was verified.

```bash
getenforce
```

Result:

```text
Enforcing
```

SELinux was actively enforcing security policies.

---

# Inspecting the Security Context

The executable's SELinux context was examined.

```bash
ls -Zd /opt/filebrowser/filebrowser
```

Result:

```text
unconfined_u:object_r:user_tmp_t:s0
```

---

# Root Cause

The executable retained the SELinux context:

```text
user_tmp_t
```

This occurred because the binary originally came from:

```text
/tmp
```

Although the file had been moved into:

```text
/opt/filebrowser
```

its SELinux label remained unchanged.

Linux file ownership changes do **not** automatically change SELinux contexts.

---

# Why did it work manually?

One important question arose during the investigation.

If SELinux blocked execution, why did:

```bash
./filebrowser
```

work successfully?

The answer is that **systemd** operates under a different SELinux domain than an interactive shell.

The shell was permitted to execute the binary.

systemd was not.

This demonstrates that:

Successful manual execution does **not** guarantee that a service can execute under systemd.

---

# Correcting the Security Context

Instead of disabling SELinux or weakening security, the correct file context was restored.

```bash
restorecon -v /opt/filebrowser/filebrowser
```

Output:

```text
Relabeled

user_tmp_t

↓

usr_t
```

The executable now matched its production location.

---

# Verifying the Solution

The service was restarted.

```bash
systemctl restart filebrowser.service
```

Status:

```bash
systemctl status filebrowser.service
```

Result:

```text
Active: active (running)

Main PID: 5062
```

File Browser successfully started under systemd.

---

# Root Cause Analysis

Incident:

```text
File Browser failed to start under systemd.
```

Observed Symptoms:

- status=203/EXEC
- Permission denied
- Restart attempts failed
- Service eventually stopped restarting

Evidence Collected:

- Executable exists.
- Permissions are correct.
- Ownership is correct.
- ExecStart is correct.
- SELinux enforcing.
- Executable labeled:

```text
user_tmp_t
```

Root Cause:

The executable inherited the SELinux label from `/tmp`.

systemd was prohibited from executing files with the:

```text
user_tmp_t
```

context.

Solution:

Restore the proper SELinux context.

```bash
restorecon -v /opt/filebrowser/filebrowser
```

Verification:

Service successfully entered:

```text
Active: active (running)
```

---

# Lessons Learned

This troubleshooting exercise demonstrates several important Linux administration concepts.

Traditional Unix permissions are only one layer of Linux security.

Even when:

- ownership is correct,
- permissions are correct,
- paths are correct,

SELinux may still deny access.

The correct approach is **not** to disable SELinux.

Instead:

- investigate,
- identify the incorrect context,
- restore the proper label,
- verify the service.

This follows enterprise Linux administration best practices.

---

# What We Learned

After completing Phase 3, we learned:

- How to troubleshoot failed systemd services.
- How to use:
  - `systemctl status`
  - `journalctl`
  - `getenforce`
  - `ls -Z`
  - `restorecon`
- Why Linux DAC permissions are different from SELinux MAC policies.
- Why files moved from `/tmp` may retain temporary security contexts.
- Why manually executing an application does not guarantee systemd can execute it.
- Why restoring the correct SELinux context is preferable to disabling SELinux.
- How structured Root Cause Analysis leads to accurate and secure solutions.
