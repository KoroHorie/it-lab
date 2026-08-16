## Phase 1: Check Disk on CentOS 9

1. lsblk

```text
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0   60G  0 disk
├─sda1        8:1    0    1G  0 part /boot
└─sda2        8:2    0   59G  0 part
  ├─cs-root 253:0    0 35.6G  0 lvm  /
  ├─cs-swap 253:1    0    6G  0 lvm  [SWAP]
  └─cs-home 253:2    0 17.4G  0 lvm  /home
sdb           8:16   0  500G  0 disk
sdc           8:32   0  500G  0 disk
sdd           8:48   0  500G  0 disk
sde           8:64   0  500G  0 disk
```
We have four disks that are seated in, no filesystem configuration yet and no partitions. and their size is 500G. The first thing to do is combine them into one software based Redundancy (RAID). We'll use mdadm for this. The mdadm is the userspace administration tool for Linux's MD (multiple-device) subsystem. The actual RAID functionality is handled by the Linux kernel; mdadm tells the kernel what array to create, assemble, monitor, etc.

2. Install `mdadm`

```bash
dnf install mdadm
```

`Package mdadm-4.4-4.el9.x86_64 is already installed.` This is what we wanted.

## Phase 2: RAID Configuration

3. Create the RAID 6 array

before we configure RAID 6, it is important to know that RAID configuration is best suited for the business needs. There are several RAID levels that we can use and each of them have use cases.

| RAID  | Minimum disks | Usable capacity with 4x500 GB | Fault tolerance | Main characteristic |
|-------|---------------|-------------------------------|-----------------|---------------------|
| RAID 0 | 2 | ~2 TB | None | Maximum capacity/performance |
| RAID 1 | 2 | ~500 GB* | Can lose disk depending on implementation | Mirroring |
| RAID 5 | 3 | ~1.5 TB | 1 disk | Striping + single parity |
| RAID 6 | 4 | ~1 TB | 2 disks | Striping + dual parity |
| RAID 10 | 2 | ~1 TB | Typically 1 disk per mirror pair | Mirroring + striping |

With four disks, we can absolutely use any of the RAID level but for this scenario we can utilize RAID 6, the fact we have four disk we can create 2 dual parity to each disks. For this lab we don't have any goals or reasons whether why we should use RAID 6 over RAID 5 or RAID 1. What we only do here is to match a bit close to the common setup of SAN Servers

To create RAID 6 array simply use this command:

```bash
sudo mdadm --create /dev/md0 
  --level=6 \
  --raid-devices=4 \
  /dev/sdb /dev/sdc /dev/sdd /dev/sde
```

Once we enter the command we'll see this prompt asking `To optimize recovery speed, it is recommended to enable write-intent bitmap, do you want to enable it now? [y/N]?`.

`mdadm` is asking us whether we want the new RAID array setup to maintain a write-intent bitmap so that future RAID synchronization/recovery can be faster. It is also useful whenever RAID gets interrupted while data is being written. Without bitmap, the RAID may need to determine which portions need synchronization or recovery. With bitmap, RAID has a much better idea of where it needs to work, because after an unexpected interruption, disk failure/replacement, or certain recovery operations, the bitmap can reduce the amount of synchronization work necessary and this is especially useful for a large arrays.

For this lab we'll select `y`.

After answering the prompt, mdadm creates the RAID and shows the following:

```text
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
```

mdadm sets the default version to 1.2 metadata. This metadata contains information that Linux needs to identify and assemble the RAID array, such as the RAID level and member information. And therefore Linux has created the software RAID block device called *md0* where, all physical disks: **sdb, sdc, sdd, and sde** have merged to it.

Now we observe the following:

```bash
cat /proc/mdstat
```

```text
Personalities : [raid4] [raid5] [raid6]
md0 : active raid6 sde[3] sdd[2] sdc[1] sdb[0]
      1048311808 blocks super 1.2 level 6, 512k chunk, algorithm 2 [4/4] [UUUU]
      [=>...................]  resync =  8.9% (46787840/524155904) finish=38.5min speed=206463K/sec
      bitmap: 4/4 pages [16KB], 65536KB chunk

unused devices: <none>
```
Here we see the active raid devices. This shows us the kernel's current RAID state, particularly useful for seeing whether the array is still initializing or rebuilding. The `resync =  8.9% (46787840/524155904) finish=38.5min speed=206463K/sec` is what's indicating that mdadm is initializing synchronization.

**NOTE:** Please make sure to finish the initialization before proceeding to the next step.

```bash
sudo mdadm --detail /dev/md0
```

```text
/dev/md0:
           Version : 1.2
     Creation Time : Sun Aug 16 17:39:51 2026
        Raid Level : raid6
        Array Size : 1048311808 (999.75 GiB 1073.47 GB)
     Used Dev Size : 524155904 (499.87 GiB 536.74 GB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

     Intent Bitmap : Internal

       Update Time : Sun Aug 16 17:44:42 2026
             State : clean, resyncing
    Active Devices : 4
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : bitmap

     Resync Status : 11% complete

              Name : localhost.localdomain:0  (local to host localhost.localdomain)
              UUID : 6d8e8e42:ee7d7762:f7994ea6:19b150c2
            Events : 57

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       3       8       64        3      active sync   /dev/sde
```
This now shows the details about md0. This tells us things such as:
- RAID level
- array size
- number of devices
- state
- active devices
- working devices
- failed devices
- spare devices
- metadata version
- bitmap information

And finally: `lsblk`

```text
NAME        MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
sda           8:0    0    60G  0 disk
├─sda1        8:1    0     1G  0 part  /boot
└─sda2        8:2    0    59G  0 part
  ├─cs-root 253:0    0  35.6G  0 lvm   /
  ├─cs-swap 253:1    0     6G  0 lvm   [SWAP]
  └─cs-home 253:2    0  17.4G  0 lvm   /home
sdb           8:16   0   500G  0 disk
└─md0         9:0    0 999.7G  0 raid6
sdc           8:32   0   500G  0 disk
└─md0         9:0    0 999.7G  0 raid6
sdd           8:48   0   500G  0 disk
└─md0         9:0    0 999.7G  0 raid6
sde           8:64   0   500G  0 disk
└─md0         9:0    0 999.7G  0 raid6
sr0          11:0    1  14.5G  0 rom
```

We see now the 4 physical disk, associated with md0.

## Phase 3: Verify the RAID, then prepare it for iSCSI

4. We'll need /dev/md0 as our backend block device for an iSCSI LUN.

The storage stack we need to follow:

```text
/dev/sdb
/dev/sdc
/dev/sdd
/dev/sde
      │
      ▼
   RAID 6
      │
      ▼
   /dev/md0
      │
      ▼
   iSCSI target
      │
      ▼
    LUN
      │
      ▼
 iSCSI initiator
```

The iSCSI server is providing raw block storage. The initiator sees that storage as if another disk were attached to it.

In this Phase, we'll install the iSCSI target software. Modern RHEL/CentOS systems uses LIO (Linux I/O Target) underneath, managed through `targetcli`.

To install:
```bash
sudo dnf install targetcli
```
Then we'll inspect the current target configuration before creating anything:

```bash
sudo targetcli ls
```
And here is the hierarchy of Linux SCSI target subsystem LIO through `targetcli`.

```text
o- / ............................................................................................................. [...]
  o- backstores .................................................................................................. [...]
  | o- block ...................................................................................... [Storage Objects: 0]
  | o- fileio ..................................................................................... [Storage Objects: 0]
  | o- pscsi ...................................................................................... [Storage Objects: 0]
  | o- ramdisk .................................................................................... [Storage Objects: 0]
  o- iscsi ................................................................................................ [Targets: 0]
  o- loopback ............................................................................................. [Targets: 0]
```

The **backstore** is like the actual storage resource that the iSCSI target is going to expose. If /dev/md0 exist, the LIO doesn't know yet that we want it to expose through iSCSI. So for this case we'll create a block backstore that points to `/dev/md0`.

**Concept Diagram**

```text
/dev/sdb ─┐
/dev/sdc ─┤
/dev/sdd ─┼── RAID 6
/dev/sde ─┘
             │
             ▼
          /dev/md0
             │
             ▼
       LIO block backstore
             │
             ▼
        iSCSI target
```
The /dev/md0 is a raw block device which is not formatted. We're going to give the iSCSI subsystem the raw block device.

5. The `targetcli` interactive shell

Targetcli provides an interactive shell that we can use to configure by adding block backstore, pointing to `/dev/md0`

```bash
sudo targetcli
```

Enters the interactive shell of Targetcli.

Then configure the following:

```bash
/backstore/block create raid6_storage /dev/md0
```
The output would be like this:

```bash
targetcli shell version 2.1.57
Copyright 2011-2013 by Datera, Inc and others.
For help on commands, type 'help'.

/> ls
o- / ............................................................................................................. [...]
  o- backstores .................................................................................................. [...]
  | o- block ...................................................................................... [Storage Objects: 0]
  | o- fileio ..................................................................................... [Storage Objects: 0]
  | o- pscsi ...................................................................................... [Storage Objects: 0]
  | o- ramdisk .................................................................................... [Storage Objects: 0]
  o- iscsi ................................................................................................ [Targets: 0]
  o- loopback ............................................................................................. [Targets: 0]
/>
/> /backstores/block create raid6_storage /dev/md0
Created block storage object raid6_storage using /dev/md0.
/>
/> ls
o- / ............................................................................................................. [...]
  o- backstores .................................................................................................. [...]
  | o- block ...................................................................................... [Storage Objects: 1]
  | | o- raid6_storage .................................................... [/dev/md0 (999.7GiB) write-thru deactivated]
  | |   o- alua ....................................................................................... [ALUA Groups: 1]
  | |     o- default_tg_pt_gp ........................................................... [ALUA state: Active/optimized]
  | o- fileio ..................................................................................... [Storage Objects: 0]
  | o- pscsi ...................................................................................... [Storage Objects: 0]
  | o- ramdisk .................................................................................... [Storage Objects: 0]
  o- iscsi ................................................................................................ [Targets: 0]
  o- loopback ............................................................................................. [Targets: 0]
/>
```
So now the RAID array we created is now successfully registered as an LIO block backstore. Given that raid6_storage identifies as that `/dev/md0` with its storage size of (999.7GiB)

6. Create the iSCSI Target.

We're going to give this storage an iSCSI identity. It is identified by an IQN (iSCSI Qualified Name). To create an IQN the command is:
```bash
/iscsi create iqn.2026-08.local.lab:storage01
```

Then let's verify using `ls`.

```bash
/> /iscsi create iqn.2026-08.local.lab:storage01
Created target iqn.2026-08.local.lab:storage01.
Created TPG 1.
Global pref auto_add_default_portal=true
Created default portal listening on all IPs (0.0.0.0), port 3260.
/>
/> ls
o- / ............................................................................................................. [...]
  o- backstores .................................................................................................. [...]
  | o- block ...................................................................................... [Storage Objects: 1]
  | | o- raid6_storage .................................................... [/dev/md0 (999.7GiB) write-thru deactivated]
  | |   o- alua ....................................................................................... [ALUA Groups: 1]
  | |     o- default_tg_pt_gp ........................................................... [ALUA state: Active/optimized]
  | o- fileio ..................................................................................... [Storage Objects: 0]
  | o- pscsi ...................................................................................... [Storage Objects: 0]
  | o- ramdisk .................................................................................... [Storage Objects: 0]
  o- iscsi ................................................................................................ [Targets: 1]
  | o- iqn.2026-08.local.lab:storage01 ....................................................................... [TPGs: 1]
  |   o- tpg1 ................................................................................... [no-gen-acls, no-auth]
  |     o- acls .............................................................................................. [ACLs: 0]
  |     o- luns .............................................................................................. [LUNs: 0]
  |     o- portals ........................................................................................ [Portals: 1]
  |       o- 0.0.0.0:3260 ......................................................................................... [OK]
  o- loopback ............................................................................................. [Targets: 0]
/>
```
Three things to worth understanding:
  1. TPG - A Target Portal Group groups the network portals and LUN/access configuration associated with this target.
  2. 0.0.0.0:3260 - A Network where LIO is listening for iSCSI connections on TCP port 3260 on all IPv4 interfaces.
  3. Target still has no storage exposed. The part where no LUNs are identified.

     We had /dev/md0 as raid6_storage and we even have an IQN: iqn.2026-08.local.lab:storage01. But these are not yet connected so we will create one.

7. Create the LUN

A LUN is the logical block-storage unit that an iSCSI initiator will see. We're going to map our existing backstore which is `raid6_storage` to LUN 0. Inside `targetcli`, run:

```bash
/iscsi/iqn.2026-08.local.lab:storage01/tpg1/luns create /backstores/block/raid6_storage
```

Then let's verify:

```bash
/> /iscsi/iqn.2026-08.local.lab:storage01/tpg1/luns create /backstores/block/raid6_storage
Created LUN 0.
/> ls
o- / ............................................................................................................. [...]
  o- backstores .................................................................................................. [...]
  | o- block ...................................................................................... [Storage Objects: 1]
  | | o- raid6_storage ...................................................... [/dev/md0 (999.7GiB) write-thru activated]
  | |   o- alua ....................................................................................... [ALUA Groups: 1]
  | |     o- default_tg_pt_gp ........................................................... [ALUA state: Active/optimized]
  | o- fileio ..................................................................................... [Storage Objects: 0]
  | o- pscsi ...................................................................................... [Storage Objects: 0]
  | o- ramdisk .................................................................................... [Storage Objects: 0]
  o- iscsi ................................................................................................ [Targets: 1]
  | o- iqn.2026-08.local.lab:storage01 ....................................................................... [TPGs: 1]
  |   o- tpg1 ................................................................................... [no-gen-acls, no-auth]
  |     o- acls .............................................................................................. [ACLs: 0]
  |     o- luns .............................................................................................. [LUNs: 1]
  |     | o- lun0 .................................................. [block/raid6_storage (/dev/md0) (default_tg_pt_gp)]
  |     o- portals ........................................................................................ [Portals: 1]
  |       o- 0.0.0.0:3260 ......................................................................................... [OK]
  o- loopback ............................................................................................. [Targets: 0]
/>
```
The LUN is now mapped to the RAID-backed storage, but we have security issues to consider. This indicator: `tpg1 [no-gen-acls, no-auth]` and `acls [ACLs: 0]`. This means our target currently has no initiator ACLs configured and no authentication. But for this case, let's test connection. We'll have to skip the security configuration for now and simply let's use another machine and install targetcli.

## Phase 4: Identify your iSCSI client

8. Install iscsi-initiator-utils

Since the first VM serves as Storage server we installed targetcli to configure the LIO pointing to our RAID block device `/dev/md0`. Now we setup another VM, this will serve as a client initiator. So the package we'll be installing is `iscsi-initiator-utils`.

```bash
sudo dnf install iscsi-initiator-utils
```

This gives the client the tools needed to:
- discover iSCSI targets
- log into targets
- maintain iSCSI sessions
- expose the remote LUN as a local block device

The mental model we needed:
- Target = provides storage -> `targetcli`
- Initiator = consumes storage -> `iscsi-initiator-utils`

9. Check the iscsi utils package

Most Linux distro especially CentOS 9 Stream has a pre-installed `iscsi-initiator-utils`. We can identify this by:

Running
```bash
rpm -ql iscsi-initiator-utils | grep -E 'initiatorname|iscsi'
```
which outputs the following filepaths relating to iscsi

```text
/etc/iscsi
/etc/iscsi/iscsid.conf
/run/lock/iscsi
/run/lock/iscsi/lock
/usr/lib/NetworkManager/dispatcher.d/04-iscsi
/usr/lib/systemd/system/iscsi-init.service
/usr/lib/systemd/system/iscsi-onboot.service
/usr/lib/systemd/system/iscsi-shutdown.service
/usr/lib/systemd/system/iscsi-starter.service
/usr/lib/systemd/system/iscsi.service
/usr/lib/systemd/system/iscsid.service
/usr/lib/systemd/system/iscsid.socket
/usr/lib/tmpfiles.d/iscsi.conf
/usr/lib64/libiscsi.so.0
/usr/lib64/libopeniscsiusr.so.0
/usr/lib64/libopeniscsiusr.so.0.2.0
/usr/libexec/iscsi-mark-root-nodes
/usr/sbin/iscsi-gen-initiatorname
/usr/sbin/iscsi-iname
/usr/sbin/iscsiadm
/usr/sbin/iscsid
/usr/sbin/iscsistart
/usr/share/doc/iscsi-initiator-utils
/usr/share/doc/iscsi-initiator-utils/README
/usr/share/man/man8/iscsi-gen-initiatorname.8.gz
/usr/share/man/man8/iscsi-iname.8.gz
/usr/share/man/man8/iscsiadm.8.gz
/usr/share/man/man8/iscsid.8.gz
/usr/share/man/man8/iscsistart.8.gz
/var/lib/iscsi
/var/lib/iscsi/ifaces
/var/lib/iscsi/isns
/var/lib/iscsi/nodes
/var/lib/iscsi/send_targets
/var/lib/iscsi/slp
/var/lib/iscsi/static
```
And this is what we should look for:

```text
/usr/sbin/iscsi-gen-initiatorname
/usr/sbin/iscsi-iname
/usr/sbin/iscsiadm
/usr/sbin/iscsid
```

We'll also check whether iscsi has a service unit.

```bash
systemctl status iscsi
```

Right now it should display as inactive state because we haven't established an iSCSI session yet, so there's nothing for the initiator to log into

10. Generate the initiator identity

We need to generate our own IQN using the installed tool. The command to run is:

```bash
sudo iscsi-iname
```

It should output something like this format: `iqn.1994-05.com.redhat:xxxxxxxxxxxx`

In this case, the iscsi generated an iqn: `iqn.1994-05.com.redhat:377c931af36a`

Then we will also run:

```bash
sudo iscsi-gen-initiatorname
```
If we didn't run this command, iscsi wouldn't create a `initiatorname.iscsi` file on `/etc/iscsi/initiatorname.iscsi`.

So if we, verify the file inside, we get this:

```bash
[root@localhost ~]# cat /etc/iscsi/initiatorname.iscsi
##
## /etc/iscsi/initiatorname.iscsi
##
## Default iSCSI Initiatorname.
##
## DO NOT EDIT OR REMOVE THIS FILE!
## If you remove this file, the iSCSI daemon will not start.
## If you change the InitiatorName, existing access control lists
## may reject this initiator. The InitiatorName must be unique
## for each iSCSI initiator. Do NOT duplicate iSCSI InitiatorNames.
InitiatorName=iqn.1994-05.com.redhat:3731c7982838
```
Now that we have the InitiatorName let's go back to the other VM and this time we'll create a rule to allow this IQN to establish iSCSI connection to it.

11. Create an ACL on targetcli

On the storage server, enter the target shell `targetcli`

Then configure the following command:

`/iscsi/iqn.2026-08.local.lab:storage01/tpg1/acls create iqn.1994-05.com.redhat:3731c7982838`

After that lets verify:

```text
/> /iscsi/iqn.2026-08.local.lab:storage01/tpg1/acls create iqn.1994-05.com.redhat:3731c7982838
Created Node ACL for iqn.1994-05.com.redhat:3731c7982838
Created mapped LUN 0.
/> ls
o- / ............................................................................................................. [...]
  o- backstores .................................................................................................. [...]
  | o- block ...................................................................................... [Storage Objects: 1]
  | | o- raid6_storage ...................................................... [/dev/md0 (999.7GiB) write-thru activated]
  | |   o- alua ....................................................................................... [ALUA Groups: 1]
  | |     o- default_tg_pt_gp ........................................................... [ALUA state: Active/optimized]
  | o- fileio ..................................................................................... [Storage Objects: 0]
  | o- pscsi ...................................................................................... [Storage Objects: 0]
  | o- ramdisk .................................................................................... [Storage Objects: 0]
  o- iscsi ................................................................................................ [Targets: 1]
  | o- iqn.2026-08.local.lab:storage01 ....................................................................... [TPGs: 1]
  |   o- tpg1 ................................................................................... [no-gen-acls, no-auth]
  |     o- acls .............................................................................................. [ACLs: 1]
  |     | o- iqn.1994-05.com.redhat:3731c7982838 ...................................................... [Mapped LUNs: 1]
  |     |   o- mapped_lun0 ............................................................. [lun0 block/raid6_storage (rw)]
  |     o- luns .............................................................................................. [LUNs: 1]
  |     | o- lun0 .................................................. [block/raid6_storage (/dev/md0) (default_tg_pt_gp)]
  |     o- portals ........................................................................................ [Portals: 1]
  |       o- 0.0.0.0:3260 ......................................................................................... [OK]
  o- loopback ............................................................................................. [Targets: 0]
/>
```
We can see here that the iqn we generated from the iscsi initiator VM exists on targetcli and it is mapped to a LUN. The `no-gen-acls` means generic/default ACL generation is disabled. So it doesn't mean the explicitly-created ACL isn't there. The `no-auth` means we're not using CHAP authentication yet.

exiting on targetcli will save the configuration. Now we're ready to connect the initiator to the server.

12. Connecting to storage server via iSCSI.

Before we do something, let's verify whether the storage server is listening to Port 3260 and check its IP address

```bash
[root@localhost ~]# ss -tlnp | grep 3260
LISTEN 0      256          0.0.0.0:3260      0.0.0.0:*
[root@localhost ~]# ip -br addr
lo               UNKNOWN        127.0.0.1/8 ::1/128
enp6s18          UP             172.16.2.66/24 fe80::be24:11ff:fe53:6248/64
```

The IP address of the server is 172.16.2.66 and confirmed listening to 3260. Now onto the iSCSI initiator VM. We'll make it discover the storage server. This is the command that we will run:

```bash
sudo iscsiadm -m discovery -t sendtargets -p <STORAGE_SERVER_IP:3260
```
The output should look like this:

```bash
[root@localhost ~]# iscsiadm -m discovery -t sendtargets -p 172.16.2.66:3260
172.16.2.66:3260,1 iqn.2026-08.local.lab:storage01
```

This indicates that the initiator was able to discover the storage server via iscsi network now we will need to login. The command we'll use:

```bash
sudo iscsiadm -m node \
  -T iqn.2026-08.local.lab:storage01 \
  -p 172.16.2.66:3260 \
  --login
```

Then it should output that the login to the target via 172.16.2.66 is successful.

```bash
[root@localhost ~]# sudo iscsiadm -m node \
  -T iqn.2026-08.local.lab:storage01 \
  -p 172.16.2.66:3260 \
  --login
Login to [iface: default, target: iqn.2026-08.local.lab:storage01, portal: 172.16.2.66,3260] successful.
```

13. Verify from the initiator VM

Now that the VM has logged in to the storage server VM, we can verify this by using the command `lsblk`

```bash
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda           8:0    0    45G  0 disk
├─sda1        8:1    0     1G  0 part /boot
└─sda2        8:2    0    44G  0 part
  ├─cs-root 253:0    0  39.5G  0 lvm  /
  └─cs-swap 253:1    0   4.5G  0 lvm  [SWAP]
sdb           8:16   0 999.7G  0 disk
```

Notice the block device name `sdb`, look at the size `999.7G` this means that the VM discovered the iSCSI network that the storage server provided, and also the disk was able to attached to the VM. This concludes it. The md0 from the storage server VM was able to provide its disk to the initiator VM and now it is using it.


14. Configure Persistent mounts and target configurations

Last thing we can do now is prevent Linux on forgetting to setup the RAID and targetcli being empty. Let's generate the mdadm configuration.

```bash
[root@localhost ~]# sudo mdadm --detail --scan | sudo tee /etc/mdadm.conf
ARRAY /dev/md0 metadata=1.2 UUID=6d8e8e42:ee7d7762:f7994ea6:19b150c2
```

This let's mdadm to persistently remember that this RAID array UUID should be assembled as `/dev/md0`

For the `targetcli`

run this inside `targetcli`

```bash
/> ls
o- / ............................................................................................................. [...]
  o- backstores .................................................................................................. [...]
  | o- block ...................................................................................... [Storage Objects: 1]
  | | o- raid6_storage ...................................................... [/dev/md0 (999.7GiB) write-thru activated]
  | |   o- alua ....................................................................................... [ALUA Groups: 1]
  | |     o- default_tg_pt_gp ........................................................... [ALUA state: Active/optimized]
  | o- fileio ..................................................................................... [Storage Objects: 0]
  | o- pscsi ...................................................................................... [Storage Objects: 0]
  | o- ramdisk .................................................................................... [Storage Objects: 0]
  o- iscsi ................................................................................................ [Targets: 1]
  | o- iqn.2026-08.local.lab:storage01 ....................................................................... [TPGs: 1]
  |   o- tpg1 ................................................................................... [no-gen-acls, no-auth]
  |     o- acls .............................................................................................. [ACLs: 1]
  |     | o- iqn.1994-05.com.redhat:3731c7982838 ...................................................... [Mapped LUNs: 1]
  |     |   o- mapped_lun0 ............................................................. [lun0 block/raid6_storage (rw)]
  |     o- luns .............................................................................................. [LUNs: 1]
  |     | o- lun0 .................................................. [block/raid6_storage (/dev/md0) (default_tg_pt_gp)]
  |     o- portals ........................................................................................ [Portals: 1]
  |       o- 0.0.0.0:3260 ......................................................................................... [OK]
  o- loopback ............................................................................................. [Targets: 0]
/> saveconfig
Last 10 configs saved in /etc/target/backup/.
Configuration saved to /etc/target/saveconfig.json
exit
```

To check if the config was saved, check the `/etc/target/saveconfig.json`

```bash
cat /etc/target/saveconfig.json

{
  "fabric_modules": [],
  "storage_objects": [
    {
      "alua_tpgs": [
        {
          "alua_access_state": 0,
          "alua_access_status": 0,
          "alua_access_type": 3,
          "alua_support_active_nonoptimized": 1,
          "alua_support_active_optimized": 1,
          "alua_support_offline": 1,
          "alua_support_standby": 1,
          "alua_support_transitioning": 1,
          "alua_support_unavailable": 1,
          "alua_write_metadata": 0,
          "implicit_trans_secs": 0,
          "name": "default_tg_pt_gp",
          "nonop_delay_msecs": 100,
          "preferred": 0,
          "tg_pt_gp_id": 0,
          "trans_delay_msecs": 0
        }
      ],
      "attributes": {
        "alua_support": 1,
        "block_size": 512,
        "emulate_3pc": 1,
        "emulate_caw": 1,
        "emulate_dpo": 1,
        "emulate_fua_read": 1,
        "emulate_fua_write": 1,
        "emulate_model_alias": 1,
        "emulate_pr": 1,
        "emulate_rest_reord": 0,
        "emulate_rsoc": 1,
        "emulate_tas": 1,
        "emulate_tpu": 0,
        "emulate_tpws": 0,
        "emulate_ua_intlck_ctrl": 0,
        "emulate_write_cache": 0,
        "enforce_pr_isids": 1,
        "force_pr_aptpl": 0,
        "is_nonrot": 0,
        "max_unmap_block_desc_count": 0,
        "max_unmap_lba_count": 0,
        "max_write_same_len": 65535,
        "optimal_sectors": 2048,
        "pgr_support": 1,
        "pi_prot_format": 0,
        "pi_prot_type": 0,
        "pi_prot_verify": 0,
        "queue_depth": 128,
        "submit_type": 0,
        "unmap_granularity": 0,
        "unmap_granularity_alignment": 0,
        "unmap_zeroes_data": 0
      },
      "dev": "/dev/md0",
      "name": "raid6_storage",
      "plugin": "block",
      "readonly": false,
      "write_back": false,
      "wwn": "1c59c32e-d5c4-4bc8-ad12-aa6c03526f63"
    }
  ],
  "targets": [
    {
      "fabric": "iscsi",
      "parameters": {
        "cmd_completion_affinity": "-1"
      },
      "tpgs": [
        {
          "attributes": {
            "authentication": 0,
            "cache_dynamic_acls": 0,
            "default_cmdsn_depth": 64,
            "default_erl": 0,
            "demo_mode_discovery": 1,
            "demo_mode_write_protect": 1,
            "fabric_prot_type": 0,
            "generate_node_acls": 0,
            "login_keys_workaround": 1,
            "login_timeout": 15,
            "prod_mode_write_protect": 0,
            "t10_pi": 0,
            "tpg_enabled_sendtargets": 1
          },
          "enable": true,
          "luns": [
            {
              "alias": "3cdfad2095",
              "alua_tg_pt_gp_name": "default_tg_pt_gp",
              "index": 0,
              "storage_object": "/backstores/block/raid6_storage"
            }
          ],
          "node_acls": [
            {
              "attributes": {
                "authentication": -1,
                "dataout_timeout": 3,
                "dataout_timeout_retries": 5,
                "default_erl": 0,
                "nopin_response_timeout": 30,
                "nopin_timeout": 15,
                "random_datain_pdu_offsets": 0,
                "random_datain_seq_offsets": 0,
                "random_r2t_offsets": 0
              },
              "mapped_luns": [
                {
                  "alias": "d70213d1e3",
                  "index": 0,
                  "tpg_lun": 0,
                  "write_protect": false
                }
              ],
              "node_wwn": "iqn.1994-05.com.redhat:3731c7982838"
            }
          ],
          "parameters": {
            "AuthMethod": "CHAP,None",
            "DataDigest": "CRC32C,None",
            "DataPDUInOrder": "Yes",
            "DataSequenceInOrder": "Yes",
            "DefaultTime2Retain": "20",
            "DefaultTime2Wait": "2",
            "ErrorRecoveryLevel": "0",
            "FirstBurstLength": "65536",
            "HeaderDigest": "CRC32C,None",
            "IFMarkInt": "Reject",
            "IFMarker": "No",
            "ImmediateData": "Yes",
            "InitialR2T": "Yes",
            "MaxBurstLength": "262144",
            "MaxConnections": "1",
            "MaxOutstandingR2T": "1",
            "MaxRecvDataSegmentLength": "8192",
            "MaxXmitDataSegmentLength": "262144",
            "OFMarkInt": "Reject",
            "OFMarker": "No",
            "TargetAlias": "LIO Target"
          },
          "portals": [
            {
              "ip_address": "0.0.0.0",
              "iser": false,
              "offload": false,
              "port": 3260
            }
          ],
          "tag": 1
        }
      ],
      "wwn": "iqn.2026-08.local.lab:storage01"
    }
  ]
}
```

The only thing we need to do now is enable and start target.service

```bash
systemctl enable --now target
```

## Phase 5: Testing Persistent configuration

Check if the iscsi client is still connected

```bash
[root@localhost ~]# iscsiadm -m session
tcp: [1] 172.16.2.66:3260,1 iqn.2026-08.local.lab:storage01 (non-flash)
```

Then disconnect

```bash
sudo iscsiadm -m node \
  -T iqn.2026-08.local.lab:storage01 \
  -p 172.16.2.66:3260 \
  --logout
```

``` bash
lsblk

NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0   45G  0 disk
├─sda1        8:1    0    1G  0 part /boot
└─sda2        8:2    0   44G  0 part
  ├─cs-root 253:0    0 39.5G  0 lvm  /
  └─cs-swap 253:1    0  4.5G  0 lvm  [SWAP]
```

As expected, `sdb` disappeared that means we've successfully disconnect the client.

Then reboot the storage server then check the following configurations using these commands:

```bash
mdadm --detail /dev/md0
targetcli ls
```

## Troubleshoot

### iscsiadm: cannot make connection

Issue:
```bash
[root@localhost ~]# iscsiadm -m discovery -t sendtargets -p 172.16.2.66:3260
iscsiadm: cannot make connection to 172.16.2.66: No route to host
iscsiadm: cannot make connection to 172.16.2.66: No route to host
iscsiadm: cannot make connection to 172.16.2.66: No route to host
iscsiadm: cannot make connection to 172.16.2.66: No route to host
iscsiadm: cannot make connection to 172.16.2.66: No route to host
iscsiadm: cannot make connection to 172.16.2.66: No route to host
iscsiadm: connection login retries (reopen_max) 5 exceeded
iscsiadm: Could not perform SendTargets discovery: iSCSI PDU timed out
[root@localhost ~]# ping 172.16.2.66
PING 172.16.2.66 (172.16.2.66) 56(84) bytes of data.
64 bytes from 172.16.2.66: icmp_seq=1 ttl=64 time=0.224 ms
64 bytes from 172.16.2.66: icmp_seq=2 ttl=64 time=0.211 ms
64 bytes from 172.16.2.66: icmp_seq=3 ttl=64 time=0.277 ms
64 bytes from 172.16.2.66: icmp_seq=4 ttl=64 time=0.256 ms
^C
--- 172.16.2.66 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3049ms
rtt min/avg/max/mdev = 0.211/0.242/0.277/0.026 ms
```

Solution:

From the Storage (SAN) Server VM
```bash
[root@localhost ~]# firewall-cmd --permanent --add-port=3260/tcp
success
[root@localhost ~]# firewall-cmd --reload
success

[root@localhost ~]# sudo firewall-cmd --list-ports
3260/tcp
```

From the Initiator VM
```bash
[root@localhost ~]# iscsiadm -m discovery -t sendtargets -p 172.16.2.66:3260
172.16.2.66:3260,1 iqn.2026-08.local.lab:storage01
```

### Targetcli configuration disappeared and md0 changed to md127 by Linux after reboot

Issue:
```bash
md127 : active (auto-read-only) raid6 sdd[2] sdb[0] sdc[1] sde[3]
  1048311808 blocks super 1.2 level 6, 512k chunk, algorithm 2 [4/4] [UUUU]
  bitmap: 0/4 pages [0KB], 65536KB chunk

unused devices: <none>
```

```bash
/dev/md127:
           Version : 1.2
     Creation Time : Sun Aug 16 17:39:51 2026
        Raid Level : raid6
        Array Size : 1048311808 (999.75 GiB 1073.47 GB)
     Used Dev Size : 524155904 (499.87 GiB 536.74 GB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

     Intent Bitmap : Internal

       Update Time : Sun Aug 16 18:23:41 2026
             State : clean
    Active Devices : 4
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : bitmap

              Name : localhost.localdomain:0  (local to host localhost.localdomain)
              UUID : 6d8e8e42:ee7d7762:f7994ea6:19b150c2
            Events : 515

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       3       8       64        3      active sync   /dev/sde


Personalities : [raid4] [raid5] [raid6]
md127 : active (auto-read-only) raid6 sdd[2] sdb[0] sdc[1] sde[3]
      1048311808 blocks super 1.2 level 6, 512k chunk, algorithm 2 [4/4] [UUUU]
      bitmap: 0/4 pages [0KB], 65536KB chunk

unused devices: <none>


[root@localhost ~]# sudo mdadm --examine --scan
ARRAY /dev/md/0  metadata=1.2 UUID=6d8e8e42:ee7d7762:f7994ea6:19b150c2
[root@localhost ~]#
```

```bash
[root@localhost ~]# targetcli
targetcli shell version 2.1.57
Copyright 2011-2013 by Datera, Inc and others.
For help on commands, type 'help'.

/> ls
o- / ............................................................................................................. [...]
  o- backstores .................................................................................................. [...]
  | o- block ...................................................................................... [Storage Objects: 0]
  | o- fileio ..................................................................................... [Storage Objects: 0]
  | o- pscsi ...................................................................................... [Storage Objects: 0]
  | o- ramdisk .................................................................................... [Storage Objects: 0]
  o- iscsi ................................................................................................ [Targets: 0]
  o- loopback ............................................................................................. [Targets: 0]
/>
```


Solution:
```bash
sudo mdadm --stop /dev/md127
```

```bash
sudo mdadm --assemble /dev/md0 /dev/sdb /dev/sdc /dev/sdd /dev/sde
mdadm: /dev/md0 has been started with 4 drives.
```
