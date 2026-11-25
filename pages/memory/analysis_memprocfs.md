---
title: Memory analysis - MemProcFS/MemNixFS
summary: 'Memory analysis with MemProcFS (Windows) and MemNixFS (Linux), which present the memory of a system as files in a virtual file system.'
keywords: memory, memory analysis, MemProcFS, memprocfs.exe, Dokany, FUSE, forensic mode, FindEvil, timeline, YARA, Elastic, memory.pmem, memory.dmp, pagefile, MemProcFS-Analyzer, MemNixFS, memnixfs, WinFsp, BTF, ISF
tags:
  - memory
  - procedures
last_updated: 2026-11-25
sidebar: sidebar
permalink: memory_analysis_memprocfs.html
folder: memory
---

## MemProcFS

[`MemProcFS`](https://github.com/ufrisk/MemProcFS) is a memory analysis tool
that presents the memory of a Windows system as files in a virtual file
system, mounted as `M:` by default on Windows (using the `Dokany` file system
library) or using `FUSE` on Linux and macOS. The memory artefacts, such as the
processes and their modules, handles, and memory, the network connections, the
registry, the services, or the scheduled tasks, can thus be browsed and
analyzed with any tool. At the root of the virtual file system, the
`memory.pmem` file exposes the raw physical memory and the `memory.dmp` file a
`WinDbg`-compatible full crash dump.

`MemProcFS` supports raw memory images, full Microsoft crash dumps, full `ELF`
core dumps, `LiME` dumps, `VMware` memory files (`vmem`, with the associated
`vmss` / `vmsn` file, or `vmsn` file with the memory included), `Hyper-V`
saved state files, and Windows hibernation files, as well as live memory
(through `DumpIt` or `WinPmem` notably). `MemProcFS` can be executed on
Windows, Linux, and macOS, but only analyzes Windows memory images.

`MemProcFS` can also use the Windows page files (`pagefile.sys` and
`swapfile.sys`), acquired at the same time as the memory image, to complement
the analysis (refer to the
[`pagefile.sys` and `swapfile.sys` note](../windows/pagefile.sys.md)).

The `forensic` mode of `MemProcFS` performs a more thorough analysis of the
memory image, producing timelines and CSV files (in `M:\forensic\timeline`
and `M:\forensic\csv`), and the `FindEvil` detections of suspicious elements
(process injection, suspicious parent / child relationships, etc.) in
`M:\forensic\findevil\findevil.txt`. The `FindEvil` detections can be
complemented with `YARA` rules, built-in (from `Elastic Security`, enabled by
accepting the associated license) or user-specified.

```bash
# Mounts the memory image as M: (Windows, requires Dokany).
memprocfs.exe -device <MEMORY_DUMP_FILE>

# Mounts the memory image with the forensic mode enabled.
# -forensic 1: forensic mode with an in-memory SQLite database (2 to 4 for an on-disk database, either temporary or kept).
# -license-accept-elastic-license-2-0: enables the built-in Elastic Security YARA rules for FindEvil.
memprocfs.exe -device <MEMORY_DUMP_FILE> -forensic 1 [-license-accept-elastic-license-2-0] [-forensic-yara-rules <YARA_RULES_FILE>]

# Complements the analysis with the page files, acquired at the same time as the memory image.
memprocfs.exe -device <MEMORY_DUMP_FILE> -pagefile0 pagefile.sys -pagefile1 swapfile.sys

# For VMware memory files, the vmss / vmsn file must be placed alongside the vmem file, with the same base name.
memprocfs.exe -device <VM_NAME>-Snapshot<N>.vmem

# Mounts the memory image on Linux (requires FUSE).
./memprocfs -device <MEMORY_DUMP_FILE> -mount <MOUNT_POINT>
```

The third-party
[`MemProcFS-Analyzer`](https://github.com/LETHAL-FORENSICS/MemProcFS-Analyzer)
`PowerShell` script can be used to automate the analysis of memory images with
`MemProcFS` (and other tools).

## MemNixFS

[`MemNixFS`](https://github.com/MemNixFS/MemNixFS) is an independent tool,
inspired by `MemProcFS` and released in 2026, that presents the memory of a
Linux system as a file system. The mounted file system notably contains the
processes information (`proc`), system-wide information such as the shell
history, kernel modules, and network connections, as well as `FindEvil`
detections (`sys`), the file system reconstructed from the page cache (`fs`),
a timeline (`forensic`), `YARA`, IOCs, and strings searches (`search`), and
the raw physical memory (`mem/phys.raw`).

`MemNixFS` supports `AVML`, `LiME`, raw, and `kdump` / `vmcore` memory images
of `x86-64` Linux systems, and can be executed on Windows (mounting requiring
`WinFsp`) and Linux (mounting requiring `FUSE`). `MemNixFS` does not
necessarily require a symbol table (`ISF`) matching the kernel of the analyzed
system: if no matching `ISF` is found (locally, or downloaded with the
`--auto-fetch` option), `MemNixFS` generates the required structure
definitions from the `BPF Type Format (BTF)` information embedded in recent
kernels.

```bash
# Lists the processes (no symbol table required).
memnixfs --dump <MEMORY_DUMP_FILE> list

# Mounts the memory image (as M: on Windows, or on an empty directory on Linux).
# --auto-fetch: automatically downloads the debug symbols of the kernel (for the main distributions).
# --symbols <ISF_FILE | ISF_DIR> / --vmlinux <VMLINUX>: uses the specified symbol table / generates the symbol table from the specified vmlinux file.
# --forensic[=<quick | smart | full>]: pre-computes in the background the files that are expensive to generate, so that they open instantly.
memnixfs --dump <MEMORY_DUMP_FILE> [--auto-fetch | --symbols <ISF_FILE | ISF_DIR> | --vmlinux <VMLINUX>] [--forensic] mount <M: | MOUNT_POINT>

# Reads a single file, or exports the files of the virtual file system to a folder, without mounting (and thus without requiring WinFsp / FUSE).
memnixfs --dump <MEMORY_DUMP_FILE> cat /sys/findevil/findevil.txt
memnixfs --dump <MEMORY_DUMP_FILE> export <OUTPUT_DIR>

# Triage verdict and timeline.
cat M:/sys/findevil/triage.txt
cat M:/forensic/timeline.txt
```

--------------------------------------------------------------------------------

## References

  - [MemNixFS - MemNixFS](https://github.com/MemNixFS/MemNixFS)

  - [Ulf Frisk - MemProcFS wiki](https://github.com/ufrisk/MemProcFS/wiki)
