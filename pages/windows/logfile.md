---
title: NTFS $LogFile
summary: 'The $LogFile is part of a journaling feature of NTFS, activated by default, which maintains a low-level record of changes made to the NTFS volume with very limited
historical data (usually only of the last few hours).'
keywords:
tags:
  - windows_ntfs_filesystem
  - windows_file_knowledge
location: '<ROOT>\$LogFile'
last_updated: 2024-01-01
sidebar: sidebar
permalink: windows_logfile.html
folder: windows
---

### Overview

The `$LogFile` is part of a journaling feature of `NTFS`, activated by default,
which maintains a low-level record of changes made to the `NTFS` volume.

Every disk operation is journalized prior to being committed. In case of
failure, such as a crash during an update, the `$LogFile` can be used to
revert disk operations.

### Information of interest

As low-level operations are journalized, the `$LogFile` contains very limited
historical data, usually only of the last few hours at most.

### References

  - [13Cubed - Introduction to MFTECmd - NTFS MFT and Journal Forensics](https://www.youtube.com/watch?v=_qElVZJqlGY)
