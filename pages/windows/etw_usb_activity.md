---
title: ETW - Devices and USB activity
summary: 'For devices and USB activity.\n\nVarious events are generated for devices and USB activity, split across a number of channels. More events
and information are available on recent versions of the Windows operating system.\n\nUsing known variables about a given device, found for example in the Windows registry, events can be used to determine timestamps of activity for the device, such as when the device was first plugged, last plugged and unplugged.\n\nAdditionally, supplementary information about devices can be retrieved from events, such as device storage sizes and an extract of their partition table.'
keywords: 'USB, Devices, Microsoft-Windows-Storage-ClassPnP/Operational, 507, 500, 502, 503, 504, 505, 506, 510, Microsoft-Windows-Kernel-PnP/Device Configuration, 400, 401, 410, 411, 420, 430, Microsoft-Windows-Kernel-PnP/Device Management, 1010, Microsoft-Windows-Partition/Diagnostic, 1006, Microsoft-Windows-Ntfs/Operational, 142, 4, 9, 10, 300, 303'
tags:
  - windows_etw
  - windows_system_information
  - windows_usb_activity
location: 'Channels:\n\nMicrosoft-Windows-Storage-ClassPnP/Operational.\n Events: 507, 500, 502, 503, 504, 505, 506, 510.\n\nMicrosoft-Windows-Kernel-PnP/Device Configuration.\n Events: 400, 401, 410, 411, 420, 430.\n\nMicrosoft-Windows-Kernel-PnP/Device Management.\n Event: 1010.\n\nMicrosoft-Windows-Partition/Diagnostic.\n Event: 1006.\n\nMicrosoft-Windows-Ntfs/Operational.\n Events: 142, 4, 9, 10, 300, 303.\n\n'
last_updated: 2024-01-15
sidebar: sidebar
permalink: windows_etw_usb_activity.html
folder: windows
---

### Overview

The Windows `Event Tracing` generate various events on devices and USB
activity. The events are split across a number of channels, with more events
and information available on recent versions of the Windows operating system.

Using known variables about a given device, found for example in the
[Windows registry](./registry_usb_activity.md), `Event Tracing` events can be
used to determine timestamps of activity for the device, such as when the
device was first plugged, last plugged and unplugged. Additionally,
supplementary information about devices can be retrieved from events, such as
device storage sizes and an extract of their partition table.

The terminology and more details on the various identifiers are available in
the [Windows devices terminology page](./devices_terminology.md).

### Microsoft-Windows-Storage-ClassPnP/Operational

{% include note.html content="Events from the `Microsoft-Windows-Storage-ClassPnP/Operational` channel (event `507` in particular) can be used to: <br><br> - Determine when a device was plugged using the device vendor and product names or `serial number`. <br><br> - Retrieve (a version of) the device `serial number` (!= registry `serial number)` and its vendor and product names. <br><br> - Identify the device `DeviceGUID` for correlation with other events." %}

| Channel | Conditions | Events |
|---------|------------|--------|
| Provider: `Microsoft-Windows-StorDiag`. <br><br> Channel: `Microsoft-Windows-Storage-ClassPnP/Operational`. | Default configuration. | Event `507`: error events. <br><br> Generated multiple times, for every connection, sometimes safe removal, and while the device is plugged-in. As the event is generated upon errors, it may however not be reliably logged. <br><br> Information of interest: <br> - Device's vendor and product names. <br> - Device `serial number` (which is however not the same as the one found in the registry and often shows up as `AA00000000000489` for different USB storage devices). <br> - Device number, which is an incremental number based on the number of devices plugged-in, for all devices, including the system drive (which would likely be device number 1). <br> - Device's `DeviceGUID` which can be used for correlation with other events. <br><br> Other events, also generated upon errors and with similar information: `500`, `502`, `503`, `504`, `505`, `506`, and `510`. |

### Microsoft-Windows-Kernel-PnP/Device Configuration

{% include note.html content="Events from the `Microsoft-Windows-Kernel-PnP/Device Configuration` channel can be used to: <br><br> - Determine when a device was first plugged. <br><br> - Identify the `vendor ID (VID)` and `product ID (PID)` of the device from its `serial number` or location information (and vice versa)." %}

| Channel | Conditions | Events |
|---------|------------|--------|
| Provider: `Microsoft-Windows-Kernel-PnP`. <br><br> `Microsoft-Windows-Kernel-PnP/Device Configuration`. | Default configuration. | The `Microsoft-Windows-Kernel-PnP/Device Configuration` channel contains information for all plug and play devices, not limited to USB storage devices. <br><br> Event `400: Device <DEVICE> was configured`. <br> Event `401: Device <DEVICE> failed configuration`. <br> Event `410: Device <DEVICE> was started`. <br> Event `411: Device <DEVICE> had a problem starting`. <br> Event `430: Device <DEVICE> requires further installation`. <br> The aforementioned events appear to be generated when a device is first plugged-in to the system. <br><br> Event `420: Device <DEVICE> was deleted`. <br> The `<DEVICE>` string is based on the event `DeviceInstanceId` field, which contains the device's `vendor ID (VID)`, `product ID (PID)` and (registry) `serial number` or location information. |

### Microsoft-Windows-Kernel-PnP/Device Management

{% include note.html content="Introduced in Windows 11, the event `1010` of the `Microsoft-Windows-Kernel-PnP/Device Management` channel can used, if a device has been removed without prior ejection, to: <br><br> - Determine when a device was unplugged with out prior ejection, from the device (registry) `serial number` or location information. <br><br> - Identify the `vendor ID (VID)` and `product ID (PID)` of the device from its `serial number` or location information (and vice versa). <br><br> - Identify the `volumes GUID` associated with the device." %}

| Channel | Conditions | Events |
|---------|------------|--------|
| Provider: `Microsoft-Windows-Kernel-PnP`. <br><br> Channel: `Microsoft-Windows-Kernel-PnP/Device Management`. | Introduced in Windows 11. | The `Microsoft-Windows-Kernel-PnP/Device Management` channel contains information for all plug and play devices, not limited to USB storage devices. <br><br> Event `1010: Device <DEVICE> has been surprise removed as it is reported as missing on the bus`. <br><br> The event is reliably generated when a device is removed / unplugged without prior ejection. Additionally, subsequent immediate event(s) are generated for each of the device volume. <br><br> Relevant information: <br> - For USB storage device: `vendor ID (VID)`, `product ID (PID)`, (registry) `serial number` or location information. Example: `USB\VID_18A5&PID_0302\1601000001586259`. <br> - For volumes: the `volume GUID` of the volume. Example: `STORAGE\Volume\<GUID>`. |

### Microsoft-Windows-Partition/Diagnostic

{% include note.html content="The event `1006` of the `Microsoft-Windows-Partition/Diagnostic` can be used to: <br><br> - Determine when a drive was plugged / unplugged. <br><br> - Identify the `vendor ID (VID)` and `product ID (PID)` of the device from its `serial number`, either in the registry-like format or of device itself (and vice versa). <br><br> - Identify the `volume id` for one of the device volume. <br><br> - Identify the device `DeviceGUID` for correlation with other events. <br><br> - Retrieve the Size in bytes of the device. <br><br> - Retrieve a raw dump of the partition table of the device." %}

| Channel | Conditions | Events |
|---------|------------|--------|
| Provider: `Microsoft-Windows-Partition`. <br><br> Channel: `Microsoft-Windows-Partition/Diagnostic`. | Default configuration. | Event `1006`. <br><br> The event is generated when a device is plugged and unplugged with or without prior ejection. <br><br> This event contains key relevant information, and notably information that are not available in other sources: <br><br> - Vendor and product names of the device. <br><br> - `vendor ID (VID)`, `product ID (PID)`, and (registry) `serial number` or location of the device (in the `ParentId` field). <br><br> - A `volume id` for one of the device volume in the `RegistryId` field. <br><br> - (A version of) the device serial number (!= registry serial number). <br><br> - The `DeviceGUID` of the device in the `DiskId`, for correlation with other events. <br><br> - The size in bytes of the device in the `Capacity` field. The capacity is set to 0 if the event match a removal. <br><br> - Raw dumps of the partition table (field `PartitionTable`), `Master Boot Record (MBR)` (field `Mbr`), and / or `Volume Boot Record (VBR)` (field `VbrX`) if available. The `VBR` dump can be used to reconstruct the `Volume Serial Number` of the device. |

### Microsoft-Windows-Ntfs/Operational

{% include note.html content="For devices that have a `NTFS` volume, the event `142` of the `Microsoft-Windows-Ntfs/Operational` channel can be used to: <br><br> - Determine the volume friendly name(s) and drive letter(s) associated with a device, either using the `volume GUIDs` of the volumes on the device or time correlation with other events. <br><br> Introduced in `Windows 11`, new events in the `Microsoft-Windows-Ntfs/Operational` channel can be used to: <br><br> - Determine when a device was plugged / unplugged (and if it was with or without prior ejection) and its associated volumes mounted / dismounted. <br><br> - Identify the volume friendly name(s) and drive letter(s) associated with a device." %}

| Channel | Conditions | Events |
|---------|------------|--------|
| Provider: `Microsoft-Windows-Ntfs`. <br><br> `Microsoft-Windows-Ntfs/Operational`. | Only generated for devices that have a `NTFS` volume. | Event `142: Summary of disk space usage, since last event`. <br><br> This event is generated with a limited delay following the plugin of the device, one occurrence for each volume of the device. <br><br> Relevant information: <br> - The volume friendly name and associated drive letter. <br> - A `volume id` for one of the device volume. |
| Provider: `Microsoft-Windows-Ntfs`. <br><br> `Microsoft-Windows-Ntfs/Operational`. | Introduced in `Windows 11`. <br><br> Only generated for devices that have a `NTFS` volume. | Event `4: The NTFS volume has been successfully mounted`. <br><br> Event `9: NTFS scanned entire volume bitmap`. <br><br> Event `10: NTFS cached run statistics`. <br><br> Event `300: The NTFS volume dismount has started`. <br><br> Event `303: The NTFS volume has been successfully dismounted`. <br><br> These events are reliably generated when a device is plugged and unplugged with or without prior ejection. <br><br> Relevant information: <br> - The volume friendly name and associated drive letter. <br> - Vendor and product names of the device. <br> - (A version of) the device serial number (!= registry serial number). <br> - `DeviceGuid` (for correlation with other events). <br> - Whether the drive was ejected ("Reason: Explicit lock") or directly unplugged ("Reason: Surprise removal"). |

### References

  - [Orion Investigations - Andrew Smith - Microsoft Windows 10 USB Forensic Artefacts](http://website.bcmsystem.com/orion/wp-content/uploads/2019/05/Microsoft-Windows-10-USB-Forensic-Artefacts.pdf)
