---
title: 'NTFS - $I30 ($INDEX_ROOT, $INDEX_ALLOCATION, and $Bitmap)'
summary: 'The NTFS index attributes $INDEX_ROOT and $INDEX_ALLOCATION are MFT attributes that represent directories and store index records.\n\nEach file in a directory is associated with an index record. The record contains information on the file it references in a $FILE_NAME (0x30) attribute: file name, size, parent directory and a set of MACB timestamps (copied from the MFT file record $STANDARD_INFORMATION of the file).'
keywords: '$I30, I30, 0x30, index attributes, index record, $INDEX_ROOT, INDEX_ROOT, $INDEX_ALLOCATION, INDEX_ALLOCATION, $FILE_NAME, FILE_NAME, $Bitmap, Bitmap'
tags:
  - windows_ntfs_filesystem
  - windows_file_knowledge
location: 'MFT $INDEX_ROOT, $INDEX_ALLOCATION, and $Bitmap attributes.'
last_updated: 2024-01-01
sidebar: sidebar
permalink: windows_i30.html
folder: windows
---

### Overview

The `NTFS` `index attributes` are `MFT` attributes, of two distinct types,
that index all the files/directories in a given directory (in a B-Tree data
structure). Each directory is represented by one or more `index attributes`.
The files and folders information displayed by the `Windows Explorer` are based
on the index attribute(s) of the directory being accessed.

The entries (files or subdirectories) in a directory's `index attribute(s)` are
stored as `index records` structures, with one dedicated record for every
entry. The `index record` structure contains a `$FILE_NAME` (`0x30`) attribute,
in which are stored the information about the file or folder.

There is two types of `index attributes`:

  - `$INDEX_ROOT`: for directories with a small number of entries. The
    `$INDEX_ROOT` attribute is always resident to the `MFT` and contains a
    small list of `index records`. A directory has at most one `$INDEX_ROOT`
    attribute.

  - `$INDEX_ALLOCATION`: additional structure for larger directories, with no
    limitation on the number of entries. The `$INDEX_ALLOCATION` attribute is
    non-resident and contains one or more `index records`. The
    `INDEX_ALLOCATION` structure starts with the `INDX` signature. The
    `$INDEX_ALLOCATION` attribute should not exist without an associated
    `$INDEX_ROOT` attribute.

The `$Bitmap` attribute keep track of the index allocations.

The `$INDEX_ROOT`, `$INDEX_ALLOCATION`, and `$Bitmap` attributes are
collectively reffered to as `$I30`.

### Information of interest

Each `index record` contains information on the file it references in a
`$FILE_NAME` (`0x30`) attribute:

 - Filename and parent directory.

 - File size.

 - A set of `MACB` timestamps.

The `$FILE_NAME` attribute of a `index record` in a directory
`index attribute` should be kept in sync with the `MFT` file record's
`$STANDARD_INFORMATION` attribute of the corresponding entry. However,
disparities may sometime occur, with the `index record` referencing older
information.

Due to their B-Tree data structure format and their frequent rebalancing,
`$INDEX_ALLOCATION` attributes often contain a significant amount of slack
space. `Index records` for deleted files no longer present in the `MFT` may be
"carvable" from this slack space.

### Tool(s)

  - [`INDXRipper`](https://github.com/harelsegev/INDXRipper)

### References

  - [Digital Forensics Myanmar - NTFS Index Attributes](https://www.forensicsmyanmar.com/2022/08/ntfs-index-attributes.html)

  - [13Cubed - Windows NTFS Index Attributes ($I30 Files)](https://www.youtube.com/watch?v=XzoYNOlJ37s)

  - [13Cubed - A File's Life - File Deletion and Recovery](https://www.youtube.com/watch?v=4zlk9ZSMa-4)

  - [DFIR.RU - MSUHANOV - $STANDARD_INFORMATION vs. $FILE_NAME](https://dfir.ru/2021/01/10/standard_information-vs-file_name/)

  - [SANS - Chad Tilbury - NTFS $I30 Index Attributes: Evidence of Deleted and Overwritten Files](https://www.sans.org/blog/ntfs-i30-index-attributes-evidence-of-deleted-and-overwritten-files/)

  - [OSForensics - How to scan NTFS $I30 (directory) entries for evidence of deleted files](https://www.osforensics.com/faqs-and-tutorials/how-to-scan-ntfs-i30-entries-deleted-files.html)
