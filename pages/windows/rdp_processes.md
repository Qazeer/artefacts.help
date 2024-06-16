---
title: RDP - Processes
summary: 'The following processes are related to RDP activity:\n\n- mstsc.exe: Windows built-in RDP. The remote host may (but not necessarily) specified using the command-line parameter "/v:".\n\n- rdpclip.exe: RDP Clipboard Monitor, executed on the remote host every time a remote interactive RDP session is successfully established.\n\n- TSTheme.exe: TSTheme Server Module, starting with Windows 7, executed on the remote host every time a remote interactive RDP session is successfully established and upon session closure.'
keywords: RDP, mstsc.exe, rdpclip.exe, RDP Clipboard Monitor, tstheme.exe, TSTheme Server Module, NLA
tags:
  - windows_lateral_movement
  - windows_lateral_movement_src
  - windows_lateral_movement_dst
  - windows_remote_desktop
  - windows_remote_desktop_src
  - windows_remote_desktop_dst
last_updated: 2024-06-16
sidebar: sidebar
permalink: windows_rdp_processes.html
folder: windows
---

### Source host

The following processes are related to `RDP` activity on the source host:

  - `mstsc.exe` (`<WINDIR>\System32\mstsc.exe`): `Windows` built-in `RDP`
    client `Microsoft Terminal Server Client`. The client can ban launched and
    the remote server then specified through the graphical interface, or the
    remote server can be specified directly using the `/v:` command-line
    parameter. If the remote server is specified through the command-line,
    [ETW process logging](./etw_process_creation.md) or
    [mstsc's `Jumplists`](./jumplists.md#remote-desktop-connection-mstscexe)
    can be leveraged to determine the remote host reached.

### Destination host

The following processes are related to `RDP` activity on the destination host:

  - `rdpclip.exe` (`<WINDIR>\System32\rdpclip.exe`): the
    `RDP Clipboard Monitor` handles the shared clipboard between the local
    computer and the remote host. `rdpclip` is executed every time (after
    `Windows XP` / `Windows Server 2003`) a remote interactive `RDP` session is
    established, even if the local clipboard is not shared with the remote
    host. An execution of `rdpclip` is thus a sign that a `RDP` connection was
    successfully authenticated and established. Even if
    `Network Level Authentication (NLA)` is disabled, `rdpclip` is only
    executed after a successful authentication.

  - `TSTheme.exe` (`<WINDIR>\System32\TSTheme.exe`): the
    `TSTheme Server Module`. Starting with `Windows 7`, `TSTheme` is executed,
    similarly to `rdpclip`, every time an interactive `RDP` session is
    successfully established. A new `TSTheme` instance also appears to be
    reliably executed upon the session closure.
