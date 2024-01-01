---
title: System Resource Usage Monitor (SRUM)
summary: 'Introduced in Windows 8, the System Resource Usage Monitor (SRUM) is a feature that records numerous metrics of system activities.\n\nThe SRUM database only stores data for the last 30 to 60 days.\n\nEntries are not associated with their timestamp of occurrence but with the timestamp of insertion in the SRUM database (every hour).\n\nInformation of interest: executable full path, executing user SID, metrics on CPU usage, I/O and network activity per execution.'
keywords: System Resource Usage Monitor, SRUM
tags:
  - windows_program_execution
  - windows_network_usage
location: '<SYSTEMROOT>\System32\SRU\SRUDB.dat'
last_updated: 2024-01-01
sidebar: sidebar
permalink: windows_srum.html
folder: windows
---

### Overview

**Introduced in Windows 8**, the `System Resource Usage Monitor (SRUM)` is a
feature that records numerous metrics of system activities, with a limited
subset of the available information displayed within the Windows `Task Manager`
("App history" tab).

The information is stored in the `SRUM` `ESE` database `SRUDB.dat`, with
historical data for the last 30 to 60 days only.

### Information of interest

Entries are not associated with their timestamp of occurrence but with the
timestamp of insertion in the `SRUM` database. As entries are only written to
the `SRUM` database every hour, timestamps are thus precise to the hour (with
multiple entries usually sharing the same insertion timestamp).

Among the various information stored, the following two tables hold the most
commonly valuable data for forensics investigations:

  - `Application Resource Usage` table (GUID
    `{D10CA2FE-6FCF-4F6D-848E-B2E99266FA89}`), that tracks programs
    execution.
    For each entry in the `Application Resource Usage` table (`SrumECmd`'s
    `AppResourceUseInfo` output), the following information may be recorded:

    - Timestamp of the `SRUM` entry creation.

    - Full path of the executable or application information / description for
      built-in components.

    - User `SID` of the user executing the process.

    - Metrics on CPU usage (CPU time in foreground and background).

    - Metrics on I/O operations (foreground / background number of read / write
      operations and bytes read / written).

  - `App Timeline Provider` table (GUID
    `{5C8CF1C7-7257-4F13-B223-970EF5939312}`), that also tracks programs
    execution.
    For each entry in the `Application Resource Usage` table (`SrumECmd`'s
    `AppTimelineProvider` output), the following information may be recorded:

    - Timestamp of the `SRUM` entry creation.

    - Name of the executable and description for built-in components.

    - Timestamp of compilation of the executable.

    - User `SID` of the user executing the process.

    - Timestamp of seemingly approximate end of execution.

    - Total duration of execution (in milliseconds).

  - `Network Data Usage` table (GUID `{973F5D5C-1D90-4944-BE8E-24B94231A174}`),
    that tracks programs execution and network usage of the executed
    programs.
    For each entry in the `Network Data Usage` table (`SrumECmd`'s
    `NetworkUsages` output), the following information may be
    recorded:

    - Timestamp of the `SRUM` entry creation.

    - Full path of the executable or application information / description for
      built-in components.

    - Metrics on network data usage (bytes sent and receive on a given network
      interface).

Some of the information recorded in the `SRUM` database be viewed using the
Windows `Task Manager` ("App history" tab).

### Tool(s)

#### Repairing the SRUDB.dat database

As the copied `SRUM` database will likely not be in a "clean state", the
database will have to be repaired. This can be accomplished using the
`esentutl` utility. It is recommended to make a copy of the `SRU` directory
before repairing the database.

```bash
# The following commands should be executed in the directory containing the UAL database files.

esentutl.exe /r sru /i

esentutl.exe /p SRUDB.dat
```

#### SrumECmd

The `SrumECmd` utility (`KAPE` `SrumECmd` module) can parse and extract
information from the `SRUDB.dat` database, and correlates information from the
`SOFTWARE` registry hive.

```bash
# Parses the specified SRUM database, using the optionally provided SOFTWARE registry hive.
SrumECmd.exe -f <SRUDB.dat | SRUM_DB_FILE> [-r <SOFTWARE>] --csv <OUTPUT_DIRECTORY>

# Recursively look for SRUDB.dat and SOFTWARE files in the specified directory.
SrumECmd.exe -d <DIRECTORY> --csv <OUTPUT_DIRECTORY>
```

### References

  - [SANS Internet Storm Center - Mark Baggett - System Resource Utilization Monitor](https://isc.sans.edu/diary/System+Resource+Utilization+Monitor/21927)

  - [13Cubed - Windows SRUM Forensics](https://www.youtube.com/watch?v=Uw8n4_o-ETM)

  - [srum-dump's SRUM_TEMPLATE2.xlsx - Mark Baggett](https://github.com/MarkBaggett/srum-dump/blob/master/SRUM_TEMPLATE2.xlsx)
