# Phase 1 — Storage
# Lesson 4 — RAID (Redundant Array of Independent Disks)

---

# What is RAID?

**RAID (Redundant Array of Independent Disks)** is a storage technology that combines multiple physical disks into a single logical storage device to achieve one or more of the following goals:

- Improve Performance
- Improve Availability
- Improve Fault Tolerance
- Increase Storage Capacity

Linux, Windows, and enterprise storage appliances all support RAID in different implementations.

---

# Why RAID Exists

Imagine your company has only one hard disk.

```text
Server

↓

SSD

↓

Company Database
```

If that SSD suddenly fails:

```text
SSD

↓

Dead
```

The entire server becomes unavailable.

The company loses access to:

- Database
- Website
- Files
- Applications

One disk becomes a **Single Point of Failure (SPOF).**

RAID was created to reduce that risk.

---

# Analogy — Teamwork

Imagine carrying heavy boxes.

One person:

```text
📦

↓

Slow
```

Four people:

```text
📦 📦 📦 📦

↓

Faster

More Reliable
```

Instead of relying on one person, the work is distributed across several people.

RAID applies the same idea to storage.

---

# RAID is NOT a Backup

One of the biggest misconceptions.

Many beginners believe:

> "I have RAID, therefore my data is backed up."

This is **false.**

RAID protects against **disk failure.**

It does **not** protect against:

- Accidental deletion
- Malware
- Ransomware
- File corruption
- Human error
- Fire
- Flood

If you delete a file:

```text
rm important.docx
```

RAID happily deletes it from every disk.

Always remember:

```text
RAID

≠

Backup
```

---

# Common RAID Levels

The RAID levels we learned are:

- RAID 0
- RAID 1
- RAID 5
- RAID 6
- RAID 10

Each one solves different problems.

---

# RAID 0 — Striping

Purpose:

Increase performance.

Minimum disks:

```text
2
```

No redundancy.

---

# How RAID 0 Works

Instead of writing a file to one disk...

Linux splits it into pieces.

```text
File

↓

Part A

Part B

Part C

Part D
```

Those parts are written simultaneously.

```text
Disk 1

Part A

Disk 2

Part B

Disk 3

Part C

Disk 4

Part D
```

This process is called:

**Striping**

---

# Analogy — Four People Painting

One painter:

```text
██████████

10 Minutes
```

Four painters:

```text
██ ██ ██ ██

2 Minutes
```

The work is divided.

Everyone paints simultaneously.

That's RAID 0.

---

# Advantages

- Fastest RAID level
- Excellent read performance
- Excellent write performance
- Full disk capacity available

---

# Disadvantages

No fault tolerance.

If one disk fails:

```text
Disk 2

↓

Dead
```

Every file becomes incomplete.

Entire array is lost.

---

# Typical Use Cases

- Temporary storage
- High-speed scratch disks
- Video editing
- Non-critical workloads

---

# RAID 1 — Mirroring

Purpose:

Redundancy.

Minimum disks:

```text
2
```

---

# How RAID 1 Works

Every write operation is duplicated.

```text
Disk 1

File A

↓

Disk 2

File A
```

Both disks always contain identical data.

---

# Analogy — Photocopy

Imagine making two identical copies of an important document.

```text
Original

↓

Copy
```

If one copy is destroyed:

The second copy still exists.

---

# Advantages

- Excellent redundancy
- Very simple recovery
- Survives one disk failure

---

# Disadvantages

Storage efficiency is only 50%.

Example:

```text
2 × 1 TB

↓

Usable

1 TB
```

Half of the storage is sacrificed for redundancy.

---

# Typical Use Cases

- Operating Systems
- Small Databases
- Critical Boot Drives

---

# RAID 5 — Striping with Distributed Parity

Purpose:

Performance + Redundancy.

Minimum disks:

```text
3
```

---

# What is Parity?

Parity is **NOT** another copy of your files.

Parity stores **mathematical information** about the data inside a stripe.

That mathematical information allows RAID to rebuild the contents of one failed disk.

---

# Analogy — Solving for the Missing Number

Suppose:

```text
A = 2

B = 5

C = 7
```

Parity stores information that relates those values.

If:

```text
B

↓

Fails
```

RAID already knows:

- A
- C
- Parity

Using that information, RAID reconstructs:

```text
B = 5
```

Notice:

Parity did **not** contain another copy of B.

It contained enough mathematical information to recreate it.

---

# Stripe Layout

Example:

```text
Stripe 1

Disk 1

A

Disk 2

B

Disk 3

Parity

------------------------

Stripe 2

Disk 1

C

Disk 2

Parity

Disk 3

D

------------------------

Stripe 3

Disk 1

Parity

Disk 2

E

Disk 3

F
```

Notice:

Parity rotates.

No single disk becomes the "Parity Disk."

Every disk shares parity responsibilities.

This prevents one disk from becoming a performance bottleneck.

---

# Why Rotate the Parity?

Imagine only Disk 3 always stores parity.

Every write operation must update Disk 3.

Eventually:

```text
Disk 3

↓

Constantly Busy
```

Performance decreases.

Instead:

Parity rotates.

```text
Stripe 1

Parity → Disk 3

Stripe 2

Parity → Disk 2

Stripe 3

Parity → Disk 1
```

The workload is distributed evenly.

---

# Disk Failure

Suppose:

```text
Disk 2

↓

Fails
```

RAID still has:

- Remaining data
- Parity information

It reconstructs the missing blocks on-the-fly.

Users often continue accessing files while the failed disk is replaced.

---

# Advantages

- Good performance
- Good storage efficiency
- One disk failure tolerated

---

# Disadvantages

Parity calculations slow down write operations.

Array rebuilds are time-consuming.

---

# Typical Use Cases

- File Servers
- NAS
- Archive Storage
- General Enterprise Storage

---

# RAID 6 — Double Distributed Parity

Purpose:

Higher fault tolerance.

Minimum disks:

```text
4
```

---

# How RAID 6 Works

Very similar to RAID 5.

The difference:

Each stripe stores:

```text
Data

+

Parity A

+

Parity B
```

Two independent parity calculations exist.

---

# Why?

Suppose RAID 5 loses:

```text
Disk 2
```

Everything still works.

But if another disk fails before rebuilding finishes:

Entire array is lost.

RAID 6 solves this.

It can tolerate:

```text
Disk 2

↓

Dead

AND

Disk 4

↓

Dead
```

at the same time.

---

# Advantages

- Survives two simultaneous disk failures
- Excellent for very large storage arrays

---

# Disadvantages

More parity calculations.

Write performance is slower than RAID 5.

Less usable capacity.

---

# Typical Use Cases

- Enterprise NAS
- Archive Storage
- Large File Servers
- Distributed Storage Systems

---

# RAID 10 — Mirroring + Striping

Purpose:

Performance + Redundancy.

Minimum disks:

```text
4
```

---

# How RAID 10 Works

First:

Mirror disks.

```text
Disk 1

↓

Disk 2

Disk 3

↓

Disk 4
```

Then:

Stripe across the mirrored pairs.

---

# Analogy

Imagine four employees.

First:

Two employees work together.

Another two employees work together.

Each pair makes identical copies.

Then:

The work is split between both teams.

Now you have:

- Redundancy
- High Performance

---

# Advantages

- Excellent read performance
- Excellent write performance
- Very fast rebuilds
- High availability

---

# Disadvantages

Only 50% usable capacity.

Requires at least four disks.

---

# Typical Use Cases

- Database Servers
- Virtualization Hosts
- High Transaction Systems
- Enterprise Applications

---

# RAID Comparison

| RAID | Min Disks | Performance | Redundancy | Survives Failure |
|--------|----------:|------------|------------|------------------|
| RAID 0 | 2 | ⭐⭐⭐⭐⭐ | ❌ | None |
| RAID 1 | 2 | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 1 Disk |
| RAID 5 | 3 | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | 1 Disk |
| RAID 6 | 4 | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 2 Disks |
| RAID 10 | 4 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | Depends on Mirror Pair |

---

# Hardware RAID vs Software RAID

## Hardware RAID

Uses:

RAID Controller

Advantages:

- Dedicated processor
- Better performance
- Enterprise standard

---

## Software RAID

Managed by Linux.

Utility:

```bash
mdadm
```

Advantages:

- No special hardware
- Flexible
- Cost effective

---

# Common Mistakes

❌ Thinking RAID is backup.

❌ Choosing RAID 0 for critical servers.

❌ Forgetting rebuild time after disk replacement.

❌ Assuming more disks automatically mean better performance.

❌ Confusing RAID with LVM.

---

# Interview Questions

## What is RAID?

RAID combines multiple physical disks into one logical storage device to improve performance, redundancy, or both.

---

## Difference between RAID 5 and RAID 6?

RAID 5:

- One distributed parity
- Survives one disk failure

RAID 6:

- Two distributed parity calculations
- Survives two simultaneous disk failures

---

## Why does RAID 5 rotate parity?

To distribute write operations evenly across all disks and prevent one disk from becoming a bottleneck.

---

## Difference between RAID and LVM?

RAID provides:

- Redundancy
- Performance

LVM provides:

- Flexible storage management
- Dynamic resizing

They solve different problems and are often used together.

---

# Senior SysAdmin Notes

One of the first questions an experienced System Administrator asks before choosing a RAID level is:

> **"What happens if a disk fails?"**

The second question is:

> **"How much downtime can this server tolerate?"**

Performance is important.

Capacity is important.

But in production environments...

Availability is usually the highest priority.

Choosing the correct RAID level is a business decision as much as a technical one.

---

# Lesson Summary

```text
RAID 0

Striping

↓

Performance

--------------------------------

RAID 1

Mirroring

↓

Redundancy

--------------------------------

RAID 5

Striping + Distributed Parity

↓

Performance + 1 Disk Fault Tolerance

--------------------------------

RAID 6

Striping + Double Distributed Parity

↓

Performance + 2 Disk Fault Tolerance

--------------------------------

RAID 10

Mirroring + Striping

↓

High Performance + High Availability
```

RAID combines multiple disks into a single logical storage system to improve performance, availability, or fault tolerance depending on the chosen RAID level. Selecting the correct RAID configuration depends on the workload, acceptable downtime, storage requirements, and the organization's recovery objectives.
