---
title: WebCacheV01
summary: 'The WebCacheV01.dat database is used by the Microsoft Internet Explorer and Microsoft Edge (legacy) web browsers to store browsing history, downloads, cache, and cookies.\n\nHowever, access to local files, not necessarily through a web browser, are also tracked in the WebCacheV01.dat database. Access to local files can be identified by the file URI scheme (such as "file:///<DRIVE_LETTER>:/folder/file").\n\nInformation of interest: full path to the file or URL, timestamp of access, and visit count.'
keywords: 'WebCacheV01, WebCacheV01.dat, ESE, file://'
tags:
  - windows_files_and_folders_access
  - windows_browsing_history
location: '<SYSTEMDRIVE>:\Users\<USERNAME>\AppData\Local\Microsoft\Windows\WebCacheV01.dat'
last_updated: 2024-01-07
sidebar: sidebar
permalink: windows_webcachev01.html
folder: windows
---

### Overview

The `WebCacheV01.dat` database is used by the `Microsoft Internet Explorer` and
`Microsoft Edge` (legacy) web browsers to store browsing history, downloads,
cache, and cookies. The database is in the `Extensible Storage Engine (ESE)`
database file format.

In addition to browsing history, **access to local files, not necessarily
through a web browser, are also tracked in the `WebCacheV01.dat` database**.

### Information of interest

**Access to local files can be identified by the `file` `URI` scheme** (such
as `file:///<DRIVE_LETTER>:/folder/file`). Note that files accessed on remote
webservers may also be using the `file` `URI` scheme (usually only as
`file://` however).

For each entry, **the full path to the file or the URL, the timestamp of
access, and the visit count** will be referenced.

### Tool(s)

The [`NirSoft's BrowsingHistoryView`](https://www.nirsoft.net/utils/browsing_history_view.html)
(`KAPE` associated module `Nirsoft_BrowsingHistoryView`) can be used to parse,
among other web browsers artefacts, the `WebCacheV01.dat` database.

### References

  - [13Cubed - Windows Browser Artifacts Cheat Sheet](https://training.13cubed.com/downloads)
