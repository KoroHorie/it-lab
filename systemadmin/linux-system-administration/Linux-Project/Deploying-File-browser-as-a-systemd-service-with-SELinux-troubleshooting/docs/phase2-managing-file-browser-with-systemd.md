# Phase 2 — Managing File Browser with systemd

---

# Objective

The objective of this phase is to integrate File Browser into the Linux operating system by creating a custom **systemd Service Unit**.

Instead of manually executing the application every time the server boots, systemd will become responsible for:

- Starting the application.
- Stopping the application.
- Restarting the application.
- Monitoring the application.
- Automatically starting the application during system boot.

This transforms File Browser from a standalone executable into a production-managed Linux service.

---

# Why use systemd?

During Phase 1, File Browser was started manually using:

```bash
./filebrowser
```

Although this successfully launched the application, several problems exist with this approach:

- The application terminates when the shell exits.
- The application must be started manually after every reboot.
- There is no centralized service management.
- The operating system does not monitor whether the application crashes.

A production Linux server should never rely on manually starting applications.

Instead, Linux uses **systemd** to manage long-running services.

---

# Creating the Service Unit

A new Service Unit was created:

```text
/etc/systemd/system/filebrowser.service
```

This directory is reserved for administrator-created Service Units.

Unlike built-in services installed by packages, manually created services are stored under:

```text
/etc/systemd/system
```

allowing systemd to manage third-party applications.

---

# File Browser Service Unit

```ini
[Unit]
Description=File Browser Service
After=network.target

[Service]
Type=simple
ExecStart=/opt/filebrowser/filebrowser
WorkingDirectory=/opt/filebrowser
Restart=always

[Install]
WantedBy=multi-user.target
```

---

# Understanding Each Section

## `[Unit]`

The `[Unit]` section describes the service itself.

```ini
Description=File Browser Service
```

Provides a human-readable description shown by commands such as:

```bash
systemctl status
```

---

```ini
After=network.target
```

Instructs systemd to start File Browser **after** the network target has been reached.

This does **not** start networking.

Instead, it defines the startup order.

---

## `[Service]`

The `[Service]` section defines how systemd should execute and manage the application.

---

### Type=simple

```ini
Type=simple
```

File Browser runs as a normal foreground process.

systemd immediately considers the service started after launching the executable.

---

### ExecStart

```ini
ExecStart=/opt/filebrowser/filebrowser
```

This specifies the exact executable systemd should launch.

Unlike interactive shells, systemd requires an **absolute path**.

Using:

```text
./filebrowser
```

would not work because systemd does not execute services relative to the current directory.

---

### WorkingDirectory

```ini
WorkingDirectory=/opt/filebrowser
```

Sets the current working directory before launching File Browser.

This ensures the application creates and accesses files relative to:

```text
/opt/filebrowser
```

instead of another location.

---

### Restart Policy

```ini
Restart=always
```

If File Browser unexpectedly exits, systemd automatically attempts to restart it.

This improves service availability in production environments.

---

## `[Install]`

The `[Install]` section controls service enablement.

```ini
WantedBy=multi-user.target
```

When enabled, systemd creates an association between:

```text
filebrowser.service
```

and

```text
multi-user.target
```

allowing File Browser to start automatically whenever the system reaches the normal multi-user operating state.

---

# Verifying the Service Unit

Before loading the service into systemd, the unit file was validated.

```bash
systemd-analyze verify filebrowser.service
```

No output indicates that the Service Unit syntax is valid.

Validating Service Units before loading them helps prevent configuration errors.

---

# Reloading systemd

After creating a new Service Unit, systemd must reload its configuration.

```bash
systemctl daemon-reload
```

This instructs systemd to detect newly created or modified Service Units.

Without reloading, systemd continues using its previously loaded configuration.

---

# Starting the Service

The service was started using:

```bash
systemctl start filebrowser.service
```

At this point, systemd reads the Service Unit and attempts to execute:

```text
ExecStart
```

---

# Understanding What Happens Internally

When the service starts, the following sequence occurs:

```
systemctl start
        ↓
systemd reads filebrowser.service
        ↓
systemd executes ExecStart
        ↓
Linux Kernel creates a new process
        ↓
Kernel assigns a Process ID (PID)
        ↓
File Browser begins running
```

systemd itself does **not** create Process IDs.

The Linux Kernel remains responsible for process creation.

systemd simply requests that the kernel execute the program defined by:

```text
ExecStart
```

---

# Service Status

The service can be inspected using:

```bash
systemctl status filebrowser.service
```

Example:

```text
Loaded: loaded
Active: active (running)
Main PID: 5062
```

---

## Loaded

```text
Loaded: loaded
```

Indicates that systemd successfully located and parsed the Service Unit.

---

## Active

```text
Active: active (running)
```

Confirms that File Browser is currently running.

---

## Main PID

```text
Main PID
```

Represents the primary process currently managed by systemd.

systemd continuously monitors this PID throughout the lifetime of the service.

---

# Enable vs Start

A common misconception is assuming that starting a service also enables it.

These are separate operations.

### Start

```bash
systemctl start filebrowser.service
```

Starts the service immediately.

---

### Enable

```bash
systemctl enable filebrowser.service
```

Configures the service to automatically start during system boot by associating it with:

```text
multi-user.target
```

Starting a service does **not** automatically enable it.

Enabling a service does **not** immediately start it.

---

# What We Learned

After completing Phase 2, we learned:

- Why manually executed applications should become systemd services.
- How custom Service Units are created.
- The purpose of:
  - `[Unit]`
  - `[Service]`
  - `[Install]`
- Why `ExecStart` requires an absolute path.
- Why `WorkingDirectory` exists.
- Why Service Units should be validated before loading.
- Why `daemon-reload` is required.
- The difference between:
  - Start
  - Enable
- How systemd launches applications.
- How the Linux Kernel creates processes and assigns Process IDs.
- How systemd supervises services using the Main PID.

Although the Service Unit was correctly configured, File Browser still failed to start.

The investigation and root cause analysis are covered in **Phase 3**.
