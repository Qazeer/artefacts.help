---
title: Linux - viminfo
summary: 'The vim text editor logs a number of operations in the text-based ".viminfo" log file.\n\n The log file notably includes information on the last 100 files accessed (by default, 20 on Red Hat-based distributions) through vim.\n\n Under the "jumplist" section, up to 50 file accesses are referenced, with each file access associated with two entries, one for the file opening and one for the file closing. Each entry includes the file path and an epoch timestamp of occurrence (file opening or closing). Subsequent openings of the same file do not erase previous entries for the file.\n\n Under the "# History of marks within files" section, up to 100 (by default) file accesses are referenced, with only a single entry for a given file, which references the last closing of the file. This section may thus contain more historical data than the "jumplist" section.'
keywords: vim, viminfo, jumplist, vimrc
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
the text-based `.viminfo` log file. As the `.viminfo` file resides in a user's
home folder, the operations logged in the file can be linked to a specific user's
usage of `vim`.

The operations of a given `vim` "session" are written when the editor exits.

The location and behavior of the `.viminfo` file can be configured in the
`vimrc` configuration files (system-wide `/etc/vimrc` on `Red Hat`-based
distributions or `/etc/vim/vimrc` on `Debian`-based distributions, and
user-scoped `~/.vimrc`, `~/.vim/vimrc`, or `~/.config/vim/vimrc`).

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
        opened (opening entry) or closed (closing entry). As the jumplist is
        ordered from newest to oldest, the closing entry appears before the
        opening entry.

    Subsequent openings of the same file do not erase previous entries for the
    file.

    The "jumplist" section stores, by default, up to 100 entries, so up to the
    last 50 files accessed. However, it appears that a single file access may
    generate multiple sets of two identical opening/closing entries, reducing
    the overall history available.

    Example of two entries associated with the first opening of the file
    (cursor at line 1, character 0) and closing (cursor at line 4,
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

    In `vim`, a mark is a saved position (line and column) in a file that the
    cursor can later jump back to. Marks are either set manually or
    automatically by `vim`:

    - `'a` to `'z`: lowercase marks, set manually (`m{a-z}`) and local to a
      file.
    - `'A` to `'Z`: uppercase marks, set manually (`m{A-Z}`) and global, as
      they also record the file name.
    - `'0` to `'9`: numbered marks, set automatically to the cursor position
      in the last files edited.
    - `"`: set automatically to the cursor position when the file was last
      exited.
    - `^`: set automatically to the position of the last insert.
    - `.`: set automatically to the position of the last change.

    The "# File marks" section stores the global marks, which point to
    positions in any file: the uppercase marks and the numbered marks. Each
    mark line (`'1  4  10  ~/.zshrc`) gives the mark name, the line, the
    column and the file. It's followed by a line starting with `|` that
    repeats the same information and adds a timestamp.

    The "# History of marks within files" section stores, for each file, the
    marks local to that file: the lowercase marks and the `"`, `^` and `.`
    marks. Lines starting with `+` record positions from the file's list of
    changes, while the line starting with `*` isn't a mark but holds the
    timestamp of the last use of the file.

    While multiple "File marks" entries are logged upon multiple accesses to
    the same file, the "# History of marks within files" section only keeps a
    single entry for a given file, which references the last closing of the
    file.

    The "# History of marks within files" section stores, by default, up to
    100 entries as well, but as each file is uniquely referenced, the section
    may contain more historical data than the "jumplist" section. The entries
    are ordered by the timestamp of the last use of the file (newest to
    oldest): when a file already referenced is accessed again, its entry is
    moved to the top of the section, and, upon reaching the limit of entries,
    the oldest entries are removed.

    The limit of entries is defined by the `'` parameter of the `viminfo`
    option (`'100` by default). The default `/etc/vimrc` configuration file of
    `Red Hat`-based distributions sets this limit to 20 entries
    (`set viminfo='20,\"50`).

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
    of occurrence. Only the last occurrence of a given command/search/
    expression is kept.
