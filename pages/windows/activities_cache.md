---
title: Windows 10 Timeline / ActivitiesCache.db
summary: 'Introduced in Windows 10 version 1803, the Windows Activity history tracks a number of operations on the system: programs used, local files opened, SharePoint documents viewed, and websites browsed (using Internet Explorer/Microsoft Edge Legacy).\n\nThe ActivitiesCache.db database only stores data for the last 30 days by default.\n\nInformation of interest, which depends on the activity type: start and end times of the activity (in UTC), executable full path for program execution, file name/SharePoint link for files accessed using certain programs, creation and last modification timestamps of the associated file, etc.\n\nThe history of the clipboard data may also be stored for a short amount of time (approximately 12 hours) in a non-default configuration.'
keywords: Windows 10 Timeline, ActivitiesCache.db, ActivitiesCache
tags:
  - windows_program_execution
  - windows_files_and_folders_access
  - windows_file_knowledge
  - windows_browsing_history
  - windows_misc
location: '<SYSTEMROOT>\Users\<USERNAME>\AppData\Local\ConnectedDevicesPlatform\[L.<USERNAME> | *]\ActivitiesCache.db'
last_updated: 2024-01-01
sidebar: sidebar
permalink: windows_activities_cache.html
folder: windows
---

### Overview

**Introduced in `Windows 10` version 1803**, the Windows Activity history
tracks a number of operations on the system: programs used, local files opened,
SharePoint documents viewed, and websites browsed (using Internet Explorer /
Microsoft Edge Legacy). The Activity history can be viewed in the Windows
Timeline (Windows + Tab keys).

The `ActivitiesCache.db` is a `SQLite` database that locally stores the
activity for its associated user.

The `ActivitiesCache.db` only stores data for the last 30 days by default.

### Information of interest

The `ActivitiesCache.db` is composed of a number of tables, with the following
tables being of interest:

  - `Activity`/`ActivityOperation` tables.

    - Data about various activities for different operation/activity types:
      - Program execution and opening of a file (5, `ExecuteOpen`)
      - Copy-pasting from a program (`CopyPaste`)
      - Application "in focus" (`InFocus`)
      - ...

    - The data available depends on the activity type:
      - Start (`startedDateTime`) and end (`lastActiveDateTime`) timestamps of
        the activity (in `UTC`).
      - The activity ID (GUID).
      - Executable full path for program execution.
      - For certain programs, the accessed file name and path and/or
        associated SharePoint link.
      - Creation and last modification timestamps of the associated file
        (local or on SharePoint).
      - The user's device timezone.
      - ...

    - Activity data can be present in either or both tables depending on the
      activity lifecycle. For example, a new activity will only be present in
      the `Activity` table, while an activity in the "upload queue" will be
      placed in the `ActivityOperation` table.

  - `Activity_PackageId`:

    - Data about the application(s)/program(s) linked to a specific activity
      (identified by its activity ID).

    - Data available:
      - The activity ID (GUID).
      - The application name/program filename.
      - Program full path (if any).
      - Activity expiration timestamp (timestamp of occurrence + 30 days by
        default).

    - Upon occurrence of an activity, one or multiple entries sharing the same
      activity ID will be created in the `Activity_PackageId` table, one for
      each program/application related to the activity.

#### Clipboard activity

The `Activity`/`ActivityOperation` tables reference clipboard activity
through two activity types: `10` and `16`. The events will be stored for
approximately 12 hours after the operation.

The activity type `16` tracks the use of the clipboard by an application, and
is logged by default. The activity type `10` tracks the clipboard content, and
is only logged if the non-default "Clipboard history" and "Sync across your
devices" clipboard settings are both enabled. The latter setting is only
available for Microsoft or Entra ID-synchronized accounts (and thus not
available for local or simple Active Directory domain accounts).

If the prerequisites are met, an activity type `10` should be logged
immediately after an activity type `16`, allowing correlation of the two
events.

The following information of interest is available for activity type `10`
events:

  - The timestamp at which the data was copied (in the `CreatedTime` column, as
    an `epoch` timestamp).

  - The content of the clipboard, encoded in base64 (in the `ClipboardPayload`
    column).

The following information of interest is available for activity type `16`
events:

  - The timestamp at which the data was copied (in the `CreatedTime` column, as
    an `epoch` timestamp).

  - The application from which the data was copied (in the `AppId` column).

  - Whether the data was copied (`Copy`) or pasted (`Paste`) (in the `Group`
    column). However, only copy operations seem to generate an entry on recent
    versions of the Windows operating system.

### Tool(s)

The [`WxTCmd`](https://github.com/EricZimmerman/WxTCmd) utility (`KAPE`
`WxTCmd` module) can parse and extract information from the
`ActivitiesCache.db` database.

```bash
# Parses the specified ActivitiesCache database.
WxTCmd.exe -f <ActivitiesCache.db | ACTIVITIESCACHE_DB_FILE> --csv <OUTPUT_DIRECTORY>
```

### References

  - [kacos2000 - An examination of Win10 ActivitiesCache.db database](https://kacos2000.github.io/WindowsTimeline/WindowsTimeline.pdf)

  - [Istrosec - Zuzana Vargova - How to trace the user: Windows 10 Timeline](https://istrosec.com/blog/windows-10-timeline/)

  - [InverseCos - How to Perform Clipboard Forensics: ActivitiesCache.db, Memory Forensics and Clipboard History](https://www.inversecos.com/2022/05/how-to-perform-clipboard-forensics.html)

  - [Forensic Focus - Windows 10 Activity Timeline: An Investigator's Gold Mine](https://www.forensicfocus.com/webinars/windows-10-activity-timeline-an-investigators-gold-mine/)
