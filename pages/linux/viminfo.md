---
title: viminfo
summary: 'The vim text editor logs a number of operations in the text-based ".viminfo" log file.\n\n The log file notably includes information on the last 100 files accessed (by default) through vim.\n\n Under the "jumplist" section up to 50 file accesses are referenced, with each file access associated with two entries, one for the file opening and one for the file closing. Each entry includes the file path and an epoch timestamp of occurrence (file opening or closing). Subsequent openings of the same file do not erase previous entries for the file.\n\n Under the "# History of marks within files" section up to 100 file accesses are referenced, with only a single entry for a given file, which references the last closing of the file. This section may thus contain more historical data than the "jumplist" section.'
keywords: vim, viminfo
tags:
  - linux_files_and_folders_access
  - linux_file_knowledge
default_location: '~/.viminfo'
last_updated: 2024-10-12
sidebar: sidebar
permalink: linux_viminfo.html
folder: linux
---

### Overview

The `vim(1)` text editor logs a number of operations, including file access, in
the text-based `.viminfo` log file. As the `.viminfo` file resides under a user
home folder, the operations logged in the file can be linked to a specific user
usage of `vim`.

The operations of a given `vim` "session" are written upon exit of the editor.

The location and behavior of the `.viminfo` file can be configured in the
`vimrc` configuration files (default locations `/etc/vimrc` and `~/.vimrc`).

### Information of interest

The `.viminfo` log file notably includes:

  - The files accessed through `vim`, in a most recently accessed list under
    the "# Jumplist (newest first)" section.

    Every file access is associated with two entries, one for the file opening
    and one for the file closing. Each entry contains:

      - The file path.

      - A Linux `epoch` timestamp, corresponding to either when the file was
        opened or closed (depending on the entry).

      - The line and character number of the cursor position when the file was
        opened (first entry) or closed (last entry).

    Subsequent openings of the same file do not erase previous entries for the
    file.

    The "jumplist" section stores, by default, up to 100 entries, so up to the
    last 50 files accessed. However, it appears that a single file access may
    generate multiple sets of two identical opening/closing entries, reducing
    the overall history available.

    Example of two entries associated with the first opening of the file
    (cursor at line 1, character 0) and closing (cursor at line 3,
    character 10):

    ```
    # Jumplist (newest first):
    -'  4  10  ~/.zshrc
    |4,39,4,10,1728596893,"~/.zshrc"
    -'  1  0  ~/.zshrc
    |4,39,1,0,1728596856,"~/.zshrc"
    ```

  - The files accessed may also be referenced under the "# File marks" and
    "# History of marks within files (newest to oldest)" sections.

    Entries in both sections also include the cursor position within the file
    and a timestamp (of when the given file was closed).

    While multiple "File marks" entries are logged upon multiple accesses to
    the same file, the "# History of marks within files" section only keep a
    single entry for a given file, which references the last closing of the
    file.

    The "# History of marks within files" section stores up to 100 entries as
    well, but as each file is uniquely referenced, the section may contain more
    historical data than the "jumplist" section. However, upon reaching the
    limit of 100 entries, it is not necessarily the latest entry that will be
    overwritten during the rotation. Thus, it is not possible to draw a
    conclusion on all files accessed during the timeframe covered by the first
    and last entries if the history reaches 100 entries.

    ```
    # File marks:
    '1  4  10  ~/.zshrc
    |4,49,4,10,1728596893,"~/.zshrc"

    # History of marks within files (newest to oldest):
    > ~/.zshrc
            *       1728596891      0
            "       4       10
    ```

  - The histories of the commands (such as `wq` to close a file), searches,
    and expressions entered in `vim`, in three distinct sections.

    Each entry in these histories is associated with an `epoch` timestamp
    of occurrence. Only the last occurrence of a given command/search /
    expression is kept.
