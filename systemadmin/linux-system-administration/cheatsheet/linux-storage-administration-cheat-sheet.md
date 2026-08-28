# Linux Storage Administration Cheat Sheet

This repository serves as my personal Linux System Administration command reference while learning Linux Storage Management.  
The purpose of this document is to quickly recall commands, syntax, examples, and common administration scenarios without having to search the internet.

## Topics Included
* Disk Discovery
* Partitioning
* Formatting Filesystems
* Mounting Filesystems
* LVM (Physical Volume, Volume Group, Logical Volume)
* Extending LVM
* Swap Management
* Common Administration Scenarios

---

## 1. Disk Discovery
Before performing any storage operation, always verify that Linux detects the storage device.

### lsblk
**Description** Displays all block storage devices connected to the system.

**Syntax**
```bash
lsblk
```

**Example**
```bash
lsblk
```

**Example Output**
```text
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   40G  0 disk
├─sda1   8:1    0    1G  0 part /boot
├─sda2   8:2    0    8G  0 part [SWAP]
└─sda3   8:3    0   31G  0 part /
sdb      8:16   0   20G  0 disk
```

**When to Use**
* Verify Linux detected a newly attached disk
* View partitions
* View mount points

### lsblk -f
**Description** Displays block devices together with their filesystem type and UUID.

**Syntax**
```bash
lsblk -f
```

**Example**
```bash
lsblk -f
```

**When to Use**
* Verify filesystem type
* View UUIDs
* Verify mounted filesystems

### fdisk -l
**Description** Lists all detected disks and partition tables.

**Syntax**
```bash
fdisk -l
```

**Example**
```bash
fdisk -l
```

**When to Use**
* Verify newly attached storage
* View partition sizes
* Inspect partition table

### blkid
**Description** Displays filesystem UUIDs and filesystem types.

**Syntax**
```bash
blkid
```

**Example**
```bash
blkid
```

**When to Use**
* Configure `/etc/fstab`
* Verify filesystem UUIDs

---

## 2. Partitioning
Linux cannot use a raw disk for storing files. A partition must first be created.

### fdisk
**Description** Creates, deletes, and manages partitions.

**Syntax**
```bash
fdisk /dev/sdX
```

**Example**
```bash
fdisk /dev/sdb
```

### Common fdisk Commands
| Command | Description |
| :--- | :--- |
| `n` | Create New Partition |
| `p` | Print Partition Table |
| `d` | Delete Partition |
| `t` | Change Partition Type |
| `w` | Save Changes |
| `q` | Quit Without Saving |

### Typical Workflow
`n`  
↓  
Choose Primary Partition  
↓  
Choose Partition Number  
↓  
Choose First Sector  
↓  
Choose Last Sector  
↓  
`w`  

---

## 3. Notify Linux of Partition Changes

### partprobe
**Description** Informs the Linux Kernel that the partition table has changed without rebooting.

**Syntax**
```bash
partprobe
```
or
```bash
partprobe /dev/sdb
```

**Example**
```bash
partprobe /dev/sdb
```

**When should you use partprobe?** Suppose you created a new partition.

Before:
```bash
lsblk
```
Output shows:
```text
sdb
```

After creating a partition:
```text
sdb
└── sdb1
```

Sometimes Linux still doesn't detect the new partition. Instead of rebooting:
```bash
partprobe
```
Linux reloads the partition table immediately.

### Real World Scenario
You increased a Virtual Machine disk from:  
40 GB  
↓  
80 GB  

Linux still sees:  
40 GB  

You modify the partition using fdisk.  
After saving:
```bash
partprobe
```
Linux immediately recognizes the updated partition table without rebooting.

---

## 4. Formatting Filesystems
After creating a partition, Linux still cannot store files. A filesystem must be created.

### EXT4
**Description** Creates an EXT4 filesystem.

**Syntax**
```bash
mkfs.ext4 /dev/sdX1
```

**Example**
```bash
mkfs.ext4 /dev/sdb1
```

**When to Use**
* General Linux Servers
* Virtual Machines
* Default filesystem for many distributions

### XFS
**Description** Creates an XFS filesystem.

**Syntax**
```bash
mkfs.xfs /dev/sdX1
```

**Example**
```bash
mkfs.xfs /dev/sdb1
```

**When to Use**
* Large storage servers
* Enterprise workloads
* Red Hat based distributions

---

## 5. Mounting Filesystems
Formatting does NOT automatically make the storage usable. The filesystem must be mounted.

### mount
**Description** Mounts a filesystem to a directory.

**Syntax**
```bash
mount DEVICE DIRECTORY
```

**Example**
```bash
mount /dev/sdb1 /mnt/storage
```

**Verify Mounted Filesystems**
```bash
mount
```
or
```bash
findmnt
```

**Unmount Filesystem** Syntax:
```bash
umount DIRECTORY
```
Example:
```bash
umount /mnt/storage
```

### Persistent Mounts
Temporary mounts disappear after reboot. To mount automatically every boot:
```bash
nano /etc/fstab
```

**Example:**
```text
UUID=2af3-67bc    /database    ext4    defaults    0 0
```
**Format:**
```
UUID/Filepath   MountPoint   FileSystem   MountOption   DumpFlag   FlagSystemCheckOrder
```

**Always Apply changes**
When we configure `/etc/fstab` we need to tell systemd to re-initialize the mount configuration on it. Use the following command:

```bash
sudo mount -a
systemctl daemon-reload
```
The `sudo mount -a` mounts all filesystem listed in the file, as for `systemctl daemon-reload` This allows systemd to register the updated configuration.

**Field 1**

What is UUID?

UUID means:

Universally Unique Identifier

Every filesystem created by Linux receives its own unique identifier.

Example:

blkid

Output:
```
/dev/sdb1:
UUID="2af3-67bc"
TYPE="ext4"
```
Linux uses the UUID because:

Imagine this.

Today:
```
sdb1
```
Tomorrow after adding another disk:
```
sdc1
```
The device name changed.

But...

The UUID never changes.

Using UUID is therefore much safer than writing:
```
/dev/sdb1
```

**Field 2**

This is called the:

Mount Point

Meaning:

"Where should Linux attach this filesystem?"

Suppose:
```
UUID=2af3-67bc
```
is your 100GB disk.

After mounting:
```
/database
```
Everything inside that disk becomes accessible through:

cd /database

**Field 3**

Filesystem Type.

Linux needs to know:

"How should I read this storage?"

Examples:

- ext4
- xfs
- btrfs
- vfat
- ntfs
- swap

For example:

UUID=xxxx

none

swap

sw

0

0

would activate Swap during boot.

**Field 4**

Mount Options.

Think of this as:

"How should Linux mount this filesystem?"

The keyword:

defaults

actually represents several commonly used options.

Equivalent to:
```
rw,suid,dev,exec,auto,nouser,async
```
Meaning:

Option	Meaning
rw	Read & Write
suid	Allow SUID programs
dev	Allow device files
exec	Allow executing programs
auto	Automatically mount during boot
nouser	Only root can mount
async	Normal asynchronous writing

Most of the time:

defaults

is all you need.

**Field 5**

Dump Flag.

Years ago Linux had a backup utility called:

dump

It used this field.

Nowadays?

Almost nobody uses it.

So you'll almost always see:

0

Meaning:

Don't use dump backup.

**Field 6**

Filesystem Check Order.

This field tells Linux:

"When should fsck check this filesystem during boot?"

Values:

0

↓

Never check
1

↓

Check FIRST

Normally only:

/

gets:

1

because it is the Root Filesystem.

Other disks:

2

Meaning:

Check after Root.
```
Linux will automatically mount the filesystem during boot.

---

## Common Administration Scenario

### Scenario: The Disk Became Larger

**Example:** Originally:  
Disk: 40 GB  

Virtualization Platform:  
Increase Disk  
↓  
80 GB  

Linux still sees:  
40 GB  

**What should a Linux Administrator do?**

* **Step 1: Verify disk**
  ```bash
  lsblk
  ```
* **Step 2: Modify the partition**
  ```bash
  fdisk /dev/sda
  ```
* **Step 3: Save changes** Enter `w`
* **Step 4: Reload partition table**
  ```bash
  partprobe
  ```
* **Step 5: Verify**
  ```bash
  lsblk
  ```
  The partition should now reflect the updated size.

### Important Note
Do NOT delete and recreate partitions on a production server unless you fully understand the implications. If the goal is to use newly added disk capacity while preserving existing data, the typical workflow is to extend the existing partition (or create a new one if appropriate), reload the partition table with `partprobe` if needed, and then continue with the appropriate filesystem or LVM resize operations. The exact method depends on whether the partition is a regular filesystem or part of an LVM setup.

## 6. Logical Volume Manager (LVM)

### Overview
Logical Volume Manager (LVM) is a storage management technology in Linux that provides flexibility when managing storage devices. Unlike traditional partitions, LVM allows storage to be expanded without recreating partitions or reinstalling the operating system.

LVM consists of three layers:  
Physical Volume (PV)  
↓  
Volume Group (VG)  
↓  
Logical Volume (LV)  
↓  
Filesystem  

### LVM Workflow
A Logical Volume cannot exist without first creating a Physical Volume and Volume Group. The recommended order is:  
Physical Disk  
↓  
Partition  
↓  
Physical Volume  
↓  
Volume Group  
↓  
Logical Volume  
↓  
Filesystem  
↓  
Mount  

---

### 6.1 Physical Volume (PV)
A Physical Volume is the first layer of LVM. It tells Linux: *"This partition will now become part of the LVM system."*

**Create Physical Volume** Syntax:
```bash
pvcreate DEVICE
```
Example:
```bash
pvcreate /dev/sdb1
```

**Verify Physical Volumes** Syntax:
```bash
pvs
```
Example:
```bash
pvs
```
Example Output:
```text
PV         VG      Fmt Attr PSize   PFree
/dev/sdb1          lvm2 --- 20.00g 20.00g
```

**Display Detailed Physical Volume Information** Syntax:
```bash
pvdisplay
```
Example:
```bash
pvdisplay
```
Displays:
* Physical Volume UUID
* Size
* Free Space
* Volume Group Membership

**When to Use**
* Verify Physical Volumes
* Troubleshoot LVM
* Check available storage

---

### 6.2 Volume Group (VG)
A Volume Group combines one or more Physical Volumes into a storage pool. Think of a Volume Group as a storage container. Instead of allocating storage directly from the disk, Linux allocates storage from the Volume Group.

**Create Volume Group** Syntax:
```bash
vgcreate VG_NAME DEVICE
```
Example:
```bash
vgcreate vg_data /dev/sdb1
```

**Verify Volume Groups** Syntax:
```bash
vgs
```
Example:
```bash
vgs
```
Example Output:
```text
VG       #PV #LV VSize   VFree
vg_data    1   0 20.00g 20.00g
```

**Display Detailed Volume Group Information** Syntax:
```bash
vgdisplay
```
Example:
```bash
vgdisplay
```

**Extend Volume Group** Suppose another disk has been added.  
* Disk 1: 20 GB  
* Disk 2: 20 GB  

Add the second Physical Volume into the Volume Group.  
Syntax:
```bash
vgextend VG_NAME DEVICE
```
Example:
```bash
vgextend vg_data /dev/sdc1
```
Now: 20 GB → 40 GB Storage Pool

---

### 6.3 Logical Volume (LV)
A Logical Volume is what Linux actually formats and mounts. Applications never directly use the Physical Volume or Volume Group. They use the Logical Volume.

**Create Logical Volume** Syntax:
```bash
lvcreate -L SIZE -n LV_NAME VG_NAME
```
Example:
```bash
lvcreate -L 10G -n lv_database vg_data
```
Meaning: Create a **10 GB** Logical Volume Named **lv_database** Inside **vg_data**.

**Verify Logical Volumes** Syntax:
```bash
lvs
```
Example:
```bash
lvs
```
Example Output:
```text
LV            VG        Attr LSize
lv_database   vg_data   -wi- 10.00g
```

**Display Detailed Logical Volume Information** Syntax:
```bash
lvdisplay
```
Example:
```bash
lvdisplay
```

---

### 6.4 Formatting the Logical Volume
A newly created Logical Volume still has no filesystem.

**EXT4**
```bash
mkfs.ext4 /dev/vg_data/lv_database
```

**XFS**
```bash
mkfs.xfs /dev/vg_data/lv_database
```

---

### 6.5 Mounting the Logical Volume
Create a directory:
```bash
mkdir /database
```

Mount it:
```bash
mount /dev/vg_data/lv_database /database
```

Verify:
```bash
findmnt
```
or
```bash
mount
```

---

## Common LVM Administration Scenario

### Scenario: The Database Server Needs More Space
Originally: Logical Volume is 10 GB.  
A few months later... the database becomes larger.  
Manager says: *"Anthony, increase the database storage."* Since the Volume Group still has free space (Volume Group: 20 GB, Free: 10 GB), no repartitioning is required. Simply extend the Logical Volume.

**Extend Logical Volume** Syntax:
```bash
lvextend -L +SIZE DEVICE
```
Example:
```bash
lvextend -L +5G /dev/vg_data/lv_database
```
Result: 10 GB → 15 GB

**Use All Remaining Free Space** Syntax:
```bash
lvextend -l +100%FREE DEVICE
```
Example:
```bash
lvextend -l +100%FREE /dev/vg_data/lv_database
```
Linux allocates every remaining free space inside the Volume Group.

### IMPORTANT
Extending the Logical Volume does not automatically extend the filesystem. Think of it this way: you built a larger room, but the furniture inside hasn't moved yet. The filesystem still thinks it has the old size. You must resize the filesystem.

**EXT4**
```bash
resize2fs /dev/vg_data/lv_database
```

**XFS**
```bash
xfs_growfs /database
```

> **Note:** EXT4 resizes using the **device path**, while XFS grows using the **mount point**. This is a common interview question.

---

### Typical LVM Expansion Workflow
Increase Disk Size  
↓  
`lsblk`  
↓  
Extend Partition (if required)  
↓  
`partprobe`  
↓  
`pvresize`  
↓  
`lvextend`  
↓  
`resize2fs` (EXT4) or `xfs_growfs` (XFS)  
↓  
Done  

### New Command: pvresize
Suppose your virtual disk increased from 20 GB → 40 GB. Your partition has already been extended and Linux sees a larger partition. However, the Physical Volume still believes it is only 20 GB. You need to tell LVM: *"Hey, the partition became larger."*

Syntax:
```bash
pvresize DEVICE
```
Example:
```bash
pvresize /dev/sdb1
```
Now the Volume Group can use the additional storage.

---

## Summary

| Layer | Purpose |
| :--- | :--- |
| **Physical Volume** | Converts a partition into LVM storage |
| **Volume Group** | Creates a storage pool |
| **Logical Volume** | Creates usable virtual partitions |
| **Filesystem** | Stores files |
| **Mount** | Makes the filesystem accessible |
