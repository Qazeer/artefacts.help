---
title: Registry - RunMRU
summary: 'The RunMRU registry tracks items launched from the Windows Run launcher (Windows + R shortcut).\n\nInformation of interest: values entered (program names, files / folders, URL, ...) in the Windows Run launcher, if associated with a successful launch.\n\nValues are ordered in a most recently used list. The timestamp of launch of the most recently launched item can thus be deduced from the last write timestamp of the registry key.'
keywords: 'RunMRU, MRU, MRUList'
tags:
  - windows_registry
  - windows_program_execution
  - windows_files_and_folders_access
location: 'File: <SYSTEMDRIVE>:\Users\<USERNAME>\NTUSER.dat\n\nRegistry key:\nHKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\RunMRU'
last_updated: 2024-01-07
sidebar: sidebar
permalink: windows_registry_runmru.html
folder: windows
---

### Overview

The `RunMRU` registry key tracks items (programs, files / folders, `URL`, ...)
launched from the `Windows Run` launcher (Windows + R shortcut) in a most
recently used list.

Entries are added / updated in near real-time.

### Information of interest

Each entry successfully launched trough the `Windows Run` launcher is stored in
a dedicated value under the `Explorer\RunMRU` key. Entry that were not found
by the operating system (error "Windows cannot find [...]") are not added in
the `RunMRU` registry key.

The values are ordered in a `Most recently used (MRU)` list, specified in the
`MRUList` value. Example: `MRUList` equals to `ba` means that the entry tagged
as `b` was launched last / the most recently, preceded by the entry tagged as
`a`.

The last write timestamp of the key indicates the timestamp of the most
recently launched item.

### References

  - [Magnet Forensics - What is MRU (Most Recently Used)?](https://www.magnetforensics.com/blog/what-is-mru-most-recently-used/)
