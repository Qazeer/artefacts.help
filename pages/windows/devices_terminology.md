---
title: Windows devices terminology
summary: 'The Windows operating system uses a number of "device identification strings" and "device instance identification strings" to identify devices that are plugged / installed on a computer, and their instances.\n\nThe following identification strings are defined: vendor ID, product ID, device ID, hardware ID, instance ID, device instance ID, and container ID.\n\nThese various identifiers can be used to uniquely identify USB drives plugged into a computer, and are referenced in various registry keys, ETW events, and log files.'
keywords: 'vendor ID, product ID, device ID, hardware ID, instance ID, device instance ID, container ID, device interface class'
tags:
  - windows_system_information
  - windows_usb_activity
last_updated: 2024-01-13
sidebar: sidebar
permalink: windows_devices_terminology.html
folder: windows
---

### Device and device instance identification strings

The Windows operating system uses a number of `device identification strings`
to identify devices that are plugged / installed on a computer. Windows notably
leverages certain identification strings to locate the driver package that best
matches a device. Additionally, `device instance identification strings` are
set to uniquely identity each device instance (to distinguish a device from
other devices of the same type / vendor / product).

The following `identification strings` are defined:

  - The `vendor ID` identifies a specific vendor, with a mapping available on
    [devicehunt.com](https://devicehunt.com/all-usb-vendors). The
    `product ID (PID)` identifies a product from that vendor.

  - The `device ID` or `hardware ID` is "a vendor-defined identification string
    that Windows uses to match a device to a driver package". The identifier
    references the vendor and product names as well as the revision version.
    Example for a `DataTraveler_3` USB key by Kingston:
    `Ven_Kingston&Prod_DataTraveler_3.0&Rev_PMAP`.

  - The `instance ID` is "a device identification string that distinguishes a
    device from other devices of the same type on a computer". It contains the
    device `serial number`, if supplied, and otherwise "some kind of location
    information". Example of an `instance ID` for a device that does not supply
    a serial number: `5&2eab04ab&0&1`.

  - The `device instance ID` is "a system-supplied device identification string
    that uniquely identifies a device in the system". It is notably composed of
    the device's `device ID` and `instance ID`.

  - The `container ID` is "a system-supplied device identification string that
    uniquely groups the functional devices associated with a single-function or
    multifunction device installed in the computer". Starting with Windows 7,
    the `Plug and Play (PnP) manager` uses the `container ID` to group one or
    more device nodes (`devnodes`) that originated from a particular physical
    device.

  - The `device interface class` represents the type of the device (storage
    devices, USB devices, Bluetooth devices, etc.). Each
    `device interface class` is associated with a unique `GUID`, defined by
    Microsoft. The list of `GUIDs` by category of device can be found
    [in the Microsoft documentation](https://learn.microsoft.com/en-us/previous-versions//ff553412(v=vs.85)).
      - External physical storage `GUID`:
        `{53f56307-b6bf-11d0-94f2-00a0c91efb8b}`.
      - Logical volumes `GUID`: `{53f5630d-b6bf-11d0-94f2-00a0c91efb8b}`.

### References

  - [Microsoft - Device ID string](https://learn.microsoft.com/en-us/windows-hardware/drivers/install/device-ids)

  - [Microsoft - Hardware ID](https://learn.microsoft.com/en-us/windows-hardware/drivers/install/hardware-ids)

  - [Microsoft - Compatible ID](https://learn.microsoft.com/en-us/windows-hardware/drivers/install/compatible-ids)

  - [Microsoft - Instance ID](https://learn.microsoft.com/en-us/windows-hardware/drivers/install/instance-ids)

  - [Microsoft - Device instance ID](https://learn.microsoft.com/en-us/windows-hardware/drivers/install/device-instance-ids)

  - [Microsoft - Container ID](https://learn.microsoft.com/en-us/windows-hardware/drivers/install/container-ids)

  - [Microsoft - Compatible ID]()
