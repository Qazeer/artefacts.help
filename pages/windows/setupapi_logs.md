---
title: Setupapi logs
summary: 'The setupapi logs are plaintext log files that track installation of devices and drivers on the system.\n\nThe logs are rotated and preserved, so historical data dating back to the system install is usually available.\n\nInformation of interest: device serial number, device id (vendor and product names) or vendor ID (VID) + product ID (PID), and when a device was first plugged (in the local timezone of the system).'
keywords: USB, device, setupapi, serial number
tags:
  - windows_usb_activity
location: 'Windows XP:\n<SYSTEMROOT>\setupapi.log\n\nStarting from Windows 7:\n<SYSTEMROOT>\INF\setupapi.dev.log\n<SYSTEMROOT>\INF\setupapi.dev.<YYYYMMDD-HMMSS>.log'
last_updated: 2024-01-15
sidebar: sidebar
permalink: windows_setupapi_logs.html
folder: windows
---

### Overview

The `setupapi` logs are plaintext log files that track installation of devices
and drivers on the system. The logs are rotated and preserved, so historical
data dating back to the system install is usually available (if the logs were
not deleted / tampered with).

The terminology and more details on the various identifiers are available in
the [Windows devices terminology page](./devices_terminology.md).

### Information of interest

{% include note.html content="The `setupapi` logs can be used to determine when a device was first plugged (in the local timezone of the system)." %}

The **device installation entries** (generated when the device is plugged-in)
**contain various information**, including the device:

  - `serial number`.

  - `Device id` (vendor and product names) or `vendor ID (VID)` +
    `product ID (PID)`.

Example of an entry for the first time an USB device was plugged-in:

```
>>>  [Device Install (Hardware initiated) - SWD\WPDBUSENUM\_??_USBSTOR#Disk&Ven_USB&Prod_Flash_Disk&Rev_1100#7&d2713f&0#{53f56307-b6bf-11d0-94f2-00a0c91efb8b}]
>>>  Section start 2021/02/07 19:11:17.101
```

Example of an entry for a device that was "deleted" through the `cleanmgr.exe`
utility:

```
>>>  [Delete Device - USB\VID_090C&PID_2000\8&1DBBAC39&0&3]
>>>  Section start 2023/03/16 16:55:26.426 <br> cmd: "%SystemRoot%\Windows\system32\cleanmgr.exe" /autoclean /d C: <br>
<<<  Section end 2023/03/16 16:55:26.473
```
