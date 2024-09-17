---
title: Amcache / RecentFileCache
summary: 'Very complex artefact, linked to an application compatibility feature.\n\nTracks program execution (or simply file presence for recent version), installed drivers, and shortcuts from a subset of folders.\n\nProgram execution / binary presence information of interest: executable full path, program size, SHA1 (of the first 30MB of the executable).'
keywords: Amcache, RecentFileCache
tags:
  - windows_program_execution
  - windows_file_knowledge
location: '<SYSTEMROOT>\AppCompat\Programs\Amcache.hve\n\nAmcache DLL 6.1.7600 and older:\n<SYSTEMROOT>\AppCompat\Programs\RecentFileCache.bcf'
last_updated: 2023-12-30
sidebar: sidebar
permalink: windows_amcache.html
folder: windows
---

### Overview

*`Amcache` is a replacement of the `RecentFileCache` (that was linked to DLL
version `6.1.7600`).*

Very complex artefact, linked to an application compatibility feature that aim
to maintain support of existing software to new versions of the Windows
operating system (like the `Shimcache` artefact). `ProgramDataUpdater` (a task
associated with the Application Experience Service) uses the registry file
`Amcache.hve` to store data during process creation. The `Amcache` is a
standalone registry hive, with multiple root keys that contain various types of
data.

The `Amcache` behavior depends on the version of the associated libraries, and
not the version of the operating system. The `Amcache` on an up-to-date
`Windows 7` and `Windows 10` will thus behave the same way.

For a very comprehensive analysis of the `Amcache` artefact, and its evolution
across different release of the underlying `DLL`, refer to the
[ANSSI's ANALYSIS OF THE AMCACHE v2 white paper](https://www.ssi.gouv.fr/uploads/2019/01/anssi-coriin_2019-amcache_investigation.pdf).

### Information of interest

The `Amcache.hve` registry hive is split in a number of root keys, with keys
being added, changed, or removed depending on the `Amcache` `DLLs` versions.

*A `Amcache` entry is created whenever a program is executed from a specific
path. However, entries may also be created for files in "scanned" directory.*

The following notable root keys can be of forensic interest:

  - `File` then `InventoryApplicationFile` starting from the version
    `10.0.14913.1002` of the `Amcache` libraries (`AmcacheParser` outputs
    `AssociatedFileEntries` and `UnassociatedFileEntries`):

    - Data about program executions if they are shimmed, programs part of an
      installed application, or programs part of scanned directories (without
      requiring execution of the associated programs).

    - Data available (depending on the `Amcache` libraries version): executable
      full path, program size, **`SHA1` of the first 30MB of the executable**
      in the `FileId` value, binary type (x86 versus x64), the compilation date
      of the program in the `LinkDate` value.

    - Additional data for entries associated with an installed application is
      available in the `InventoryApplication` key. The `ProgramId` value
      from the `InventoryApplicationFile` subkey of a given program matches the
      subkey's name under the `InventoryApplication` key of the associated
      application. The `InventoryApplication` key provide metadata information
      about the application: name, publisher, install date, etc.

    - For non up-to-date systems still using a `File` key, the last write time
      of an entry key under the `File` key coincides with the execution time of
      an executable that is not associated to an application. For executables
      that are part of an application, the last write time coincides with
      either the application installation time or the first execution if the
      executable needed shimming. For entries under the
      newer `InventoryApplicationFile` key, the last write time of the keys
      always coincides with an execution of
      `Microsoft Compatibility Appraiser` and is thus no longer a timestamp of
      execution time.

    - `AmcacheParser`'s `AssociatedFileEntries` output references programs
      associated with an application and `UnassociatedFileEntries` output
      references "loose" programs (that are not associated with an installed
      application).

  - `InventoryDeviceContainer` and `InventoryDevicePnp` (`AmcacheParser`
    outputs ``DeviceContainers`` and `DevicePnp`):

    - Data about devices plugged in on the system.

    - Data available: device type (usb; Bluetooth, media, etc.), device
      friendly name, self reported description, manufacturer, associated
      driver, etc.

  - `InventoryDriverBinary` (`AmcacheParser` output `DriveBinaries`):

    - Data about installed drivers.

    - Data available: driver name, full path, size, associated service name,
     compilation timestamp (`DriverTimestamp`), driver file last write
     timestamp, etc.

  - `InventoryDriverPackage` (`AmcacheParser` output `DriverPackages`):

    - Data about drivers package file (INF file) that contains information
      about the driver.

    - Data available: driver package file name, path, last write timestamp, etc.

  - `Programs` then `InventoryApplication` (`AmcacheParser` output
    `ProgramEntries`):

    - Data about installed programs, as referenced in the `Uninstall` and / or
      a `Run` key of the `SOFTWARE` hive.

    - Data available: application name, executable full path and SHA1,
      publisher, install date, etc.

  - `InventoryApplicationShortcut` (`AmcacheParser` output `ShortCuts`):

    - Data about the shortcuts (`LNK` files) that were present at one time (and
      that may still be present or may have been removed) from a subset of
      scanned folders (Start Menu and / or Desktop folders).

    - Data available: full path of the shortcut. The last write timestamp of
      the associated subkey can also be a general indicator of when the
      activity occurred but does not seem to match any `MACB` timestamps of the
      shortcut file.

### Tool(s)

- [EricZimmerman's `AmcacheParser`](https://github.com/EricZimmerman/AmcacheParser).

- `KAPE`'s `AmcacheParser` module.

### References

  - [ANSSI - Blanche Lagny - ANALYSIS OF THE AMCACHE v2 white paper](https://cyber.gouv.fr/sites/default/files/2019/01/anssi-coriin_2019-analysis_amcache.pdf)
