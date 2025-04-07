---
title: Thumbs.db and Thumbcache
summary: 'The Thumbs.db and Thumbcache files contain cached thumbnail previews for files (pictures, some document and media file types) in folders that were interactively accessed with the Windows Explorer. Some document types, such as PDF files, will have their first page as their thumbnail preview.\n\nThe cached thumbnail previews persist even after deletion of the associated files.\n\nThe Thumbs.db files are stored in their associated folders, with one individual Thumbs.db file per folder. Since Windows Vista, Thumbs.db files are only generated for access through UNC paths (in the remote/share directory).\n\nStarting with Windows Vista, the Thumbcache files centralize thumbnails in a central location. Each Thumbcache file, labeled "thumbcache_<RESOLUTION>.db", contains thumbnails from all locations. The location of the file linked to a thumbnail is not stored in the Thumbcache file. However, an unique identifier may be used to retrieve the location of the associated file (mostly for non deleted files).'
keywords: Thumbs.db, Thumbcache, picture, thumbnail, preview
tags:
  - windows_files_and_folders_access
  - windows_file_knowledge
  - windows_misc
location: 'Thumbs.db:\nIndividual hidden files in their associated folders.\n\nStarting from Windows Vista, Thumbcache:\n<SYSTEMDRIVE>:\Users\<USERNAME>\AppData\Local\Microsoft\Windows\Explorer\thumbcache_<RESOLUTION>.db files.'
last_updated: 2024-01-16
sidebar: sidebar
permalink: windows_thumbs-db_thumbcache.html
folder: windows
---

### Overview

The `Thumbs.db` and `Thumbcache` files contain cached thumbnail previews for
files (pictures, some document and media file types) in folders that were
interactively accessed with the `Windows Explorer`. Some document types, such
as `PDF` files, will have their first page as their thumbnail preview.

The thumbnail previews are stored in these databases as it takes less system
resources (CPU time and memory) to retrieve an already generated thumbnail as
opposed to generating it every time the directory is accessed.

For a `Thumbs.db` file to be generated in a given folder, or for entries to be
added to the central `Thumbcache` files, the access must have been done with
some sort of files' thumbnail/icon preview enabled.

The cached thumbnail previews persist even after deletion of the associated
files.

### Information of interest

#### Thumbs.db

The `Thumbs.db` files are stored in their associated folders, with one
individual `Thumbs.db` file per folder (that was interactively accessed with
files preview). However, since Windows Vista, `Thumbs.db` files are only
generated for access through `UNC` paths (such as
`\\<HOST>\<SHARE_NAME>\<FOLDER>` or `\\<HOST>\c$\<FOLDER>`) in the remote /
share directory.

Each thumbnail created in a directory is represented in the `Thumbs.db` file as
a small `JPEG` file, regardless of the file's original format. The images are
resized to 96 x 96 pixels by default. As each `Thumbs.db` file is associated
with a given directory, the location of the cached thumbnails can be easily
deduced.

#### Thumbcache

Starting with `Windows Vista`, the `Thumbcache` files centralize thumbnails in
a central location. Each `Thumbcache` file, labeled
`thumbcache_<RESOLUTION>.db`, contains thumbnails from all locations. The
`<RESOLUTION>` indicate the resolution of the thumbnail previews, such as the
`thumbcache_1280.db` file for thumbnails in 1280 x 720 pixels resolution.

The location of the file linked to a thumbnail is not stored in the
`Thumbcache` file. However, each thumbnail in the `Thumbcache` file is
associated with an unique identifier `ThumbnailcacheID`. This identifier/hash
can be used to retrieve the location of the associated file, mostly for non
deleted files:

  - By scanning and computing the identifier for every files on the volume.
    This requires the file to still be present on the volume.

  - By searching the `Windows Search` database (`Windows.edb`) for the
    `ThumbnailcacheID`, as a table of this database notably references the file
    original full path and size. As the `Windows Search` database is updated in
    near real time and does not store information on deleted files, this also
    requires the original file to still be present.

### Tool(s)

The [`Thumbs Viewer`](https://thumbsviewer.github.io/) and
[`Thumbcache Viewer`](https://thumbcacheviewer.github.io/) can be used to,
respectively, parse `Thumbs.db` and `Thumbcache` files.

The command-line version
[`thumbcache_viewer_cmd`](https://thumbcacheviewer.github.io/#Thumbcache_Viewer_CMD)
can be used to extract thumbnail images and generate HTML and CSV report(s) on
the thumbnails extracted. The
[Execute-ThumbcacheViewer.ps1](https://gist.github.com/Qazeer/cb3a0cf306bc1f75a2d5a8cef5b9ffa9)
PowerShell script (`KAPE` associated module
`PowerShell_Execute-ThumbcacheViewer`) can recursively process the specified
input folder to execute `thumbcache_viewer_cmd.exe` over each thumbcache
subfolder found. The PowerShell script is basically a wrapper to make
`thumbcache_viewer_cmd.exe` recursive, as the tool can natively only process a
thumbcache subfolder directly.

### References

  - [Wikiepedia - Windows thumbnail cache](https://en.wikipedia.org/wiki/Windows_thumbnail_cache)

  - [erickutcher - THUMBCACHE VIEWER](https://thumbcacheviewer.github.io/)

  - [13Cubed - Two Thumbs Up Thumbnail Forensics](https://www.youtube.com/watch?v=5efCp1VXhfQ)
