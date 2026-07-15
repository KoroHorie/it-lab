# Phase 1 — Storage
# Lesson 6 — Disk Monitoring & Storage Troubleshooting

---

# Why Disk Monitoring Matters

Storage is one of the most critical resources in a Linux server.

A server can continue running with:

- High CPU usage
- High Memory usage
- Heavy Network traffic

But once storage becomes completely full:

```text
Storage

100%
```

Applications begin to fail.

Examples:

- Users cannot upload files.
- Databases cannot write transactions.
- Logs stop being written.
- Docker containers may fail.
- Services may unexpectedly stop.

This is why Linux System Administrators continuously monitor storage utilization.

---

# The SysAdmin Mindset

One of the biggest lessons we learned during Phase 1 is:

> **Never assume adding more storage is the solution.**

When someone reports:

> "The server is full."

Do **not** immediately expand storage.

Instead, investigate first.

A professional System Administrator follows this workflow:

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

Verify

↓

Prevent Future Recurrence
```

---

# Step 1 — Check Filesystem Usage

The first command most Linux administrators use is:

```bash
df -h
```

---

# What does `df` mean?

`df`

↓

**Disk Free**

It reports the usage of mounted filesystems.

The `-h` option displays values in a human-readable format.

---

# Example

```bash
df -h
```

Output:

```text
Filesystem      Size  Used Avail Use% Mounted on

/dev/sda2       100G   95G    5G   95% /

/dev/sdb1       500G  120G  380G   24% /database
```

---

# Understanding Every Column

```text
Filesystem
```

The storage device.

---

```text
Size
```

Total filesystem capacity.

---

```text
Used
```

Current storage consumed.

---

```text
Avail
```

Remaining usable space.

---

```text
Use%
```

Percentage currently used.

---

```text
Mounted on
```

Where the filesystem exists inside Linux.

---

# Analogy — Apartment Building

Imagine every filesystem is an apartment.

```text
Apartment A

100 Rooms

95 Occupied

5 Available

↓

95%
```

`df` tells you:

- How many rooms exist
- How many are occupied
- How many remain

---

# What if One Filesystem is 100%?

Suppose:

```text
/

100%
```

Now we know **which filesystem** is full.

But we still don't know:

> **What actually consumed the storage?**

That is where `du` becomes useful.

---

# Step 2 — Investigate Directory Sizes

Command:

```bash
du
```

---

# What does `du` mean?

`du`

↓

**Disk Usage**

Instead of reporting filesystem usage,

`du` reports the size of directories and files.

---

# Example

```bash
du -sh /var
```

Output:

```text
43G

/var
```

Now we know:

```text
/var

↓

43 GB
```

But...

That still isn't enough.

---

# Continue Investigating

Suppose:

```text
/var

43 GB
```

We continue:

```bash
du -sh /var/*
```

Example:

```text
30G

/var/log

5G

/var/lib

2G

/var/cache
```

Now we have narrowed the investigation.

---

# Continue Again

```bash
du -sh /var/log/*
```

Eventually:

```text
28G

access.log
```

Now we found the culprit.

---

# Analogy — Detective Work

Imagine your electricity bill suddenly doubles.

Do you immediately buy a bigger power supply?

No.

You investigate.

You discover:

```text
Air Conditioner

Running 24 Hours
```

Problem found.

Storage troubleshooting works exactly the same way.

---

# lsblk

Command:

```bash
lsblk
```

Displays:

- Disks
- Partitions
- LVM
- Mount Points

Example:

```text
NAME

SIZE

TYPE

MOUNTPOINT

sda

1T

disk

├─sda1

500M

part

/boot

├─sda2

100G

part

/

└─sda3

899G

part

LVM
```

---

# When do SysAdmins use `lsblk`?

Before modifying storage.

For example:

Suppose:

```text
/

100%
```

After investigation:

Storage really needs expansion.

Now we must understand:

- Which partition?
- Is it LVM?
- Which disk?
- How much storage exists?

`lsblk` provides the storage layout.

---

# blkid

Command:

```bash
blkid
```

Displays:

- UUID
- Filesystem
- Labels

Example:

```text
/dev/sda2

UUID=xxxxx

TYPE=ext4
```

---

# When is `blkid` useful?

Mainly when editing:

```text
/etc/fstab
```

Instead of:

```text
/dev/sdb1
```

we use:

```text
UUID=xxxxxxxx
```

---

# findmnt

Command:

```bash
findmnt
```

Shows:

Current mounted filesystems.

Example:

```text
TARGET

SOURCE

/

/dev/sda2

/database

/dev/vg_data/lv_db
```

Useful for verifying mount locations.

---

# Production Troubleshooting

Suppose a ticket arrives.

---

## Ticket

```text
Users cannot upload files.

Storage Full.
```

---

## Junior SysAdmin

Immediately increases storage.

Problem returns two weeks later.

---

## Senior SysAdmin

Starts investigating.

```bash
df -h
```

Find:

```text
/

100%
```

Next:

```bash
du -sh /*
```

Find:

```text
/var

43G
```

Continue:

```bash
du -sh /var/*
```

Find:

```text
/var/log

39G
```

Continue:

```bash
du -sh /var/log/*
```

Find:

```text
access.log

38G
```

Root Cause discovered.

---

# Root Cause First

This became one of the biggest lessons we learned.

Just because storage is full does **not** mean storage must be expanded.

Sometimes:

The application itself is misbehaving.

Examples:

- Infinite logging
- Debug mode enabled
- Huge temporary files
- Malware
- Self-replicating files
- Runaway Docker logs

Always understand **why** storage became full.

---

# Never Delete Random Files

One real lesson from our discussion.

Deleting random files because:

> "Maybe these aren't important."

can destroy production systems.

Applications depend on many files.

Instead:

Investigate first.

Understand ownership.

Plan the solution.

Then act.

---

# Log Rotation

Suppose:

```text
access.log

38 GB
```

Do we delete it?

Usually:

No.

Instead:

Archive it.

Compress it.

Rotate it.

---

# What is Log Rotation?

Instead of keeping:

```text
access.log

38 GB
```

Linux renames it:

```text
access.log

↓

access.log.1
```

Then compresses it:

```text
access.log.1.gz
```

Finally:

Creates a brand-new:

```text
access.log
```

Applications continue writing logs normally.

Old logs remain available for auditing.

---

# logrotate

Linux provides:

```text
logrotate
```

to automate this process.

It can:

- Rotate logs
- Compress logs
- Remove old logs
- Keep only recent history

Example policies:

```text
Rotate weekly

Keep 4 weeks

Compress old logs
```

---

# Why Log Rotation Exists

Imagine never cleaning your inbox.

Eventually:

```text
500,000 Emails
```

Searching becomes slow.

Storage grows.

Organization disappears.

Log rotation performs housekeeping automatically.

---

# Storage Expansion

Suppose investigation concludes:

Storage truly needs expansion.

Now:

Use:

```bash
lsblk
```

Check:

- Physical Disk
- Volume Group
- Free Space

If LVM exists:

```text
PV

↓

VG

↓

LV

↓

Filesystem
```

Expand the Logical Volume instead of repartitioning whenever possible.

---

# Common Mistakes

❌ Expanding storage before investigating.

❌ Deleting random files.

❌ Ignoring log growth.

❌ Forgetting log rotation.

❌ Assuming every "Disk Full" issue requires larger disks.

---

# Interview Questions

## First command when storage becomes full?

```bash
df -h
```

Determine which filesystem is affected.

---

## After identifying the filesystem?

Use:

```bash
du
```

to locate the directories consuming storage.

---

## Should you immediately expand storage?

No.

Investigate first.

Determine the root cause.

---

## Why use logrotate?

To automatically archive, compress, and remove old log files, preventing storage exhaustion.

---

# Senior SysAdmin Notes

One lesson separates Junior and Senior System Administrators.

Junior Admin:

> "Storage is full."

↓

"Let's increase storage."

---

Senior Admin:

> "Why did storage become full?"

Only after answering that question does the solution become obvious.

Sometimes:

The correct solution is:

- Fix logging.
- Remove temporary files.
- Configure retention.
- Archive data.
- Tune applications.

Expanding storage should usually be the **last** step, not the first.

---

# Lesson Summary

```text
User Reports

↓

Storage Full

↓

df -h

↓

Identify Filesystem

↓

du

↓

Locate Large Directories

↓

Locate Root Cause

↓

Plan Solution

↓

Implement

↓

Verify

↓

Prevent Future Recurrence
```

Disk monitoring is not simply checking available storage. It is the process of continuously observing storage health, identifying abnormal growth, investigating root causes, and implementing long-term solutions that keep Linux servers reliable, stable, and maintainable in production environments.
