---
title: Windows - Registry - Terminal Server Client\Servers
summary: 'The Terminal Server Client\Servers registry key tracks the remote hosts the associated user connected to using the built-in mstsc.exe Remote Desktop client.\n\nInformation of interest: IP address of the remote host and saved username associated with the remote host (if any).\n\nThe last write timestamp may be an indicator of the first access to the remote host.'
keywords: 'Terminal Server Client\Servers'
tags:
  - windows_registry
  - windows_lateral_movement
  - windows_lateral_movement_src
  - windows_remote_desktop
  - windows_remote_desktop_src
location: 'File: <SYSTEMDRIVE>:\Users\<USERNAME>\NTUSER.dat\n\nRegistry key: HKCU\SOFTWARE\Microsoft\Terminal Server Client\Servers\<IP>'
last_updated: 2024-01-18
sidebar: sidebar
permalink: windows_registry_terminalserverclient.html
folder: windows
---

### Overview

The `Terminal Server Client\Servers` registry key tracks the remote hosts the
associated user connected to (from the local system) using the built-in
`mstsc.exe` `Remote Desktop` client. Remote hosts are only added if the remote
host screen was reached, i.e. `Network Level Authentication` succeeded (if the
remote host requires `NLA`) and the certificate warning was accepted (if a
warning was displayed).

Entries are added/updated in near real time.

### Information of interest

Each remote host is referenced as a dedicated subkey under the
`Terminal Server Client\Servers` registry key. This subkey is named after the
IP address of the remote host.

For each host, its dedicated subkey references the saved username for the
connection (if any) in the `UsernameHint` value.

Additionally, the last write timestamp may be an indicator of the first access
to the remote host (but may also have been updated for various other reasons).

### References

  - [JPCERT - RDP (Remote Desktop Protocol)](https://jpcertcc.github.io/ToolAnalysisResultSheet/details/mstsc.htm)
