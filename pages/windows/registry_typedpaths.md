---
title: Registry - TypedPaths
summary: 'The TypedPaths registry key tracks the last 25 entries entered in the Windows Explorer path bar. Entries can be paths or programs.\n\nEntries are only committed to registry after the Windows Explorer window is closed.'
keywords: 'TypedPaths, Explorer, MRUList'
tags:
  - windows_registry
location: 'File: <SYSTEMDRIVE>:\Users\<USERNAME>\NTUSER.dat\n\nRegistry key: HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\TypedPaths'
last_updated: 2024-03-16
sidebar: sidebar
permalink: windows_registry_typedpaths.html
folder: windows
---

### Overview

The `TypedPaths` registry key tracks the last 25 entries entered in the
`Windows Explorer` path bar. Entries can be paths (local or on a remote share)
or programs (executed by full path or from the system or the user's `PATH`).

If an item entered is not found (e.g. nonexistent paths or programs for
instance), the entry may lead to a web search using the default web browser
configured on the system but will not be added to the `TypedPaths` registry
key.

Entries are only committed to registry after the `Windows Explorer` window is
closed. If two `Windows Explorer` windows are opened at the same
time, the entries of the lastly closed window will overwrite the entries added
when the first window was closed. 

### Information of interest

The entries are stored as registry strings (`REG_SZ`), with the key names being
ordered from `url1` to `url25`. The last entered entry will be `url1` (with
rotation of previous entries and eventual deletion if the maximum of 25 entries
is reached).

The last write timestamp of the `TypedPaths` registry key is an indicator of
when the `Windows Explorer` session that populated the entries was closed (but
not an indicator of when the last entry was entered).

### References

  - [Forensafe - Investigating Typed Paths](https://forensafe.com/blogs/typedpaths.html)
