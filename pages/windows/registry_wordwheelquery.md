---
title: Windows - Registry - WordWheelQuery
summary: 'Introduced in Windows 7, and not present in Windows Server operating systems, the WordWheelQuery registry key tracks the keywords searched in the Windows Explorer search box.\n\nInformation of interest: terms/keywords entered in the Windows Explorer search box.\n\nValues are ordered in a Most Recently Used (MRU) list. The search timestamp of the most recently searched item can thus be deduced from the last write timestamp of the registry key.'
keywords: 'WordWheelQuery, MRU'
tags:
  - windows_registry
  - windows_files_and_folders_access
location: 'File: <SYSTEMDRIVE>:\Users\<USERNAME>\NTUSER.dat\n\nRegistry key:\nHKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\WordWheelQuery'
last_updated: 2024-01-07
sidebar: sidebar
permalink: windows_registry_wordwheelquery.html
folder: windows
---

### Overview

Introduced in `Windows 7`, and not present in `Windows Server` operating
systems, the `WordWheelQuery` registry key tracks the keywords searched in
the `Windows Explorer` search box, potentially resulting in access to files or
folders.

### Information of interest

Each term/keyword entered is stored in a dedicated value under the
`Explorer\WordWheelQuery` key (as a Unicode string).

The values are ordered in a `Most Recently Used (MRU)` list, with the most
recently entered term having the position of `0`.

The last write timestamp of the key indicates the timestamp of the most
recently entered term.

### References

  - [The 4N6 Post - Registry - WordWheelQuery](https://www.4n6post.com/2023/02/registry-wordwheelquery.html)
