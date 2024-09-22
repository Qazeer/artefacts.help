---
title: ETW - Active Directory Domain Services (Domain Controllers) ntds.dit dumping
summary: 'Secrets stored in the Active Directory database (ntds.dit) can be retrieved a number of ways:\n\n- By leveraging the DRSUAPI replication functions, normally used by Domain Controllers to replicate objects (replicated) properties. This attack can be conducted over the network (with out executing code on a Domain Controller) and is known as "DCSync".\n\n - By executing code / commands on a Domain Controller and exfiltrating the ntds.dit database directly. While the ntds.dit database can be accessed and copied using various tools and techniques, the "ntdsutil" built-in administration utility is often leverage by threat actors to do so.'
keywords: Active Directory, Domain Controllers, NTDS, ntds.dit, DRSUAPI, DCSync, 4662, 1131f6aa-9c07-11d1-f79f-00c04fc2dcd, 1131f6ad-9c07-11d1-f79f-00c04fc2dcd2, ntdsutil, ESENT, 325, 326, 327, 206
tags:
  - windows_etw
  - windows_misc
location: 'DCSync (DRSUAPI):\nChannel: Security.\nEvent: 4662 (Property "1131f6aa-9c07-11d1-f79f-00c04fc2dcd" or "1131f6ad-9c07-11d1-f79f-00c04fc2dcd2").\n\n NTDS export using ntdsutil:\nChannel: ESENT (Application.evtx).\nEvents: 206, 325, 326, 327'
last_updated: 2024-07-09
sidebar: sidebar
permalink: windows_etw_ad_ntds_dumping.html
folder: windows
---

### Overview

Secrets stored in the Active Directory database (`ntds.dit`) can be retrieved a
number of ways:

  - By leveraging the `DRSUAPI` replication functions, normally used by
    `Domain Controllers` to replicate objects (replicated) properties. This
    attack can be conducted over the network (with out executing code on a
    `Domain Controller`) and is known as `DCSync`.

  - By executing code / commands on a `Domain Controller` and exfiltrating
    the `ntds.dit` database directly. While the `ntds.dit` database can be
    accessed and copied using various tools and techniques, the `ntdsutil`
    built-in administration utility is often leverage by threat actors to
    do so.

### DCSync (DRSUAPI)

The `DCSync` attack consists in leveraging the Active Directory `DRSUAPI`
replication functions (part of the `Directory Replication Service (DRS)`
protocol) to remotely retrieve the specified Active Directory objects' sensible
information. The `DRSUAPI` functions are normally used by the
`Domain Controllers` to replicate the modifications made to AD objects and keep
the AD objects consistent across all the `Domain Controllers` of the forest.
The `DRSUAPI` replication functions are exposed on the network by the
`Microsoft Remote Procedure Call (MSRPC)` `DRSUAPI` interface on each `Domain
Controller`. Thus, contrary to the others methods explicated so far, no local
code execution on a `Domain Controller` is required to retrieve information
from the `ntds.dit` database.

While multiples `DRSUAPI` intermediate functions are used in the replication
process, the `DSGetNCChanges` function implements the replication request.

The following privileges on the `domain root object` are necessary to make
replication requests through the `DRSUAPI`:
  - Replicating Directory Changes (`Ds-Replication-Get-Changes`,
    `ACE GUID: 1131f6aa-9c07-11d1-f79f-00c04fc2dcd2`)
  - Replicating Directory Changes All (`Ds-Replication-Get-Changes-All`,
    `ACE GUID: 1131f6ad-9c07-11d1-f79f-00c04fc2dcd2`)

Those privileges are, in a default Active Directory configuration, granted to
the `Domain Controllers`, `ENTERPRISE DOMAIN CONTROLLERS`, `Domain Admins`,
`Enterprise Admins` and `Administrators` domain groups.

| Channel | Conditions | Events |
|---------|------------|--------|
| `Security` | Default configuration. <br><br> Events will be generated only if the operation was not conducted under a `Domain Controller` identity. | Upon replication operations, such as the retrieval of Active Directory secrets (`DCSync` attack), the following event will be generated: <br><br> - Event `4662`: `An operation was performed on an object` with the `Property` attribute equal to the `GUID` `1131f6aa-9c07-11d1-f79f-00c04fc2dcd2` or `1131f6ad-9c07-11d1-f79f-00c04fc2dcd2`. |

### ntdsutil usage

If code execution is achieved on a `Domain Controller`, multiples Windows
utilities can be used to export the `ntds.dit` database, file:
`%SystemDrive%\Windows\NTDS\ntds.dit` (given sufficient privileges).

On a standard `Domain Controller` installation, the `NT AUTHORITY\SYSTEM`
built-in Windows account and the `Administrators` domain group have
`full control` access on the file. Additionally, yet again in a standard
configuration, members of the `Backup Operators` (`SID: S-1-5-32-551`) domain
group have the necessary privileges to open an interactive (and remote) session
on the `Domain Controllers` (`SeInteractiveLogonRight`) and can make use of the
`SeBackupPrivilege` privilege to open files with the
`FILE_FLAG_BACKUP_SEMANTICS` flag in order to bypass the file access
controls.

As the `ntds.dit` file is being continuously accessed, the file cannot be
directly copied ("The action can't be completed because the file is open in
another program"). The copy must be done through the Windows `shadow copy`
mechanism, which leverage a temporary freezing of the I/O requests on the
file. The freezing is requested by the `Volume Shadow Copy Service (VSS)`
Windows built-in service, which orchestrate the creation of the `shadow copy`.

The sensitive information in the `ntds.dit` file are encrypted using the system
`Boot Key` (also known as the `System Key`, or `SysKey`). This key is located
in the `HKEY_LOCAL_MACHINE\SYSTEM` registry hive
(`%SystemDrive%\Windows\system32\config\SYSTEM` file) and is unique to each
`Domain Controller`. The `SYSTEM` registry hive (or the `Boot Key` directly)
must thus be exported from the `Domain Controller` the `ntds.dit` was copied
from.

the `ntdsutil` built-in administration utility is often leverage by threat
actors to export the `ntds.dit` database and the `SECURITY` and `SYSTEM`
registry hives.

| Channel | Conditions | Events |
|---------|------------|--------|
| Channel: <br> `ESENT` <br><br> EVTX file: <br> `Application.evtx` | Default configuration. | Upon execution of the `ntdsutil` command to dump the Active Directory `ntds.dit` database, the following events (containing the `ntds` keyword) will be generated: <br><br> - Event `325`: `The database engine created a new database [...]` <br><br> - Event `326`: `The database engine attached a database [...]` <br><br> - Event `327`: `The database engine detached a database [...]` <br><br> - Event `206`: `A database location change was detected [...]` |

### References

  - [adsecurity - Mimikatz DCSync Usage, Exploitation, and Detection](https://adsecurity.org/?p=1729)

  - [Microsoft - drsuapi RPC Interface](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-drsr/58f33216-d9f1-43bf-a183-87e3c899c410)

  - [SambaWiki - DRSUAPI](https://wiki.samba.org/index.php/DRSUAPI)

  - [giuliano108 - SeBackupPrivilege](https://github.com/giuliano108/SeBackupPrivilege/blob/master/README.md)
