---
title: Image acquisition - Physical devices
summary: 'Image acquisition of physical devices, through a write blocker or by booting a forensic Linux distribution (CAINE or Kali Linux in forensics mode), with FTK Imager (Windows), and dd, dc3dd, and Guymager (Linux).'
keywords: image, disk, image acquisition, physical device, raw, img, Advanced Forensics Format, AFF, EnCase, E01, EWF, write blocker, TX1, Tableau, CAINE, Kali Linux, forensics mode, FTK Imager, block device, SATA, NVMe, MBR, GPT, lsblk, fdisk, gdisk, dd, dc3dd, Guymager, sha256sum, hash
tags:
  - guides
last_updated: 2025-09-01
sidebar: sidebar
permalink: image_acquisition_physical_devices.html
folder: image
---

For physical drives, the drive to be imaged should be extracted from the system
and plugged into an acquisition station through an (ideally hardware) write
blocker. Dedicated hardware, such as the `TX1 Tableau Forensic Imager`, can
also be used to directly make a drive-to-drive or drive-to-file (such as a
`raw` file) copy.

If the disk to image cannot be extracted from the system, a bootable USB drive
can be used to boot into a temporary OS (if the BIOS boot order of the target
system can be changed). A Linux distribution suitable for forensic imaging
should be used, such as the [`CAINE`](https://www.caine-live.net/) distribution
(based on `Ubuntu`) or
[`Kali Linux in Forensics Mode`](https://www.kali.org/docs/general-use/kali-linux-forensics-mode/).
In such distributions, the devices are protected against accidental writes and
a number of forensic tools are installed for acquisition: `CAINE` sets all
block devices to read-only by default (the `UnBlock` and `Mounter` utilities
allowing to make a device writable), while `Kali Linux` in forensics mode
never uses the internal disks (including swap partitions) and disables the
auto-mounting of the internal disks and removable media (without, however,
setting the devices to read-only).

## Using a Windows OS

Windows should only be used as an acquisition platform if the drive to image can
be connected through a hardware write blocker, as Windows will automatically
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

## Using a Linux OS

### Devices overview

Linux storage devices, such as hard disks or SSDs, are typically block
devices represented as files under `/dev`. Block devices can be divided into
one or more logical disks called partitions. This partitioning is recorded in
the partition table, such as `MBR` or `GPT`, found at the beginning of the
disk: the `MBR` partition table is located in sector 0 (`LBA 0`), while the
`GPT` header is located in sector 1 (`LBA 1`), with sector 0 containing a
protective `MBR` and a backup `GPT` header being located in the last sector of
the disk.

| Type | Block device | Partition(s), if any |
|------|--------------|-----------------------|
| SATA drives | Each drive is represented as `/dev/sdx`. <br><br> Example of two drives: <br> `/dev/sda` <br> `/dev/sdc` | Each partition is represented as `/dev/sdxN`. <br><br> Example of two partitions on the same drive: <br> `/dev/sda1` <br> `/dev/sda2` |
| NVMe drives | Each drive is represented as `/dev/nvmeXn1`. <br><br> Examples: <br> `/dev/nvme0n1` <br> `/dev/nvme1n1` | Each partition is represented as `/dev/nvmeXnYpZ`. <br><br> Examples: <br> `/dev/nvme0n1p1` <br> `/dev/nvme0n1p2` |
| Optical media (CD or DVD drives) | `/dev/srx` <br><br> Example of two CD / DVD drives: <br> `/dev/sr0` <br> `/dev/sr1` | NA |

The `lsblk` utility can be used to list the block devices present on a system,
and the `fdisk` or `gdisk` (for `GPT` handling) utilities to retrieve more
information on a specific device and its partitions.

```bash
# Lists the block devices present on the system.
lsblk

# Retrieves information on the specified block device and its partitions (if any).
# <DEVICE> example: /dev/sda
fdisk -l <DEVICE>
gdisk -l <DEVICE>
```

### Imaging using dd

`dd` is the standard copy utility that, while not forensics-oriented, can be
used to create an image of a device / partition. `dd` has the advantage of
being available in most Linux distributions. A more forensically sound utility,
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

### Imaging using dc3dd

`dc3dd` is a more forensics-oriented utility, based on `dd`, to make disk or
partition images. `dc3dd` automatically detects bad sectors, natively integrates
hashing of the input and output bytes, and integrates logging features, making
it more suitable for forensic imaging.

```bash
# log option: output log file.
# hlog option: output file that will contain the image hash(es).

dc3dd if=<DEVICE | PARTITION> hof=/dest_media/image.raw log=<LOG_FILE> hash=sha256 hlog=<HASH_FILE>
```

### Imaging using Guymager

[`Guymager`](https://guymager.sourceforge.io/) is a GUI forensic imager that
can produce image files in raw, EWF, and AFF formats. `Guymager` is
multi-threaded for faster collection and integrates hash calculation.

```
Right-click the device -> Acquire image
  -> Format selection (raw, EWF, AFF)
  -> Evidence collection metadata
  -> Selection of the image destination folder and name
  -> Calculate SHA-256
```

--------------------------------------------------------------------------------

## References

  - [Barry J. Grundy - The Law Enforcement and Forensic Examiner’s Introduction to Linux](https://www.linuxleo.com/Docs/LinuxLEO_4.98.pdf)

  - [BlueMonkey 4n6 - CAINE 02 forensic imaging and cloning using Guymager](https://www.youtube.com/watch?v=FoEO9p-J15w)
