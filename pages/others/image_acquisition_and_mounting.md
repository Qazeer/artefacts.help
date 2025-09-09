---
title: Procedure - Image acquisition and mounting
summary: ''
keywords: image, disk, mount, raw, img, Advanced Forensics Format, AFF, EnCase, E01, FTK Imager, dd, dc3dd, Guymager, CAINE, TX1, Tableau, MBR, GPT, vmdk, VHD, VHDX, VDI, OVF, ESXi, vSphere, lsblk, SSHFS, Arsenal Image Mounter, guestmount, ewfmount, Logical Volume Manager, LVM, kpartx, pvs, lvdisplay, vgchange, mmls, loop, noload, noatime, noexec, show_sys_files
tags:
  - procedures
location: ''
last_updated: 2025-09-01
sidebar: sidebar
permalink: procedure_image_acquisition_and_mounting.html
folder: others
---

## Image acquisition

### Physical devices

For physical drives, the drive to be imaged should be extracted from the system
and plugged on an acquisition station through a (ideally) hardware write
blocker. Dedicated hardware, such as the `TX1 Tableau Forensic Imager`, can
also be used to directly make a drive to drive or drive to file (such as a
`raw` file) copy.

If the disk to image cannot be extracted from the system, a bootable USB drive
can be used to boot into a temporary OS (if the BIOS boot order of the target
system can be changed). A Linux distribution suitable for forensic imaging
should be used, such as the [`CAINE`](https://www.caine-live.net/) distribution
(based on `Ubuntu`) or [`Kali Linux in Forensics Mode`](https://www.kali.org/docs/general-use/kali-linux-forensics-mode/). In such distributions all devices are blocked in read-only and
auto-mounting is disabled, and a number of forensics tools are installed for
acquisition.

#### Using a Windows OS

Windows should only be used as a acquisition platform if the drive to image can
be connected through an hardware write-blocker, as Windows will automatically
attempt to mount any recognized partitions, potentially altering the data.

The `FTK Imager` utility can be used to create a forensic image of a physical
drive or logical drive / partition. `FTK Imager` can create images in raw,
EnCase (`E01`), or `Advanced Forensics Format (AFF)` formats.

The procedure is as follows:

```
-> File -> Create Disk Image...
   -> Physical Drive -> Selection of the source drive / device
   -> Add image destination -> Selection of the image type (raw, E01, etc.)
   -> Evidence collection metadata
   -> Selection of the image destination folder and name
```

#### Using a Linux OS

*Devices overview*

Linux storage devices, such as hard disks or SSDs, are typically block
devices represented as files under `/dev`. Block devices can be divided into
one or more logical disks called partitions. This partitioning is recorded in
the partition table, such as `MBR` or `GPT`, usually found in sector 0 of the
disk.

| Type | block device | Eventual partition(s) |
|------|--------------|-----------------------|
| SATA drives | Each drive is represented as `/dev/sdx`. <br><br> Example of two drives: <br> `/dev/sda` <br> `/dev/sdc` | Each partition is represented as `/dev/sdxN`. <br><br> Example of two partitions on the same drive: <br> `/dev/sda1` <br> `/dev/sda2` |
| NVMe drives | Each drive is represented as `/dev/nvmeXn1`. <br><br> Examples: <br> `/dev/nvme0n1` <br> `/dev/nvme1n1` | Each drive is partition as `/dev/nvmeXpX`. <br><br> Examples: <br> `/dev/nvme0n1p1` <br> `/dev/nvme0n1p2` |
| Optical medias (CD or DVD drives) | `/dev/srx` <br><br> Example of two CD / DVD drives: <br> `/dev/sr0` <br> `/dev/sr1` | NA |

The `lsblk` utility can be used to list the block devices present on a system
and the `fdisk` or `gdisk` (for handling of `GPT`) utilities to retrieve more
information on a specific device and its partitions.

```bash
# Lists the block devices present on the system.
lsblk

# Retrieves information on the specific block device and its eventual partitions.
# <DEVICE> example: /dev/sda
fdisk -l <DEVICE>
gdisk -l <DEVICE>
```

*Imaging using dd*

`dd` is the standard copy utility that, while not forensic oriented, can be
used to create image of a device / partition. `dd` presents the advantage of
being available in most Linux distributions. A more forensics sound utility,
such as `dc3dd` (presented below), should generally be used if possible.

```bash
# <DEVICE> example: /dev/sda
# <PARTITION> example: /dev/sda1

sha256sum <DEVICE | PARTITION> > /dest_media/SHA256_original

# The device / partition block size can be retrieved using the fdisk utility.
# The "conv=noerror,sync" option instructs dd to continue the copy if a bad sector is encountered (otherwise dd terminates).
dd if=<DEVICE | PARTITION> of=/dest_media/image.raw bs=<2k | 4k | BLOCK_SIZE> status=progress [conv=noerror,sync]

sha256sum /dest_media/image.raw  > /dest_media/SHA256_image
```

*Imaging using dc3dd*

`dc3dd` is a more forensic oriented utility, based on `dd`, to make disk or
partition image. `dc3dd` automatically detects bad sectors, natively integrates
hashing of the input and output bytes, and integrates logging functionalities,
making it more suitable for forensic imaging.

```bash
# log option: output log files.
# hlog option: output file that will contains the image hash(es).

dc3dd if=<DEVICE | PARTITION> hof=/dest_media/image.raw log=<LOG_FILE> hash=sha256 hlog=<HASH_FILE>
```

*Imaging using Guymager*

[`Guymager`](https://guymager.sourceforge.io/) is a GUI forensic imager, that
can produce image files in raw, EWF, and AFF formats. `Guymager` is
multi-threaded for faster collection and integrates hashing calculation.

```
Right click device -> Acquire image
  -> Format selection (raw, EWF, AFF)
  -> Evidence collection metadata
  -> Selection of the image destination folder and name
  -> Calculate SHA-256
```

### Virtual Machines

#### VMware vSphere

*Procedure in brief*

  1. Take a snapshot of the virtual machine to save the VM virtual memory (the
     corresponding option must be enabled when taking the snapshot). This can
     be done through the `ESXi Host client` or in `SSH`.

  2. Shutdown the virtual machine and copy all the files in the VM folder from
     the datastore. Depending on the number of past snapshot(s) taken, relevant
     data may be present in different virtual drives. Additionally, metadata
     files (`vmsm` for virtual memory or disk descriptor files) can be 
     required / relevant for analysis.  

*Impacts of snapshots on virtual drives*

When taking a snapshot of a virtual machine, a new virtual drive will be
created. Modifications to the filesystem of the virtual machine after that
snapshot standpoint will be applied to this newly created virtual drive. The
drive is associated with its own disk descriptor file, that references the
parent virtual disk (including in an human readable format in the
`parentFileNameHint` field).

The following files are thus created after a snapshot is taken:

  - The virtual drive `vmdk`: `disk-X-sesparse.vmdk`, such as
    `disk-000001-sesparse.vmdk`, with `X` being the snapshot number.

  - The disk descriptor file: `disk-X.vmdk`, such as `disk-000001.vmdk`
    (with `X` also being the snapshot number).

*Known limitations*

If downloading the virtual disk(s) from the `ESXi Host client` `Datastore`
browser web interface, a virtual drive and its associated
disk descriptor file are listed as a single file. For instance as `disk.vmdk`
for `disk.vmdk` (disk descriptor file) and `disk-flat.vmdk` (virtual disk) or
`disk-000001.vmdk` instead of `disk-000001.vmdk` and
`disk-000001-sesparse.vmdk`. While the associated virtual machine is running,
this file will be mapped to the disk descriptor file of the VM and when the
VM is powered-off, the file will correspond to the actual virtual disk.

It is not possible to download any of the virtual drive(s), either using the
`ESXi Host client` web interface or through `SCP`, if the virtual machine is
running. The downloaded `vmdk` will indeed appear empty.

Before shutting down the virtual machine, it is recommended to make a snapshot
of the virtual memory:

  - In `ESXi Host client`, on the virtual machine page:
    `Actions -> Snapshots -> Take snapshot -> Check "Snapshot the virtual machine's memory" -> Take snapshot`.
  
  - In `ESXi Shell`:
    1. The `VMID` of the virtual machine to snapshot must first be retrieved,
       for instance with `vim-cmd vmsvc/getallvms`.
    2. Then a snapshot of the virtual machine, with the virtual memory included,
       can be taken using:
       `vim-cmd vmsvc/snapshot.create <VMID> "<SNAPSHOT_NAME>" "<SNAPSHOT_DESCRIPTION>" true true Create Snapshot:`

The virtual memory of the virtual machine at the time of the snapshot will be
stored in the `vmem` file and the `vmsm` file will contain metadata (such as
offsets to specific structures) required by memory forensics tools.

### Unix / Linux appliances

For some appliances, such as physical network appliances, taking an image of
the filesystem can be more easily achieved from a mounted partition / device
over `SSH` of the live system. If taking an image is not practical, the
filesystem of the remote appliance can also be mounted locally over `SSH`
using `SSHFS`.

#### Disk imaging over SSH

The mounted device(s) / partition(s) can be copied over `SSH` by
using `dd` both on the server and client sides.

```bash
# Running ssh client from the system that will receive the image:
ssh <REMOTE_USER>@<REMOTE_HOST> "dd if=<DEVICE | PARTITION>" | dd of=<IMAGE_FILE>

# Depending on the targeted appliance, it might be required to "drop" into a shell using a specific appliance command (such as "shell" in the example below):
ssh <REMOTE_USER>@<REMOTE_HOST> 'shell "dd if=<DEVICE | PARTITION>"' | dd of=<IMAGE_FILE>

# If running ssh client directly on the remote appliance:
dd if=<DEVICE | PARTITION> | ssh <LOCAL_USER>@<LOCAL_SYSTEM> "dd of=<OUTPUT_FILE>"
```

Following the copy, any eventual message printed by the remote host to `stdout`
upon an `SSH` connection will have to be removed to get the image in a clean
state:

```bash
# Checks the first NUMBER_BYTES bytes of the image for any printed string.
od -A d -N <NUMBER_BYTES> <IMAGE_FILE>

# Trims the first NUMBER_BYTES bytes of the image to remove any eventual stdout printed string.
# Note: tail is much faster than using dd.
tail -c+<NUMBER_BYTES+1> <IMAGE_FILE> > <TRIMMED_IMAGE_FILE>
dd iflag=skip_bytes skip=<NUMBER_BYTES> if=<IMAGE_FILE> of=<TRIMMED_IMAGE_FILE>
```

#### Remote filesystem mounting using SSHFS

```bash
sudo apt install sshfs

sudo mkdir -p /mnt/<SSH_MOUNT_POINT>

# -o IdentityFile: if needed, private key to use instead of the user's password.
sudo sshfs [-odebug,sshfs_debug,loglevel=debug] -o reconnect [-o IdentityFile=<KEY_FILE>] <REMOTE_USER>@<REMOTE_HOST>:/ /mnt/<SSH_MOUNT_POINT>/
```
 
## Image mounting

Following the imaging of a system disk, the image taken must be mounted as a
partition for analysis. Numerous tools can be used, from either the Windows or
Linux operating systems, to do so. An important aspect of image mounting is the
preservation of the artefacts integrity. The image should be mounted in
read-only (or temporary write), with a few specificities to preserve timestamps
and data integrity.

Some utilities, such as `Autoruns`, require writable partitions. To use such
utilities, the image should be mounted using an utility supporting temporary
writes.

### On Windows OS

The [`Arsenal Image Mounter`](https://arsenalrecon.com/downloads/) is a
powerful graphical utility that can be used to mount multiple image types. It
supports temporary write using a diff file that will store the modifications.
`Arsenal Image Mounter` will automatically mount the different partitions of
a given image and implements decryption of `BitLocker` protected partition.

`Arsenal Image Mounter` supports the following disk image format:
  - `raw` / `DD`
  - Multi-parts `raw`
  - `EnCase Evidence File (E01)`
  - `Advanced Forensics Format (AFF)`
  - `VDI` / `VMDK` / `VHD`

Other utilities such as `FTK Imager` or `OSF Mount` may be used as well.

### On Linux OS

#### Virtual Machine disks

The `guestmount` utility can be used to mount a virtual machine disk (`vmdk`,
`vhdx`, `qcow` / `qcow2`, `vdi`, etc.) directly:

```bash
# Attempts to automatically find the device(s) to mount.
guestmount --ro -i -a <VM_DISK_FILE> </mnt/mounted | MOUNT_POINT>

# Requires knowledge of the device to mount.
guestmount -a <VM_DISK_FILE> -m </dev/sda1 | DEVICE> --ro </mnt/mounted_vmdk | MOUNT_POINT>

# Unmounts the mounted device.
guestunmount <MOUNT_POINT>
```

Alternatively, the `qemu-img` utility can be used to convert a virtual machine
disk to a `raw` image:

```
qemu-img convert -O raw <VM_DISK_FILE> <OUTPUT_IMAGE>
```

#### Expert Witness/EnCase (EWF) image

The following procedure can be following to mount disk images in the
`Expert Witness/EnCase (EWF)` format:

```bash
# Mount the raw EWF image. Following the ewfmount, an "ewf1" file should be present in the <RAW_EWF_DIR_PATH> directory.
# The ewfmount utility is part of the "ewf-tools" package on Debian / Kali Linux.
mkdir <RAW_EWF_DIR>
ewfmount <EWF_FILE_PATH> <RAW_EWF_DIR_PATH>

# Mount the image as a loop device.
# show_sys_files and streams_interace=windows are options for Windows NTFS partitions.
mkdir <MOUNTPOINT>
mount <RAW_EWF_DIR_PATH>/ewf1 <MOUNTPOINT_PATH> -o ro,loop,noatime,noexec,noload,norecovery[,show_sys_files,streams_interace=windows]
```

#### Logical Volume Manager image

`Logical Volume Manager (LVM)` is a device mapper framework that provides
logical volume management (for the Linux kernel) to provide a system of
partitions independent of underlying disk layout.

The `LVM` feature is composed of the following building blocks:

  - `Physical volume (PV)`: Unix block device node, such as a hard disk, an
    `MBR` or `GPT` partition, usable for (physical) storage.

  - `Volume group (VG)`: group of `physical volume(s)` that serves as a
    container for `logical volume(s)`.

  - `Logical volume (LV)`: virtual/logical partition that resides in a
    `volume group` and is composed of `physical extents`. `LV` can be directly
    formatted with a file system.

  - `Physical extent (PE)`: smallest contiguous extent in a `physical volume`
    that can be assigned to a `logical volume`.

Examples:

  - Physical disks: `/dev/sda1` and `/dev/sdb1`.

  - Volume Group: `/dev/MyVolGroup/` = `/dev/sda1` + `/dev/sdb1`

  - Logical volumes: `/dev/MyVolGroup/rootvol`, `/dev/MyVolGroup/homevol`,
    `/dev/MyVolGroup/mediavol `

The following commands can be used to mount a `LVM` disk image:

```bash
# Mappings for the LVM's volumes.
kpartx -av <IMAGE_PATH>

# Checks the LVM's PV(s) available.
pvs

# Checks the LVM's VG(s) and LV(s) available.
lvdisplay

# If the LV does not appear, the associated VG may need to be activated.
vgchange -ay <VG_NAME>

# Mounts the specified LV.
# LV path example: /dev/<VG_NAME>/<root | LV_NAME>
mount -o ro,noatime,noexec,noload,norecovery <LV_PATH> <MOUNT_POINT>
```

#### Generic / raw image types

The image partitions can be first determined using the `TSK`'s `mmls` or
`fdisk` utilities. The utilities will retrieve the image sector size and the
partition(s) offsets, both required to mount the partition.

```bash
mmls [ -o offset ] <IMAGE_FILE>
fdisk -l <IMAGE_FILE>

# Units are in <SECTOR_SIZE>-byte sectors
# Slot      Start             End   Length  Description
# [...]
# 02: 00:00 <PARTITION_START> XXX   YYY     NTFS
```

```bash
# mount options:
# ro : read-only.
# noatime : preserve the atime (last access time) timestamps.
# noexec : files from the mounted partition cannot be executed.
# norecovery/noload : prevent replaying of the partition journal to preserve integrity.
# loop : explicitly tells mount to use a loop device (optional on newer version of mount).
# show_sys_files and streams_interace=windows are options for Windows NTFS partitions.

# OFFSET = SECTOR_SIZE * PARTITION_START.

sudo mount -o ro,loop,noload,noatime,noexec,[show_sys_files,streams_interace=windows,]offset=<OFFSET | $((<SECTOR_SIZE> * <PARTITION_START>))> <IMAGE_FILE> </mnt/ | MOUNT_POINT>
```

--------------------------------------------------------------------------------

## References

https://www.linuxleo.com/Docs/LinuxLeo_4.95.1.pdf

https://tmairi.github.io/posts/forensic-aquisition-with-dd-tools/

https://www.youtube.com/watch?v=FoEO9p-J15w
