---
title: Memory acquisition - Windows
summary: 'Memory acquisition of Windows systems with WinPmem and DumpIt, and memory-related files on disk (pagefile.sys, swapfile.sys, and hiberfil.sys).'
keywords: memory, RAM, volatile memory, memory dump, memory image, memory acquisition, raw, crash dump, Windows, WinPmem, AFF4, DumpIt, Magnet, pagefile.sys, swapfile.sys, hiberfil.sys
tags:
  - memory
  - guides
last_updated: 2026-11-25
sidebar: sidebar
permalink: memory_acquisition_windows.html
folder: memory
---

## WinPmem

[`WinPmem`](https://github.com/Velocidex/WinPmem) is an open-source utility
that can be used to conduct a local capture of memory. `WinPmem` implements
three independent acquisition methods, the default method being the `PTE`
remapping on 64-bit systems and the `\Device\PhysicalMemory` device on 32-bit
systems.

`WinPmem` used to produce images in the
`Advanced Forensics File Format 4 (AFF4)` format (which includes metadata
about the capture, compression of the output, etc.), but current versions
only produce images in the raw format. The older versions are available in the
[`c-aff4`](https://github.com/Velocidex/c-aff4) repository.

```bash
# Depending on the release, the executable is named winpmem_mini_x64.exe / winpmem_mini_x86.exe or winpmem64.exe / winpmem32.exe.
winpmem_mini_x64.exe <OUTPUT_RAW_DUMP>
winpmem_mini_x64.exe \\<IP | HOSTNAME>\<SHARE>\<OUTPUT_RAW_DUMP>

# --- Older (AFF4) versions
# -p <PAGEFILE_PATH>: instructs WinPmem to also collect the page file.

# Retrieves the page file path (wmic being deprecated on recent Windows versions).
wmic pagefile list
Get-CimInstance -ClassName Win32_PageFileUsage | Select-Object Name

winpmem.exe -p <PAGEFILE_PATH> -o <OUTPUT_DUMP_AFF4>
```

## DumpIt

`DumpIt` (now
[`Magnet DumpIt for Windows`](https://www.magnetforensics.com/resources/magnet-dumpit-for-windows/),
following the acquisition of `Comae` by `Magnet Forensics`) is a reliable
utility that can be used to conduct a local capture of memory on Windows
systems (`x86`, `x64`, and `ARM64`). `DumpIt` generates full memory Microsoft
crash dumps, which are supported by most memory analysis tools (`WinDbg`,
`Volatility`, `MemProcFS`, etc.).

Depending on the version used, different options are implemented. In a basic
use case, `DumpIt` can simply be executed without any argument to create a
memory dump in the current folder.

```
DumpIt.exe
```

## Memory files on disk

### pagefile.sys and swapfile.sys

The `pagefile.sys` and `swapfile.sys` files, used by Windows for memory
paging, contain memory pages swapped from the main memory to disk. While the
files cannot be analyzed on their own as memory images, they can be collected
at the same time as the memory image to complement its analysis (with
`MemProcFS` or `Volatility 3`). Refer to the
[`pagefile.sys` and `swapfile.sys` note](../windows/pagefile.sys.md) for more
information on the files and their analysis.

### hiberfil.sys

The `hiberfil.sys` file, linked to the hibernation, hybrid sleep, and
`Fast Startup` features, can contain a full or partial snapshot of the memory
of the system. Refer to the [`hiberfil.sys` note](../windows/hiberfil.sys.md)
for more information on the file and its conversion to a raw memory image.

`MemProcFS` natively supports `hiberfil.sys` files (from `Windows 10` and
later) as input. As of November 2026, the `windows.hibernation` plugins of
`Volatility 3` are only available in a
[pull request](https://github.com/volatilityfoundation/volatility3/pull/1036)
that is not yet merged.

--------------------------------------------------------------------------------

## References

  - [FORENSICXLAB - Volatility3: Modern Windows Hibernation file analysis](https://www.forensicxlab.com/posts/hibernation/)

  - [Magnet Forensics - Magnet DumpIt for Windows](https://www.magnetforensics.com/resources/magnet-dumpit-for-windows/)

  - [Ulf Frisk - MemProcFS wiki](https://github.com/ufrisk/MemProcFS/wiki)

  - [Velocidex - WinPmem](https://github.com/Velocidex/WinPmem)
