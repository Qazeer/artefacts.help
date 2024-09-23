---
title: User Access Logging / SUM
summary: 'Introduced in Windows Server 2012 and enabled by default, User Access Logging (UAL) is a feature that consolidates data on user access to Windows Server roles (such as "Active Directory Domain Services" on Domain Controllers).\n\nUAL store data for the 2 years.\n\nInformation of interest:\n- Accessed Windows Server role (such as ADDS, CIFS, ADCS, etc.)\n- The client domain and username.\n The client IPv4 or IPv6 address.\n- First, last, and daily access timestamps.\n- Total number of access.\n\nAs machine accounts of domain-joined systems also authenticate to ADDS, UAL of Domain Controllers can be used to map hostnames with past IP addresses.'
keywords: User Access Logging, SUM, UAL, DNS, Current.mdb
tags:
  - windows_lateral_movement
  - windows_lateral_movement_dst
  - windows_active_directory
location: 'Files under <SYSTEMROOT>\System32\Logfiles\SUM\ folder:\n\nCurrent.mdb (data for the last 24-hours).\nUp to three "<GUID>.mdb" files (current year and history up to 2 years).\nSystemidentity.mdb (mapping on roles GUIDs and names).'
last_updated: 2024-01-01
sidebar: sidebar
permalink: windows_ual.html
folder: windows
---

### Overview

`User Access Logging (UAL)` is a feature introduced, and enabled by default, in
`Windows Server 2012` that consolidates data on client activity. Among other
information, user access on specific Windows Server roles (such as
`Active Directory Domain Services` on Domain Controller) are logged by the
`UAL`. The specific activity triggering an entry to be logged for a given role
is not documented.

On Domain Controllers, `UAL` thus yield information on **sessions opening on
domain-joined computers** (if the given DC was reached for authentication or
`Group Policy` retrieval).

The information is stored locally in up to five (5)
`Extensible Storage Engine (ESE)` database files (`.mdb`):

  - The `Current.mdb` file which contains data for the last 24-hour.

  - Up to three `<GUID>.mdb` files, which contain data for an entire year
    (first to last day), going back to 2 years. The data in the `Current.mdb`
    database is copied each day to the corresponding (`<GUID>.mdb`) database
    for the current year.

  - `Systemidentity.mdb` which contains metadata on the local server, including
    a mapping on roles' GUIDs and names.

**Historical data going back to 2 years may thus be retrieved in the `UAL`
database files**.

### Information of interest

The `CLIENTS` table of the `Current.mdb` and `<GUID>.mdb` database files
contain multiple information of interest:

  - Accessed Windows Server role `GUID` and description. Among others, the
    following roles can be encountered:
      - `Active Directory Domain Services` (GUID:
        `ad495fc3-0eaa-413d-ba7d-8b13fa7ec598`).
      - `File Server` (GUID: `10a9226f-50ee-49d8-a393-9a501d47ce04`).
      - `Active Directory Certificate Services` (GUID:
        `c50fcc83-bc8d-4df5-8a3d-89d7f80f074b`).

  - The client domain and username.

  - Total number of access.

  - First, last, and daily access timestamps.

  - The client `IPv4` or `IPv6` address. On Domain Controllers, the hostname
    associated the `IP` address at that time may be retrievable as machine
    accounts of domain-joined computers also authenticate on `AD DS`.

Each entry in the `CLIENTS` table is composed of a unique set of a Windows
Server role, a client's domain / username, and a source `IP` address.

The `DNS` table of the aforementioned database files contain information about
`DNS` resolutions: hostname, associated `IP` address, and timestamp of last
resolution.

### Tool(s)

#### Live forensics

The PowerShell cmdlets of the `UserAccessLogging` module can be used to
retrieve `UAL` data on a live system:

```bash
# Enumerates the roles installed on the system.
Get-UalOverview

# Retrieves UAL data for user access (data stored in the CLIENTS table).
Get-UalUserAccess

# Retrieves UAL data for client access by device for a given service, ordered by date (data stored in the CLIENTS table).
# The cmdlets returns the date that the client accessed the service and how many times the client accessed the service during that day.
Get-UalDailyAccess

# Retrieves information on DNS resolutions (data stored in the DNS table).
Get-UalDns
```

#### UAL database files

##### Repairing the SUM databases

As the databases copied, through a `shadow copy` volume or using utilities
implementing raw disk reads (such as
[`Velociraptor`](https://github.com/Velocidex/velociraptor) or
[`RawCopy`](https://github.com/jschicht/RawCopy)), will not be in a
"clean state", the database files will have to be repaired.

This can be accomplished using the `esentutl` built-in utility:

```bash
# The following commands should be executed in the directory containing the UAL database files.

esentutl.exe /r sru /i

esentutl.exe /p <Current.mdb | UAL_DB_FILE>
```

The Windows operating system used to parse the `SUM` databases must be of the
same major version, or of a more recent version, that the system from which the
databases were acquired. For instance, `SUM` databases acquired from a
`Windows Server 2022` operating system must be parsed on `Windows 11` or
`Windows Server 2022`.

##### SumECmd

The Eric Zimmerman's [`SumECmd`](https://github.com/EricZimmerman/Sum) tool or
the [`KStrike`](https://github.com/brimorlabs/KStrike) Python script can be
used to parse repaired `UAL` database files:

```bash
# Parses the specified individual UAL database file.
KStrike.py <Current.mdb | UAL_DB_FILE>

# Parses the UAL database files (Current.mdb, SystemIdentity.mdb, etc.) in the specified directory.
# The results will be aggregated in single CSV files per category (client access, DNS requests, etc.).
SumECmd.exe --csv <CSV_DIRECTORY_OUTPUT> -d <DIRECTORY_WITH_UAL_DB_FILES>
```

The `PowerShell_SumECmd_SUM-RepairAndParse` `KAPE` module, leveraging the
PowerShell script
[`SUM-Repair.ps1`](https://github.com/AndrewRathbun/DFIRPowerShellScripts/blob/main/SUM-Repair.ps1)
can be used to automate the repairing and parsing process (with `SumECmd`).

### References

  - [Microsoft - Get Started with User Access Logging](https://learn.microsoft.com/en-us/windows-server/administration/user-access-logging/get-started-with-user-access-logging)

  - [CrowdStrike - Patrick Bennett - UAL Thank Us Later: Leveraging User Access Logging for Forensic Investigations](https://www.crowdstrike.com/blog/user-access-logging-ual-overview/)

  - [13Cubed - User Access Logging (UAL) Forensics](https://www.youtube.com/watch?v=rVHKXUXhhWA)

  - *KPMG - UAL - DOWN*
