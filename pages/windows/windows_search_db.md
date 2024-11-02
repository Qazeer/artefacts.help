---
title: Windows Search database
summary: 'The Windows Search database provides an index to the Windows Search feature to improve search speed by indexing content from a subset of folders and files.\n\nInformation of interest: files and folders from the Users folders (file name, path, size, attributes, MAC timestamps, and sometimes part of the content of smaller files), Outlook mail data (timestamp of reception and possible mail content), OneNote notes title, and Internet Explorer history.'
keywords: Search database, Windows.edb, Windows.db, Windows-gather.db
tags:
  - windows_file_knowledge
  - windows_browsing_history
location: 'Starting from Windows 11:\n<SYSTEMDRIVE>:\ProgramData\Microsoft\Search\Data\Applications\Windows\Windows.db\n<SYSTEMDRIVE>:\ProgramData\Microsoft\Search\Data\Applications\Windows\Windows-gather.db\n\nWindows 7 to Windows 10:\n<SYSTEMDRIVE>:\ProgramData\Microsoft\Search\Data\Applications\Windows\Windows.edb\n\nWindows XP:\n<SYSTEMDRIVE>:\Documents and Settings\All user\Application Data\Microsoft\Search\Data\Application\Windows\Windows.edb'
last_updated: 2024-01-01
sidebar: sidebar
permalink: windows_search_db.html
folder: windows
---

### Overview

The `Windows Search` database provides an index to the Windows Search feature
to improve search speed by indexing content. The Windows Search index is used
for searches made through Windows taskbar, the Windows Explorer, and some
`Universal Windows Platform (UWP)` applications (such as Outlook, OneDrive,
etc.).

By default, only a subset of folders and files are indexed (to reduce the
Windows Search database size and CPU usage). The folders scanned and number of
items indexed can be consulted in the "Windows search settings" menu.

From `Windows Vista` / `Windows Server 2008` to `Windows 10` /
`Windows Server 2019`, the `Windows Search` used an
`Extensible Storage Engine (ESE)` database (`Windows.edb`). Starting with
`Windows 11` / `Windows Server 2022`, the `Windows Search` switched to two
`SQLite` databases (`Windows.db` and `Windows-gather.db`).

### Information of interest

By default, only items from the following sources are scanned and indexed:

  - Files and folders from the folders `C:\Users\*` (excluding `AppData`
    directories) and `C:\ProgramData\Microsoft\Windows\Start Menu\Programs\*`
    (which include startup `LNK` files).

    > Data available: file name, path, size, attributes, `MAC` timestamps.
      For small file, part of the content of the file may be indexed as well.

  - Outlook mail data (with timestamp of reception, possible mail content).

  - OneNote notes title.

  - Internet Explorer history (URLs, timestamp of last visit).

### Tool(s)

The [Search Index DB Reporter (SIDR)](https://github.com/strozfriedberg/sidr)
utility (`SIDRWindowsIndexSearchParser` KAPE module) can be used to parse the
`Windows Search` database (in both `ESE` and `SQLite` formats).

```bash
# Recursively scan the <INPUT_DIRECTORY> for Windows.edb and Windows.db files.

sidr.exe -f <json | csv> <INPUT_DIRECTORY> -o <DESTINATION_DIRECTORY>
```

Alternatively, the [WinSearchDBAnalyzer](https://github.com/moaistory/WinSearchDBAnalyzer)
graphical utility can be used to parse and explore the `Windows Search`, in
`ESE` database format only.

### References

  - [SANS DFIR Summit 2023 - Phalgun Kulkarni & Julia Paluch - Windows Search Index: The Forensic Artifact You've Been Searching For](https://www.youtube.com/watch?v=X4WTcRdIDAM)

  - [Aon - Phalgun Kulkarni, Julia Paluch - Windows Search Index: The forensic artifact you’ve been searching for](https://www.aon.com/cyber-solutions/aon_cyber_labs/windows-search-index-the-forensic-artifact-youve-been-searching-for/)
