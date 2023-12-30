---
title: NTFS $MFT, $MFTMir, and $Bitmap
keywords: Filesystem, MFT, MFTMir, LogFile, Bitmap, $MFT, $MFTMir, $Bitmap
tags:
  - windows_ntfs_filesystem
  - windows_file_knowledge
location: '<SYSTEMDRIVE>:\$MFT\n\n<SYSTEMDRIVE>:\$MFTMir\n\n<SYSTEMDRIVE>:\$Bitmap'
last_updated: 2023-12-30
sidebar: sidebar
permalink: windows_mft.html
folder: windows
---

### Overview

The `Master File Table (MFT)`, filename `$MFT`, is the main element of any
`NTFS` partition.

The `MFT` contains an entry for all existing files written on the partition.
Deleted files that were once written on the partition may also still
(temporally) have a record in the `MFT`.

The `Partition Boot Sector` `$Boot` metadata file, which starts at sector 0 and
can be up to 16 sectors long, describes the basic `NTFS` volume information and
indicates the location of the `$MFT`.

The `$MFTMirr` file is statically-located as the first entry in the `MFT` and
contains the first 4 entries of the `MFT` (`MFT`, `$MFTMir`, `$LogFile`, and
`$Volume`) as a recovery mechanism.

The `$Bitmap` file tracks the allocation status (allocated or unused) of the
clusters of the volume. Each cluster is associated with a bit, set to `0x1` if
the cluster is in use. Upon deletion of a `non resident file`, the `$Bitmap`
file is updated to tag the cluster(s) associated with the file as free. The
clusters are not overwritten during the deletion process, and the file data can
thus be carved as long as the cluster(s) are not re-used.

The `$MFT`, `$MFTMirr`, and `$Bitmap` files have both the `Hidden (H)` and
`System (S)` attributes and will thus not be shown by the Windows Explorer
application or the `dir` utility by default.

### Information of interest

Each file on an `NTFS` volume is represented in the `MFT` in a file record.

Small files and directories (typically 512 bytes or smaller), can be entirely
contained within their associated `MFT` file record. These files are called
`resident files`. Files larger than that threshold are written on allocated
clusters, and are called `non resident files`.

Directory records are stored within the master file table just like file
records. Instead of data, directories contain index information.

A file record (`FILE0` data structure) notably includes:

  - The filename.

  - The file size.

  - The file unique (under the `NTFS` volume) `Security ID` in the
    `$STANDARD_INFORMATION` attribute.

  - Two or three set of timestamps:
    - The file creation, last modified, last accessed, last changed `SI`
      timestamps (`MACB`) in the `$STANDARD_INFORMATION` attribute.
    - The file creation, last modified, last accessed, last changed `FN`
      timestamps (`MACB`) in the `$FILE_NAME` attribute. Two sets of
      `$FILE_NAME` timestamps will be available for files with a short
      (`DOS`) and long filenames.

 - File access permissions.

 - One or multiple `DATA` attribute, that either contain the file data for
   `resident file` or reference the clusters of disk space where the file is
   stored for `nonresident file`.

 - Whether the `file record` is in use. When a file is deleted from the volume,
   its associated `MFT` `file record` is set as no longer in use, but is not
   directly deleted during the file deletion process. Metadata information, and
   content for `MFT` resident files, can thus be retrieved for recently deleted
   files (as long as the `file record` is not overwritten by a new entry).

#### $STANDARD_INFORMATION vs $FILE_NAME

The `$STANDARD_INFORMATION` and `$FILE_NAME` attributes are updated
differently for the same file action. The changes produced on the attributes
for a file creation, access, modification, renaming, etc. can be found on the
[SANS `Windows Forensic Analysis` poster](https://www.sans.org/security-resources/posters/windows-forensic-analysis/170/download).

### Tool(s)

#### MFTECmd

The [`MFTECmd`](https://github.com/EricZimmerman/MFTECmd) utility can parse and
extract information from the `$MFT` (as well as other NTFS filesystem artefacts
such as the `UsnJrnl`'s `$J` stream, the file ownership `$Secure:$SDS` data
stream, and the transaction log file `$Logfile`).

Associated `KAPE` compound module: `MFTECmd` (includes `MFTECmd_$Boot`,
`MFTECmd_$MFT`, `MFTECmd_$J`, and `MFTECmd_$SDS`).

```bash
# A $MFT file on a mounted partition should be specified.
# For instance, to extract $MFT data from a forensics image, the image should first be mounted and the $MFT specified as <DRIVER_LETTER:\$MFT to MFTECmd.exe.

MFTECmd.exe -f '<$MFT_FILE>' --csv <OUTPUTDIR_PATH>
```

#### Mft2Csv

The [`Mft2Csv`](https://github.com/jschicht/Mft2Csv) utility can parse, decode,
and log information from the MFT to a CSV. It supports getting the `$MFT` from
a variety of sources and notably:
  - a raw/dd image of disk or partition.
  - an extracted `$MFT` file.
  - a live host.

Note that `Mft2Csv` can only output in one format at a time.

```bash
# Opens a GUI.
Mft2Csv.exe

# Command line.
Mft2Csv.exe /Volume:<NTFS_VOLUME> /OutputPath:"<OUTPUT_FOLDER>" /OutputFormat:all /Separator:"<CSV_SEPARATOR>"
Mft2Csv.exe /MftFile:<MFT_FILE> /OutputPath:"<OUTPUT_FOLDER>" /OutputFormat:all /Separator:"<CSV_SEPARATOR>"
```

### References

  - [Velociraptor - NTFS Analysis](https://docs.velociraptor.app/docs/forensic/ntfs/)

  - [13Cubed - Introduction to MFTECmd - NTFS MFT and Journal Forensics](https://www.youtube.com/watch?v=_qElVZJqlGY)

  - [13Cubed - Anatomy of an NTFS FILE Record - Windows File System Forensics](https://www.youtube.com/watch?v=l4IphrAjzeY)

  - [13Cubed - A File's Life - File Deletion and Recovery](https://www.youtube.com/watch?v=4zlk9ZSMa-4)
