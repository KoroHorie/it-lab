## Proxmox GPU Passthrough

## Phase 1 - Checking GPU IOMMU Group
1. Find the PCI address of the GPU (RTX 3060).

`lspci`

`01:00.0 VGA compatible controller: NVIDIA Corporation GA104 [GeForce RTX 3060] (rev a1)`
`01:00.1 Audio device: NVIDIA Corporation GA104 High Definition Audio Controller (rev a1)`

2. For a VM passthrough, we generally want to pass both 01:00.0 and 01:00.1 to the same VM. The .1 function is the HDMI/DisplayPort audio device from the GPU

3. The next thing we need to determine is whether the RTX 3060 is isolated in its own IOMMU group.
Run: `find /sys/kernel/iommu_groups/ -type 1`

`/sys/kernel/iommu_groups/9/devices/0000:01:00.0`
`/sys/kernel/iommu_groups/9/devices/0000:01:00.1`

A better way to identify the IOMMU Group of the GPU, we can use this:

for g in /sys/kernel/iommu_groups/*; do
    echo "IOMMU Group ${g##*/}:"
    for d in "$g"/devices/*; do
        lspci -nns "${d##*/}"
    done
    echo
done`

which it will display something like this:

`IOMMU Group 9:`
`01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GA104 [GeForce RTX 3060] [10de:2487] (rev a1)`
`01:00.1 Audio device [0403]: NVIDIA Corporation GA104 High Definition Audio Controller [10de:228b] (rev a1)`

4. We also need to determine the current driver owned by the GPU

`lspci -nnk -s 01:00.0`

01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GA104 [GeForce RTX 3060] [10de:2487] (rev a1)
        Subsystem: Micro-Star International Co., Ltd. [MSI] Device [1462:397d]
        Kernel driver in use: nouveau
        Kernel modules: nvidiafb, nouveau, nova_core

`lspci -nnk -s 01:00.1`

01:00.1 Audio device [0403]: NVIDIA Corporation GA104 High Definition Audio Controller [10de:228b] (rev a1)
        Subsystem: Micro-Star International Co., Ltd. [MSI] Device [1462:397d]
        Kernel driver in use: snd_hda_intel
        Kernel modules: snd_hda_intel

5. Checking whether IOMMU is actually enabled

`dmesg | grep -Ei 'iommu|amd-vi'`

[    0.160946] AMD-Vi: Using global IVHD EFR:0x206d73ef22254ade, EFR2:0x0
[    0.410466] iommu: Default domain type: Translated
[    0.410466] iommu: DMA domain TLB invalidation policy: lazy mode
[    0.449819] pci 0000:00:00.2: AMD-Vi: IOMMU performance counters supported
[    0.449922] pci 0000:00:01.0: Adding to iommu group 0
[    0.449942] pci 0000:00:01.1: Adding to iommu group 1
[    0.449980] pci 0000:00:02.0: Adding to iommu group 2
[    0.449999] pci 0000:00:02.1: Adding to iommu group 3
[    0.450019] pci 0000:00:02.2: Adding to iommu group 4
[    0.450048] pci 0000:00:08.0: Adding to iommu group 5
[    0.450065] pci 0000:00:08.1: Adding to iommu group 6
[    0.450095] pci 0000:00:14.0: Adding to iommu group 7
[    0.450110] pci 0000:00:14.3: Adding to iommu group 7
[    0.450177] pci 0000:00:18.0: Adding to iommu group 8
[    0.450191] pci 0000:00:18.1: Adding to iommu group 8
[    0.450207] pci 0000:00:18.2: Adding to iommu group 8
[    0.450222] pci 0000:00:18.3: Adding to iommu group 8
[    0.450237] pci 0000:00:18.4: Adding to iommu group 8
[    0.450254] pci 0000:00:18.5: Adding to iommu group 8
[    0.450268] pci 0000:00:18.6: Adding to iommu group 8
[    0.450283] pci 0000:00:18.7: Adding to iommu group 8
[    0.450319] pci 0000:01:00.0: Adding to iommu group 9
[    0.450343] pci 0000:01:00.1: Adding to iommu group 9
[    0.450388] pci 0000:02:00.0: Adding to iommu group 10
[    0.450409] pci 0000:02:00.1: Adding to iommu group 10
[    0.450431] pci 0000:02:00.2: Adding to iommu group 10
[    0.450439] pci 0000:03:04.0: Adding to iommu group 10
[    0.450450] pci 0000:03:09.0: Adding to iommu group 10
[    0.450458] pci 0000:04:00.0: Adding to iommu group 10
[    0.450467] pci 0000:05:00.0: Adding to iommu group 10
[    0.450485] pci 0000:06:00.0: Adding to iommu group 11
[    0.450520] pci 0000:07:00.0: Adding to iommu group 12
[    0.450539] pci 0000:07:00.1: Adding to iommu group 13
[    0.450558] pci 0000:07:00.2: Adding to iommu group 14
[    0.450577] pci 0000:07:00.3: Adding to iommu group 15
[    0.450596] pci 0000:07:00.4: Adding to iommu group 16
[    0.450616] pci 0000:07:00.6: Adding to iommu group 17
[    0.452656] AMD-Vi: Extended features (0x206d73ef22254ade, 0x0): PPR X2APIC NX GT IA GA PC GA_vAPIC
[    0.452668] AMD-Vi: Interrupt remapping enabled
[    0.452669] AMD-Vi: X2APIC enabled
[    0.672579] AMD-Vi: Virtual APIC enabled
[    0.673907] perf/amd_iommu: Detected AMD IOMMU #0 (2 banks, 4 counters/bank).

## Phase 2 - VFIO Configuration 

Pre-requisites checklists:

✅ AMD IOMMU enabled
✅ Interrupt remapping enabled
✅ RTX 3060 detected at 01:00.0
✅ GPU audio at 01:00.1
✅ Both are in IOMMU Group 9
✅ Group 9 contains only those two functions
⚠️ Host currently uses nouveau → we need to detach it and bind the GPU to vfio-pci

1. Check loaded GPU modules

`lsmod | grep -E 'nouveau|nvidia'`

nouveau              2969600  0
gpu_sched              69632  1 nouveau
drm_gpuvm              53248  1 nouveau
mxm_wmi                12288  1 nouveau
drm_ttm_helper         20480  1 nouveau
ttm                   126976  2 drm_ttm_helper,nouveau
drm_exec               12288  2 drm_gpuvm,nouveau
drm_display_helper    286720  1 nouveau
i2c_algo_bit           16384  1 nouveau
video                  77824  1 nouveau
wmi                    32768  5 video,gigabyte_wmi,wmi_bmof,mxm_wmi,nouveau

This output tells us that nouveau has been loaded

`lsof /dev/nvidia* 2>/dev/null`
return 0

`ls -l /dev/dri/`
total 0
drwxr-xr-x 2 root root         80 Aug 13 19:37 by-path
crw-rw---- 1 root video  226,   0 Aug 13 19:37 card0
crw-rw---- 1 root render 226, 128 Aug 13 19:37 renderD128

card0 and renderD128 are being provided by the DRM stack so these are most likely nouveau

What we're going to do is we need to take out RTX 3060 from using nouveau from Linux host and transition it to make the RTX 3060 to passthrough its PCI to VM via vfio-pci.

The GPU is isolated in IOMMU Group 9 so we don't need any ACS override or other questionable workarounds.

2. Blacklisting nouveau

Create a blacklist configuration:
`nano /etc/modprobe.d/blacklist-nouveau.conf`

Configure the following inside blacklist-nouveau.conf

blacklist nouveau
options nouveau modeset=0

Then we need to make sure VFIO modules are available, to check that run:

`lsmod | grep vfio`

It is expected that there's no output, they are not loaded yet.

3. Identify the PCI IDs

The IDs are:

10de:2487 → RTX 3060 GPU
10de:228b → RTX 3060 Audio

We'll use these IDs to tell vfio-pci that they are belong to VFIO. To do that we need to create a vfio configuration file

`nano /etc/modprobe.d/vfio.conf`

And put this inside:

`options vfio-pci ids=10de:2487,10de:228b`

Then we'll rebuild the initramfs so the configuration is applied early during boot.

cat /etc/modprobe.d/blacklist-nouveau.conf && cat /etc/modprobe.d/vfio.conf 
blacklist nouveau
options nouveau modeset=0
options vfio-pci ids=10de:2487,10de:228b

In this configuration, we're telling the host to not load the nouveau and never initialize nouveau modsetting.

Bind the NVIDIA device 10de:2487 to vfio-pci aswell as the NVIDIA audio device 10de:228b


4. Make the VFIO load early

Before rebuilding the initramfs, check Proxmox kernel modules configuration

`cat /etc/modules`

# /etc/modules is obsolete and has been replaced by /etc/modules-load.d/.
# Please see modules-load.d(5) and modprobe.d(5) for details.
#
# Updating this file still works, but it is undocumented and unsupported.

As we can see there's nothing here, this is where we will add the following vfio modules such as:
- vfio
- vfio_iommu_type1
- vfio_pci

Save and Exit then run this command:

`update-initramfs -u -k all`

update-initramfs: Generating /boot/initrd.img-7.0.12-1-pve
Running hook script 'zz-proxmox-boot'..
Re-executing '/etc/kernel/postinst.d/zz-proxmox-boot' in new private mount namespace..
No /etc/kernel/proxmox-boot-uuids found, skipping ESP sync.
update-initramfs: Generating /boot/initrd.img-7.0.2-6-pve
Running hook script 'zz-proxmox-boot'..
Re-executing '/etc/kernel/postinst.d/zz-proxmox-boot' in new private mount namespace..
No /etc/kernel/proxmox-boot-uuids found, skipping ESP sync.

One small cleanup

Because Proxmox kernel explicitly says /etc/modules is obsolete, we prefer to put those VFIO modules in the modern location:

`nano /etc/modules-load.d/vfio.conf`

cat /etc/modules-load.d/vfio.conf 
vfio 
vfio_iommu_type1 
vfio_pci

Then run the update-initramfs command again.

5. Reboot the Proxmox host

6. Verify VFIO took ownership

root@pve-node1:~# lspci -nnk -s 01:00.0
01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GA104 [GeForce RTX 3060] [10de:2487] (rev a1)
        Subsystem: Micro-Star International Co., Ltd. [MSI] Device [1462:397d]
        Kernel driver in use: **vfio-pci**
        Kernel modules: nvidiafb, nouveau, nova_core
root@pve-node1:~# lspci -nnk -s 01:00.1
01:00.1 Audio device [0403]: NVIDIA Corporation GA104 High Definition Audio Controller [10de:228b] (rev a1)
        Subsystem: Micro-Star International Co., Ltd. [MSI] Device [1462:397d]
        Kernel driver in use: **vfio-pci**
        Kernel modules: snd_hda_intel

On lsmod we can literally see VFIO modules loaded and no nouveau module.

root@pve-node1:~# lsmod | grep -E 'nouveau|vfio'
vfio_pci               20480  0
vfio_pci_core          94208  1 vfio_pci
irqbypass              16384  2 vfio_pci_core,kvm
vfio_iommu_type1       53248  0
vfio                   73728  4 vfio_pci_core,vfio_iommu_type1,vfio_pci
iommufd               131072  1 vfio

7. 
