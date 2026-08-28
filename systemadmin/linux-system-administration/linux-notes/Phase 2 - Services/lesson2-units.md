# Phase 2 — Services

# Lesson 2 — Understanding systemd Units

---

# Introduction

In the previous lesson, we learned that **systemd** is the init system, service manager, and system manager of modern Linux systems. We also learned that after the Linux kernel finishes booting, it starts **systemd (PID 1)**, which becomes responsible for managing the userspace environment.

At first, it is easy to think that systemd only manages **Services** such as Apache, MariaDB, SSH, or Docker.

However, that understanding is incomplete.

The correct way to think about systemd is:

> **systemd manages Units.**

Services are only **one type** of Unit.

Understanding Units is one of the most important milestones in learning modern Linux administration because almost everything that systemd controls is represented internally as a Unit.

This lesson introduces the concept of Units, explains why they exist, how Unit Files describe them, and how systemd uses those Unit Files to manage Linux components.

---

# Why This Topic Exists

Imagine if Linux had completely different management tools for every subsystem.

For example:

- One utility to manage services
- One utility to manage swap
- One utility to manage mounted filesystems
- One utility to manage timers
- One utility to manage sockets
- Another utility to manage boot states

Every subsystem would have different commands, different configuration files, and different management philosophies.

Enterprise Linux would become difficult to learn and even harder to administer.

Instead, systemd introduces a common abstraction called a **Unit**.

Rather than treating services, timers, mounts, swap devices, sockets, and many other resources as completely different things, systemd represents all of them as Units.

This provides administrators with one consistent architecture for managing different parts of the operating system.

As a Linux System Administrator, understanding Units is important because whenever you interact with systemd, you are almost always interacting with Units—even if you do not realize it.

---

# Analogies

## Office Manager Analogy

Imagine a company.

The company has many different departments.

- Human Resources
- Accounting
- Security
- IT Department
- Customer Service

Although each department performs different tasks, the Office Manager oversees all of them.

The Office Manager does not perform accounting.

The Office Manager does not answer customer calls.

The Office Manager simply manages and coordinates everyone.

systemd behaves the same way.

Apache, MariaDB, Docker, SSH, timers, mounts, and sockets all perform different jobs.

systemd simply manages them.

---

## Library Analogy

Imagine a library.

A library contains many different resources.

- Books
- Magazines
- Newspapers
- Journals

Although every resource is different, the librarian manages them using one catalog system.

Likewise,

systemd manages many different Linux resources through one common management model called **Units**.

Instead of having different management systems for every resource, they are all represented consistently.

---

## Blueprint Analogy

Imagine constructing a house.

There are two important things.

The blueprint.

The house itself.

The blueprint tells the builders:

- where the walls go
- where the windows go
- where the doors go

However, the blueprint is **not** the house.

Likewise,

A Unit File tells systemd:

- what to start
- how to stop it
- when to start it
- what it depends on

The Unit File is **not** the application.

It is only the blueprint.

---

# Core Concepts

## What is a Unit?

A Unit is an object that systemd knows how to manage.

Think of a Unit as a manageable resource inside Linux.

Examples include:

- Web servers
- Database servers
- Mounted filesystems
- Swap
- Timers
- Sockets
- Devices

Instead of creating different management frameworks for each one, systemd treats them all as Units.

This provides consistency across the operating system.

---

## Unit Types

systemd supports many different Unit types.

The most common include:

| Unit Type | Description |
|------------|-------------|
| `.service` | Background services and daemons |
| `.mount` | Mounted filesystems |
| `.swap` | Swap partitions or swap files |
| `.socket` | Network or IPC sockets |
| `.timer` | Scheduled execution |
| `.device` | Hardware devices |
| `.path` | Filesystem path monitoring |
| `.target` | System operating states |
| `.slice` | Resource grouping |
| `.scope` | Externally created processes |

Among these, Service Units are the ones Linux administrators interact with most frequently.

---

## Understanding Path Units

One Unit type that often confuses beginners is the **Path Unit**.

A Path Unit does **not** manage directories.

It does **not** modify files.

It does **not** create files.

Its job is very simple.

It **monitors a filesystem path**.

When a configured event occurs, systemd can automatically start another Unit.

For example,

Suppose a company has an upload directory.

```text
/srv/uploads
```

Whenever a new file appears, the company wants to scan it for malware.

The workflow becomes:

```text
Watch:

/srv/uploads

        │
        ▼

New File Appears

        │
        ▼

Trigger

virus-scan.service
```

The Path Unit simply watches.

Another Unit performs the actual work.

---

## What is a Unit File?

Now that we know what a Unit is, another question naturally appears.

> How does systemd know how to manage that Unit?

The answer is:

**Unit Files**

A Unit File is a configuration file.

Its purpose is to describe how systemd should manage a particular Unit.

This is one of the most important distinctions to understand.

A Unit is the object.

A Unit File describes that object.

---

## Unit vs Unit File

These two terms are often confused.

A Unit is something managed by systemd.

A Unit File is a configuration file.

Relationship:

```text
Apache

        │

        ▼

httpd.service

(Unit File)

        │

        ▼

systemd reads it

        │

        ▼

Knows how to manage Apache
```

Notice that Apache is not the Unit File.

Apache is the application.

The Unit File simply tells systemd how to manage it.

---

## Unit Files are Configuration Files

A common beginner misconception is:

> The `.service` file is the application.

This is incorrect.

The application is the executable.

For Apache:

Executable

```text
/usr/sbin/httpd
```

Configuration

```text
httpd.service
```

The executable performs work.

The Unit File contains instructions.

---

## Why Doesn't systemd Hardcode Every Application?

Suppose systemd were written like this:

```text
If Apache

Run /usr/sbin/httpd

If SSH

Run /usr/sbin/sshd

If MariaDB

Run /usr/sbin/mysqld
```

Now imagine your company develops its own application.

```text
inventoryd
```

How would systemd know about it?

It wouldn't.

Instead,

systemd simply says:

> Give me a Unit File.

Inside the Unit File,

tell me:

- what executable to run
- how to stop it
- how to restart it
- what it depends on

Because of this design, systemd can manage virtually any application without needing to know anything about it beforehand.

---

## Anatomy of a Service Unit

A Service Unit typically contains three major sections.

```ini
[Unit]

[Service]

[Install]
```

Each section has a specific responsibility.

---

### [Unit]

This section contains information **about** the Unit.

Examples include:

- Description
- Documentation
- Startup ordering
- Dependencies

It answers questions like:

- What is this Unit?
- What does it depend on?

---

### [Service]

This section contains instructions describing how systemd should manage the process.

Typical directives include:

- ExecStart
- ExecStop
- ExecReload
- Restart

This section answers:

> How should systemd manage this process?

---

### [Install]

This section determines how the Unit integrates into system startup.

It answers:

> When should this Unit start automatically?

---

## Understanding ExecStart

One of the most important directives inside a Service Unit is:

```ini
ExecStart=
```

Example:

```ini
ExecStart=/usr/sbin/httpd
```

This tells systemd:

> Execute this program when starting the service.

Notice something important.

systemd does not know how Apache works.

systemd only knows:

"I should execute the program located here."

---

## Relationship Between systemd, Unit Files and Executables

```text
Administrator

        │

systemctl start httpd

        │

        ▼

systemd

        │

Reads

httpd.service

        │

        ▼

ExecStart=/usr/sbin/httpd

        │

        ▼

Kernel executes

/usr/sbin/httpd

        │

        ▼

Apache starts
```

This demonstrates an important separation of responsibilities.

- Unit File → configuration
- systemd → manager
- Kernel → process creation
- Executable → performs the work

---

# Diagrams

## systemd Management Model

```text
systemd

    │

    ├── Services

    ├── Mounts

    ├── Swap

    ├── Timers

    ├── Sockets

    ├── Devices

    ├── Paths

    ├── Targets

    ├── Slices

    └── Scopes
```

---

## Unit Relationship

```text
Application

        │

        ▼

Unit

        │

        ▼

Unit File

        │

        ▼

systemd

        │

        ▼

Kernel

        │

        ▼

Running Process
```

---

# Commands

## systemctl start

### Purpose

Requests systemd to start a Unit.

### Syntax

```bash
systemctl start UNIT_NAME
```

### Example

```bash
systemctl start httpd
```

### What happens internally?

systemd:

1. Reads the Unit File.
2. Looks for `ExecStart=`.
3. Executes the specified program.
4. The kernel creates the process.

### Production Usage

Used when:

- Starting newly installed services
- Restarting applications after maintenance
- Recovering failed services

---

# Production Scenarios

## Scenario 1 — Custom Enterprise Application

A software development team creates:

```text
/opt/company/bin/inventoryd
```

The administrator creates:

```text
inventory.service
```

systemd now knows how to manage the application without requiring any changes to systemd itself.

---

## Scenario 2 — Missing Executable

A Unit File contains:

```ini
ExecStart=/usr/sbin/httpd
```

However,

```text
/usr/sbin/httpd
```

has been accidentally deleted.

systemd reads the Unit File.

Attempts to execute the program.

The kernel reports that the executable does not exist.

The service enters a failed state.

An experienced administrator investigates the Unit File before blaming Apache itself.

---

## Scenario 3 — Malware Scanning

A company stores uploaded files inside:

```text
/srv/uploads
```

A Path Unit watches the directory.

Whenever a new file appears,

systemd automatically starts:

```text
virus-scan.service
```

No administrator intervention is required.

---

# Common Mistakes

## Mistake 1

Believing systemd only manages Services.

Correct understanding:

systemd manages Units.

---

## Mistake 2

Thinking a `.service` file is the application.

Correct understanding:

It is only configuration.

---

## Mistake 3

Thinking ExecStart contains source code.

Correct understanding:

ExecStart simply points to the executable.

---

## Mistake 4

Believing Path Units modify files.

Correct understanding:

Path Units only monitor filesystem paths and trigger other Units.

---

## Mistake 5

Assuming systemd knows how to start every application automatically.

Correct understanding:

systemd depends entirely on Unit Files.

---

# Interview Questions

## What is a Unit?

A Unit is an object managed by systemd.

---

## What is a Unit File?

A Unit File is a configuration file describing how systemd should manage a Unit.

---

## What is the purpose of ExecStart?

ExecStart specifies the executable that systemd should launch.

---

## What is the difference between a Unit and a Unit File?

A Unit is the object.

A Unit File describes how systemd should manage it.

---

## What is a Path Unit?

A Path Unit monitors filesystem paths and triggers another Unit when configured filesystem events occur.

---

# Senior System Administrator Notes

One of the biggest differences between junior and senior Linux administrators is understanding the separation between **configuration** and **execution**.

Junior administrators often think:

> "The `.service` file runs Apache."

It does not.

The `.service` file contains configuration.

systemd reads the configuration.

The kernel creates the process.

Apache performs the actual work.

When troubleshooting failed services, experienced administrators avoid immediately restarting the service repeatedly.

Instead, they verify:

- Does the Unit File exist?
- Is `ExecStart=` correct?
- Does the executable exist?
- Are the dependencies satisfied?

Following this workflow leads to faster and more accurate troubleshooting.

Always remember:

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

Verify

↓

Prevent Future Recurrence
```

Never make changes before understanding the problem.

---

# Lesson Summary

## Summary

- systemd manages Units rather than only Services.
- A Unit represents an object managed by systemd.
- A Unit File is a configuration file describing how systemd manages a Unit.
- The executable performs the work.
- systemd reads the Unit File.
- The kernel creates the process.
- ExecStart tells systemd which executable should be launched.
- Understanding Units provides the foundation for learning Services, Targets, Timers, and the rest of the systemd ecosystem.

---

## Final Diagram

```text
                 Linux Kernel

                      │

                      ▼

                systemd (PID 1)

                      │

          Manages Different Units

                      │

      ┌───────────────┼───────────────┐

      │               │               │

      ▼               ▼               ▼

 Services        Mounts         Timers

      │               │               │

      └───────────────┼───────────────┘

                      ▼

                 Unit Files

                      │

          systemd reads configuration

                      │

                      ▼

         Executes the specified program

                      │

                      ▼

           Running Linux Components
```

Understanding Units is the next major step after learning what systemd is. Once you understand that everything systemd manages is represented as a Unit and that every Unit is described by a Unit File, the remaining topics—Services, Targets, Timers, Boot Process, and Service Troubleshooting—become much easier to understand because they all build upon this single, consistent architecture.
