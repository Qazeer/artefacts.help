---
title: Registry - TypedURLs
summary: 'The TypedURLs registry tracks URL entered (typed, pasted, or auto-completed) in the Internet Explorer (IE) web browser search bar. Web searches are not stored, only the URLs entered are tracked.\n\nInformation of interest: URL entered in the IE search bar.\n\nValues are stored in inverse chronological order. The timestamp of last visit of the most recently visited URL can thus be deduced from the last write timestamp of the registry key.'
keywords: 'TypedURLs, URL, Internet Explorer, IE, MRUList'
tags:
  - windows_registry
  - windows_browsing_history
location: 'File: <SYSTEMDRIVE>:\Users\<USERNAME>\NTUSER.dat\n\nRegistry key: HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\TypedURLs'
last_updated: 2024-01-16
sidebar: sidebar
permalink: windows_registry_typedurls.html
folder: windows
---

### Overview

The `TypedURLs` registry tracks URLs entered (typed, pasted, or auto-completed)
in the `Internet Explorer (IE)` web browser search bar. Web searches are not
stored, only the URLs entered are tracked.

Starting with `IE8`, entries are added / updated in near real-time.

### Information of interest

Each URL is stored in a dedicated value under the `Explorer\TypedURLs` key, as
`url1` to `url[N]`. The values are stored in inverse chronological order.

The last write timestamp of the key is thus the timestamp of visit of the most
recently visited `URL`.

### References

  - [Crucial Security Forensics Blog - Paul Nichols - TypedURLs (Part 1)](https://crucialsecurity.files.wordpress.com/2011/04/figure-1-typedurls-key-viewed-with-regedit_tcm19-16019.jpg)

  - [Crucial Security Forensics Blog - Paul Nichols - TypedURLs (Part 2)](https://crucialsecurity.wordpress.com/2011/03/23/typedurls-part-2/)
