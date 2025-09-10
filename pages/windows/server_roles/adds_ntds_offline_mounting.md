---
title: Procedure - Active Directory Domain Services NTDS offline mounting
summary: 'An exported NTDS database file can be mounted using dsamain, to then be queried in LDAP or with PowerShell cmdlets through the "AD Web Services".\n\n '
keywords: Active Directory, Active Directory Domain Services, Domain Controllers, DC, replication metadata, msDS-ReplAttributeMetaData, msDS-ReplValueMetaData, Get-ADReplicationAttributeMetadata, ADTimeline, FarsightAD, esentutl, dsamain.exe
tags:
  - windows_active_directory
  - procedures
location: 'NTDS database location: %SystemRoot%/ntds\NTDS.DIT\n\n Commands for offline mounting:\n esentutl /p <NTDS_DIT_PATH>\n dsamain.exe -dbpath <NTDS_DIT_PATH> -ldapport 3266 -allownonadminaccess'
last_updated: 2024-09-08
sidebar: sidebar
permalink: adds_ntds_offline_mounting.html
folder: windows
---

### Overview and pre-requisites

An exported NTDS database file can be mounted using `dsamain`, to then be
queried in LDAP or with PowerShell cmdlets through the `AD Web Services`
(ADWS).

For instance, the `Get-ADReplicationAttributeMetadata` cmdlet or `ADTimeline`
can be used on a "mounted" NTDS database to retrieve historical
[AD replication metadata](./adds_replication_metadata.md) from backup images.

The following pre-requisites are required to mount a NTDS database:

  1. The Windows Server major version used must be superior or equal to the
     Windows Server version of the Domain Controller the NTDS is coming
     from.

  2. The `Active Directory Lightweight Directory Services` (`AD LDS`) server
     role must be installed.

  3. The `Remote Server Administration Tools` (`RSAT`) must be installed and
     the `Active Directory Web Services` service enabled and running.

     Note: the `ADWS` service may need to be restarted after "mounting" the
     NTDS database.
     
     ```bash
     Set-Service -Name "ADWS" -StartupType Manual

     Stop-Service -Name "ADWS"
     Start-Service -Name "ADWS"
     ```

### NTDS "mounting"

If needed, for instance if the NTDS database was exported while the AD DS
service was running, the NTDS must be repaired using `esentutl`:

```bash
# /r: Replays the last transaction logs (files "edb*.log") from the current folder, if any, to apply the last AD objects changes.
esentutl /r edb

# /p: Repairs the specified NTDS database.
esentutl /p <NTDS_DIT_PATH>
```

Then the `dsamain` utility can be used to "mount" the NTDS (in a clean state):

```bash
# Mount the specified NTDS database, exposing a LDAP service on port 3266 (and allowing access through the ADWS service).

dsamain.exe -dbpath <NTDS_DIT_PATH> -ldapport 3266 -allownonadminaccess
```

### Accessing AD objects from the mounted NTDS

The objects from the mounted NTDS database can be accessed through LDAP or
ADWS, generally using the same tools as one would use on a live AD domain
(with some caveats).

#### RSAT module

The cmdlets from the RSAT AD module can be used to query the mounted NTDS
through the ADWS:

```bash
$PSDefaultParameterValues.Add("*-AD*:Server", "127.0.0.1:3266")

Get-AD*
```

#### Human-readable ACL using Get-ACL

While RSAT cmdlets or GUI tools can be used to retrieve ACL in a `SDDL`
notation, the `Get-Acl` cmdlet can be used to query and retrieve ACL in an
human-readable format:

```bash
# A custom AD PS drive is needed in order to use Get-Acl on a mounted NTDS database.
New-PSDrive -Name ADOffline -PSProvider ActiveDirectory -Root "//RootDSE/" -Server 127.0.0.1:3266

Get-Acl "ADOffline://<OBJECT_DN> | Select -ExpandProperty Access
```

#### AD Replication metadata

The `Get-ADReplicationAttributeMetadata` PowerShell cmdlet and
[ADTimeline](https://github.com/ANSSI-FR/ADTimeline) can be used to retrieve
[AD replication metadata](./adds_replication_metadata.md):

```bash
Get-ADReplicationAttributeMetadata -Server "127.0.0.1:3266" -IncludeDeletedObjects –ShowAllLinkedValues "<DISTINGUISHED_NAME>"

.\AD-timeline.ps1 -server "127.0.0.1:3266"
```

#### Hunting with BloodHound

While `SharpHound` can not be directly used on a mounted NTDS database, a
snapshot taken with
[`ADExplorer`](https://learn.microsoft.com/en-us/sysinternals/downloads/adexplorer)
can be converted to JSON files compatible with `BloodHound` using
[ADExplorerSnapshot.py](https://github.com/c3c/ADExplorerSnapshot.py):

```bash
ADExplorer -> File -> Connect -> Connect to: 127.0.0.1:3266, no username or password -> OK
    Select the 127.0.0.1:3266 instance -> File -> Create snapshot -> Enter the desired snapshot filename -> OK

ADExplorerSnapshot.py [-o <OUT_DIRECTORY>] -m BloodHound <ADEXPLORER_SNAPSHOT>
```
