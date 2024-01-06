---
title: Registry - RecentApps
summary: 'Introduced in Windows 10 1607 and removed in Windows 10 1709 (with the key not present on subsequent versions), the RecentApps is an undocumented registry key that tracks program executions and files accessed by the tracked programs.\n\nInformation of interest: filename, last access timestamp, and run count execution of the application.\n\nAdditionally, 10 files accessed by the application (not necessarily the last files accessed) are tracked. For each file, the file name and file full path are referenced and the last access timestamp can be deduced (from the last write timestamp of the associated registry key).'
keywords: 'RecentApps'
tags:
  - windows_program_execution
  - windows_files_and_folders_access
  - windows_registry
location: 'File: <SYSTEMDRIVE>:\Users\<USERNAME>\NTUSER.dat\n\nRegistry key:\nHKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Search\RecentApps\<GUID>'
last_updated: 2024-01-06
sidebar: sidebar
permalink: windows_registry_recentapps.html
folder: windows
---

### Overview

**Introduced in `Windows 10 1607` and removed in `Windows 10 1709`** (with the
key not present on subsequent versions), the `RecentApps` is an undocumented
registry key that **tracks program executions and files accessed by the tracked
programs**.

### Information of interest

Each subkey, identified with a GUID, under the `RecentApps` key correspond to
an executed program. In **these application GUID subkeys, the filename, last
access timestamp, and run count of the application** are stored as values.

Additionally, **each application GUID subkey can have up to 10 subkeys**, also
identified with a GUID, that correspond to **files accessed using the
application**. In these file GUID subkeys, **the file name, file full path, and
(on some OS version) a non-updated timestamp of last access** are stored as
values. The cycling order of the file subkeys [appears to be based on the
alphabetical order of the GUID and not on a first-in-first-out basis](https://df-stream.com/2017/10/recentapps/).
If a new file is accessed while 10 files are already referenced, the subkeys
are sorted alphabetically and the last one is removed in place of the new
entry. Thus the current files referenced are not necessarily the last 10 files
accessed.

The **last write timestamp of an application subkey can indicate when the
program was last executed**. While the **last write timestamp of a file subkey
can indicate when the file was accessed** (by the associated program).

### References

  - [Digital Forensics Stream - JASON HALE - RecentApps Registry Key](https://df-stream.com/2017/10/recentapps/)

  - [ThinkDFIR - PHILL MOORE - When did RecentApps go?](https://thinkdfir.com/2020/10/23/when-did-recentapps-go/)
