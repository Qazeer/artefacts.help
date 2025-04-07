---
title: Registry - Background Activity Moderator (BAM) and Desktop Activity Moderator (DAM)
summary: 'Introduced in Windows 10 Fall Creators update - version 1709, the Background Activity Moderator (BAM) is a mostly undocumented feature that controls the programs executed in the background.The Desktop Activity Moderator (DAM) is a feature for mobile devices, that support the "Connected Standby" mode (and thus hold no data on Windows desktop or server).\n\nIf a file is deleted, the eventual associated entry in the BAM is deleted as well after the system reboot. Additionally, BAM entries older than 7 days are deleted upon system boot.\n\nInformation of interest: program full path, timestamp of execution, and executing user (as the values are grouped by user SID).'
keywords: 'Background Activity Moderator, BAM, Desktop Activity Moderator, DAM'
tags:
  - windows_registry
  - windows_program_execution
location: 'File: <SYSTEMROOT>\System32\config\SYSTEM\n\nRegistry key:\nHKLM\SYSTEM\CurrentControlSet\Services\bam\UserSettings\<SID>\*\nHKLM\SYSTEM\CurrentControlSet\Services\dam\UserSettings\<SID>\*\n\nStarting from Windows 10 1809:\n HKLM\SYSTEM\CurrentControlSet\Services\bam\State\UserSettings\<SID>\*\nHKLM\SYSTEM\CurrentControlSet\Services\dam\State\UserSettings\<SID>\*'
last_updated: 2024-01-05
sidebar: sidebar
permalink: windows_registry_bam_dam.html
folder: windows
---

### Overview

Introduced in `Windows 10`'s Fall Creators update - version 1709, the
`Background Activity Moderator (BAM)` is a mostly undocumented feature that
controls the programs executed in the background. The
`Desktop Activity Moderator (DAM)` is a feature for devices supporting the
"Connected Standby" mode (i.e. when a device is turned on, but its display will
be turned off). As a result, the `BAM` registry keys will contain data on any
devices, while `DAM` registry keys will only contain data on mobile devices.

### Information of interest

The `BAM` registry key contains multiple subkeys under
`bam\State\UserSettings`, with one subkey per user, identified with the user
`SID`. While the key is in the `SYSTEM` registry hive, program executions can
thus still be tied to a specific user using this `SID`.

Each user-specific key contains a list of executed programs, with one value per
program. The value name is the program full path and the value data is the
timestamp of last execution.

If a file is deleted, the eventual associated entry in the `BAM` is deleted as
well after the system reboot. Additionally, `BAM` entries older than 7 days are
deleted upon system boot. The `BAM` thus provides limited information on
historic execution of programs.

No entries are created in the `BAM` keys for executables on removable media
and/or on network shares.

### References

  - [dfir.ru - MSUHANOV - BAM internals](https://dfir.ru/2020/04/08/bam-internals/)
