# Linux Boot Process — System Administration Notes

## Overview

The Linux boot process is the sequence of events that occurs from the moment a machine is powered on until the operating system reaches a usable state.

A Linux System Administrator needs to understand the boot process because troubleshooting depends on knowing **which stage failed**.

A failure before the kernel starts is completely different from a failure after systemd starts.

The overall boot flow:

```text
Power Button
      |
      v
Firmware (BIOS / UEFI)
      |
      v
Bootloader (GRUB2)
      |
      v
Linux Kernel
      |
      v
systemd (PID 1)
      |
      v
Targets and Units
      |
      v
Login / Graphical Interface
```

---

# 1. Firmware (BIOS / UEFI)

## Purpose

Firmware is the first software executed after the machine receives power.

Its responsibility is to initialize hardware and locate a bootable device.

Common firmware:

- BIOS (Legacy)
- UEFI (Modern)

---

## Firmware Responsibilities

During startup, firmware performs:

- CPU initialization
- Memory (RAM) detection
- Storage controller initialization
- PCI/PCIe device detection
- Hardware checks (POST)
- Finding a bootable device

Flow:

```text
Power Button
      |
      v
BIOS / UEFI
      |
      v
Find Bootable Device
```

---

## Administrator Perspective

If the machine never reaches GRUB, Linux has not started yet.

Possible causes:

- Hardware failure
- Boot disk not detected
- Incorrect boot order
- Firmware configuration issues

At this stage:

- Kernel is not running.
- systemd does not exist.
- Linux services cannot be investigated.

---

# 2. Bootloader (GRUB2)

## Purpose

GRUB2 is the bootloader commonly used by Enterprise Linux distributions:

- RHEL
- CentOS Stream
- Rocky Linux
- AlmaLinux

Its purpose is to locate and load the Linux kernel.

The bootloader acts as the bridge between firmware and Linux.

```text
Firmware
      |
      v
GRUB2
      |
      v
Linux Kernel
```

---

## GRUB2 Responsibilities

GRUB2:

1. Starts after firmware.
2. Reads its configuration.
3. Locates the Linux kernel.
4. Locates initramfs.
5. Passes kernel parameters.
6. Loads required files into memory.
7. Transfers control to the kernel.

---

# 3. GRUB Configuration (grub.cfg)

## Purpose

`grub.cfg` is the configuration file that provides instructions to GRUB.

It is not the bootloader itself.

It tells GRUB:

- Which kernel to load.
- Where the kernel is located.
- Which initramfs file to use.
- Which kernel parameters to pass.

Example concept:

```text
Load kernel:

vmlinuz-5.x.x

Load initramfs:

initramfs-5.x.x.img

Kernel parameters:

root=UUID=xxxx
ro
quiet
```

---

## GRUB Menu Editing

At the GRUB menu, pressing:

```text
e
```

allows temporary editing of the boot entry.

This does not permanently modify:

```text
grub.cfg
```

The changes only apply to the current boot.

---

## Common Uses

Administrators use temporary GRUB edits for:

- Password recovery
- Rescue mode
- Testing kernel parameters
- Troubleshooting boot problems

Examples:

```text
rd.break
```

or:

```text
systemd.unit=rescue.target
```

---

## Example Recovery Scenario

Forgotten root password:

1. Reboot system.
2. Enter GRUB editor.
3. Modify boot parameters.
4. Boot into recovery environment.
5. Remount filesystem as read-write.
6. Repair the system.

---

# 4. Linux Kernel

## Purpose

After GRUB loads the kernel, control is transferred to the Linux kernel.

The kernel is responsible for managing the core operating system.

Responsibilities include:

- Hardware management
- Memory management
- Process management
- Device management
- Security enforcement

---

## Kernel During Boot

The kernel:

- Initializes hardware drivers.
- Initializes memory management.
- Detects devices.
- Starts the userspace environment.
- Starts the first userspace process.

---

# 5. initramfs

## Purpose

initramfs means:

```text
Initial RAM Filesystem
```

It is a temporary filesystem loaded into memory before the real root filesystem becomes available.

---

## Why initramfs Exists

Some systems require additional preparation before accessing the real root filesystem.

Examples:

- LVM activation
- RAID assembly
- Disk encryption unlocking
- Storage drivers

The kernel may need initramfs to prepare the environment first.

Flow:

```text
Kernel
   |
   v
initramfs
   |
   v
Real Root Filesystem
```

---

# 6. systemd (PID 1)

## Purpose

After kernel initialization completes, the kernel starts:

```text
systemd
```

Systemd becomes:

```text
PID 1
```

PID 1 is the first userspace process.

---

## Why PID 1 Matters

Systemd becomes the parent process of normal userspace processes.

Example:

```text
systemd (PID 1)
      |
      +-- sshd
      |
      +-- nginx
      |
      +-- NetworkManager
```

---

## Kernel vs systemd Responsibilities

The kernel manages:

```text
Hardware
CPU
Memory
Drivers
Devices
```

Systemd manages:

```text
Services
Targets
Mounts
Sockets
Timers
User Sessions
```

---

# 7. systemd Boot Process

After systemd starts, it begins building the operating system environment.

Systemd:

1. Determines the default target.
2. Resolves dependencies.
3. Activates required units.
4. Reaches the requested system state.

Example:

```text
default.target
        |
        v
graphical.target
        |
        v
multi-user.target
        |
        v
basic.target
        |
        v
sysinit.target
```

---

# 8. Targets During Boot

Targets represent system states or milestones.

They do not perform work themselves.

Instead, they organize which units should be activated.

---

## sysinit.target

The early system initialization stage.

Responsible for preparing the foundation of the operating system.

Examples:

- Early filesystem preparation
- Device initialization
- Basic system setup

---

## basic.target

Represents a system after basic initialization.

It depends on earlier initialization stages.

---

## multi-user.target

Represents a fully operational server environment.

Common in server systems.

Usually includes:

- Networking
- Login services
- Background services

---

## graphical.target

Represents a graphical environment.

Usually adds:

- Display manager
- Desktop environment

Example:

A CentOS workstation with GNOME commonly uses:

```text
graphical.target
```

as the default target.

---

# 9. Boot Troubleshooting Mental Model

When troubleshooting boot problems, do not ask:

> "Why doesn't Linux boot?"

Ask:

> "How far did the boot process reach?"

---

## Firmware Failure

Symptoms:

- No GRUB screen
- Hardware errors
- Boot device missing

Investigate:

- Hardware
- Firmware settings
- Storage detection

---

## GRUB Failure

Symptoms:

```text
grub>
```

or:

```text
grub rescue>
```

Possible causes:

- Missing or corrupted grub.cfg
- Cannot locate /boot
- Cannot find kernel
- Broken bootloader installation

At this stage:

- Firmware succeeded.
- Kernel has not started.
- systemd has not started.

---

## Kernel Failure

Symptoms:

- Kernel panic
- Driver failures
- Root filesystem mounting problems

Investigate:

- Kernel messages
- initramfs
- Storage drivers

---

## systemd Failure

Symptoms:

- Emergency mode
- Failed services
- Target not reached

Useful commands:

```bash
systemctl status
systemctl --failed
journalctl -b
```

---

# Key Administrator Takeaways

## 1. Boot is a chain of responsibility

```text
Firmware
    |
    v
GRUB2
    |
    v
Kernel
    |
    v
systemd
    |
    v
Units
```

Each component has a specific responsibility.

---

## 2. The kernel starts systemd

Systemd does not start the kernel.

The kernel starts systemd.

---

## 3. Targets are goals

Targets are not programs.

They represent desired system states.

Systemd reaches them by activating units.

---

## 4. Troubleshooting follows the boot stage

Always determine:

```text
Where did the system stop?
```

before making changes.

---

# Final Mental Model

A Linux Administrator should think:

```text
Hardware initialized?
        |
        v
Firmware successful?
        |
        v
GRUB loaded?
        |
        v
Kernel running?
        |
        v
systemd started?
        |
        v
Target reached?
        |
        v
Services healthy?
```

Understanding this sequence allows administrators to quickly identify the layer responsible for a boot failure and investigate the correct area.
