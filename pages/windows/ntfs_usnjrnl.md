---
title: NTFS - UsnJrnl
summary: 'The USN Journal is a feature of NTFS, activated by default on Vista and later, which maintains a record of changes made to the NTFS volume.\n\n The $J stream stores the actual change log records, with usually historical data of the last few days.\n\n Each change log record is notably composed of: the timestamp, filename, and reason / operation of the change.\n\n Additionally, each change log record contains the MFT "entry" and "sequence" numbers and the direct parent "entry" and "sequence" numbers of the file concerned by the change, making it is possible to retrieve the location of the file using the MFT. The UsnJrnl can be "rewinded" to exhaustively and accurately rebuild the location of every files in the journal.'
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

  - The `$Max` stream stores the metadata of the change.

  - The `$J`  stream stores the actual change log records.

Each change log record is notably composed of:

  - an `Update Sequence Number (USN)`.

  - The timestamp of the change.

  - The reason / operation of the record (`USN_REASON_FILE_CREATE`,
    `USN_REASON_FILE_DELETE`, `USN_REASON_DATA_OVERWRITE`,
    `USN_REASON_RENAME_NEW_NAME`, etc.).

  - The filename of the file impacted by the change.

  - The `MFT` `entry` and `sequence` numbers of the file impacted by the change,
    as well as its parent's `MFT` `entry` and `sequence` numbers.

#### Deducing an entry path using the MFT

As each change log record contains the `MFT` `entry` and
`sequence` numbers and the direct parent `entry` and `sequence` numbers of the
file concerned by the change, it is possible to retrieve the location of the
file using the `MFT`.

However, as the files and parent folder(s) of files referenced in the `UsnJrnl`
may have been deleted and may no longer be present in the `MFT`, simply
searching for records with matching `entry` and `sequence` numbers in the `MFT`
can lead to unresolved or inconsistent paths. Indeed, when a file is deleted
from the volume, its associated `MFT` `file record` is set as no longer in use
and the metadata of the `file record` are overwritten with that of the new
entry (when the `file record` gets re-used). The `MFT` `file record` `entry`
number remains unchanged with only the `file record` `sequence` number being
increased by one.

Thus, matching the `UsnJrnl` to the current state of the `MFT` will result:

  - In some invalid paths if only the `file record`'s `entry` numbers are used
    to find the files location. For example, if the `file record` of the folder
    "C:\Windows\Temp\TMP" is re-used (after deletion of the "TMP" folder) for
    the folder "C:\Users\user\Other", only using the `entry` numbers would lead
    to locating the files from the "TMP" folder to the current "Other" folder
    instead.

  - In some unknown paths if both the `file record`'s `entry` and `sequence`
    numbers are used, as deleted entries referenced in the `UsnJrnl` have an
    `entry` and `sequence` numbers combination that no longer exist in the
    `MFT`.

Additionally, renaming or moving files or folders does not change the `MFT`
`file record` `entry` or `sequence` numbers, potentially leading to invalid
paths if only the `MFT` is used to the location of files referenced in the
`UsnJrnl`.

A technique to exhaustively and accurately rebuild the location of files found
in the `UsnJrnl` is implemented by
[usnjrnl_rewind](https://github.com/CyberCX-DFIR/usnjrnl_rewind).
`usnjrnl_rewind` "rewinds" the `UsnJrnl`, reading the journal in reverse (from
the last, most recent change to the first, oldest change) and keeping stateful
information (in a local `SQL` database) about every entry's `entry` and
`sequence` numbers and parent's `entry` and `sequence` numbers. Using this
technique it is thus possible to rebuild a file location as it was when the
change operation ocurred. Indeed every entry's `entry` and `sequence` numbers
in the file's path are either in the `MFT` (if unchanged since the operation)
or referenced by an operation in the `UsnJrnl` (as changes impacting the file
location would have occurred after the file change operation).

More details on this technique can be found in
[CyberCX Blog's NTFS Usnjrnl Rewind](https://cybercx.com.au/blog/ntfs-usnjrnl-rewind/).

```bash
python usnjrnl_rewind.py -m "<MFT_CSV>" -u "<USNJRNL_CSV>" "<OUTPUT_FOLDER>"
```

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

  - [CyberCX Blog's NTFS Usnjrnl Rewind](https://cybercx.com.au/blog/ntfs-usnjrnl-rewind/)
