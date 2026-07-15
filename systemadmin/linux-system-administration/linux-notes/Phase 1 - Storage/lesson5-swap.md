# Phase 1 — Storage
# Lesson 5 — Swap Space

---

# What is Swap?

**Swap** is a dedicated storage area on a Linux system that acts as an **extension of RAM** when physical memory becomes scarce.

It is important to understand one thing immediately:

> **Swap is NOT RAM.**

Swap resides on a storage device (SSD or HDD), making it **much slower** than physical memory.

Its purpose is not to replace RAM, but to provide Linux with additional flexibility in memory management.

---

# Why Does Swap Exist?

Imagine your server has:

```text
RAM

8 GB
```

Everything runs normally.

Suddenly:

- Apache receives thousands of requests.
- MariaDB consumes more memory.
- Docker containers start.
- Another application is launched.

RAM usage begins to increase.

Eventually:

```text
RAM

100%
```

Now Linux has a problem.

It cannot simply say:

> "Sorry, memory is full."

Instead, Linux tries to **free RAM** by moving less important or inactive memory pages to Swap.

This allows active applications to continue running.

---

# Analogy — Classroom

Imagine a classroom with only **10 chairs**.

```text
Classroom

10 Chairs
```

Ten students are already seated.

Suddenly...

Another student arrives.

There are no more chairs.

Does the teacher kick someone out?

No.

The teacher notices that one student has been sleeping for the past hour.

The teacher politely asks:

> "Can you continue reading in the library for now?"

The sleeping student leaves temporarily.

A chair becomes available.

The new student sits down.

Later...

When the sleeping student is needed again, they return from the library.

---

In this analogy:

```text
Classroom

↓

RAM

Library

↓

Swap

Students

↓

Processes
```

Linux does something very similar.

Instead of terminating inactive processes, it temporarily moves their memory pages to Swap.

---

# Swap is NOT Extra RAM

Many beginners say:

> "I have 8 GB RAM and 8 GB Swap, so I have 16 GB RAM."

This is incorrect.

```text
8 GB RAM

+

8 GB Swap

≠

16 GB RAM
```

RAM and Swap have completely different performance characteristics.

Example:

| Memory Type | Speed |
|-------------|-------|
| RAM | Extremely Fast |
| SSD Swap | Much Slower |
| HDD Swap | Significantly Slower |

Linux always prefers using RAM.

Swap is only used when necessary.

---

# How Linux Uses Swap

Linux constantly monitors memory usage.

When memory pressure increases, Linux decides:

Which memory pages have not been used recently?

Those inactive pages are moved to Swap.

This process is called:

**Swapping**

---

# Analogy — Warehouse

Imagine a warehouse.

Near the entrance:

```text
Frequently Used Items
```

Farther inside:

```text
Old Inventory
```

Workers keep frequently used products close because they're accessed constantly.

Less frequently used inventory is moved deeper into storage.

Linux does exactly this.

Active memory stays inside RAM.

Inactive memory may be moved into Swap.

---

# Does RAM Need to Be 100% Full Before Swap Is Used?

Not always.

This surprised us during the lesson.

Many people assume Linux only uses Swap when RAM reaches:

```text
100%
```

In reality...

Linux may begin swapping **before RAM is completely full.**

Why?

Because Linux tries to optimize memory usage.

If inactive pages have not been accessed for a long time, Linux may proactively move them to Swap to keep more RAM available for applications that actually need fast access.

---

# The Linux Kernel Thinks Ahead

Linux doesn't simply react.

It predicts.

Suppose:

```text
RAM

90%
```

An application requiring 2 GB suddenly starts.

Instead of waiting until RAM completely fills up, Linux may already have moved inactive memory into Swap.

This makes room for the incoming workload.

---

# Why Doesn't Linux Just Kill Applications?

Imagine running:

- Database
- Apache
- Docker
- Monitoring
- SSH Session

Memory becomes tight.

Should Linux terminate MariaDB?

Probably not.

Should Linux terminate Apache?

Probably not.

Instead:

Linux attempts to preserve running applications by moving inactive memory pages to Swap.

Only in severe situations, when memory is exhausted and nothing else can be reclaimed, does Linux invoke the:

**OOM Killer (Out Of Memory Killer)**

which terminates processes to protect system stability.

---

# Swap Partition vs Swap File

Linux supports two methods.

---

## Swap Partition

Dedicated partition.

Example:

```text
/dev/sda3

↓

Swap
```

Advantages:

- Slightly faster
- Traditional
- Common in enterprise environments

---

## Swap File

A regular file stored inside an existing filesystem.

Example:

```text
/swapfile
```

Advantages:

- Easier to resize
- Easier to create
- Very common in cloud environments

Modern Linux distributions frequently use swap files by default.

---

# Viewing Swap

Display memory usage:

```bash
free -h
```

Example:

```text
              total   used   free

Mem:            8G     6G     2G

Swap:           8G     0G     8G
```

---

Display active swap devices:

```bash
swapon --show
```

or

```bash
swapon -s
```

---

# Creating a Swap Partition

Suppose:

```text
/dev/sda3
```

was created as a partition.

Initialize it:

```bash
mkswap /dev/sda3
```

Enable it:

```bash
swapon /dev/sda3
```

Verify:

```bash
swapon --show
```

---

# Making Swap Persistent

Like filesystems...

Swap disappears after reboot unless configured.

Edit:

```text
/etc/fstab
```

Example:

```text
UUID=xxxxxxxx    none    swap    sw    0    0
```

---

# Understanding the fstab Entry

```text
UUID

↓

Swap Device

none

↓

No Mount Point Needed

swap

↓

Filesystem Type

sw

↓

Swap Mount Option

0 0

↓

Dump / fsck Options
```

Notice:

Swap is **not mounted into a directory.**

Unlike filesystems:

```text
/database
```

Swap simply becomes available to the Linux kernel.

---

# Disabling Swap

Temporarily disable:

```bash
swapoff /dev/sda3
```

Disable every configured swap:

```bash
swapoff -a
```

---

# Why Servers Still Use Swap

Many beginners ask:

> "If RAM is faster, why not simply install more RAM?"

The answer:

You absolutely should.

However:

Swap provides:

- Additional protection
- Temporary memory overflow
- Better memory management
- Increased system stability

Swap is not there because RAM is insufficient.

Swap is there because Linux is designed to manage memory intelligently.

---

# Typical Swap Sizes

There is no universal rule.

Examples:

| RAM | Typical Swap |
|------|-------------:|
| 2 GB | 2–4 GB |
| 4 GB | 2–4 GB |
| 8 GB | 2–8 GB |
| 16 GB | 4–8 GB |
| 32 GB+ | Depends on workload |

Enterprise servers often size Swap based on application requirements rather than fixed formulas.

---

# Common Mistakes

❌ Thinking Swap is extra RAM.

❌ Assuming Linux only swaps when RAM reaches 100%.

❌ Creating huge Swap expecting better performance.

❌ Forgetting to configure Swap inside `/etc/fstab`.

❌ Blaming Swap for slow performance instead of investigating memory pressure.

---

# Production Scenario

A web server normally consumes:

```text
RAM

6 GB

Available

2 GB
```

Suddenly...

Traffic spikes.

Apache creates additional worker processes.

MariaDB consumes more cache.

Linux notices several inactive background processes.

Instead of terminating them...

Linux swaps those inactive pages to disk.

RAM becomes available for active requests.

Users never notice.

---

# Interview Questions

## What is Swap?

Swap is reserved storage used as an extension of RAM when Linux experiences memory pressure.

---

## Is Swap RAM?

No.

Swap resides on storage devices and is significantly slower than RAM.

---

## Why does Linux use Swap before RAM reaches 100%?

Because Linux proactively manages memory.

Inactive pages may be moved to Swap to keep RAM available for applications that require fast access.

---

## Difference between Swap Partition and Swap File?

Swap Partition:

Dedicated storage partition.

Swap File:

Regular file inside an existing filesystem.

Both provide the same functionality.

---

# Senior SysAdmin Notes

One of the biggest lessons we learned is:

**Swap is not created to improve performance.**

It exists to improve **system stability.**

A server that completely runs out of memory becomes unpredictable.

Swap gives Linux another tool for managing memory intelligently before the situation becomes critical.

Good System Administrators do not monitor only RAM usage.

They monitor:

- RAM
- Swap
- Memory pressure
- Application behavior

because high Swap usage is often a symptom of another problem that deserves investigation.

---

# Lesson Summary

```text
Applications
        │
        ▼
RAM (Fast)
        │
Memory Pressure
        │
        ▼
Inactive Pages
        │
        ▼
Swap (Slow Storage)
```

Swap is a reserved area on storage that Linux uses to temporarily hold inactive memory pages when physical RAM becomes scarce. It does not replace RAM, but instead provides the Linux kernel with additional flexibility to maintain system stability and keep active applications running efficiently during periods of high memory usage.
