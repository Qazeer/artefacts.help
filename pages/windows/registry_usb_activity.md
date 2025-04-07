---
title: Registry - Devices and USB activity
summary: 'The registry hold numerous information on currently and previously plugged devices, such as USB devices. The information is stored across a number of registry keys.\n\nGiven a known variable about a device as input (such as the device serial number for example), other identifiers can be retrieved from the registry: serial number, vendor ID, product ID, device id (vendor and product names), instance ID, device interface class, associated volume friendly name and volume letter, etc.\n\nThe first and last plugged-in timestamps, and last unplugged timestamp (for Windows 7 / 8 and later) of a device are also stored in the registry (Enum\USB and Enum\USBSTOR registry keys).'
keywords: 'USB, USBSTOR, WPDBUSENUM, Enum\USB, Enum\USBSTOR, Enum\SWD\WPDBUSENUM, MountedDevices, DeviceClasses, Windows Portable Devices, VolumeInfoCache, EMDMgmt, MountPoints2'
tags:
  - windows_registry
  - windows_system_information
  - windows_usb_activity
location: 'HKLM\SYSTEM - Enum\USB\n\nHKLM\SYSTEM - Enum\USBSTOR\n\nHKLM\SYSTEM - Enum\SWD\WPDBUSENUM\n\nHKLM\SYSTEM - MountedDevices\n\nHKLM\SYSTEM - DeviceClasses\n\nHKLM\SOFTWARE - Windows Portable Devices\n\nHKLM\SOFTWARE - VolumeInfoCache\n\nHKLM\SOFTWARE - EMDMgmt\n\nHKCU\SOFTWARE - MountPoints2'
last_updated: 2024-01-13
sidebar: sidebar
permalink: windows_registry_usb_activity.html
folder: windows
---

### Overview

The registry hold numerous information on currently and previously plugged
devices, such as USB devices. The information is stored across a number of
registry keys.

Given a known variable about a device as input (such as the device
`serial number` or vendor and product names for example), other identifiers can
be retrieved from the registry: `serial number`, `vendor ID`, `product ID`,
`device id` (vendor and product names), `instance ID`,
`device interface class`, associated volume friendly name and volume letter,
etc.

The first and last plugged-in timestamps, and last unplugged timestamp (for
`Windows 7 / 8` and later) of a device are also stored in the registry
(`Enum\USB` and `Enum\USBSTOR` registry keys).

The terminology and more details on the various identifiers are available in
the [Windows devices terminology page](./devices_terminology.md).

### Enum\USB

{% include note.html content="`Enum\USB` can be used to: <br><br> - Identity the `vendor ID (VID)` and `product ID (PID)` of an USB device from its `serial number` or location information (and vice versa). <br><br> - Determine when the device was first and last plugged-in and last unplugged for `Windows 7 / 8` and later." %}

| Hive | Location |
| `HKLM\SYSTEM` | File: `%SystemRoot%\System32\config\SYSTEM`. <br><br> Registry key: <br> `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\USB\`. |

The `Enum\USB` registry key contains system-wide information about the
currently or previously connected USB devices.

Each USB device is associated with a dedicated subkey under the `Enum\USB`
key. This subkey is named after the device `vendor ID (VID)` and
`product ID (PID)` of the device. For example: `VID_1B1C&PID_4242`.

FUnderneath the `VID`/`PID` subkey, another subkey is named after the
`instance ID` of the device (referencing either the device's `serial number` or
location information).

This subkey references in turn information and parameters for the USB device as
values and under the `Properties` and `Device Parameters` subkeys, notably:

  - The `ClassGUID` key value references the `device interface class` `GUID` of
    the device.

  - The `ContainerID` key value references the `container ID` of the device.

  - The `Properties\{83da6326-97a6-4088-9453-a1923f573b29}` subkey notably
    references three child subkeys of interest, each containing a timestamp
    value:
      - `0064` (starting from Windows 7): timestamp of when the device was
        first plugged-in/installed.

      - `0066` (starting from Windows 8): timestamp of when the device was last
        connected.

      - `0067` (starting from Windows 8): timestamp of when the device was last
        removed.

### Enum\USBSTOR

{% include note.html content="`Enum\USBSTOR` can be used to: <br><br> - Identity the `device id` (vendor and product names) of an USB device from its `serial number` or location information (and vice versa). <br><br> - Determine when the device was first and last plugged-in and last unplugged for `Windows 7 / 8` and later. <br><br> - Retrieve a `volume id` for the (or one of the) volume(s) of the device." %}

| Hive | Location |
| `HKLM\SYSTEM` | File: `%SystemRoot%\System32\config\SYSTEM`. <br><br> Registry key: <br> `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\USBSTOR\`. |

The `Enum\USBSTOR` registry key contains system-wide information about the currently or
previously connected USB devices **that are related to storage**.

Each USB device is associated with a dedicated subkey under the `Enum\USBSTOR`
key. This subkey is named after the device `device ID` or `hardware ID` of the
device, which references the vendor and product names.
Example: `Disk&Ven_SanDisk&Prod_Extreme&Rev_0001`.

Underneath the `device ID` subkey, another subkey is named after the
`instance ID` of the device (referencing either the device's `serial number` or
location information).

This subkey in turn references:

  - The same information as the `Enum\USB` key.

  - A `volume id` for one of the device volume in the `DiskId` key value under
    the `Device Parameters\Partmgr`.

### Enum\SWD\WPDBUSENUM

{% include note.html content="`Enum\SWD\WPDBUSENUM` can be used to: <br><br> - Identity the `device id` (vendor and product names) of an USB device from its `serial number` or location information (and vice versa). <br><br> - Determine when the device was first and last plugged-in and last unplugged for `Windows 7 / 8` and later. <br><br> - Retrieve a friendly name of the (or one of the) volume associated with the device." %}

| Hive | Location |
| `HKLM\SYSTEM` | File: `%SystemRoot%\System32\config\SYSTEM`. <br><br> Registry key: <br> `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\SWD\WPDBUSENUM`. |

The `Enum\SWD\WPDBUSENUM` registry key contains system-wide information about
the currently or previously connected USB devices.

Each device is associated with a dedicated subkey under the `WPDBUSENUM` key.

This key is named with a string containing either:

  - The device's `device instance ID` (that includes the device's vendor and
    product names and `serial number`) and `device interface class` `GUID`.
    Example: `SWD#WPDBUSENUM#_??_USBSTOR#DISK&VEN_SAMSUNG&PROD_TYPE-C&REV_1100#0376022080001660&0#{53F56307-B6BF-11D0-94F2-00A0C91EFB8B}`.

  - The `volume id`.
    Example: `SWD#WPDBUSENUM#{44B06C95-F0BA-11ED-9802-6C9466A63B90}#000000000C900000`.

This subkey references:

  - The same information as the `Enum\USB` key.

  - A "friendly name" or display name of the (or one of the) volume associated
    with the device in the `FriendlyName` key value.

### MountedDevices

{% include note.html content="`MountedDevices` can be used to identity the `volume GUID` or the drive letter associated with the device from its `serial number` or location information (and vice versa)." %}

| Hive | Location |
| `HKLM\SYSTEM` | File: `%SystemRoot%\System32\config\SYSTEM`. <br><br> Registry key: <br> `HKEY_LOCAL_MACHINE\SYSTEM\MountedDevices`. |

The `MountedDevices` registry key is the persistent database of the
`Mount manager` (component responsible for managing volume names), and contains
system-wide information about the currently or previously mounted drives
(such as the system drive or USB devices).

Each device is associated with a separate binary value, composed of:

  - The device `volume GUID` (as the key's value name).

  - The `device/hardware  ID`, `instance ID`, and `device interface class`
    `GUID` in a `#` separated string (as the key's value data).

      - The `device ID`/`hardware ID` references the vendor and product
        names.

      - The `instance ID` contains the device's `serial number` or location
        information.

      - The `device interface class` represents the type of the device and
        each `class` is associated with a unique `GUID`.

Full example value for an USB key:
`_??_USBSTOR#Disk&Ven_Kingston&Prod_DataTraveler_3.0&Rev_PMAP#60A44C42568CB041B98902A4&0#{53f56307-b6bf-11d0-94f2-00a0c91efb8b}`

### DeviceClasses

{% include note.html content="`DeviceClasses` can be used to identity the `device id` (vendor and product names) of an USB device from its `serial number` or location information (and vice versa). <br><br> The last written timestamp of a device subkey can be an indicator of when the device was last plugged on the system or the first time the device was plugged following a reboot. However, the subkey does not appear to be reliably written to on recent versions of the Windows operating system and thus the timestamp should not be considered by itself as a reliable indicator of the device's last activity." %}

| Hive | Location |
| `HKLM\SYSTEM` | File: `%SystemRoot%\System32\config\SYSTEM`. <br><br> Registry key: <br> `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\DeviceClasses`. |

**Introduced in Windows Vista**, the `DeviceClasses` registry key contains
system-wide information about the currently or previously connected plug and
play devices (such as storage devices, volumes, network devices, Bluetooth
devices, etc.).

Each `device class` (physical disk, volume, USB device, Bluetooth device,
etc.) is referenced by its own subkey. The subkeys are named after the
[`device classes` `GUID`](https://learn.microsoft.com/en-us/previous-versions//ff553412(v=vs.85)).

For examples:
  - The external physical storages of the device would be referenced under the
    `{53f56307-b6bf-11d0-94f2-00a0c91efb8b}` `GUID` subkey.

  - The logical volumes on the device would be referenced under the
    `{53f5630d-b6bf-11d0-94f2-00a0c91efb8b}` `GUID` subkey.

Under each `device classes` `GUID` subkey, the devices of the given type are
referenced as their own subkey, whose name is a `#` separated string composed
of the `device/hardware ID`, `instance ID`, and `device interface class`
`GUID` of the device.

Example of a `{53f56307-b6bf-11d0-94f2-00a0c91efb8b}` subkey (physical
storage): <br>
`##?#SCSI#Disk&Ven_Samsung&Prod_SSD_870_EVO_2TB#4&cd4f6d&0&040000#{53f56307-b6bf-11d0-94f2-00a0c91efb8b}`.

Example of a `{53f5630d-b6bf-11d0-94f2-00a0c91efb8b}` subkey (logical volume): <br>
`##?#STORAGE#Volume#{d446d066-ade9-11ed-8679-eae9fe3c14cf}#0000000000100000#{53f5630d-b6bf-11d0-94f2-00a0c91efb8b}`.

### Windows Portable Devices

{% include note.html content="`Windows Portable Devices` can be used to identity a volume friendly name from (a) a device `serial number`/location information or (b) a `volume id` (and vice versa)." %}

| Hive | Location |
| `HKLM\SOFTWARE` | File: `%SystemRoot%\System32\config\SOFTWARE`. <br><br> Registry key: `HKLM\SOFTWARE\Microsoft\Windows NT\Windows Portable Devices`. |

The `Windows Portable Devices` registry key contains information on currently
or previously attached media and storage devices, notably the device
volume(s)' "friendly name" or display name.

Each device is associated with a dedicated subkey under the
`Windows Portable Devices\Devices` key.

This key is named with a string containing either:

  - The device's `device instance ID` (that includes the device's vendor and
    product names and `serial number`) and `device interface class` `GUID`.

    Example: `SWD#WPDBUSENUM#_??_USBSTOR#DISK&VEN_SAMSUNG&PROD_TYPE-C&REV_1100#0376022080001660&0#{53F56307-B6BF-11D0-94F2-00A0C91EFB8B}`.

  - The `volume id`.

    Example: `SWD#WPDBUSENUM#{44B06C95-F0BA-11ED-9802-6C9466A63B90}#000000000C900000`.

This device subkey references in turn, in the `FriendlyName` value, the
"friendly name" or display name of the (or one of the) volume associated with
the device.

### VolumeInfoCache

{% include note.html content="`VolumeInfoCache` can be used to: <br> - Identity a volume drive letter from a volume friendly name (and vice versa), if the volume was the last one to be associated with the given letter. <br> - Determine when a volume was last associated with a given drive letter." %}

| Hive | Location |
| `HKLM\SOFTWARE` | File: `%SystemRoot%\System32\config\SOFTWARE`. <br><br> Registry key: `HKLM\SOFTWARE\Microsoft\Windows Search\VolumeInfoCache`. |

The `VolumeInfoCache` registry key contains information on currently or
previously mounted volumes, notably a mapping between drive letters and their
last associated volume(s)'s "friendly name" or display name.

Each previously referenced drive letter (`A:` to `Z:`, including `C:`) is
associated with a dedicated subkey under the `VolumeInfoCache` key.

This subkey contains information about the volume last associated with the
corresponding drive letter:

  - The volume friendly name in the `VolumeLabel` value.

  - The associated [drive type](https://learn.microsoft.com/en-us/dotnet/api/system.io.drivetype)
    in the `DriveType` value. Both hard disks/SSDs and storage devices (such
    as USB keys) appear to be associated with the value `3` (on Windows 10).

The last written timestamp of the key is an indicator of when a volume was last
associated with a given drive letter.

### EMDMgmt

{% include note.html content="`EMDMgmt` can be used to identity the device `serial number`, associated volume `serial number`, and possibly the volume friendly name of an USB device from its `device ID` or `hardware ID` (and vice versa)." %}

| Hive | Location |
| `HKLM\SOFTWARE` | File: `%SystemRoot%\System32\config\SOFTWARE`. <br><br> Registry key: `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\EMDMgmt`. |

**Only available if the system drive is not an SSD**, the `EMDMgmt` registry
key is related to the `ReadyBoost` feature and contains system-wide information
about the currently or previously connected USB devices.

Each USB device is associated with a dedicated subkey under the `EMDMgmt` key.
This subkey is named after the device `device ID` or `hardware ID` of the
device, which references the vendor and product names.

Example: `Disk&Ven_SanDisk&Prod_Extreme&Rev_0001`.

The device subkey contains:

  - The device's `serial number`.

  - The associated volume `serial number`.

  - Possibly the volume "friendly name" (if the mounted volume has a name).

[Example](http://windowsir.blogspot.com/2013/04/plugin-emdmgmt.html):

```
Disk&Ven_Best_Buy&Prod_Geek_Squad_U3&Rev_6.15

LastWrite: Sun Jul 17 12:13:25 2011 Z
SN: 0C90195032E36889&0
Vol Name: TEST
VSN: 6403-CD1C
```

### [MountPoints2](./registry_mountpoints2.md)

{% include note.html content="`MountPoints2` can be used to determine which user interacted with a given USB device. However entries are not reliably created, so the absence of an entry is not an indicator that the given user didn't interact with the device." %}

### References

  - [Orion Investigations - Andrew Smith - Microsoft Windows 10 USB Forensic Artefacts](http://website.bcmsystem.com/orion/wp-content/uploads/2019/05/Microsoft-Windows-10-USB-Forensic-Artefacts.pdf)

  - [SANS - Rob Lee - Computer Forensic Guide To Profiling USB Device Thumbdrives on Win7, Vista, and XP](https://www.sans.org/blog/computer-forensic-guide-to-profiling-usb-device-thumbdrives-on-win7-vista-and-xp/)

  - [13Cubed - Windows Registry Cheat Sheet](https://training.13cubed.com/downloads)

  - [hecfblog - David Cowen - Daily Blog #67 - Understanding the artifacts DeviceClasses](https://www.hecfblog.com/2013/08/daily-blog-67-understanding-artifacts.html)

  - [Microsoft - Supporting Mount Manager Requests in a Storage Class Driver](https://learn.microsoft.com/en-us/windows-hardware/drivers/storage/supporting-mount-manager-requests-in-a-storage-class-driver)

  - [windowsir.blogspot.com - H. Carvey - Plugin: EMDMgmt](https://windowsir.blogspot.com/2013/04/plugin-emdmgmt.html)
