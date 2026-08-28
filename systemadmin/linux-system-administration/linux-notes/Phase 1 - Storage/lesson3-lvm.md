# Phase 1 — Storage
# Lesson 3 — Logical Volume Manager (LVM)

---

# What is LVM?

**Logical Volume Manager (LVM)** is a storage management technology in Linux that provides an abstraction layer between physical storage devices and the filesystems that use them.

Unlike traditional partitioning, LVM allows storage to be expanded, resized, and managed dynamically without being limited by fixed partition sizes.

LVM is one of the reasons Linux is widely used in enterprise environments because it provides **flexibility** in managing storage.

---

# The Problem with Traditional Partitions

Suppose you created a server with the following partition layout:

```text
Disk

+---------------------------+
| /        | /home          |
| 100 GB   | 400 GB         |
+---------------------------+
```

Months later...

The operating system partition (`/`) becomes full.

Unfortunately:

```text
/

100 GB (100%)

-----------------------

/home

400 GB (Only 80 GB Used)
```

You have plenty of free storage.

The problem is...

The free space exists **inside another partition.**

Traditional partitions cannot simply borrow unused space from one another.

Usually you would need to:

- Unmount filesystems
- Resize partitions
- Possibly reboot
- Risk data loss if done incorrectly

---

# Why LVM Exists

LVM solves this problem.

Instead of allocating storage directly to partitions, Linux introduces another storage management layer.

Now storage becomes **flexible**.

Unused storage can be allocated whenever needed.

---

# Analogy — Water Tank System

Imagine a house with three water tanks.

Without LVM:

```text
Kitchen Tank

100 L

Bathroom Tank

100 L

Garden Tank

100 L
```

If the kitchen runs out of water...

The bathroom still has plenty.

Unfortunately...

The kitchen cannot borrow water from the bathroom.

Each tank is isolated.

---

Now imagine connecting all tanks into one large reservoir.

```text
Large Water Reservoir

300 L
```

Now every room simply requests water whenever needed.

That reservoir is exactly how a **Volume Group** works.

---

# LVM Architecture

LVM introduces three new components.

```text
Physical Volume (PV)

↓

Volume Group (VG)

↓

Logical Volume (LV)
```

Only after creating a Logical Volume do we create a filesystem.

Storage workflow now becomes:

```text
Physical Disk

↓

Partition

↓

Physical Volume (PV)

↓

Volume Group (VG)

↓

Logical Volume (LV)

↓

Filesystem

↓

Mount Point
```

---

# Physical Volume (PV)

A **Physical Volume** is a storage device prepared for LVM.

Examples:

- Entire disks
- Partitions

Linux converts them into storage that LVM can manage.

Example:

```text
/dev/sdb1

↓

Physical Volume
```

Create a Physical Volume:

```bash
pvcreate /dev/sdb1
```

View Physical Volumes:

```bash
pvs
```

Detailed information:

```bash
pvdisplay
```

---

# Analogy — Building Materials

Imagine building a warehouse.

The physical volumes are simply stacks of construction materials.

By themselves...

They are not useful yet.

They must first be gathered together.

---

# Volume Group (VG)

A **Volume Group** combines one or more Physical Volumes into one large storage pool.

Think of it as a giant storage reservoir.

Example:

```text
Disk A

500 GB

+

Disk B

500 GB

↓

Volume Group

1000 GB
```

Linux no longer thinks about individual disks.

It simply sees one storage pool.

Create Volume Group:

```bash
vgcreate vg_data /dev/sdb1
```

Display Volume Groups:

```bash
vgs
```

Detailed information:

```bash
vgdisplay
```

---

# Analogy — Company Budget

Imagine three departments.

Each department contributes money.

```text
Sales

₱100,000

IT

₱200,000

HR

₱50,000
```

The company collects everything into one budget.

```text
Company Budget

₱350,000
```

Whenever a department needs funds...

The money comes from the shared budget.

The Volume Group behaves exactly like this.

---

# Logical Volume (LV)

A **Logical Volume** is the storage actually presented to Linux.

It behaves exactly like a traditional partition.

Applications never interact with the Volume Group.

They interact with Logical Volumes.

Example:

```text
Volume Group

1 TB

↓

LV Root

200 GB

LV Database

500 GB

LV Backup

300 GB
```

Create Logical Volume:

```bash
lvcreate -L 200G -n lv_root vg_data
```

Display Logical Volumes:

```bash
lvs
```

Detailed information:

```bash
lvdisplay
```

---

# Analogy — Bank Account

The company budget contains all available money.

Employees don't spend directly from the company budget.

Instead...

Money is allocated into departments.

Each department receives a budget.

Logical Volumes are those department budgets.

---

# Creating a Filesystem

After creating the Logical Volume:

```bash
mkfs.ext4 /dev/vg_data/lv_root
```

Now Linux can store files.

---

# Mounting

Create mount point:

```bash
mkdir /database
```

Mount:

```bash
mount /dev/vg_data/lv_root /database
```

Persistent mount:

```text
UUID=xxxxxxxx    /database    ext4    defaults    0    0
```

inside:

```text
/etc/fstab
```

---

# Why Enterprise Linux Uses LVM

Suppose a database grows unexpectedly.

Traditional partitions:

```text
Database

200 GB

100%

Free Space

300 GB

Elsewhere
```

Very difficult to resize.

With LVM:

Administrator simply allocates more storage from the Volume Group.

No repartitioning.

Much less downtime.

---

# Extending Storage

Suppose:

```text
Volume Group

1 TB

Logical Volume

200 GB

Unused

800 GB
```

The administrator extends it:

```bash
lvextend -L +100G /dev/vg_data/lv_root
```

Now:

```text
Logical Volume

300 GB
```

The filesystem must then be expanded.

For ext4:

```bash
resize2fs /dev/vg_data/lv_root
```

For XFS:

```bash
xfs_growfs /database
```

Storage becomes immediately available.

---

# When the Volume Group Has No Free Space

Suppose:

```text
Volume Group

100%

Used
```

Now there is no storage left.

Solution:

Install another disk.

Create Physical Volume:

```bash
pvcreate /dev/sdc1
```

Add it:

```bash
vgextend vg_data /dev/sdc1
```

The Volume Group immediately grows.

Now Logical Volumes can expand again.

---

# Production Scenario

A production database reaches 95% storage utilization.

Investigation shows:

The Volume Group still contains 500 GB of free space.

Administrator performs:

```text
lvextend

↓

resize2fs

↓

Filesystem Grows

↓

Database Continues Running
```

No repartitioning.

Minimal downtime.

---

# Common Mistakes

❌ Forgetting to resize the filesystem after extending the Logical Volume.

❌ Extending the wrong Logical Volume.

❌ Assuming the Volume Group has free storage without checking.

❌ Forgetting to update monitoring after storage expansion.

❌ Thinking LVM replaces RAID.

---

# LVM vs RAID

Many beginners confuse them.

They solve different problems.

RAID:

Provides:

- Redundancy
- Performance
- Fault tolerance

LVM:

Provides:

- Flexible storage management
- Easy resizing
- Dynamic allocation

Many enterprise servers use:

```text
RAID

↓

LVM

↓

Filesystem
```

Both technologies complement each other.

---

# Interview Questions

## What is LVM?

LVM is a storage management technology that allows Linux administrators to manage storage dynamically using Physical Volumes, Volume Groups, and Logical Volumes.

---

## Components of LVM

```text
Physical Volume

↓

Volume Group

↓

Logical Volume
```

---

## Why use LVM?

- Easier storage expansion
- Better flexibility
- Minimal downtime
- Enterprise storage management

---

## Difference between a Partition and a Logical Volume

Partition:

Fixed.

Logical Volume:

Flexible and can grow or shrink depending on available storage.

---

## Difference between RAID and LVM

RAID protects or improves storage hardware.

LVM manages storage allocation.

They solve different problems and are commonly used together.

---

# Senior SysAdmin Notes

One of the biggest advantages of LVM is **planning for the future.**

Storage requirements almost always increase over time.

A good System Administrator assumes today's storage layout will eventually need to grow.

Rather than rebuilding partitions years later, LVM provides a flexible architecture that allows storage to evolve with the server.

This is why many enterprise Linux deployments place LVM between the physical disks and the filesystem.

---

# Lesson Summary

```text
Physical Disk
        │
        ▼
Partition
        │
        ▼
Physical Volume (PV)
        │
        ▼
Volume Group (VG)
        │
        ▼
Logical Volume (LV)
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

Logical Volume Manager introduces a flexible storage layer that separates physical storage from logical storage. Instead of being limited by fixed partitions, administrators can expand storage dynamically, combine multiple disks into one storage pool, and allocate storage whenever applications require additional capacity. This flexibility is one of the reasons LVM has become a standard storage technology in enterprise Linux environments.
