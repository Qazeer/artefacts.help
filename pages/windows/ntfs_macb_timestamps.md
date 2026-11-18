---
title: NTFS - MACB timestamps and timestomping
summary: 'On NTFS filesystems, each file possesses (at least) two attributes that hold (among other information) Modification, Access, Change, and Birth (MACB) timestamps: the $STANDARD_INFORMATION and $FILENAME attributes.\n\nThe $STANDARD_INFORMATION and $FILENAME timestamps are updated differently depending on the operation.\n\nTimestomping is the action of modifying the timestamps of a file, generally to evade detection.\n\nTimestomping may be detected using a number of techniques:\n - Identifying $STANDARD_INFORMATION timestamps older than $FILENAME timestamps.\n - Using UsnJrnl records.\n - Identifying non nano-second precise $STANDARD_INFORMATION timestamps.\n - Using MFT entry numbers.\n\nHowever, each technique is prone to false positives and false negatives.'
keywords: Filesystem, MACB timestamps, STANDARD_INFORMATION, $STANDARD_INFORMATION, FILENAME, $FILENAME, timestomping
tags:
  - windows_ntfs_filesystem
  - windows_defense_evasion
location: 'A given file may be associated with up to 20 timestamps: $STANDARD_INFORMATION + 2 * $FILENAME + 2 * NTFS $I30 $FILENAME (duplicate $FILENAME for files with short and long names).'
last_updated: 2024-01-04
sidebar: sidebar
permalink: windows_macb_timestamps.html
folder: windows
---

### NTFS $STANDARD_INFORMATION & $FILENAME MACB timestamps

On `NTFS` filesystems, each file possesses (at least) two attributes that hold
(among other information) `Modification, Access, Change, and Birth (MACB)`
timestamps:
  - `$STANDARD_INFORMATION`
  - `$FILENAME`

The impact of a number of operations on each timestamp for the
`$STANDARD_INFORMATION` and `$FILENAME` attributes is detailed in the
[SANS's `Windows Time Rules` poster](https://www.sans.org/security-resources/posters/windows-forensic-analysis/170/download).
Overall, the following points should be noted:

  - `$FILENAME` `MACB` timestamps are updated on file creation/copy/volume
    move with the date of the operation itself, but are not reliably updated
    on regular file operations (access, modification, rename, deletion).
    **However, as the `$FILENAME` `MAB` timestamps are updated/copied from the
    `$STANDARD_INFORMATION` `MAB` timestamps on file rename or volume-local
    file move, they are prone to false negatives.** Indeed, by timestomping the
    `$STANDARD_INFORMATION` timestamps and then renaming or moving the file,
    the `$FILENAME` timestamps will be indirectly timestomped as well.

  - On file copy (between two `NTFS` partitions): the `$STANDARD_INFORMATION`
    `MC` timestamps are inherited from the original file, but the
    `$STANDARD_INFORMATION` `AB` timestamps (and the `$FILENAME` `MACB`
    timestamps) are those of the copy itself.

  - On local file moves (on the same `NTFS` partition), the
    `$STANDARD_INFORMATION` `C` and `$FILENAME` `C` timestamps are updated with
    the timestamp of the move. On file moves (between `NTFS` partitions), the
    `$STANDARD_INFORMATION` `AC` timestamps are updated, also with the
    timestamp of the move.

  - The update of the `$STANDARD_INFORMATION` `A` timestamp is unreliable and
    depends on the value of the
    `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem\NtfsDisableLastAccessUpdate`
    registry key. The following values may be encountered:

      - `0x0` (default on Windows XP), `0x80000000` / `2147483648` (User
        managed), `0x80000002` / 2147483650 (System managed) mean that last
        access updates are enabled.
        
        Starting from `Windows Redstone 4` (`Build 1803` of 2018-04), last
        access updates seem to be enabled (back) by default if the system
        partition size is <= 128 GiB. Starting from `Windows 10 20H1`
        (`Build 18970` of 2020-05), last access updates seem to be enabled by
        default regardless of the system partition size.

      - `1` (default from Windows Vista to early Windows 10 versions),
        `0x80000001` / `2147483649` (User managed), `0x80000003` / `2147483651`
        mean that last access updates are disabled.

Depending on its filename length, a given file may have one or two `$FILENAME`
attributes:

  - files with short names will have a single `$FILENAME` attribute.

  - files with long names will be associated with two `$FILENAME` attributes,
    one for the long file name and a second for the MS-DOS-compatible short
    file name (`FILENA~1.TXT` for example).

Additionally, another `$FILENAME` attribute can be found for each file in the
directory index of its parent directory. Indeed, directories are stored on
`NTFS` partitions as `B+ tree data structures`, with the keys (representing
files and subdirectories) stored as `$FILENAME` attributes. `MACB` timestamps
for each file and subdirectory of a given directory can thus be found in the
directory index. The directory indexes are stored in `NTFS Index Attribute`
files, also known as `INDX` files and named `$I30` on disk.

A given file may thus be associated with either:

  - **12 timestamps**: `$STANDARD_INFORMATION` + `$FILENAME` + `NTFS $I30`'s
    `$FILENAME`.

  - **20 timestamps**: `$STANDARD_INFORMATION` + 2 * `$FILENAME` +
    2 * `NTFS $I30`'s `$FILENAME` (duplicate timestamps for files with long
    names).

### Timestomping detection

Timestomping is the action of modifying the timestamps of a file (on Windows
systems, on an `NTFS` partition). It can notably be used to evade digital
forensic investigation by making malicious files look legitimate or appear to
be outside of the presumed attack timeframe.

This technique is identified by
[MITRE ATT&CK T1070.006](https://attack.mitre.org/techniques/T1070/006/).

The `MACB` timestamps in the `$STANDARD_INFORMATION` attributes can be modified
by standard users, while the `$FILENAME` attributes can only be modified by /
through the Windows kernel. The modification of a file's
`$STANDARD_INFORMATION` attribute requires the right to modify the file
attributes (`FullControl`, `Modify`, `Write`, `WriteAttributes`), which is
granted by default to the file owner.

Note that in addition to being the ones that can be easily modified, the
`MACB` timestamps from the `$STANDARD_INFORMATION` attribute are conveniently
the ones (generally) displayed by the `Windows Explorer`.

Most of the timestomping detection methods below rely on information stored in
the `$MFT` file. Refer to the [$MFT page](./ntfs_mft.md) for more information
on how to parse the `$MFT` artefact.

#### $STANDARD_INFORMATION timestamps older than $FILENAME timestamps

Timestomping can be detected by comparing the `$STANDARD_INFORMATION` and
`$FILENAME` timestamps of a given file in the `MFT`. Indeed, if the timestamps
from `$STANDARD_INFORMATION` (easily modifiable) are older than the `$FILENAME`
timestamps (not (easily) modifiable), the file timestamps may have been
timestomped.

**However, as the `$FILENAME` `MAB` timestamps are updated/copied from the
`$STANDARD_INFORMATION` `MAB` timestamps on file rename or volume-local file
move, `$FILENAME` timestamps can also be (indirectly) tampered with.**

Additionally, this detection method is prone to false positives, as some
applications or installers may modify the `$STANDARD_INFORMATION` timestamps.

`MFTECmd` can be used to parse the `MFT` of an `NTFS` volume and automatically
highlight the files having `$STANDARD_INFORMATION` timestamps older than their
`$FILENAME` timestamps.

#### UsnJrnl records

Data from the `UsnJrnl` artefact may reveal recent operations on timestomped
files. For instance, a `USN_REASON_FILE_CREATE` record logged in the `UsnJrnl`
for a seemingly older file could be an indicator of timestomping.

Additionally, a `USN_REASON_BASIC_INFO_CHANGE` (+ `USN_REASON_CLOSE`) record
would be logged in the `UsnJrnl` following the timestomping of a file. The
presence of such a record is however not necessarily a strong indicator of
timestomping, as many other attribute changes would also trigger a similar
record to be logged in the `UsnJrnl`.

However, this detection method is prone to false negatives, as the `UsnJrnl`
usually has limited historical data.

Refer to the [`$UsnJrnl` page](./ntfs_usnjrnl.md) for more information on how
to parse the `UsnJrnl` artefact.

#### Non nano-second precise $STANDARD_INFORMATION timestamps

The timestomping tool used may have limitations on the time precision it
uses for timestomped timestamps. For example, the tool may only allow precision
down to the second level, while the `$STANDARD_INFORMATION` timestamps are
precise down to the ten-millionth of a second. In such a case, the timestomped
timestamps will be padded with zeros in place of the actual milliseconds:
`YYYY-MM-DD hh:mm:ss.0000000`.

However, this detection method is prone to false positives, as some utilities
or file formats, such as file archives, may truncate timestamps down to the
second level.

#### MFT entry numbers

`$MFT` entry numbers grow sequentially, with older files generally having lower
entry numbers than more recent files. The `$MFT` entry numbers should thus grow
linearly with the `$STANDARD_INFORMATION` created/birth timestamp (with the
expected exceptions that files older by a few days may sometimes have slightly
higher entry numbers than relatively more recent files).

However, this detection method is prone to false positives, as `$MFT` entry
numbers of deleted files may be reused (especially for `NTFS` partitions on
SSDs).

### References

  - [Andrea Fortuna - MAC(b) times in Windows forensic analysis](https://andreafortuna.org/2017/10/06/macb-times-in-windows-forensic-analysis/)

  - [dfir.ru - MSUHANOV - $STANDARD_INFORMATION vs. $FILE_NAME](https://dfir.ru/2021/01/10/standard_information-vs-file_name/)

  - [dfir.ru - MSUHANOV - The "Last Access" updates are almost back](https://dfir.ru/2018/12/08/the-last-access-updates-are-almost-back/)

  - [Matt B - A Journey into NTFS: Part 4](https://bromiley.medium.com/a-journey-into-ntfs-part-4-f2865c39ac83)

  - [SANS - Dave Hull - Digital Forensics: Detecting time stamp manipulation](https://www.sans.org/blog/digital-forensics-detecting-time-stamp-manipulation/)

  - [Alexandru Stamate - AlexSta - How to detect timestomping (on a Windows system)](https://alexsta-cybersecurity.com/how-to-detect-timestomping-on-a-windows-system/)

  - *forensicswiki.xyz - MAC_times - DOWN*
