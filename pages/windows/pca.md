---
title: Program Compatibility Assistant (PCA)
summary: 'Introduced in Windows 11, the Program Compatibility Assistant (PCA) is an application compatibility feature that aim to maintain support of existing desktop applications to new versions of the Windows operating system.\n\nInformation of interest, only for programs executed as GUI: file full path and timestamp of execution. More information available for executions resulting in non 0x0 exit code.'
keywords: Program Compatibility Assistant, PCA, PcaAppLaunchDic, nPcaGeneralDb0, nPcaGeneralDb1
tags:
  - windows_program_execution
location: 'Files under <SYSTEMROOT>\appcompat\pca\:\nPcaAppLaunchDic.txt\nPcaGeneralDb0.txt\nPcaGeneralDb1.txt'
last_updated: 2024-01-01
sidebar: sidebar
permalink: windows_pca.html
folder: windows
---

### Overview

Introduced in Windows 11, the `Program Compatibility Assistant (PCA)` is an
application compatibility feature that aim to maintain support of existing
desktop applications to new versions of the Windows operating system (like the
`Shimcache` and `Amcache` artefacts). `PCA` is linked to the `pcasvc` service.

`PCA` only hold information about executions of programs with a graphical
interface, installed or from a portable executable, or command line programs
executed as GUI programs (such as by double-clicking on the CLI executable
from `Windows Explorer`).

### Information of interest

The information stored by the `PCA` is split in 3 text based files:
  - `PcaAppLaunchDic.txt`:

    - Most valuable file from a forensic standpoint and reliable source of
      program execution.

    - One entry per line, containing the full path of the executable and the
      timestamp of execution in `UTC` (in a pipe separated string).

    - Example: `%SystemRoot%\FOLDER\executable.exe\|2023-05-25 01:20:30.123`.

  - `PcaGeneralDb0.txt` and `PcaGeneralDb1.txt`:

    - Fewer entries than in the `PcaAppLaunchDic.txt` file, with most entries
      seemingly related to non `0x0` execution exit code.

    - One entry per line, containing the following information in a pipe
      delimited string:
      - Execution timestamp.
      - Execution status.
      - Full path of the executable.
      - Description of the executable and its vendor name.
      - File version.
      - `ProgramId` referenced in the `Amcache` registry hive
       (`InventoryApplicationFile` key).
      - Exit code of the execution.

### References

  - [AboutDFIR.com - Andrew Rathbun and Lucas Gonzalez - New Windows 11 Pro (22H2) Evidence of Execution Artifact!](https://aboutdfir.com/new-windows-11-pro-22h2-evidence-of-execution-artifact/)

  - [13Cubed - A New Program Execution Artifact - Windows 11 22H2 Update!](https://www.youtube.com/watch?v=rV8aErDj06A)
