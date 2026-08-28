# Linux System Administration

# Phase 2 -- Services

# Lesson 3 -- Service Units (.service)

> **Lesson Objective**
>
> Understand what a Service Unit is, why it exists, how `systemd` uses
> it, and how the monitored process behaves during service execution.

------------------------------------------------------------------------

# Introduction

In Lesson 2 we learned that **systemd** is the first userspace process
started by the Linux kernel (PID 1). It manages services, sockets,
mounts, timers and many other unit types.

This raises an important question:

> **How does systemd know HOW to start a service?**

The answer is through a **Service Unit**.

A Service Unit is not the application itself.

Instead, it is a configuration file that tells systemd exactly how a
service should be managed.

------------------------------------------------------------------------

# Why This Topic Exists

Imagine a server containing hundreds of services.

Examples:

-   SSH
-   Apache
-   MariaDB
-   Docker

Questions naturally appear.

-   Which executable should systemd run?
-   Should the service restart if it crashes?
-   How should it stop?
-   Should it start during boot?

Applications do not magically answer these questions.

Instead, systemd reads a **Service Unit (.service)** which contains the
instructions needed to manage that application.

Think of a Service Unit as an instruction manual written for systemd.

------------------------------------------------------------------------

# Analogy

Restaurant Example

Application = Chef

systemd = Restaurant Manager

.service file = Recipe Card

The chef knows how to cook.

The manager knows how to supervise.

The recipe card tells the manager exactly how today's work should begin.

Without the recipe card, the manager has no instructions.

------------------------------------------------------------------------

# Core Concepts

## What is a Service?

A Service is an application intended to provide functionality in the
background.

Examples:

-   sshd
-   httpd
-   nginx
-   mariadb

Unlike graphical applications, services usually continue running without
user interaction.

Their purpose is to continuously provide functionality.

------------------------------------------------------------------------

## What is a Service Unit?

A Service Unit is a configuration file ending with:

``` text
.service
```

Its purpose is to describe **how systemd should manage a service**.

It contains instructions such as:

-   what command to execute
-   how to stop it
-   restart behaviour
-   startup behaviour

A Service Unit does **not** contain the application.

It only describes how to manage it.

------------------------------------------------------------------------

# Anatomy of a Service Unit

``` ini
[Unit]
Description=Inventory Application

[Service]
ExecStart=/opt/inventory/start.sh

[Install]
WantedBy=multi-user.target
```

## \[Unit\]

### Why does this section exist?

When Linux boots, systemd may need to manage hundreds of units.

Not every service can start randomly.

Some services depend on networking.

Some require storage.

Some should start before others.

The **\[Unit\]** section exists to describe the unit itself and its
relationships with other units.

Think of it as the identity card of the service.

Today's lesson only introduced the Description directive.

Future lessons will cover dependency directives such as After=, Before=,
Requires= and Wants=.

Notice that **\[Unit\] does not tell systemd how to execute the
application.**

Its responsibility is describing the unit.

------------------------------------------------------------------------

## \[Service\]

### Why does this section exist?

Once systemd understands what the unit is, it still needs instructions
describing **how the service should run**.

That is the responsibility of the **\[Service\]** section.

This section contains execution-related directives.

Examples include:

-   ExecStart
-   ExecStop
-   Restart

Today's lesson focuses only on **ExecStart**.

Everything inside this section answers one question:

> How should systemd manage this service while it is running?

------------------------------------------------------------------------

## \[Install\]

### Why does this section exist?

Managing a service is different from deciding **when** that service
should become part of the system.

The \[Install\] section describes how the unit integrates with the boot
process.

The directive:

``` ini
WantedBy=multi-user.target
```

is simply an instruction telling systemd where this unit belongs when
enabled.

The details of targets will be covered in a later lesson.

For now remember:

-   \[Unit\] describes the unit.
-   \[Service\] describes execution.
-   \[Install\] describes installation into the boot process.

------------------------------------------------------------------------

# ExecStart

``` ini
ExecStart=/opt/inventory/start.sh
```

This is the command executed by systemd.

Internally:

systemctl start

↓

systemd

↓

reads .service

↓

finds ExecStart

↓

executes command

systemd does not understand your application.

It simply executes the command specified by ExecStart.

------------------------------------------------------------------------

# Foreground Process Behavior

Suppose `ExecStart` executes the following script:

``` bash
#!/bin/bash

java -jar inventory.jar
```

Many beginners assume that Bash starts Java and immediately exits.

That is **not** what happens.

When Bash starts a foreground program, it **waits** until that program
finishes.

Execution flow:

``` text
systemd
   │
   ▼
start.sh (bash)
   │
   ▼
java -jar inventory.jar
```

While Java is running:

-   Bash is still alive.
-   Bash is waiting.
-   systemd still sees the Bash process running.

If Java exits normally or crashes:

``` text
Java exits
     │
     ▼
Bash wakes up
     │
Reaches end of script
     │
Bash exits
     │
systemd detects the monitored process has exited
```

This explains why systemd reacts only after the process started by
`ExecStart` ends.

------------------------------------------------------------------------

# Deleting the Executable

Consider the running service:

``` ini
ExecStart=/opt/inventory/start.sh
```

Now someone deletes the script:

``` bash
rm /opt/inventory/start.sh
```

Does the service immediately stop?

**No.**

## Why?

The script was already executed.

The Bash process already exists in memory.

Deleting the file only removes it from disk.

It does **not** terminate the running process.

Think of it like reading a book.

If someone removes the book from the shelf while you're already holding
it, you can still finish reading the copy in your hands.

The failure only appears when systemd tries to execute `ExecStart`
again.

Example:

``` bash
systemctl restart inventory.service
```

Linux attempts to execute:

``` text
/opt/inventory/start.sh
```

Since the file no longer exists, startup fails.

------------------------------------------------------------------------

# Commands

## ps -f

### Why does this command exist?

Linux may have hundreds of running processes.

Administrators need a quick way to inspect them and understand their
relationships.

`ps -f` displays processes in **full format**.

### Columns

**UID** --- User account that owns the process.

**PID** --- Unique Process ID.

**PPID** --- Parent Process ID. Identifies which process created the
current process.

**TTY** --- Controlling terminal. `?` usually indicates a background
service with no terminal.

**TIME** --- Total CPU time consumed by the process.

**CMD** --- Command used to start the process.

### Production Usage

Useful when tracing parent-child relationships and confirming which
process owns another.

## echo \$\$

Displays the Process ID of the current Bash shell.

Useful when identifying your active shell during troubleshooting.

## ps -fp \$\$

Displays detailed information about the current Bash process, including
its PPID.

## systemctl status

Provides an overview of:

-   Loaded units
-   Active units
-   Failed units

Usually one of the first commands executed during service
troubleshooting.

------------------------------------------------------------------------

# Production Scenario

**Ticket #1034**

Users report that the Inventory application is unavailable.

Professional investigation:

1.  Run `systemctl status inventory.service`.
2.  Determine whether the monitored process exited.
3.  Verify the `ExecStart` path still exists.
4.  Review logs before restarting.
5.  Restart only after identifying the cause.

------------------------------------------------------------------------

# Common Mistakes

-   Confusing a Service Unit with the application itself.
-   Assuming deleting an executable immediately stops a running process.
-   Assuming systemd directly monitors every child process.
-   Restarting services before investigating the root cause.

------------------------------------------------------------------------

# Lesson Summary

By the end of Lesson 3 you should understand:

-   What a Service is.
-   What a Service Unit is.
-   The purpose of `[Unit]`, `[Service]`, and `[Install]`.
-   Why `ExecStart` exists.
-   Why Bash waits for foreground programs.
-   Why deleting an executable does not immediately stop a running
    service.
-   Why systemd detects service completion after the monitored process
    exits.
