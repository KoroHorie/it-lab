# Targets (Part 1) — Understanding Default Targets and `.wants` Directories

## Learning Objective

Understand what a **systemd Target** is, how Linux determines the default boot target, and how services become associated with a target.

---

# What is a Target?

A **Target** is a special type of **systemd Unit** that represents a **desired operating state** of the Linux system.

Unlike a Service Unit, a Target **does not run a process**.

Instead, it groups together other units (services, paths, sockets, mounts, etc.) that should be active when the system reaches that state.

Think of a Target as a **checkpoint** or **goal** that systemd tries to reach during boot.

Example:

```text
multi-user.target

        │
        ├── sshd.service
        ├── chronyd.service
        ├── firewalld.service
        └── nginx.service
```

The target itself does nothing.

It simply tells systemd:

> "These units should also be active."

---

# Default Target

To determine which operating state Linux should reach after boot:

```bash
systemctl get-default
```

Example:

```text
graphical.target
```

This command **does not** show the currently running target.

Instead, it shows the **configured default boot target** that systemd (PID 1) attempts to reach after taking control from the Linux kernel.

---

# Why My System Boots to `graphical.target`

My CentOS Stream 9 installation includes the GNOME Desktop Environment.

Because of this, the installer configured:

```text
default.target
        ↓
graphical.target
```

A minimal CentOS installation would normally use:

```text
multi-user.target
```

instead.

---

# Investigating Target Unit Files

Target definitions are stored under:

```text
/usr/lib/systemd/system/
```

Example:

```bash
ls -l /usr/lib/systemd/system | grep graphical
```

Output:

```text
default.target -> graphical.target
graphical.target
graphical.target.wants/
runlevel5.target -> graphical.target
```

Important observations:

- `graphical.target` is the actual Target Unit file.
- `graphical.target.wants/` is **not another target**.
- It is a directory that stores dependency relationships.
- `default.target` is simply a symbolic link pointing to the configured default boot target.
- `runlevel5.target` is provided for compatibility with the traditional SysV Runlevel 5.

---

# Investigating `multi-user.target`

```bash
ls -ld /usr/lib/systemd/system/multi-user.target*
```

Output:

```text
multi-user.target
multi-user.target.wants/
```

This confirms the same structure:

- One Target Unit file.
- One `.wants` directory.

---

# Understanding `.wants` Directories

Every Target may have its own:

```text
<target>.wants/
```

directory.

Example:

```text
multi-user.target.wants/
```

This directory contains **symbolic links** to other unit files.

Example:

```text
multi-user.target.wants/

├── systemd-logind.service
├── systemd-user-sessions.service
├── getty.target
└── ...
```

These symbolic links tell systemd:

> "When this target becomes active, these units are also wanted."

The directory itself does **not** execute anything.

It simply stores the dependency relationships that systemd reads.

---

# What `systemctl enable` Actually Does

When enabling a service:

```bash
systemctl enable nginx.service
```

systemd performs the following steps:

```text
Read nginx.service
        │
        ▼
Locate:

[Install]
WantedBy=multi-user.target

        │
        ▼
Create symbolic link

multi-user.target.wants/nginx.service

        │
        ▼
Point to the real unit file
```

This does **not** start the service.

Instead, it creates the relationship that allows systemd to include the service whenever `multi-user.target` is activated.

---

# Key Takeaways

- Targets represent a desired operating state.
- Targets do **not** run processes.
- `systemctl get-default` displays the configured default boot target.
- Target Unit files are stored under:

```text
/usr/lib/systemd/system/
```

- Every Target may have a corresponding:

```text
<target>.wants/
```

directory.

- `.wants` directories store symbolic links to units that systemd should activate when reaching that target.
- `systemctl enable` works by creating symbolic links inside the appropriate `.wants` directory based on the service's `[Install]` section.
