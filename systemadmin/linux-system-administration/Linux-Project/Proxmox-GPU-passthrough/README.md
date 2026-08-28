# Proxmox GPU Passthrough

## Phase 1 - Checking GPU IOMMU Group

### 1. Find the PCI address of the GPU (RTX 3060)

Run:

```bash
lspci
```

Output:

```text
01:00.0 VGA compatible controller: NVIDIA Corporation GA104 [GeForce RTX 3060] (rev a1)
01:00.1 Audio device: NVIDIA Corporation GA104 High Definition Audio Controller (rev a1)
```

### 2. Identify the GPU Functions

For VM passthrough, we generally want to pass both `01:00.0` and `01:00.1` to the same VM.

- `01:00.0` → RTX 3060 GPU
- `01:00.1` → NVIDIA HDMI/DisplayPort Audio

The `.1` function is the GPU's audio device and should normally be passed through together with the GPU.

### 3. Check the GPU's IOMMU Group

Determine whether the RTX 3060 is isolated in its own IOMMU group.

Run:

```bash
find /sys/kernel/iommu_groups/ -type l
```

Relevant output:

```text
/sys/kernel/iommu_groups/9/devices/0000:01:00.0
/sys/kernel/iommu_groups/9/devices/0000:01:00.1
```

A better way to identify the devices belonging to each IOMMU group is:

```bash
for g in /sys/kernel/iommu_groups/*; do
    echo "IOMMU Group ${g##*/}:"
    for d in "$g"/devices/*; do
        lspci -nns "${d##*/}"
    done
    echo
done
```

Relevant output:

```text
IOMMU Group 9:
01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GA104 [GeForce RTX 3060] [10de:2487] (rev a1)
01:00.1 Audio device [0403]: NVIDIA Corporation GA104 High Definition Audio Controller [10de:228b] (rev a1)
```

This confirms that both RTX 3060 functions are in **IOMMU Group 9** and that the group contains only the GPU and its audio function.

### 4. Check the Current Driver

Check which driver currently owns the GPU:

```bash
lspci -nnk -s 01:00.0
```

Output:

```text
01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GA104 [GeForce RTX 3060] [10de:2487] (rev a1)
        Subsystem: Micro-Star International Co., Ltd. [MSI] Device [1462:397d]
        Kernel driver in use: nouveau
        Kernel modules: nvidiafb, nouveau, nova_core
```

Check the GPU audio device:

```bash
lspci -nnk -s 01:00.1
```

Output:

```text
01:00.1 Audio device [0403]: NVIDIA Corporation GA104 High Definition Audio Controller [10de:228b] (rev a1)
        Subsystem: Micro-Star International Co., Ltd. [MSI] Device [1462:397d]
        Kernel driver in use: snd_hda_intel
        Kernel modules: snd_hda_intel
```

At this point, the host is currently using:

```text
01:00.0 → nouveau
01:00.1 → snd_hda_intel
```

The GPU must later be detached from these host drivers and assigned to `vfio-pci`.

### 5. Verify That IOMMU Is Enabled

Run:

```bash
dmesg | grep -Ei 'iommu|amd-vi'
```

```text
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
```

```text
AMD-Vi: Using global IVHD EFR:0x206d73ef22254ade, EFR2:0x0
iommu: Default domain type: Translated
iommu: DMA domain TLB invalidation policy: lazy mode
pci 0000:01:00.0: Adding to iommu group 9
pci 0000:01:00.1: Adding to iommu group 9
AMD-Vi: Interrupt remapping enabled
AMD-Vi: X2APIC enabled
AMD-Vi: Virtual APIC enabled
perf/amd_iommu: Detected AMD IOMMU #0 (2 banks, 4 counters/bank).
```

The important entries are:

```text
pci 0000:01:00.0: Adding to iommu group 9
pci 0000:01:00.1: Adding to iommu group 9
AMD-Vi: Interrupt remapping enabled
```

This confirms that AMD IOMMU is enabled and the RTX 3060 devices have been assigned to IOMMU Group 9.

---

## Phase 2 - VFIO Configuration

### Pre-requisites Checklist

- ✅ AMD IOMMU enabled
- ✅ Interrupt remapping enabled
- ✅ RTX 3060 detected at `01:00.0`
- ✅ GPU audio detected at `01:00.1`
- ✅ Both devices are in IOMMU Group 9
- ✅ IOMMU Group 9 contains only the RTX 3060 functions
- ⚠️ Host currently uses `nouveau` → GPU needs to be detached and bound to `vfio-pci`

### 1. Check Loaded GPU Modules

Run:

```bash
lsmod | grep -E 'nouveau|nvidia'
```

Output:

```text
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
```

This confirms that `nouveau` is currently loaded.

Check whether any NVIDIA device files are in use:

```bash
lsof /dev/nvidia* 2>/dev/null
```

No output is expected.

Check the DRM devices:

```bash
ls -l /dev/dri/
```

Example output:

```text
total 0
drwxr-xr-x 2 root root         80 Aug 13 19:37 by-path
crw-rw---- 1 root video  226,   0 Aug 13 19:37 card0
crw-rw---- 1 root render 226, 128 Aug 13 19:37 renderD128
```

The goal is to remove the RTX 3060 from the host's `nouveau` driver and assign it to `vfio-pci` so it can be passed through to a VM.

Because the GPU is isolated in IOMMU Group 9, no ACS override is required.

### 2. Blacklist nouveau

Create the blacklist configuration:

```bash
nano /etc/modprobe.d/blacklist-nouveau.conf
```

Add:

```text
blacklist nouveau
options nouveau modeset=0
```

### 3. Configure vfio-pci

Identify the PCI device IDs:

```text
10de:2487 → RTX 3060 GPU
10de:228b → RTX 3060 Audio
```

Create the VFIO configuration:

```bash
nano /etc/modprobe.d/vfio.conf
```

Add:

```text
options vfio-pci ids=10de:2487,10de:228b
```

Verify both configurations:

```bash
cat /etc/modprobe.d/blacklist-nouveau.conf
cat /etc/modprobe.d/vfio.conf
```

Expected:

```text
blacklist nouveau
options nouveau modeset=0

options vfio-pci ids=10de:2487,10de:228b
```

This configuration prevents the host from loading `nouveau` and tells `vfio-pci` to claim both RTX 3060 PCI functions.

### 4. Load VFIO Modules Early

Check the existing module configuration:

```bash
cat /etc/modules
```

The Proxmox kernel indicates that `/etc/modules` is obsolete and has been replaced by `/etc/modules-load.d/`.

Create the modern VFIO module configuration:

```bash
nano /etc/modules-load.d/vfio.conf
```

Add:

```text
vfio
vfio_iommu_type1
vfio_pci
```

Rebuild the initramfs:

```bash
update-initramfs -u -k all
```

Example output:

```text
update-initramfs: Generating /boot/initrd.img-7.0.12-1-pve
Running hook script 'zz-proxmox-boot'..
Re-executing '/etc/kernel/postinst.d/zz-proxmox-boot' in new private mount namespace..
No /etc/kernel/proxmox-boot-uuids found, skipping ESP sync.
update-initramfs: Generating /boot/initrd.img-7.0.2-6-pve
Running hook script 'zz-proxmox-boot'..
Re-executing '/etc/kernel/postinst.d/zz-proxmox-boot' in new private mount namespace..
No /etc/kernel/proxmox-boot-uuids found, skipping ESP sync.
```

### 5. Reboot the Proxmox Host

Reboot the host so the new driver configuration takes effect:

```bash
reboot
```

### 6. Verify VFIO Took Ownership

After reboot, check the GPU:

```bash
lspci -nnk -s 01:00.0
```

Expected:

```text
01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GA104 [GeForce RTX 3060] [10de:2487] (rev a1)
        Subsystem: Micro-Star International Co., Ltd. [MSI] Device [1462:397d]
        Kernel driver in use: vfio-pci
        Kernel modules: nvidiafb, nouveau, nova_core
```

Check the GPU audio device:

```bash
lspci -nnk -s 01:00.1
```

Expected:

```text
01:00.1 Audio device [0403]: NVIDIA Corporation GA104 High Definition Audio Controller [10de:228b] (rev a1)
        Subsystem: Micro-Star International Co., Ltd. [MSI] Device [1462:397d]
        Kernel driver in use: vfio-pci
        Kernel modules: snd_hda_intel
```

Verify that VFIO modules are loaded and `nouveau` is no longer loaded:

```bash
lsmod | grep -E 'nouveau|vfio'
```

Output:

```text
vfio_pci               20480  0
vfio_pci_core          94208  1 vfio_pci
irqbypass              16384  2 vfio_pci_core,kvm
vfio_iommu_type1       53248  0
vfio                   73728  4 vfio_pci_core,vfio_iommu_type1,vfio_pci
iommufd               131072  1 vfio
```

The important result is:

```text
Kernel driver in use: vfio-pci
```

for both `01:00.0` and `01:00.1`, while `nouveau` is no longer loaded.

### 7. VFIO Configuration Complete

At this point, the Proxmox host-side GPU passthrough configuration is complete.

The RTX 3060 is now:

```text
RTX 3060
    ↓
IOMMU Group 9
    ↓
vfio-pci
    ↓
Ready for VM PCI passthrough
```
