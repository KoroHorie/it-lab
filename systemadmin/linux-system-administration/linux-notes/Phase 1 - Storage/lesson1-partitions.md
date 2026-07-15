# Phase 1 — Storage
# Lesson 1 — Partitions

---

# What is a Partition?

A **partition** is a logical division of a physical storage device.

A physical disk is just one large storage device. Before Linux can organize data efficiently, the storage is divided into smaller logical sections called **partitions**.

Each partition behaves as an independent storage area even though they all reside on the same physical disk.

---

# Analogy — A Piece of Land

Imagine you bought a **10-hectare land**.

Without dividing it, you could technically build everything on it:

- Your house
- A warehouse
- A farm
- A parking lot

But that would become disorganized very quickly.

Instead, you divide the land into lots.

```text
10 Hectares

+------------------------------------------------------+
|                                                      |
|                 Entire Piece of Land                 |
|                                                      |
+------------------------------------------------------+

↓

Partitioning

+-----------+------------+-----------------------------+
| House Lot | Farm Lot   | Warehouse Lot              |
+-----------+------------+-----------------------------+
```

It is still one piece of land.

You simply assigned different purposes to different sections.

Linux does the exact same thing to a storage device.

---

# Physical Disk vs Partition

Before partitioning:

```text
SSD

+-------------------------------------------+
|                                           |
|                1 TB SSD                   |
|                                           |
+-------------------------------------------+
```

After partitioning:

```text
SSD

+---------+-----------+----------------------+
|   100G  |   400G    |       500G           |
+---------+-----------+----------------------+
```

Linux now sees three independent storage areas.

---

# Why do we partition disks?

Without partitions, everything would exist inside one massive storage space.

Imagine your operating system, databases, logs, user files and backups all living inside one directory.

Eventually:

- logs grow
- databases grow
- users upload files
- applications create temporary files

Everything competes for the same storage.

Partitioning allows administrators to separate workloads.

Example:

```text
/

Operating System

100 GB

----------------------------

/home

User Files

500 GB

----------------------------

/var

Logs

Databases

Applications

300 GB

----------------------------

Swap

16 GB
```

Now each area has its own responsibility.

---

# Why Enterprise Servers Use Multiple Partitions

Suppose your web server suddenly receives millions of requests.

Apache starts writing logs.

Those logs live inside:

```text
/var/log
```

If there was only one partition:

```text
/

1 TB
```

Those logs could eventually consume the entire storage.

The operating system itself may stop functioning correctly.

However, if:

```text
/var

300 GB
```

becomes full,

the operating system partition still has free space.

The server is much easier to recover.

This is called **isolation**.

One workload cannot easily destroy another.

---

# Senior SysAdmin Mindset

Partitions are **not created for performance**.

They exist for:

- Organization
- Isolation
- Easier administration
- Easier recovery
- Better storage planning

Whenever you see multiple partitions on an enterprise server, someone intentionally designed that layout.

---

# Partition Table

Before Linux can use partitions, it must first understand where every partition begins and ends.

This information is stored inside something called a **Partition Table**.

Think of it as the table of contents of a book.

Without it, Linux has no idea where each partition exists.

---

# Analogy — Table of Contents

Imagine opening a textbook.

The first page says:

```text
Chapter 1

Page 1

Chapter 2

Page 43

Chapter 3

Page 87
```

The operating system does something similar.

The partition table tells Linux:

```text
Partition 1

Starts here

Ends here

Partition 2

Starts here

Ends here
```

Without that information, Linux cannot locate partitions.

---

# MBR (Master Boot Record)

MBR is the older partition table format.

Characteristics:

- Maximum disk size of approximately **2 TB**
- Maximum of **4 Primary Partitions**
- Used mostly with Legacy BIOS systems

Example:

```text
Disk

↓

MBR

↓

Partition 1

Partition 2

Partition 3

Partition 4
```

If more than four partitions are needed, an Extended Partition must be created.

---

# GPT (GUID Partition Table)

GPT is the modern partition table.

Characteristics:

- Supports disks larger than 2 TB
- Up to 128 partitions by default
- Used together with UEFI firmware
- Stores backup partition tables for improved reliability

Example:

```text
Disk

↓

GPT

↓

Partition 1

Partition 2

Partition 3

...

Partition 128
```

---

# Which should you use?

Today:

Always prefer **GPT**.

Only use MBR if compatibility with legacy hardware is required.

---

# Linux Device Naming

Linux represents hardware devices as files.

Storage devices appear under:

```bash
/dev/
```

Examples:

```bash
/dev/sda
/dev/sdb
/dev/sdc
```

Meaning:

```text
/dev/sda

↓

First Storage Device

/dev/sdb

↓

Second Storage Device

/dev/sdc

↓

Third Storage Device
```

---

# Partition Naming

Partitions simply receive numbers.

```bash
/dev/sda1
/dev/sda2
/dev/sda3
```

Meaning:

```text
sda

↓

Whole Disk

sda1

↓

Partition 1

sda2

↓

Partition 2
```

For NVMe drives:

```bash
/dev/nvme0n1

↓

Whole Disk

/dev/nvme0n1p1

↓

Partition 1
```

---

# Common Partition Utilities

## fdisk

Traditional partitioning utility.

Commonly used in:

- RHCSA
- Basic administration
- MBR
- GPT

Example:

```bash
fdisk /dev/sda
```

---

## parted

Modern partitioning utility.

Better suited for:

- GPT disks
- Large storage devices
- Advanced partitioning

Example:

```bash
parted /dev/sda
```

---

# Viewing Storage

Display block devices:

```bash
lsblk
```

Example:

```text
NAME

SIZE

TYPE

sda

1T

disk

├─sda1

100G

part

├─sda2

400G

part

└─sda3

500G

part
```

---

Display detailed partition information:

```bash
fdisk -l
```

---

# Why does Linux need `partprobe`?

This was one of the most important commands we learned.

After changing partitions, the physical disk has changed.

However...

The Linux Kernel may still believe the old partition layout exists.

Imagine someone hands you a new blueprint for a building.

You haven't looked at it yet.

You're still using yesterday's blueprint.

Linux behaves the same way.

---

# Analogy — Updated Blueprint

Yesterday:

```text
Room

100 m²
```

Today the architect expanded it.

```text
Room

300 m²
```

You haven't looked at the updated blueprint.

You'll continue believing the room is only:

```text
100 m²
```

until someone tells you:

> Look again.

That is exactly what:

```bash
partprobe
```

does.

It tells the kernel:

> "Re-read the partition table."

---

# Production Scenario

Suppose VMware expanded a virtual disk from:

```text
100 GB

↓

300 GB
```

Inside Linux:

```bash
lsblk
```

still reports:

```text
100 GB
```

The administrator runs:

```bash
partprobe
```

Now Linux detects the updated partition table.

No reboot is required (when supported).

Only then can LVM or the filesystem be expanded.

---

# Common Mistakes

❌ Assuming partitioning automatically creates a filesystem.

❌ Forgetting to notify the kernel after partition changes.

❌ Using MBR for modern enterprise storage.

❌ Partitioning without planning future storage growth.

❌ Creating partitions without understanding what each workload requires.

---

# Interview Questions

## What is a partition?

A partition is a logical division of a physical storage device that allows the operating system to organize storage into separate independent areas.

---

## Difference between MBR and GPT?

| MBR | GPT |
|------|------|
| Older | Modern |
| Up to 2 TB | Supports very large disks |
| 4 Primary Partitions | Up to 128 Partitions |
| BIOS | UEFI |

---

## Why do administrators partition disks?

- Better organization
- Workload isolation
- Easier administration
- Easier recovery
- Better storage planning

---

## Why use `partprobe`?

Because after changing partitions, Linux may still be using the old partition table currently loaded in memory.

`partprobe` tells the kernel to re-read the partition table without requiring a reboot whenever possible.

---

# Senior SysAdmin Notes

Storage administration always begins with planning.

Before touching a production disk, ask yourself:

- What is this server used for?
- What applications will run here?
- How much storage will each application consume?
- Will storage need to grow in the future?
- Should LVM be used instead of traditional partitions?

Professional System Administrators don't create partitions randomly.

They design storage layouts based on how the server will be used over the next several years.

---

# Lesson Summary

```text
Physical Disk
        │
        ▼
Partition Table (GPT / MBR)
        │
        ▼
Partitions
        │
        ▼
Filesystem
        │
        ▼
Mount Point
        │
        ▼
Applications
```

Partitions are the **foundation** of Linux storage. Every storage technology we learned later: Filesystems, LVM, RAID, Swap, and Disk Monitoring builds upon this first layer.