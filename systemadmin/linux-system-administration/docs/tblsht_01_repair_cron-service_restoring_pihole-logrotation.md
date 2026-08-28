# Repairing a Broken `cron.service` and Restoring Pi-hole Log Rotation

## Introduction

This document records the investigation and recovery of a Pi-hole container whose log file continuously grew because scheduled log rotation was no longer functioning. Although the immediate symptom was excessive disk usage, the actual root cause was a corrupted `cron.service` unit file, preventing scheduled maintenance tasks from executing.

This incident demonstrates the importance of identifying the root cause instead of applying temporary fixes such as deleting log files.

---

# Overview

## Environment

- Debian 12 (Bookworm)
- Pi-hole running inside an LXC container
- systemd as the init system
- Pi-hole configured to use its own logrotate configuration
- Daily log rotation executed through cron

---

## Problem

Running:

```bash
df -h /
```

showed the root filesystem continuously increasing in usage.

Using:

```bash
du -xh --max-depth=1 /
```

revealed that the largest directory was:

```text
/var
```

Further investigation showed:

```text
/var/log/pihole
```

occupying approximately **777 MB**, with the active log file:

```text
/var/log/pihole/pihole.log
```

reaching approximately **757 MB**.

---

# Investigation Process

## Step 1 — Identify Disk Usage

Filesystem usage was verified using:

```bash
df -h /
```

Directory usage was then analyzed using:

```bash
du -xh --max-depth=1 /
du -xh --max-depth=1 /var
du -xh --max-depth=1 /var/log
```

This narrowed the issue to:

```text
/var/log/pihole
```

---

## Step 2 — Verify Log Rotation

Searching the system logrotate configuration:

```bash
logrotate --debug /etc/logrotate.conf
```

showed no Pi-hole logs.

Further searching revealed that Pi-hole ships its own configuration:

```bash
cat /etc/pihole/logrotate
```

Configuration:

```conf
daily
rotate 5
copytruncate
compress
delaycompress
```

This confirmed Pi-hole manages its own logs separately from the system's default logrotate configuration.

---

## Step 3 — Verify Scheduled Execution

The Pi-hole cron file was inspected:

```bash
cat /etc/cron.d/pihole
```

Important entry:

```cron
00 00 * * * root pihole flush once quiet
```

The daily maintenance job invokes:

```bash
logrotate /etc/pihole/logrotate
```

Therefore, log rotation depends on the cron daemon running successfully.

---

## Step 4 — Verify Cron Service

Checking the service:

```bash
systemctl status cron
```

returned:

```text
Loaded: error
UnitFileState=bad
```

Further inspection:

```bash
systemctl cat cron.service
```

failed because the unit could not be loaded.

---

## Step 5 — Investigate Unit File

Viewing the unit file:

```bash
head /lib/systemd/system/cron.service
```

showed binary garbage instead of a valid systemd unit.

Verification:

```bash
file /lib/systemd/system/cron.service
```

returned:

```text
data
```

instead of:

```text
ASCII text
```

This confirmed the unit file itself was corrupted.

---

## Step 6 — Additional Corruption Found

systemd also reported:

```text
/usr/lib/systemd/system-preset/90-systemd.preset
```

as invalid.

Inspection:

```bash
file /lib/systemd/system-preset/90-systemd.preset
```

also returned:

```text
data
```

instead of ASCII text.

---

## Step 7 — Repair systemd

The package was reinstalled:

```bash
apt update
apt install --reinstall systemd
```

After reinstalling:

```bash
file /lib/systemd/system-preset/90-systemd.preset
```

correctly returned:

```text
ASCII text
```

However:

```bash
cron.service
```

was still corrupted because it belongs to the **cron** package rather than **systemd**.

---

## Step 8 — Repair cron

Package ownership was verified:

```bash
dpkg -S /lib/systemd/system/cron.service
```

Result:

```text
cron
```

The package was reinstalled:

```bash
apt install --reinstall cron
```

Verification:

```bash
file /lib/systemd/system/cron.service
```

returned:

```text
ASCII text
```

Reloading systemd:

```bash
systemctl daemon-reload
```

Service status:

```bash
systemctl status cron
```

returned:

```text
Loaded: loaded
Active: active (running)
```

The cron daemon was successfully restored.

---

## Step 9 — Verify Log Rotation

Immediately after cron started, the Pi-hole reboot cron jobs executed.

Before:

```text
pihole.log
757 MB
```

After:

```text
pihole.log
9 KB

pihole.log.1
757 MB
```

Additional files:

```text
FTL.log
webserver.log
```

were also rotated successfully.

This confirmed that:

- cron was running
- Pi-hole's scheduled maintenance executed
- logrotate functioned correctly
- new log files were created
- logging continued normally

---

# Root Cause

The growing log file was **not** caused by Pi-hole itself.

The actual failure chain was:

```text
cron.service corrupted
        │
        ▼
cron daemon failed
        │
        ▼
Pi-hole scheduled jobs never executed
        │
        ▼
logrotate never ran
        │
        ▼
pihole.log continued growing
        │
        ▼
Disk usage steadily increased
```

---

# Lessons Learned

## 1. Investigate Before Making Changes

Deleting log files would only have treated the symptom.

Understanding why maintenance stopped prevented the issue from recurring.

---

## 2. Always Differentiate `df` and `du`

`df`

- Filesystem usage

`du`

- Directory and file usage

Both commands answer different questions and are often used together during disk investigations.

---

## 3. Applications May Use Their Own Logrotate Configuration

Not every application relies on:

```text
/etc/logrotate.conf
```

Pi-hole provides:

```text
/etc/pihole/logrotate
```

Understanding application-specific configurations is essential when troubleshooting.

---

## 4. Scheduled Maintenance Depends on Supporting Services

Log rotation was correctly configured.

The real failure was that cron never executed the scheduled task.

Always verify the scheduler before assuming the configuration is incorrect.

---

## 5. Verify Package Ownership

When a file is corrupted:

```bash
dpkg -S <file>
```

identifies which package owns it.

Reinstalling only the affected package is often safer and faster than rebuilding the system.

---

## 6. Verify the Repair

Do not assume a repair succeeded.

Confirm:

- Service status
- Configuration validity
- Expected scheduled execution
- Actual application behavior

Verification is the final step of every production change.

---

# Recommendations

- Monitor disk usage regularly using `df` and `du`.
- Verify that scheduled services such as `cron` or `systemd` timers are running.
- Investigate root causes instead of deleting files to reclaim space.
- Use package verification (`dpkg -V`) when corruption is suspected.
- Restore corrupted files by reinstalling the owning package rather than manually recreating system files.
- Validate every repair by confirming the service operates normally and the original problem no longer occurs.
- Treat maintenance failures (cron, timers, logrotate) as infrastructure issues because they can silently cause secondary problems over time.

---

# Conclusion

This incident began as a disk usage investigation but ultimately revealed corruption in critical system service files. By following a structured troubleshooting methodology—observe, verify, investigate, determine the root cause, implement the fix, and verify the outcome—the issue was resolved without unnecessary data loss or manual cleanup.

Rather than deleting a large log file, the underlying scheduling infrastructure was repaired, allowing Pi-hole's built-in log rotation to resume normal operation. This approach not only solved the immediate problem but also restored automated maintenance, preventing the issue from recurring.
