---
title: Recycle Bin
summary: 'Deleted files and folders (if deleted through a recycle bin aware application), associated with a given user (by their SID).\n\nTwo kind of files:\n $I, which contain the path and timestamp of deletion of the original file.\n$R, which contain the original file content.'
keywords: Recycle Bin, $Recycle.Bin, $I, $R
tags:
  - windows_file_knowledge
location: '$I, $R files under <SYSTEMDRIVE>:\$Recycle.Bin\<USER_SID>\'
last_updated: 2024-01-01
sidebar: sidebar
permalink: windows_recycle_bin.html
folder: windows
---

### Overview

Deleted files and folders (if deleted through a recycle bin aware application).

The deleted files are placed in a subfolder (under
`%SystemDrive%:\$Recycle.Bin`) named after the `SID` of the user that performed
the deletion. Deleted files can thus be associated with a given user.

### Information of interest

Two kind of files are present in the `Recycle Bin`:

  - `$I` (for "Information") files, which contain the path and timestamp of
    deletion of the original file.

 - `$R` (for "Resource") files, which contain the original file content.

### Tool(s)

  - [RBCmd](https://github.com/EricZimmerman/RBCmd)

### References

  - [AT&T Cybersecurity - Kushalveer Singh Bachchas - Digital dumpster diving: Exploring the intricacies of recycle bin forensics](https://cybersecurity.att.com/blogs/security-essentials/digital-dumpster-diving-exploring-the-intricacies-of-recycle-bin-forensics)

  - [13Cubed - Recycle Bin Forensics](https://www.youtube.com/watch?v=Gkir-wGqG2c)