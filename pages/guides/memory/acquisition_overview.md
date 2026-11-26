---
title: Memory acquisition - Overview
summary: 'The volatile memory should be acquired as early as possible, with the acquisition utility preferably executed from, and the memory image written to, an external drive or a network share. The memory of virtual machines should preferably be acquired from the hypervisor.'
keywords: memory, RAM, volatile memory, memory dump, memory image, memory acquisition, remediation, footprint, external drive, network share, virtual machine, hypervisor, memory snapshot, Windows, Linux
tags:
  - memory
  - guides
last_updated: 2026-11-25
sidebar: sidebar
permalink: memory_acquisition_overview.html
folder: memory
---

The volatile memory should be acquired as early as possible, before any
remediation action and before the system is shut down, as the memory is
altered by every action taken on the system and lost upon shutdown. To limit
the footprint on the system, the acquisition utility should preferably be
executed from, and the memory image written to, an external drive or a network
share.

The memory of virtual machines should preferably be acquired from the
hypervisor (through a memory snapshot for instance), which does not require
executing any utility in the virtual machine and thus limits the alteration of
its memory.

The acquisition guides are detailed in the following notes:

  - [Windows](./acquisition_windows.md): `WinPmem`, `DumpIt`, and
    memory-related files on disk (`pagefile.sys`, `swapfile.sys`, and
    `hiberfil.sys`).

  - [Linux](./acquisition_linux.md): `AVML` and `LiME`.

  - [Virtual machines](./acquisition_virtualmachines.md):
    `VMware vSphere` / `ESXi` and `Microsoft Hyper-V` memory snapshots.

The analysis of the acquired memory images is covered in the
[memory analysis overview](./analysis_overview.md).
