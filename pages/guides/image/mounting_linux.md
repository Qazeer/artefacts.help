---
title: Image mounting - Linux
summary: 'Image mounting on Linux, with mount options preserving integrity: virtual machine disks (guestmount and qemu-img), Expert Witness / EnCase (EWF) images (ewfmount), Logical Volume Manager (LVM) images (kpartx), generic / raw images (mmls and mount), and automated mounting (EverReady Disk Mount).'
keywords: image, disk, mount, image mounting, Linux, read-only, guestmount, guestunmount, qemu-img, vmdk, vhdx, qcow2, vdi, raw, Expert Witness, EnCase, EWF, E01, ewfmount, ewf-tools, loop, Logical Volume Manager, LVM, PV, VG, LV, kpartx, pvs, lvdisplay, vgchange, mmls, fdisk, offset, sector size, noload, norecovery, norecover, noatime, noexec, nouuid, show_sys_files, streams_interface, ntfs-3g, ext3, ext4, NTFS, XFS, EverReady Disk Mount, er2.sh, ermount
tags:
  - guides
last_updated: 2025-09-01
sidebar: sidebar
permalink: image_mounting_linux.html
folder: image
---

Following the imaging of a system disk, the image taken must be mounted as a
partition for analysis. Numerous tools can be used, from either the
[Windows](./mounting_windows.md) or Linux operating systems, to do so. An
important aspect of image mounting is the preservation of the artefacts'
integrity. The image should be mounted read-only (or with temporary writes),
with a few specific options to preserve timestamps and data integrity.

## Virtual machine disks

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

## Expert Witness/EnCase (EWF) image

The following procedure can be followed to mount disk images in the
`Expert Witness/EnCase (EWF)` format:

```bash
# Mounts the raw EWF image. Following the ewfmount execution, an "ewf1" file should be present in the <RAW_EWF_DIR_PATH> directory.
# The ewfmount utility is part of the "ewf-tools" package on Debian / Kali Linux.
mkdir <RAW_EWF_DIR>
ewfmount <EWF_FILE_PATH> <RAW_EWF_DIR_PATH>

# Mounts the image as a loop device.
# For an image of a full disk (with a partition table), the offset of the partition to mount must be specified with the "offset=<OFFSET>" option (refer to the "Generic / raw image types" section below).
# noload is an option for ext3 / ext4 partitions, while show_sys_files and streams_interface=windows are options of the ntfs-3g driver for Windows NTFS partitions (refer to the "Generic / raw image types" section below).
mkdir <MOUNTPOINT>

# ext3 / ext4 partitions.
mount <RAW_EWF_DIR_PATH>/ewf1 <MOUNTPOINT_PATH> -o ro,loop,noload,noatime,noexec

# Windows NTFS partitions.
mount -t ntfs-3g <RAW_EWF_DIR_PATH>/ewf1 <MOUNTPOINT_PATH> -o ro,loop,norecover,noatime,noexec,show_sys_files,streams_interface=windows
```

## Logical Volume Manager image

`Logical Volume Manager (LVM)` is a device mapper framework that provides
logical volume management for the Linux kernel, offering a system of
partitions independent of the underlying disk layout.

The `LVM` feature is composed of the following building blocks:

  - `Physical volume (PV)`: Unix block device node, such as a hard disk, an
    `MBR` or `GPT` partition, usable for (physical) storage.

  - `Volume group (VG)`: group of `physical volume(s)` that serves as a
    container for `logical volume(s)`.

  - `Logical volume (LV)`: virtual/logical partition that resides in a
    `volume group` and is composed of `physical extents`. `LVs` can be directly
    formatted with a file system.

  - `Physical extent (PE)`: smallest contiguous extent in a `physical volume`
    that can be assigned to a `logical volume`.

Examples:

  - Physical disks: `/dev/sda1` and `/dev/sdb1`.

  - Volume group: `/dev/MyVolGroup/` = `/dev/sda1` + `/dev/sdb1`

  - Logical volumes: `/dev/MyVolGroup/rootvol`, `/dev/MyVolGroup/homevol`,
    `/dev/MyVolGroup/mediavol `

The following commands can be used to mount an `LVM` disk image:

```bash
# Creates the device mappings for the LVM volumes.
kpartx -av <IMAGE_PATH>

# Lists the available LVM PV(s).
pvs

# Lists the available LVM VG(s) and LV(s).
lvdisplay

# If the LV does not appear, the associated VG may need to be activated.
vgchange -ay <VG_NAME>

# Mounts the specified LV.
# LV path example: /dev/<VG_NAME>/<root | LV_NAME>
# ext3 / ext4 LV.
mount -o ro,noload,noatime,noexec <LV_PATH> <MOUNT_POINT>

# XFS LV (XFS being the default filesystem on Red Hat-based distributions).
mount -o ro,norecovery,noatime,noexec[,nouuid] <LV_PATH> <MOUNT_POINT>
```

## Generic / raw image types

The image partitions can first be determined using `TSK`'s `mmls` or the
`fdisk` utility. The utilities will retrieve the image sector size and the
partition offset(s), both required to mount the partition.

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
# ro: read-only.
# noatime: preserves the atime (last access time) timestamps.
# noexec: files from the mounted partition cannot be executed.
# noload (ext3 / ext4), norecovery (XFS), or norecover (NTFS): prevents the replay of the partition journal/logfile, to preserve integrity.
# loop: explicitly tells mount to use a loop device (optional on newer versions of mount).

# OFFSET = SECTOR_SIZE * PARTITION_START.

# ext3 / ext4 partitions.
sudo mount -o ro,loop,noload,noatime,noexec,offset=<OFFSET | $((<SECTOR_SIZE> * <PARTITION_START>))> <IMAGE_FILE> </mnt/ | MOUNT_POINT>

# Windows NTFS partitions.
# show_sys_files and streams_interface=windows: options of the ntfs-3g driver for Windows NTFS partitions, to respectively list the NTFS metafiles (such as $MFT) and access the alternate data streams (using the <FILE>:<STREAM> syntax).
sudo mount -t ntfs-3g -o ro,loop,norecover,noatime,noexec,show_sys_files,streams_interface=windows,offset=<OFFSET | $((<SECTOR_SIZE> * <PARTITION_START>))> <IMAGE_FILE> </mnt/ | MOUNT_POINT>

# XFS partitions.
# nouuid (XFS): allows mounting an XFS filesystem with the same UUID as an already mounted filesystem (such as the filesystem of the analysis system or of another mounted image).
sudo mount -o ro,loop,norecovery,noatime,noexec,[nouuid,]offset=<OFFSET | $((<SECTOR_SIZE> * <PARTITION_START>))> <IMAGE_FILE> </mnt/ | MOUNT_POINT>
```

## Automated script

The [`EverReady Disk Mount`](https://github.com/dfir-scripts/EverReady-Disk-Mount)
scripts (`er2.sh` and the original interactive
[`ermount.sh`](https://github.com/dfir-scripts/EverReady-Disk-Mount/blob/master/ermount.sh)
script) can be used to automate the read-only mounting of disk images (`raw`,
`E01`, `AFF`, `VMDK`, `VHD` / `VHDX`, `VDI`, `QCOW2`, etc.), including images
containing `LVM` volumes. The original `ermount.sh` script additionally
supports the mounting of `Volume Shadow Copies` and `BitLocker`-encrypted
volumes.

```bash
# -l: scans and activates LVM volumes.
sudo er2.sh -i <IMAGE_FILE> -m <MOUNT_POINT> [-l]

# Unmounts the image.
sudo er2.sh -u -m <MOUNT_POINT>
```

--------------------------------------------------------------------------------

## References

  - [Barry J. Grundy - The Law Enforcement and Forensic Examiner’s Introduction to Linux](https://www.linuxleo.com/Docs/LinuxLEO_4.98.pdf)
