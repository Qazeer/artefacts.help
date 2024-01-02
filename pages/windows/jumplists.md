---
title: Jumplists
summary: 'Introduced in Windows 7, Jumplists are linked to a taskbar user experience-enhancing feature that allows users to "jump" to files, folders or others elements by right clicking on open applications in the Windows taskbar.\n\nInformation of interest: target file absolute path, size, attributes, and Modified, Access, and Birth timestamps (updated whenever the file is "jumped" to).'
keywords: Jumplists
tags:
  - windows_files_and_folders_access
  - windows_file_knowledge
location: 'AutomaticDestinations:\n<SYSTEMDRIVE>:\Users\<USERNAME>\AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations\<APP_ID>.automaticDestinations-ms\n\nCustomDestinations:\n<SYSTEMDRIVE>:\Users\<USERNAME>\AppData\Roaming\Microsoft\Windows\Recent\CustomDestinations\<APP_ID>.customDestinations-ms'
last_updated: 2023-12-31
sidebar: sidebar
permalink: windows_jumplists.html
folder: windows
---

### Overview

Introduced in `Windows 7`, `Jumplists` are linked to a taskbar user
experience-enhancing feature that allows users to "jump" to files, folders
or others elements by right clicking on open applications in the `Windows
taskbar`. The `Windows Explorer`'s `Quick Access` feature also stores entries
in `Jumplists`.

Two forms of `Jumplists` are created:
  - automatic entries for recently accessed items, stored in
    `*.automaticDestinations-ms` files.

  - custom entries in `*.customDestinations-ms` files for items manually
    "pinned" elements (by users or the applications themselves) to the
    `Windows taskbar` or an application's `Jumplist`.

Each application `AutomaticDestinations` and `CustomDestinations` `JumpLists`
information are thus stored in two unique and separated files, of different
format:
  - `AutomaticDestinations` `JumpLists` files are stored as
    `AUTOMATICDESTINATIONS-MS` file, in the `MS OLE Structured Storage` format.
    This file format contains multiple streams, each stream composed of data
    similar to `shortcut files (.LNK)`.

  - `CustomDestinations` `JumpLists` are stored as `CUSTOMDESTINATIONS-MS`
    file, also assimilable to a series of `shortcut files`.

### Information of interest

`JumpLists` hold information similar in nature to `shortcut files` for each
file referenced in an application's `AutomaticDestinations` /
`CustomDestinations` `JumpLists`:
  - the target file's **absolute path, size and attributes** (hidden,
    read-only, etc.).

  - the target file **`Modified, Access, and Birth` timestamps**,
    updated whenever the file is "jumped" to.

  - the **number of times the target file was "jumped" to**.

As `JumpLists` are linked to an application, through an `AppId`, knowledge of
the application that was used to open the files can be deducted if the
application associated to the `AppId` is known. A number of `AppId` is
documented in
[`EricZimmerman` 's `JumpList` GitHub repository](https://github.com/EricZimmerman/JumpList/blob/master/JumpList/Resources/AppIDs.txt).

Specific applications may define custom `JumpLists` entries that store
information of forensic interest. For example, the `Google Chrome` and
`Microsoft Edge` web browsers store the recently closed tabs in their
respective `CustomDestinations` `JumpLists`.

### Tool(s)

Eric Zimmerman's [`JumpListExplorer`](https://f001.backblazeb2.com/file/EricZimmermanTools/net6/JumpListExplorer.zip)
and [`JLECmd`](https://github.com/EricZimmerman/JLECmd) tools can be used to
process `JumpLists` files.

Associated `KAPE` compound module: `JLECmd`.

```bash
# Parses the specified JumpLists file.
JLECmd.exe [-q --csv <CSV_DIRECTORY_OUTPUT>] -f <JUMPLIST_FILE>

# Recursively retrieves and parses the JumpLists files in the specified directory.
JLECmd.exe [-q --csv <CSV_DIRECTORY_OUTPUT>] -d <C:\Users\<USERNAME>\AppData\Roaming\Microsoft\Windows\Recent\ | C:\ | DIRECTORY>
```

### References

  - [13Cubed - LNK Files and Jump Lists](https://www.youtube.com/watch?v=wu4-nREmzGM)
