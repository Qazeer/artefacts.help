---
title: NTFS - UsnJrnl
summary: 'The USN Journal is a feature of NTFS, activated by default on Vista and later, which maintains a record of changes made to the NTFS volume.\n\nThe $J stream stores the actual change log records, with usually historical data of the last few days.\n\nEach change log record is notably composed of: the timestamp, filename, and reason / operation of the change.'
keywords: Filesystem, NTFS, UsnJrnl, USN, Journal
tags:
  - windows_ntfs_filesystem
  - windows_file_knowledge
location: '$Max and $J named data streams under <ROOT>\$Extend\$UsnJrnl'
last_updated: 2024-01-01
sidebar: sidebar
permalink: windows_usnjrnl.html
folder: windows
---

### Overview

The `Update Sequence Number Journal (USN) Journal` is a feature of NTFS,
activated by default on Vista and later, which maintains a record of changes
made to the NTFS volume. The creation, deletion or modification of files or
directories are for instance journalized.

The records in the `UsnJrnl` are progressively overwritten once the max size of
the journal has been reached. The `UsnJrnl` usually contains historical data on
the last few days (1-3 days for system full time use, < 7 days for regular
system use).

### Information of interest

The `UsnJrnl` is composed of two named data streams:

  - The `$Max` stream stores the meta data of the change.

  - The `$J`  stream stores the actual change log records.

Each change log record is notably composed of:

  - an `Update Sequence Number (USN)`.

  - The timestamp of the change.

  - The filename of the file impacted by the change.

  - The reason / operation of the record (`USN_REASON_FILE_CREATE`,
    `USN_REASON_FILE_DELETE`, `USN_REASON_DATA_OVERWRITE`,
    `USN_REASON_RENAME_NEW_NAME`, etc.).

  - MFT reference and reference sequence number.

### Tool(s)

#### UsnJrnl metadata

The Windows `fsutil` and the PowerShell cmdlet `Get-ForensicUsnJrnlInformation`
of the [`PowerForensics`](https://github.com/Invoke-IR/PowerForensics) suite
can be used to retrieve metadata about the `UsnJrnl`:

```
# First and current USN, maximum size notably
fsutil usn queryjournal <NTFS_VOLUME>

Get-ForensicUsnJrnlInformation
Get-ForensicUsnJrnlInformation -VolumeName <NTFS_VOLUME>
Get-ForensicUsnJrnlInformation -Path <USN_JRNL_PATH>
```

#### MFTECmd

The [`MFTECmd`](https://github.com/EricZimmerman/MFTECmd) utility can parse and
extract information from the `UsnJrnl` (as well as other NTFS filesystem
artefacts such as the `MFT`'s `$J` stream, the file ownership `$Secure:$SDS`
data stream, and the transaction log file `$Logfile`).

Associated `KAPE` compound module: `MFTECmd` (includes `MFTECmd_$Boot`,
`MFTECmd_$MFT`, `MFTECmd_$J`, and `MFTECmd_$SDS`).

```bash
MFTECmd.exe -f '<UsnJrnl>' --csv <OUTPUTDIR_PATH>
```

### References

  - [Junghoon Oh - Advanced $UsnJrnl Forensics](http://forensicinsight.org/wp-content/uploads/2013/07/F-INSIGHT-Advanced-UsnJrnl-Forensics-English.pdf)

  - [COUNT UPON SECURITY - Luis Rocha - DIGITAL FORENSICS NTFS CHANGE JOURNAL](https://countuponsecurity.com/2017/05/25/digital-forensics-ntfs-change-journal/)
