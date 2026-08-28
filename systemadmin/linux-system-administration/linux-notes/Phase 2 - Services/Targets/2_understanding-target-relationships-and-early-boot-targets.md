# Targets (Part 2) — Understanding Target Relationships and Early Boot Targets

---

# Objective

In this lesson, we investigated how **Target Units** define relationships between different system states rather than executing programs.

We also began tracing the Linux boot sequence from the default target down to the earliest initialization targets to understand **why the operating system reaches them in a specific order**.

---

# Lesson Recap

Previously, we learned:

- What a Target Unit is.
- The purpose of `default.target`.
- `.wants` directories.
- How `systemctl enable` creates symbolic links.
- Why legacy runlevels still exist through compatibility targets.

In this lesson, we explored **what a Target Unit actually contains** and how early boot targets fit together.

---

# Investigating Target Unit Files

Example:

```bash
systemctl cat graphical.target
```

Output:

```ini
[Unit]
Description=Graphical Interface
Documentation=man:systemd.special(7)
Requires=multi-user.target
Wants=display-manager.service
Conflicts=rescue.service rescue.target
After=multi-user.target rescue.service rescue.target display-manager.service
AllowIsolate=yes
```

Unlike Service Units, Target Units contain only the **[Unit]** section.

Target Units do **not** contain:

```ini
[Service]
ExecStart=
Restart=
Type=
```

This immediately tells us an important design principle.

---

# Target Units Do Not Execute Programs

A Target Unit does **not**:

- execute processes
- launch applications
- become a running daemon

Instead, a Target Unit describes:

- relationships
- dependencies
- ordering
- desired system state

A Target is simply a coordination point.

---

# Understanding Common Target Directives

## Description

```ini
Description=Graphical Interface
```

Human-readable description displayed by systemctl.

---

## Documentation

```ini
Documentation=man:systemd.special(7)
```

Provides the manual page explaining the purpose of the target.

Useful command:

```bash
man systemd.special
```

---

# Requires=

Example:

```ini
Requires=multi-user.target
```

Meaning:

> This target cannot exist without `multi-user.target`.

If `graphical.target` is started, systemd must also activate:

```text
multi-user.target
```

`Requires=` defines a **dependency**.

It answers:

> **"What must exist for me to reach my goal?"**

---

# Wants=

Example:

```ini
Wants=display-manager.service
```

Meaning:

> When this target becomes active, systemd should also activate the Display Manager.

Unlike `Requires=`, this expresses a desired accompanying unit.

Think of it as:

> "My purpose is better fulfilled if this unit is also activated."

The Display Manager provides the graphical login screen, making it a natural companion for `graphical.target`.

---

# After=

Example:

```ini
After=multi-user.target
```

This **does not** create a dependency.

Instead, it controls activation order.

It answers:

> "If both units are started, which one should start first?"

---

# Why Requires= and After= Are Used Together

These directives solve different problems.

Requires:

```ini
Requires=multi-user.target
```

Answers:

> "Who do I depend on?"

After:

```ini
After=multi-user.target
```

Answers:

> "In what order should we start?"

One creates the dependency.

The other establishes the startup sequence.

Both are commonly used together.

---

# Conflicts=

Example:

```ini
Conflicts=rescue.target
```

Meaning:

Two units cannot be active simultaneously.

Example:

- Graphical Interface
- Rescue Mode

These represent different operating states.

---

# AllowIsolate=

Example:

```ini
AllowIsolate=yes
```

Allows the target to become the active system target through:

```bash
systemctl isolate <target>
```

This will be explored later in the Targets lesson.

---

# Important Observation

Service Units and Target Units use the **same dependency language**.

Both support directives such as:

- Requires=
- Wants=
- After=
- Conflicts=

The difference is their purpose.

Service Units:

> Execute processes.

Target Units:

> Coordinate system state.

---

# Following the Boot Sequence

While tracing dependencies, we naturally discovered that boot follows a layered structure.

Conceptually:

```text
Kernel
    │
    ▼
local-fs-pre.target
    │
    ▼
local-fs.target
    │
    ▼
sysinit.target
    │
    ▼
basic.target
    │
    ▼
multi-user.target
    │
    ▼
graphical.target
```

Each target represents another stage in preparing the operating system.

---

# local-fs-pre.target

Purpose:

Preparation stage before mounting local filesystems.

This stage exists to ensure the system is ready for filesystem mounting.

Typical preparation tasks include:

- waiting for storage devices
- preparing encrypted volumes
- preparing block devices
- other storage initialization tasks

Think of it as:

> "Prepare everything needed before mounting local filesystems."

It does **not** perform the mounts.

---

# local-fs.target

Purpose:

Represents the point where all required local filesystems have been mounted successfully.

Examples include:

- /
- /boot
- /home
- /var

By the time this target is reached, these filesystems should already be available.

Important realization:

Opening the unit file reveals:

```bash
systemctl cat local-fs.target
```

contains no mount commands.

This demonstrates another systemd principle:

Targets coordinate.

They do not perform the work themselves.

---

# Relationship Between local-fs.target and /etc/fstab

During Phase 1, persistent mounts were configured using:

```text
/etc/fstab
```

Systemd reads `/etc/fstab` during boot.

It generates individual **Mount Units** for each filesystem.

Conceptually:

```text
/etc/fstab
        │
        ▼
Systemd reads entries
        │
        ▼
Generates Mount Units

home.mount
var.mount
boot.mount
...
        │
        ▼
Mount Units execute
        │
        ▼
local-fs.target reached
```

This demonstrates that:

- `/etc/fstab` describes what should be mounted.
- Mount Units perform the mounting.
- `local-fs.target` represents the successful completion of those mounts.

---

# sysinit.target

Purpose:

Represents completion of essential operating system initialization.

This stage prepares the operating system itself before higher-level services begin.

Examples include:

- essential system initialization
- swap activation
- device preparation
- low-level operating system readiness

Think of it as:

> "The operating system is now initialized and ready for higher-level functionality."

---

# basic.target

Purpose:

Represents completion of the minimum environment required before user-facing services begin.

At this stage:

- the operating system is initialized
- local filesystems are available
- essential initialization has completed

This creates the foundation for later targets such as:

```text
multi-user.target
```

and eventually

```text
graphical.target
```

---

# Major Design Principle Learned

One of the most important realizations from this lesson is:

Targets are **coordination points**, not workers.

Examples:

- `graphical.target` does not launch GNOME.
- `multi-user.target` does not launch application services.
- `local-fs.target` does not mount filesystems.

Instead:

- Services execute processes.
- Mount Units perform mounts.
- Targets coordinate the successful completion of related units.

This architecture is one of the core design philosophies of systemd.

---

# Key Takeaways

- Target Units contain only relationship metadata.
- They define system state rather than execute processes.
- `Requires=` creates dependencies.
- `After=` defines activation order.
- `Wants=` activates complementary units that help fulfill the target's purpose.
- Early boot proceeds through multiple initialization targets before reaching user-facing targets.
- `local-fs.target` represents successful filesystem mounting rather than performing the mounts itself.
- `/etc/fstab` is parsed by systemd to generate Mount Units.
- Understanding Targets reveals how storage concepts from Phase 1 integrate into the systemd boot process.
