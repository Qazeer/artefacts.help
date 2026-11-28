---
title: Linux - ext4 file system (inodes, timestamps, and journal)
summary: 'ext4 is the default file system of most Linux distributions. Unlike NTFS, which centralizes file records in the MFT, the ext4 file system information is split across block groups, each composed of (optional) superblock and group descriptor backups, block and inode bitmaps, an inode table, and data blocks.\n\nThe superblock references information on the file system: creation, last mount, and last write timestamps, last mount point, volume UUID and label, etc.\n\nEach file is associated with an inode, which stores its metadata (size, owner UID / GID, permissions, and timestamps) but not its name (stored in the parent directory entries). ext4 inodes record four timestamps, with nanosecond precision: access (atime), modification (mtime), inode change (ctime), and creation / birth (crtime), as well as a deletion time (dtime) for deleted inodes.\n\nThe atime is usually only partially updated (relatime mount option). The atime and mtime can be arbitrarily set from user space (timestomping with touch), while the ctime and crtime cannot.\n\nThe ext4 journal (jbd2) records metadata changes and may hold previous versions of inodes and directory blocks, which can be used to recover information on deleted files.'
keywords: ext4, ext3, ext2, superblock, block group, inode, extent, atime, mtime, ctime, crtime, birth, dtime, relatime, noatime, timestomping, touch, stat, debugfs, dumpe2fs, istat, fls, mactime, jbd2, journal, logdump, ext4magic, lost+found
tags:
  - linux_filesystem
  - linux_file_knowledge
  - linux_defense_evasion
default_location: 'File system metadata (superblock, group descriptors, inode tables, journal) on ext2 / ext3 / ext4 partitions.\n\nNotable inodes:\n - inode 2: root directory\n - inode 8: journal\n - inode 11: lost+found directory (usually)'
last_updated: 2026-10-23
sidebar: sidebar
permalink: linux_ext4.html
folder: linux
---

### Overview

`ext4` (fourth extended file system) is the default file system of most Linux
distributions (Debian, Ubuntu, etc. Red Hat / CentOS / Rocky use `XFS` by
default). It is backward compatible with `ext2` and `ext3`, and notably adds
extents, nanosecond-precision timestamps, and creation timestamps.

Unlike `NTFS`, which centralizes file records in the
[`MFT`](../windows/ntfs_mft.md), the `ext4` file system information is split
across block groups (of 32768 blocks, i.e. 128 MiB with 4 KiB blocks). Each
block group is composed of:

  - A backup of the superblock and of the group descriptors (only in some
    block groups, with the `sparse_super` feature).

  - A block bitmap and an inode bitmap, tracking the allocation status of the
    data blocks and inodes of the group.

  - An inode table, holding the inodes of the group.

  - The data blocks.

#### Superblock

The primary superblock is located at byte offset 1024 (`0x400`) of the file
system, with the magic number `0xEF53` (stored in little-endian) at offset
`0x38` of the superblock (shared with `ext2` / `ext3`). It references
information of forensic interest:

  - The file system creation timestamp.

  - The last mount, last write, and last check timestamps.

  - The last mount point (directory where the file system was last mounted).

  - The volume UUID and label (name).

  - The mount count since the last check, and the amount of data (KiB) written
    during the lifetime of the file system.

  - The enabled features (`has_journal`, `extent`, `huge_file`,
    `metadata_csum`, etc.) and the inode size.

#### Inodes and directory entries

Each file or directory is associated with an inode, identified by its number.
The inode stores the file metadata: type and permissions (mode), owner `UID`
and `GID`, size, links count, timestamps, extended attributes (`ACL`,
`SELinux` context, etc.), and the location of the file data (extent tree, or
block pointers on `ext2` / `ext3`). **The inode does not store the file
name**: the names are stored in the directory entries of the parent directory,
which map a name to an inode number. A file can thus have multiple names (hard
links).

A few inodes are reserved: inode 2 is the root directory, inode 8 the
journal, and inode 11 (the first non-reserved inode) is usually the
`lost+found` directory. Inode numbers are preferably allocated in the block
group of the parent directory, so files created at the same time in the same
directory usually have close inode numbers.

### Information of interest

#### Timestamps

`ext4` inodes record the following timestamps:

| Timestamp | Name | Description |
|-----------|------|-------------|
| `atime` | Access | Last access to the file content (read, execution). Its update depends on the mount options (see below). |
| `mtime` | Modification | Last modification of the file content. For directories, last creation / deletion / renaming of an entry in the directory. |
| `ctime` | Change | Last modification of the inode (permissions, owner, links count, extended attributes, renaming), or of the file content. |
| `crtime` | Creation / birth | Creation of the inode. Only available on `ext4` file systems with 256-byte inodes (default), and not on `ext2` / `ext3`. |
| `dtime` | Deletion | Deletion of the inode (when the inode is freed). Only set for deleted inodes. |

The timestamps are stored as `UTC` epoch values. On `ext4` file systems with
256-byte inodes, the extra inode fields add a nanosecond precision (and extend
the supported date range beyond 2038). On file systems with 128-byte inodes
(legacy `ext3`), timestamps have a one-second precision and the `crtime` is
not available.

The impact of common operations on the timestamps of a file is as follows:

| Operation | atime | mtime | ctime | crtime |
|-----------|-------|-------|-------|--------|
| File creation | Set | Set | Set | Set |
| Content read / execution | Updated (depending on the mount options) | - | - | - |
| Content modification | - | Updated | Updated | - |
| Permissions / owner change (`chmod`, `chown`) | - | - | Updated | - |
| Rename or move within the same file system (`mv`) | - | - | Updated | - |
| Copy (`cp`), the copy being a new inode | Set | Set (preserved with `cp -p` / `cp -a`) | Set | Set |
| Move to another file system (`mv`, copy then delete) | Preserved | Preserved | Set | Set |
| Extraction from an archive / package (`tar`, `dpkg`, `rpm`, etc.) | Set or preserved | Preserved (from the archive) | Set | Set |

Creating, deleting, or renaming a file also updates the `mtime` and `ctime` of
its parent directory.

#### Access time and mount options

The update of the `atime` timestamp depends on the file system mount options
(as configured in `/etc/fstab`):

  - `relatime` (default since Linux 2.6.30): the `atime` is only updated if
    the previous `atime` is older than the `mtime` or `ctime`, or older than
    24 hours. The `atime` is thus an indicator of the first access following
    a modification, and of at least one access per day.

  - `noatime`: the `atime` is never updated (`nodiratime` for directories
    only).

  - `strictatime`: the `atime` is updated upon every access.

  - `lazytime`: the timestamp updates are only kept in memory and written to
    disk periodically (up to 24 hours) or when the inode is written for other
    reasons.

#### Timestomping

The `atime` and `mtime` timestamps can be arbitrarily set from user space by
the owner of a file (or `root`), for example with `touch`:

```bash
# Sets the atime and mtime to the specified timestamp.
touch -d "2021-03-04 10:11:12" <FILE>

# Copies the atime and mtime from the specified reference file.
touch -r <REFERENCE_FILE> <FILE>
```

**The `ctime` and `crtime` cannot be set from user space**: setting the
`atime` / `mtime` updates the `ctime` to the current time, and the `crtime`
is only set at the inode creation. They can, however, be modified by changing
the system clock before the operation, or by directly editing the inode on the
unmounted file system (`debugfs -w -R 'set_inode_field <FILE> crtime <VALUE>'`).

Timestomping may be detected by:

  - An `mtime` older than the `crtime`. This is, however, also the case for
    files extracted from archives or packages, or copied with their
    timestamps preserved.

  - A `ctime` significantly more recent than the `mtime`, without a
    corresponding permission / owner change or rename.

  - `atime` / `mtime` timestamps with a nanosecond part of 0 (when set with a
    second precision, for example `touch -d`). Files extracted from packages
    often have whole-second timestamps as well.

  - A `crtime` inconsistent with the inode number (compared to the inodes of
    neighboring files in the same directory).

#### Deleted files

When a file is deleted on `ext4`, its directory entry is unlinked (the
previous entry's length is extended to cover it), its inode is marked as
unallocated with the `dtime` set and the links count set to 0, and **the
extent tree of the inode is cleared**. Unlike on `ext2`, the file content
thus cannot be directly retrieved from the inode, and recovery usually
relies on the journal or on file carving.

The following information may, however, remain:

  - The deleted directory entry (name and inode number) in the slack of the
    directory blocks, until overwritten (listed by `fls -d` or `debugfs -R
    'ls -d'`).

  - The deleted inode metadata (timestamps including `dtime`, size, owner,
    mode), until the inode is reused.

#### Journal

`ext3` and `ext4` file systems are journaled (`jbd2`), with the journal
stored in inode 8 by default. In the default `data=ordered` mode, the
journal only records metadata blocks (inode tables, directory blocks, bitmaps,
etc.), as full block copies grouped in transactions with a sequence number
and a commit timestamp.

As the journal is a circular buffer, it usually only covers a limited period
of time (depending on the activity), but **may hold previous versions of
inodes (including the extent tree of deleted files) and directory blocks**,
which can be used to recover the name, metadata, and content location of
recently deleted files, or previous timestamps of a file.

### Tool(s)

#### Live system and mounted file systems

The `stat` utility (`coreutils` 8.31+ on Linux 4.11+, using `statx`) displays
the four timestamps of a file, including the birth (`crtime`) timestamp:

```bash
stat <FILE>

# Generates a timeline of the file system, in CSV format.
# %W: birth, %Y: modification, %Z: change, %X: access (epoch, 0 or '-' if unknown).
find <MOUNT_POINT> -xdev -print0 | xargs -0 stat --printf='%W|%Y|%Z|%X|%s|%U|%G|%a|%F|%n\n' > <OUTPUT_FILE>
```

Note that accessing files on a mounted file system may update their `atime`
(if not mounted read-only or with `noatime`).

#### debugfs and dumpe2fs

The `e2fsprogs` utilities `dumpe2fs` and `debugfs` can be used on a partition
image (or a loop device), to parse the superblock and the inodes (including
deleted ones):

```bash
# Displays the superblock information (without the block group details).
dumpe2fs -h <PARTITION_IMAGE>

# Displays the metadata of the specified inode or file, including the crtime and dtime.
# debugfs expects inode numbers between angle brackets (for example: stat <1234>).
debugfs -R 'stat <INODE_NUMBER>' <PARTITION_IMAGE>
debugfs -R 'stat <FILE_PATH>' <PARTITION_IMAGE>

# Lists the content of a directory, including deleted entries.
debugfs -R 'ls -drl <DIRECTORY_PATH>' <PARTITION_IMAGE>

# Extracts the content of the specified inode.
debugfs -R 'dump <INODE_NUMBER> <OUTPUT_FILE>' <PARTITION_IMAGE>

# Dumps the journal content (transactions and blocks).
debugfs -R 'logdump -a' <PARTITION_IMAGE>
```

#### The Sleuth Kit

[`The Sleuth Kit (TSK)`](https://www.sleuthkit.org/sleuthkit/) tools
support `ext2` / `ext3` / `ext4` file systems, and can be used directly on a
full disk image (with the partition offset, in sectors, as listed by
`mmls`):

```bash
# Displays the superblock information.
fsstat -o <PARTITION_OFFSET> <IMAGE>

# Lists files and directories recursively, including deleted entries.
fls -o <PARTITION_OFFSET> -r -p <IMAGE>

# Displays the metadata of the specified inode.
istat -o <PARTITION_OFFSET> <IMAGE> <INODE>

# Generates a timeline (body file then CSV) of all files, including deleted entries.
fls -o <PARTITION_OFFSET> -r -m / <IMAGE> > <BODY_FILE>
mactime -b <BODY_FILE> -d -y > <TIMELINE_CSV>
```

#### Deleted files recovery

[`ext4magic`](https://sourceforge.net/projects/ext4magic/) leverages the
journal to recover deleted files and their metadata, notably for files deleted
after a given timestamp. File carving tools (`photorec`, `foremost`, etc.) can
be used to recover file content without the file system metadata.

```bash
# Recovers all the files deleted after the specified epoch timestamp.
ext4magic <PARTITION_IMAGE> -a <EPOCH_TIMESTAMP> -r -d <OUTPUT_FOLDER>
```

### References

  - [kernel.org - ext4 Data Structures and Algorithms](https://www.kernel.org/doc/html/latest/filesystems/ext4/index.html)

  - [Linux manual page - inode(7)](https://man7.org/linux/man-pages/man7/inode.7.html)

  - [Linux manual page - mount(8) - atime options](https://man7.org/linux/man-pages/man8/mount.8.html)

  - [Linux manual page - debugfs(8)](https://man7.org/linux/man-pages/man8/debugfs.8.html)

  - [Righteous IT - Hal Pomeranz - Linux and EXT4 forensics blog posts](https://righteousit.com/)
