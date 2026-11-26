---
title: Image mounting - Windows
summary: 'Image mounting on Windows, read-only or with temporary writes, with Arsenal Image Mounter (raw, E01, AFF, VDI, VMDK, and VHD images, and BitLocker-protected partitions), FTK Imager, or OSFMount.'
keywords: image, disk, mount, image mounting, Windows, read-only, temporary writes, Autoruns, Arsenal Image Mounter, BitLocker, raw, DD, E01, EnCase, AFF, VDI, VMDK, VHD, FTK Imager, OSFMount
tags:
  - guides
last_updated: 2025-09-01
sidebar: sidebar
permalink: image_mounting_windows.html
folder: image
---

Following the imaging of a system disk, the image taken must be mounted as a
partition for analysis. Numerous tools can be used, from either the Windows or
[Linux](./mounting_linux.md) operating systems, to do so. An important aspect
of image mounting is the preservation of the artefacts' integrity. The image
should be mounted read-only (or with temporary writes), with a few specific
options to preserve timestamps and data integrity.

Some utilities, such as `Autoruns`, require writable partitions. To use such
utilities, the image should be mounted using a utility supporting temporary
writes.

The [`Arsenal Image Mounter`](https://arsenalrecon.com/downloads/) is a
powerful graphical utility that can be used to mount multiple image types. It
supports temporary writes using a diff file that will store the modifications.
`Arsenal Image Mounter` will automatically mount the different partitions of
a given image and supports the decryption of `BitLocker`-protected partitions.

`Arsenal Image Mounter` supports the following disk image formats:
  - `raw` / `DD`
  - Multi-part `raw`
  - `EnCase Evidence File (E01)`
  - `Advanced Forensics Format (AFF)`
  - `VDI` / `VMDK` / `VHD`

Other utilities, such as `FTK Imager` or `OSFMount`, may be used as well.
