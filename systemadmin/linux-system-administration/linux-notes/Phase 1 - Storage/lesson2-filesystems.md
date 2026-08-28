# Phase 1 — Storage
# Lesson 2 — Filesystems

---

# What is a Filesystem?

A **filesystem** is the method an operating system uses to organize, store, retrieve, and manage data on a storage device.

Without a filesystem, a partition is simply an empty space capable of storing electrical charges (SSD) or magnetic data (HDD), but Linux has no idea how to organize files inside it.

Think of a filesystem as the **language** that Linux understands when reading and writing data.

---

# Analogy — A Library

Imagine a newly constructed library.

The building already exists.

The rooms are finished.

The shelves are installed.

However...

There are no labels.

There are no book categories.

There is no numbering system.

There is no librarian.

Every book is thrown randomly on the floor.

Technically...

The books are inside the library.

But nobody can find anything.

A filesystem is the librarian.

It decides:

- where files are stored
- where folders exist
- how files are named
- where free space exists
- how deleted files are reclaimed

---

# Relationship Between Partition and Filesystem

One of the biggest misconceptions beginners have is believing that a partition can already store files.

It cannot.

Storage workflow looks like this:

```text
Physical Disk

↓

Partition

↓

Filesystem

↓

Mount

↓

Store Files
```

A partition only reserves an area of storage.

The filesystem makes that partition usable.

---

# Real World Example

Suppose we created:

```text
/dev/sda1

100 GB
```

At this point...

Linux cannot yet store files.

We must first create a filesystem.

Example:

```bash
mkfs.ext4 /dev/sda1
```

Now Linux understands how to organize data inside that partition.

---

# Common Linux Filesystems

## ext4

The default filesystem used by most Linux distributions.

Characteristics:

- Stable
- Reliable
- Mature
- Excellent compatibility
- Journaling support

Recommended for:

- General Linux Servers
- Workstations
- RHCSA
- Enterprise Linux

---

## XFS

Designed for high performance.

Characteristics:

- Excellent for very large files
- Excellent scalability
- Online filesystem growth
- Default on RHEL and Rocky Linux

Recommended for:

- Enterprise Servers
- Database Servers
- Large Storage Arrays

---

## Btrfs

Modern Copy-on-Write filesystem.

Characteristics:

- Snapshots
- Compression
- Checksums
- Built-in RAID features

Recommended for:

- Advanced Linux environments
- Snapshot-heavy workloads

---

## FAT32

Commonly used for:

- USB Flash Drives
- Older devices

Characteristics:

- Excellent compatibility
- Maximum file size of 4 GB
- No Linux permissions

---

## NTFS

Windows filesystem.

Linux can read and write NTFS using appropriate drivers.

Commonly encountered when:

- Dual boot systems
- External hard drives
- Windows data recovery

---

# Which Filesystem Should You Use?

| Filesystem | Typical Use |
|------------|-------------|
| ext4 | General Linux Servers |
| XFS | Enterprise Linux Servers |
| Btrfs | Advanced Linux Features |
| FAT32 | Flash Drives |
| NTFS | Windows Storage |

---

# Creating a Filesystem

The command:

```bash
mkfs
```

means:

> Make Filesystem

Examples:

Create ext4:

```bash
mkfs.ext4 /dev/sdb1
```

Create XFS:

```bash
mkfs.xfs /dev/sdb1
```

---

# WARNING

Creating a filesystem destroys existing data on that partition.

Why?

Because Linux writes a completely new filesystem structure onto the partition.

Always verify the correct device before running:

```bash
mkfs
```

---

# Viewing Filesystem Information

Display block devices together with filesystems:

```bash
lsblk -f
```

Example:

```text
NAME    FSTYPE LABEL UUID

sda

├─sda1 ext4

├─sda2 xfs

└─sda3 LVM2_member
```

---

Display UUID information:

```bash
blkid
```

Example:

```text
/dev/sda1:

UUID="C4E2-31F4"

TYPE="ext4"
```

---

# Mounting Filesystems

Creating a filesystem does **not** automatically make it accessible.

Linux must mount it.

Mounting means:

> Connecting a filesystem into the Linux directory tree.

---

# Analogy — Plugging in a USB

Imagine plugging a USB flash drive into your laptop.

Before opening its contents:

The operating system must recognize it.

Assign it.

Connect it.

Linux does exactly that.

---

# Linux Directory Tree

Linux has one single directory hierarchy.

Everything eventually appears somewhere underneath:

```text
/
```

Instead of giving drives letters like Windows:

```text
C:

D:

E:
```

Linux attaches storage into directories.

Example:

```text
/

├── home

├── var

├── database

└── backup
```

---

# Mount Example

Suppose:

```text
/dev/sdb1
```

contains an ext4 filesystem.

Create a mount point:

```bash
mkdir /database
```

Mount it:

```bash
mount /dev/sdb1 /database
```

Now:

Everything written inside:

```text
/database
```

is actually stored on:

```text
/dev/sdb1
```

---

# Temporary Mounts

Mounting using:

```bash
mount
```

creates a **temporary mount**.

After reboot:

The filesystem disappears.

Why?

Because Linux forgets temporary mounts once the operating system restarts.

---

# Persistent Mounts

To mount automatically every boot:

```bash
nano /etc/fstab
```

---

# What is `/etc/fstab`?

`fstab` stands for:

> File System Table

It tells Linux:

> During boot...

> Mount these filesystems automatically.

---

# Example Entry

```text
UUID=2af3-67bc    /database    ext4    defaults    0    0
```

---

# Understanding Every Column

## Column 1

```text
UUID=2af3-67bc
```

This identifies the storage device.

Linux prefers UUID instead of:

```text
/dev/sdb1
```

because device names can change.

A UUID never changes unless the filesystem is recreated.

---

## Why UUID?

Imagine adding another SSD.

Yesterday:

```text
/dev/sdb
```

Today:

Linux detects disks differently.

Now:

```text
/dev/sdc
```

Your boot would fail.

UUID solves that problem.

---

## Column 2

```text
/database
```

The mount point.

Where the filesystem becomes accessible.

---

## Column 3

```text
ext4
```

Filesystem type.

Linux now knows which filesystem driver to load.

Examples:

```text
ext4

xfs

btrfs

vfat

ntfs
```

---

## Column 4

```text
defaults
```

Mount options.

`defaults` usually means:

- read/write
- allow execution
- automatic mounting
- normal permissions

Custom options can also be specified.

---

## Column 5

```text
0
```

Used by the old backup utility:

```text
dump
```

Today:

Almost always:

```text
0
```

---

## Column 6

```text
0
```

Filesystem check order during boot.

Values:

```text
0

Don't check

1

Check first

Usually root filesystem

2

Check afterwards
```

Example:

```text
/

1

/home

2
```

---

# Verify fstab Before Reboot

One of the best practices we learned.

Instead of rebooting:

```bash
mount -a
```

Linux attempts to mount every entry inside:

```text
/etc/fstab
```

If there are no errors:

Your configuration is probably correct.

---

# Common Mistakes

❌ Forgetting to create a filesystem after partitioning.

❌ Forgetting to mount the filesystem.

❌ Assuming mounting is permanent.

❌ Using `/dev/sdb1` instead of UUID inside `/etc/fstab`.

❌ Editing `/etc/fstab` incorrectly and making the system unbootable.

---

# Production Scenario

A company installs a new 2 TB SSD.

Workflow:

```text
Install SSD

↓

Partition Disk

↓

Create Filesystem

↓

Create Mount Point

↓

Mount

↓

Configure /etc/fstab

↓

Verify

↓

Applications Begin Using Storage
```

---

# Interview Questions

## What is a filesystem?

A filesystem is the structure Linux uses to organize, store, retrieve, and manage files inside a partition.

---

## Difference between a partition and a filesystem?

Partition:

Reserves storage space.

Filesystem:

Organizes storage space so Linux can actually store files.

---

## Why doesn't Linux use drive letters?

Linux uses a single directory hierarchy.

Storage devices are mounted into directories instead of receiving letters like Windows.

---

## Why should UUID be used in `/etc/fstab`?

Because Linux device names may change after hardware modifications.

UUID remains constant, making boot-time mounting reliable.

---

# Senior SysAdmin Notes

A partition without a filesystem is like buying an empty warehouse.

The warehouse exists.

But there are:

- no shelves
- no inventory system
- no organization

The filesystem turns empty storage into something Linux can actually use.

When troubleshooting storage, always think in layers:

```text
Disk

↓

Partition

↓

Filesystem

↓

Mount

↓

Applications
```

Whenever a storage issue occurs, determine which layer is failing before making changes.

---

# Lesson Summary

```text
Physical Disk
        │
        ▼
Partition
        │
        ▼
Filesystem (ext4, XFS, etc.)
        │
        ▼
Mount Point
        │
        ▼
Applications
```

A filesystem is what transforms a partition into usable storage. It provides the structure Linux needs to create directories, store files, track free space, manage metadata, and retrieve information efficiently. Without a filesystem, a partition is nothing more than an empty reserved space on a disk.
