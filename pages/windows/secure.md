---
title: NTFS $Secure
summary: 'The $Secure file contains the security descriptor for all the files and folders on a NTFS volume.'
keywords: Filesystem, Secure, $Secure
tags:
  - windows_ntfs_filesystem
location: '<SYSTEMDRIVE>:\$Secure'
last_updated: 2023-12-30
sidebar: sidebar
permalink: windows_secure.html
folder: windows
---

### Overview

The `$Secure` file contains the `security descriptor` for all the files and
folders on a `NTFS` volume.

The `security descriptors` are stored within the `$SDS` named data stream of
the `$Secure` file. The `$Secure` file additionally defines two other named
streams (`$SDH` and `$SII`) for lookup in the `$SDS` stream.

### Information of interest

Each file or folder is referenced in the `$Secure` file with its volume-unique
`Security ID` and `security descriptor`.

The `Security ID` of the file is referenced in the `MFT` file record associated
with the file (in the `$STANDARD_INFORMATION` attribute). While no metadata
information are present in the `$Secure` file (only the file's
`security descriptor`), the file's `Security ID` can be used to map the file's
information / data from the `MFT` to its `security descriptor` in the `$Secure`
file.

The `security descriptor` (`SECURITY_DESCRIPTOR` data structure) references:

  - The owner of the file (as a pointer to a `SID` structure).

  - The access rights to the file in the
    `Discretionary Access Control List (DACL)` attribute.

 - The audit rights that control how access is audited (which access will
   generate events) in the `System Access Control List (SACL)` attribute.

### Tool(s)

  - [`Secure2Csv`](https://github.com/jschicht/Secure2Csv)

### References

  - [`Secure2Csv`'s README](https://github.com/jschicht/Secure2Csv)
