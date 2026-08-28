# Linux 01 - Logical Volume Manager

A hands-on project demonstrating the fundamentals of **Logical Volume Manager (LVM)** in Linux. This project covers the core concepts of LVM and provides a step-by-step implementation using **CentOS Stream 9** running on **VMware Workstation Pro**.

The objective of this project is to understand how LVM abstracts physical storage and enables flexible storage management by separating physical disks from filesystems. Unlike traditional partitioning, LVM allows storage to be expanded with minimal downtime and without repartitioning existing disks.

---

## Project Objectives

- Understand the architecture of Logical Volume Manager (LVM)
- Learn the purpose of Physical Volumes (PV), Volume Groups (VG), and Logical Volumes (LV)
- Create and manage Logical Volumes
- Create XFS filesystems
- Configure persistent mounting using `/etc/fstab`
- Perform online Logical Volume expansion
- Expand a Volume Group by adding an additional physical disk

---

## Lab Environment

| Component | Specification |
|----------|---------------|
| Operating System | CentOS Stream 9 |
| Virtualization | VMware Workstation Pro |
| vCPU | 2 |
| Memory | 4 GB RAM |

### Storage Configuration

| Disk | Purpose |
|------|---------|
| 60 GB NVMe | Operating System |
| 30 GB SATA Disk | Initial LVM Storage |
| 30 GB SATA Disk | Volume Group Expansion |

---

## Project Workflow

This project is divided into the following phases:

1. Adding two virtual hard disks
2. Creating a Physical Volume (PV)
3. Creating a Volume Group (VG)
4. Creating Logical Volumes (LV)
5. Creating XFS filesystems
6. Mounting Logical Volumes and configuring `/etc/fstab`
7. Expanding Logical Volumes and XFS filesystems
8. Expanding the Volume Group using an additional physical disk

---

## Technologies Used

- Linux
- CentOS Stream 9
- Logical Volume Manager (LVM)
- XFS Filesystem
- VMware Workstation Pro
- Bash

---

## Learning Outcomes

After completing this project, you will understand how to:

- Initialize Physical Volumes
- Build Volume Groups
- Allocate storage through Logical Volumes
- Format Logical Volumes with XFS
- Mount filesystems permanently
- Expand Logical Volumes while preserving existing data
- Increase storage capacity by extending Volume Groups
- Apply enterprise storage management practices using LVM

---

## Documentation

The complete step-by-step guide, explanations, screenshots, and command outputs are available in the accompanying project documentation:

**📄 Linux 01 - LVM Basics.pdf**
