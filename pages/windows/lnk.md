---
title: Windows - Shortcut files / LNK
summary: 'Shortcut files (*.lnk) are Windows Shell Items that reference an original file, folder, or application.\n\nWhile LNK files can be created manually, Windows also creates LNK files upon numerous user activities, such as the opening of a non-executable file.\n\nInformation of interest, per LNK file:\n - Target file absolute path, size, and attributes.\n - Target file Modified, Access, and Created (MAC) timestamps at the time of the last access.\n - Sometimes information on the volume that stored the target file (local or network share, serial number, and label).\n - Additionally, for automatically created LNK files, the creation and modification timestamps of the LNK itself will usually indicate when the target file was first and last opened.\n\nShortcut files can also be automatically executed upon an interactive user logon, by being placed under the system-wide Startup folder (under %ALLUSERSPROFILE%) or the user-scoped Startup folders (under each user''s %APPDATA% folder).'
keywords: 'LNK, Shortcut files, Recent, Startup, Startup folder, Start Menu'
tags:
  - windows_files_and_folders_access
  - windows_file_knowledge
  - windows_local_persistence
  - windows_local_persistence_user
location: 'Automatically created LNK files on file access:\n<SYSTEMDRIVE>:\Users\<USERNAME>\AppData\Roaming\Microsoft\Windows\Recent\*.lnk\n\nAutomatically created LNK files for documents opened using Microsoft Office products:\n<SYSTEMDRIVE>:\Users\<USERNAME>\AppData\Roaming\Microsoft\Office\Recent\*.lnk\n\nOther common LNK locations:\n\nUser Desktop folder:\n<SYSTEMDRIVE>:\Users\<USERNAME>\Desktop\*.lnk\n\nStartup folders:\n<SYSTEMDRIVE>:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\*.lnk\n<SYSTEMDRIVE>:\Users\<USERNAME>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\*.lnk'
last_updated: 2024-01-03
sidebar: sidebar
permalink: windows_lnk.html
folder: windows
---

### Overview

`Shortcut files (*.lnk)` are `Windows Shell Items` that reference an
original file, folder, or application. The effect of double-clicking a
`shortcut file` is intended to be the same as double-clicking the application
or file to which it refers. In addition, command line parameters and the folder
in which the target should be opened can be specified in the shortcut. The
`shortcut files` have a magic number of `0x4C` (`4C 00 00 00`).

While `shortcut files` can be created manually, the Windows operating system
also creates `shortcut files` upon numerous user activities, such as the
opening of a non-executable file. For instance, a `shortcut file` is created
under `[...]\AppData\Roaming\Microsoft\Windows\Recent\` whenever a file is
opened from the `Windows Explorer`. `Shortcut files` created in such
circumstances are referenced in the
`NTUSER.DAT\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs`
registry keys.

These automatically created and updated `shortcut files` are not deleted upon
deletion of their associated files.

The `shortcut file` format is also used for entries within the
`AutomaticDestinations` and `CustomDestinations`
[`JumpLists` files](./jumplists.md) (introduced in `Windows 7`).

### Information of interest

The **creation and modification timestamps of the shortcut file itself** will
usually respectively indicate **when the target file was first and last
opened** for automatically created `shortcut files`.

Each shortcut file additionally yields the following information:

  - The **target file's absolute path, size, and attributes** (hidden,
    read-only, etc.). The size and attributes are updated upon every access to
    the target file (that induces an update to the `shortcut file`).

  - The **target file **`Modified,  Access, and Created (MAC)` timestamps at
    the time of the last access to the target file**.

  - Whether the **target file was stored locally or on a remote network share**
    through the specification of a `LocalPath` or `NetworkPath`.

  - Occasionally **information on the volume that stored the target file**:
    drive type (fixed vs removable storage media), serial number, and label /
    name if any.

  - Occasionally **information on the host on which the shortcut file is
    present**: system's NetBIOS hostname and MAC address.

### Startup Folders

`Shortcut files` can also be automatically executed upon an interactive user
logon, by being placed under the `Startup` folders. A user interactive logon
is required for the entries to be executed, a system boot or a network logon
will not trigger the execution of the `Startup` folder entries.

There is a system-wide `Startup` folder (under `%ALLUSERSPROFILE%`) and
user-scoped `Startup` folders (under each user's `%APPDATA%` folder). While
administrative privileges are required to create an entry in the system-wide
`startup folder`, regular users can add entries to their own `startup folder`. 

### Tool(s)

Eric Zimmerman's [`LECmd`](https://github.com/EricZimmerman/LECmd) tool
(`KAPE`'s `LECmd` module) can be used to process `shortcut files`.

```bash
# Parses the specified shortcut file.
LECmd.exe [-q --csv <CSV_DIRECTORY_OUTPUT>] -f <LNK_FILE>

# Recursively retrieves and parses the shortcut files in the specified directory.
LECmd.exe [-q --csv <CSV_DIRECTORY_OUTPUT>] -d <C:\Users\<USERNAME>\AppData\Roaming\Microsoft\Windows\Recent\ | C:\ | DIRECTORY>
```

### References

  - [Windows 10 Jump List and Link File Artifacts - Saved, Copied and Moved](https://dfir.pubpub.org/pub/wfuxlu9v/release/1)

  - [Magnet Forensics - Jamie McQuaid - Forensic Analysis of LNK files](https://www.magnetforensics.com/blog/forensic-analysis-of-lnk-files/#:~:text=LNK%20files%20are%20a%20relatively,LNK%20extension)

  - [13Cubed - LNK Files and Jump Lists](https://www.youtube.com/watch?v=wu4-nREmzGM)

  - [Forensics Wiki - Lnk](https://forensics.wiki/lnk/)

  - [Microsoft - [MS-SHLLINK] - 2.1 ShellLinkHeader](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-shllink/c3376b21-0931-45e4-b2fc-a48ac0e60d15)
