---
title: Prefetch
summary: 'Windows Prefetch is a performance enhancement feature that enables prefetching of applications to make system boots or application startups faster. Prefetch is disabled by default on Windows Server operating systems and is limited to 128 entries on Windows XP to Windows 7, and 1024 entries starting from Windows 8.\n\nPrefetch files are created whenever a program is executed from a specific path.\n\nInformation of interest: file name and size, first and, starting from Windows 8, last eight execution timestamps, run count, and list of files and directories accessed during the first ten seconds of execution (often including the full path of the executed file itself).'
keywords: Prefetch
tags:
  - windows_program_execution
location: '<SYSTEMROOT>\Prefetch\<EXECUTABLE.EXE>-<RANDOM_ID>.pf\n\nFilename example: POWERSHELL.EXE-022A1004.pf'
last_updated: 2024-01-01
sidebar: sidebar
permalink: windows_prefetch.html
folder: windows
---

### Overview

`Windows Prefetch` is a performance enhancement feature that enables
prefetching of applications to make system boots or application startups
faster. `Prefetch` files are created whenever a program is executed from a
specific path. If the same binary is executed from different locations,
separate `Prefetch` files will be created for each different location.
A `Prefetch` file can be created even if the executable did not successfully
run.

Whether the `Prefetch` feature is enabled is configured by the
`EnablePrefetcher` registry key:
  - `0`/undefined: disabled (default on Windows Server operating systems).
    **`Prefetch` is thus disabled by default on Windows Server operating
    systems**.
  - `0x1`: Partially enabled (application prefetching only).
  - `0x2`: Partially enabled (boot prefetching only).
  - `0x3`: Enabled (application and boot prefetching).

`Prefetch` files are not automatically deleted if the related executable is
deleted, and can thus be a source of historical information. However, as the
`Prefetch` directory is limited to 128 entries on `Windows XP` to `Windows 7`
and 1024 entries starting from `Windows 8`, Prefetch files may be overwritten
and information lost.

### Information of interest

The `Prefetch` filenames are based on the executed program name and a hash,
computed using a proprietary algorithm and based on the full path (and
for some binaries, such as `dllhost.exe` or `svchost.exe`, command line
parameters) of the executed program.

The `Prefetch` files can yield the following information of forensic interest:

  - The file name and size of the binary executed.

  - The first and, starting from Windows 8, last eight execution timestamps.

  - The `Prefetch` file `NTFS` created and last modified timestamps also
    indicate the first and last time the program was executed.

  - Run count (number of times the binary was executed).

  - List of files and directories accessed during the first ten seconds of
    execution (including any `DLLs` loaded).
    The full path to the executable file can often be determined from the list
    of files accessed (duplicates are possible if a given binary accesses
    another binary with the same name).

Note that the `Prefetch` files can be easily deleted, potentially invalidating
the trace of execution and timestamps (notably of first execution).

#### Prefetch files indirect information

The creation or modification of `Prefetch` files observed in other artefacts
(`$MFT`, `UsnJrnl`, etc.) reflects an execution of the binary linked to the
`Prefetch` file (and whose name can be deduced from the `Prefetch` filename).

#### Prefetch information related to PowerShell execution

The `POWERSHELL.EXE-[...].pf` Prefetch file may contain references to
recently executed PowerShell scripts. For an entry to be created in the
Prefetch file, the given script must be executed within the first ten seconds
of the `powershell.exe` execution.

The accessed file list does retain entries from previous instances of a program
execution. Accessed file information may thus persist through subsequent
`powershell.exe` runs.

### Tool(s)

[`PECmd`](https://github.com/EricZimmerman/PECmd) (associated `KAPE`
`PECmd` module) can be used to parse `Prefetch` file(s).

```bash
# Parses the specified Prefetch file.
PECmd.exe [-q --csv <CSV_DIRECTORY_OUTPUT>] -f <PF_FILE>

# Recursively retrieves and parses the Prefetch files in the specified directory.
PECmd.exe [-q --csv <CSV_DIRECTORY_OUTPUT>] -d <C:\Windows\Prefetch | C:\ | DIRECTORY>
```

### References

  - [Forensic Focus - Oleg Skulkin - Hunting For Attackers' Tactics And Techniques With Prefetch Files](https://www.forensicfocus.com/articles/hunting-for-attackers-tactics-and-techniques-with-prefetch-files/)

  - [13Cubed - Prefetch Deep Dive](https://www.youtube.com/watch?v=f4RAtR_3zcs)