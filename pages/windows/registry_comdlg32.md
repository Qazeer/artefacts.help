---
title: Registry - Common Dialogs (ComDlg32)
summary: 'The registry keys under ComDlg32 are linked to the Common Dialogs boxes, such as the "Open" and "Save as" dialog boxes.\n\nOpenSaveMRU / OpenSavePidlMRU information of interest: full path of the last 20 files, for each file extension, opened or saved through a Common Dialogs box.\n\nLastVisitedMRU / LastVisitedPidlMRU / LastVisitedPidlMRULegacy information of interest: some of the programs used to open / save the files tracked in the OpenSaveMRU / OpenSavePidlMRU registry key. The application filename and last folder accessed through a dialog box is tracked. The created and last accessed timestamps of each subfolder in the path of the last accessed folder are also stored.\n\nCIDSizeMRU information of interest: filename of the applications linked to Common Dialogs activity.'
keywords: 'Common Dialogs, ComDlg32, OpenSaveMRU, OpenSavePidlMRU, LastVisitedMRU, LastVisitedPidlMRU, LastVisitedPidlMRULegacy, CIDSizeMRU'
tags:
  - windows_files_and_folders_access
  - windows_program_execution
  - windows_registry
location: 'File: <SYSTEMDRIVE>:\Users\<USERNAME>\NTUSER.dat\n\nRegistry subkeys under:\nHKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\\n\nOpenSaveMRU / OpenSavePidlMRU\n\nLastVisitedMRU / LastVisitedPidlMRU / LastVisitedPidlMRULegacy\n\nCIDSizeMRU'
last_updated: 2024-01-07
sidebar: sidebar
permalink: windows_registry_comdlg32.html
folder: windows
---

The registry keys under `ComDlg32` are linked to the `Common Dialogs` boxes,
such as the "Open" and "Save as" dialog boxes.

Files or folders accessed, and the executing programs, are stored to maintain
a dialog box state across operations. For instance, the last path a file was
saved into will be directly opened when opening the "Save as" dialog box using
Microsoft Word.

### OpenSaveMRU / OpenSavePidlMRU

| Hive | Description | Location |
| `HKCU\SOFTWARE` | Renamed from `OpenSaveMRU` to `OpenSavePidlMRU` in `Windows Vista` and later. <br><br> Records information on files opened or saved through the "Open File" or "Save File" `Common Dialogs` box. <br><br> The `OpenSaveMRU`/ `OpenSavePidlMRU` keys has multiple subkeys, one for each different file extension (for the files opened / saved on the given system). <br><br> Each subkey contains an ordered `Most recently used (MRU)` list of opened / saved files (full path of the file). The list can go up to 20 entries, with entries over 20 being overwritten. <br><br> The last write timestamp of each subkey thus corresponds to the timestamp of opening / saving of the file in MRU position 0 (for a given file extension). | File: `%SystemDrive%:\Users\<USERNAME>\NTUSER.dat` <br><br> Registry key: <br> `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\OpenSaveMRU` <br> `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\OpenSavePidlMRU` |

### LastVisitedMRU / LastVisitedPidlMRU / LastVisitedPidlMRULegacy

| Hive | Description | Location |
| `HKCU\SOFTWARE` | Renamed from `LastVisitedMRU` to `LastVisitedPidlMRU` in `Windows Vista` and later. <br><br> Records the programs used to open / save (some of) the file tracked in the `OpenSaveMRU` / `OpenSavePidlMRU` registry key. <br><br> Notably used to track the last folder used by a given program in an "Open File" / "Save File" `Common Dialogs` box. <br><br> Applications tracked by their file name and are stored in an ordered `Most Recently Used (MRU)` list. The last write timestamp of the key thus corresponds to the timestamp of execution of the most recently executed program (first in the MRU list). <br><br> For each application, the full path of the folder can be constructed from information blocks on each subfolder in the location. For example, for the "%SystemRoot%\Users\Public\Documents" location, three blocks will be present: "Users", "Public", and "Documents". For each block, the created and last accessed timestamps and the MFT entry / sequence associated with the folder are referenced. | File: `%SystemDrive%:\Users\<USERNAME>\NTUSER.dat` <br><br> Registry key: <br> `HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\LastVisitedMRU` <br> `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\LastVisitedPidlMRU` <br> `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\LastVisitedPidlMRULegacy` |

### CIDSizeMRU

| Hive | Description | Location |
| `HKCU\SOFTWARE` | Recently executed programs, linked to `Common Dialogs` activity (pop boxes to open / save file, print, find / replace, ...). <br><br> The key contains an ordered `Most Recently Used (MRU)` list of executed programs, identified through their filename. <br><br> The last write timestamp of the key thus corresponds to the timestamp of execution of the most recently executed program (first in the MRU list). | File: `%SystemDrive%:\Users\<USERNAME>\NTUSER.dat` <br><br> Registry key: <br> `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\CIDSizeMRU` |

### References

  - [SANS - Chad Tilbury - OpenSaveMRU and LastVisitedMRU](https://www.sans.org/blog/opensavemru-and-lastvisitedmru/)

  - [digitalf0rensics - Ajith Ravindran - Windows Registry and Forensics (Part2)](https://digitalf0rensics.wordpress.com/2014/01/17/windows-registry-and-forensics-part2/)
