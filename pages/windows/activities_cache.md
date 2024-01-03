---
title: Windows 10 Timeline / ActivitiesCache.db
summary: 'Introduced in Windows 10 version 1803, the Windows Activity history tracks a number of operations on the system: programs used, local files opened, SharePoint documents consulted, and websites browsed (using Internet Explorer / Microsoft Edge Legacy).\n\nThe ActivitiesCache.db database only stores data for the last 30 days by default.\n\nInformation of interest, that depends on the activity type: start and end times of the activity (in UTC), executable full path for program execution, file name / SharePoint link for files accessed using certain programs, created and last modified timestamp of the associated file, etc.'
keywords: Windows 10 Timeline, ActivitiesCache.db, ActivitiesCache
tags:
  - windows_program_execution
  - windows_files_and_folders_access
  - windows_file_knowledge
  - windows_browsing_history
location: '<SYSTEMROOT>\Users\<USERNAME>\AppData\Local\ConnectedDevicesPlatform\[L.<USERNAME> | *]\ActivitiesCache.db'
last_updated: 2024-01-01
sidebar: sidebar
permalink: windows_activities_cache.html
folder: windows
---

### Overview

**Introduced in `Windows 10` version 1803**, the Windows Activity history
tracks a number of operations on the system: programs used, local files opened,
SharePoint documents consulted, and websites browsed (using Internet Explorer /
Microsoft Edge Legacy). The Activity history can be consulted in the Windows
Timeline (Windows + Tab keys).

The `ActivitiesCache.db` is a `SQLite` database that locally stores the
activity for its associated user.

The `ActivitiesCache.db` only stores data for the last 30 days by default.

### Information of interest

The `ActivitiesCache.db` is composed of a number of tables, with the following
tables being of interest:

  - `Activity` / `ActivityOperation` tables.

    - Data about various activities for different operation / activity type:
      - Program execution and opening of a file (5, `ExecuteOpen`)
      - Copy-pasting from a program (`CopyPaste`)
      - Application "in focus" (`InFocus`)
      - ...

    - The data available depends on the activity type:
      - Start (`startedDateTime`) and end (`lastActiveDateTime`) timestamps of
        the activity (in `UTC`).
      - The activity ID (GUID).
      - Executable full path for program execution.
      - For certain programs, the accessed file name and path and / or
        associated SharePoint link.
      - Created and last modified timestamp of the associated file (local or
        on SharePoint).
      - The user's device timezone.
      - ...

    - An activity data can be present in either or both tables depending on the
      activity lifecycle. For example, a new activity will only be present in
      the `Activity` table, while an activity in the "upload queue" will be
      placed in the `ActivityOperation` table.

  - `Activity_PackageId`:

    - Data about the application(s) / program(s) linked to a specific activity
      (identified by its activity ID).

    - Data available:
      - The activity ID (GUID).
      - The application name / program filename.
      - Eventual program full path.
      - Activity expiration timestamp (timestamp of occurrence + 30 days by
        default).

   - Upon occurrence of an activity, one or multiple entries sharing the same
     activity ID will be created in the `Activity_PackageId` table, one for
     each program / application related to the activity.

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
