---
title: Registry - Map Network Drive MRU
summary: 'The Map Network Drive MRU registry key references the recently used network shares.\n\nInformation of interest: UNC path of the network shares (such as "<IP | HOSTNAME>\<SHARE_NAME>").\n\nValues are ordered in a most recently used list. The timestamp of access of the most recently access share can thus be deduced from the last write timestamp of the registry key.'
keywords: 'Map Network Drive MRU, Network share, MRUList'
tags:
  - windows_registry
  - windows_system_information
  - windows_lateral_movement
  - windows_lateral_movement_src
location: 'File:\n<SYSTEMDRIVE>:\Users\<USERNAME>\NTUSER.dat.\n\nRegistry key:\nHKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Map Network Drive MRU'
last_updated: 2024-01-13
sidebar: sidebar
permalink: windows_registry_mapnetworkdrivemru.html
folder: windows
---

### Overview

The `Map Network Drive MRU` registry key references the network shares recently
used by the associated user. The network share do not have to be persistently
mapped to be referenced by the `Map Network Drive MRU` key.

### Information of interest

Each network share is stored in a dedicated value under the
`Map Network Drive MRU` key. The value data contains the `UNC` path of the
network share (such as `<IP | HOSTNAME>\<SHARE_NAME>`).

The values are ordered in a `Most recently used (MRU)` list, specified in the
`MRUList` value. Example: `MRUList` equals to `ba` means that the entry tagged
as `b` was accessed last / the most recently, preceded by the entry tagged as
`a`.

The last write timestamp of the key indicates the timestamp of the most
recently accessed share.

### References

  - [Artifast - Investigating Windows Mapped Network Drives](https://forensafe.com/blogs/mappednetworkdrive.html)
