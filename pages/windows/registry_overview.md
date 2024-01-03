---
title: Registry - Overview
summary: 'The Registry is a feature to store settings, for the operating system and applications, in system-wide or per users hierarchical databases or hives.\n\nBefore being written / committed to a file on disk, registry modifications can be written to Registry Transaction logs (such as SYSTEM.LOG1 and SYSTEM.LOG2 for the SYSTEM registry hive).'
keywords: 'HKEY_LOCAL_MACHINE, HKLM, SYSTEM, SOFTWARE, SAM, HKEY_CURRENT_USER, HKCU, NTUSER.dat, UsrClass, Registry Transaction logs, SYSTEM.LOG'
tags:
  - windows_registry
location: 'System-wide registry is mapped to the HKEY_LOCAL_MACHINE (HKLM) root key in memory.\nAssociated files on disk, under <SYSTEMROOT>\System32\config\: SYSTEM, SOFTWARE, SECURITY, SAM.\n\nPer user registry is mapped to the HKEY_CURRENT_USER (HKCU) root key in memory.\nAssociated files on disk:\n<SYSTEMDRIVE>\Users\<USERNAME>\NTUSER.dat\n<SYSTEMDRIVE>\Users\<USERNAME>\AppData\Local\Microsoft\Windows\UsrClass.dat.'
last_updated: 2024-01-03
sidebar: sidebar
permalink: windows_registry_overview.html
folder: windows
---

The **Windows Registry is a Windows feature to store low-level settings**, for
the operating system and for applications (that opt to use the registry), **in
the form of system-wide or per users hierarchical databases or hives**.

A registry hive is a group of keys, subkeys, and values in the registry, with
supporting file(s) on disk. Registry hives are loaded in memory upon system
boot or user logon from their associated files on disk.

**Before being written / committed to a file on disk**, registry modifications
can be **written to `Registry Transaction logs`** (notably if the hives cannot
be written to directly due to locking). `Transaction logs` are files named, and
stored in the same directory, as their corresponding registry hives. Such as
`SYSTEM.LOG1` and `SYSTEM.LOG2` for the `SYSTEM` registry file.

### System-wide registry hives

The **system-wide registry is mapped to the `HKEY_LOCAL_MACHINE`** (`HKLM`)
root key in memory.

The following notable system-wide root subkeys are defined:

  - `HKEY_LOCAL_MACHINE\SYSTEM`.
    File on disk: `%SystemRoot%\System32\config\SYSTEM`.

  - `HKEY_LOCAL_MACHINE\SOFTWARE`.
     File on disk: `%SystemRoot%\System32\config\SOFTWARE`.

  - `HKEY_LOCAL_MACHINE\SECURITY`.
     File on disk: `%SystemRoot%\System32\config\SECURITY`.

  -  `HKEY_LOCAL_MACHINE\SAM`.
     File on disk: `%SystemRoot%\System32\config\SAM`.

  - `HKEY_USERS`, contains all the actively loaded user profile registry hives
    on the computer.
    The `.DEFAULT` key is populated from the
    `%SystemRoot%\Users\Default\NTUSER.DAT` file.
    File on disk: users' `NTUSER.dat` and `UsrClass.dat` files (of logon
    users).

The `SYSTEM`, `SOFTWARE`, `SECURITY`, and `SAM` registry hives used to be
backed up periodically (every 10 days by default) under the
`%SystemRoot%\System32\config\RegBack` folder by the `RegIdleBackup` scheduled
task. Starting with the Windows 10 operating system, this mechanism is no
longer in use and no registry hive backups are stored under the `RegBack`
folder.

### Per-user registry hives

The **user specific registry is mapped to the `HKEY_CURRENT_USER`** (`HKCU`)
root key in memory:

  - `HKEY_CURRENT_USER`.
    File on disk: `%SystemDrive%:\Users\<USERNAME>\NTUSER.dat`.

  - `HKEY_CURRENT_USER\SOFTWARE\Classes`.
    File on disk:
    `%SystemDrive%:\Users\<USERNAME>\AppData\Local\Microsoft\Windows\UsrClass.dat`.

  - `HKEY_CLASSES_ROOT`, which define the programs and file extensions
    association.
    Mapped to the keys `HKEY_LOCAL_MACHINE\SOFTWARE\Classes`, for default
    settings, and `HKEY_CURRENT_USER\SOFTWARE\Classes`, for user specific
    settings that override the default settings.

### References

  - [Wikipedia - Windows_Registry](https://en.wikipedia.org/wiki/Windows_Registry)
