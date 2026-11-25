---
title: Pagefile.sys and swapfile.sys
summary: 'The pagefile.sys and swapfile.sys files are used by Windows for memory paging, to store memory pages from the main memory (RAM) on disk. The pagefile.sys file is the system-wide page file, while the swapfile.sys file is used to suspend and resume Universal Windows Platform (UWP) applications.\n\nAs the files only store unstructured / unordered memory pages, their analysis on their own is limited to carving and strings extraction. MemProcFS and Volatility 3 can however use the page files, acquired at the same time as the memory image, to complement the analysis of the memory image.\n\nThe page file is cleared upon system shutdown if the ClearPageFileAtShutdown setting is enabled.'
keywords: pagefile.sys, swapfile.sys, page file, paging file, swap, virtual memory, memory, PagingFiles, ClearPageFileAtShutdown, EncryptPagingFile, memory compression, carving, strings, bstrings, bulk_extractor, YARA, MemProcFS, Volatility, single-swap-locations
tags:
  - memory
  - windows_misc
location: 'pagefile.sys (default location, configurable):\n<SYSTEMDRIVE>:\pagefile.sys\n\nswapfile.sys:\n<SYSTEMDRIVE>:\swapfile.sys\n\nPage file(s) configuration:\nHKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management\PagingFiles'
last_updated: 2026-11-25
sidebar: sidebar
permalink: windows_pagefilesys.html
folder: windows
---

### Overview

The `pagefile.sys` and `swapfile.sys` files are used by the Windows operating
system for memory paging, i.e., to store memory pages from the main memory
(RAM) to disk and retrieve them as needed, which allows the system to extend
the total amount of memory available. Less frequently used memory pages are
swapped to disk, and loaded back into the main memory on page fault events.

  - `pagefile.sys` is the system-wide page file, which stores memory pages
    from the whole main memory. The page file is located by default at the
    root of the system drive, but its location (possibly on another volume, or
    with multiple page files on different volumes) and size can be configured.
    The configuration of the page file(s) is stored in the `PagingFiles` value
    of the
    `HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management`
    registry key.

  - `swapfile.sys`, introduced in `Windows 8`, is used to suspend and resume
    `Universal Windows Platform (UWP)` applications (usually from the
    `Microsoft Store`), and thus only stores memory pages of these
    applications.

Starting with `Windows 10`, memory pages can be compressed and kept in memory
(memory compression feature) instead of being written to the page file. As
the pages holding the compressed memory can themselves be paged out, the
`pagefile.sys` file may contain compressed memory pages, whose content is not
directly visible to carving or strings extraction.

If the `ClearPageFileAtShutdown` registry setting
(`Shutdown: Clear virtual memory pagefile` policy) is enabled (set to `0x1`),
the page file is cleared upon system shutdown. The page file can also be
encrypted (setting that can be checked with
`fsutil behavior query EncryptPagingFile`), using a key that is not persisted
across reboots, which prevents its offline analysis.

As the `pagefile.sys` and `swapfile.sys` files are locked on a running system,
they must be collected from a disk image or using tools that access the raw
volume (such as `FTK Imager` or `KAPE`). To complement the analysis of a
memory image, the files should be collected at the same time (or very close in
time) as the memory image (refer to the
[Windows memory acquisition note](../memory/acquisition_windows.md)).

### Tool(s)

As `pagefile.sys` and `swapfile.sys` only store unstructured / unordered
memory pages, the files cannot be analyzed on their own using memory analysis
tools such as `Volatility` or `MemProcFS`. Both
[`MemProcFS`](../memory/analysis_memprocfs.md) and
[`Volatility 3`](../memory/analysis_volatility.md) can however use the page
files, if acquired at the same time (or very close in time) as the memory
image, to complement the analysis of the memory image.

On their own, the analysis of the `pagefile.sys` and `swapfile.sys` files is
thus limited to carving and strings extraction. As memory pages are typically
4 KB in size, only files of less than 4 KB can reliably be fully carved out.
Scanning the `pagefile.sys` and `swapfile.sys` files for known malware
indicators, with `YARA` rules for example, may result in false positives, as
such indicators may be present in pages swapped from security products.

Tools such as `strings` / [`bstrings`](https://ericzimmerman.github.io/) or
[`bulk_extractor`](https://github.com/simsong/bulk_extractor) can be used to
extract strings such as URLs, IP addresses, or email addresses, or files (for
`bulk_extractor`).

```bash
# Extracts strings (URLs, email addresses, etc.) and carves files.
bulk_extractor -o <OUTPUT_DIRECTORY> <PAGEFILE | SWAPFILE>

# Extracts the ASCII and UTF-16LE strings.
strings -a <PAGEFILE | SWAPFILE> > <STRINGS_FILE>
strings -a -e l <PAGEFILE | SWAPFILE> >> <STRINGS_FILE>

# Analysis of a memory image complemented with the page files.
memprocfs.exe -device <MEMORY_DUMP_FILE> -pagefile0 pagefile.sys -pagefile1 swapfile.sys
vol --single-swap-locations <PAGEFILE_SYS> -f <MEMORY_DUMP_FILE> <PLUGIN>
```

### References

  - [Microsoft - Shutdown: Clear virtual memory pagefile](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/security-policy-settings/shutdown-clear-virtual-memory-pagefile)

  - [Ulf Frisk - MemProcFS wiki](https://github.com/ufrisk/MemProcFS/wiki)

  - [Volatility Foundation - Volatility 3 documentation - volatility manual page](https://volatility3.readthedocs.io/en/latest/vol-cli.html)
