---
title: Registry - MountPoints2
summary: 'The MountPoints2 registry key references the currently or previously mapped drives (such as the system drive, USB devices, or network shares) mounted by the associated user.\n\nInformation of interest: each drive is represented by a subkey, which is named as either the volume GUID, a letter, or, for network shares "##<IP | HOSTNAME>#<SHARE_NAME>".'
keywords: 'MountPoints2'
tags:
  - windows_registry
  - windows_system_information
  - windows_lateral_movement
  - windows_lateral_movement_src
  - windows_usb_activity
location: 'File:\n<SYSTEMDRIVE>:\Users\<USERNAME>\NTUSER.dat.\n\nRegistry key:\nHKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\MountPoints2'
last_updated: 2024-01-10
sidebar: sidebar
permalink: windows_registry_mountpoints2.html
folder: windows
---

### Overview

The `MountPoints2` registry key references the currently or previously mapped
drives (such as the system drive, USB devices, or network shares) mounted by
the associated user.

### Information of interest

Each drive is represented by a subkey, which is named as either the
`volume GUID`, a letter, or, for network shares, using a specific nomenclature
(`##<IP | HOSTNAME>#<SHARE_NAME>`).

For devices, the `volume GUID` can be used to retrieve more information on the
device from the `HKLM\SYSTEM\MountedDevices` registry key, including the
`device/hardware  ID` (vendor and product name) and `instance ID` (with the
`serial number` if existing).

**This key can be used to determine which user interacted with a given USB
device**. However entries are not reliably created, so **the absence of an
entry is not an indicator that the given user didn't interact with the
device**.
