---
title: ETW - Remote Desktop - Destination host
summary: 'Destination host of a Remote Desktop access.\n\nMain events:\n\nChannel: Security.\nEvent ID 4624: "An account was successfully logged on", with LogonType 10.
\n\nChannel:\nMicrosoft-Windows-TerminalServices-RemoteConnectionManager/Operational.\nEvent ID 1149: "Remote Desktop Services: User authentication succeeded".\n\nChannel:\nMicrosoft-Windows-TerminalServices-LocalSessionManager/Operational.\nEvent ID 21: "Remote Desktop Services: Session logon succeeded".\nEvent ID 23: "Remote Desktop Services: Session logoff succeeded".\nEvent ID 25: "Remote Desktop Services: Session reconnection succeeded".'
keywords: 'RDP, Remote Desktop, Terminal Services, RemoteConnectionManager, LocalSessionManager, RdpCoreTS, LogonType 10, 1149, 21, 22, 23, 25, 131'
tags:
  - windows_etw
  - windows_lateral_movement
  - windows_lateral_movement_dst
  - windows_remote_desktop
  - windows_remote_desktop_dst
location: 'Channels:\n\nSecurity.\nEvent: 4624 (LogonType 10).\n\nMicrosoft-Windows-TerminalServices-RemoteConnectionManager/Operational.\nEvent: 1149.\n\nMicrosoft-Windows-TerminalServices-LocalSessionManager/Operational.\nEvents: 21, 22, 23, 25.\n\nMicrosoft-Windows-RemoteDesktopServices-RdpCoreTS/Operational.\nEvents: 131.'
last_updated: 2024-01-08
sidebar: sidebar
permalink: windows_etw_remote_desktop_dst_host.html
folder: windows
---

### Overview

| Channel | Conditions | Events |
|---------|------------|--------|
| `Security` | Default configuration. <br><br> Also logged for other logon types (`Network`, `Console`, `Batch`, `Service`, ...). | Event [`4624: An account was successfully logged on`](./etw_authentication_dst_host.md#security-event-id-4624), with `LogonType` `10`. |
| `Microsoft-Windows-TerminalServices-RemoteConnectionManager/Operational` | Default configuration. | Event `1149: Remote Desktop Services: User authentication succeeded`. <br><br> Access to the Windows login screen, not necessarily a successful session opening. This event is however only generated upon successful authentication if `Network Level Authentication (NLA)` is required. |
| `Microsoft-Windows-TerminalServices-LocalSessionManager/Operational` | Default configuration. | Event `21: Remote Desktop Services: Session logon succeeded`. |
| `Microsoft-Windows-TerminalServices-LocalSessionManager/Operational` | Default configuration. | Event `22: Remote Desktop Services: Shell start notification received` |
| `Microsoft-Windows-TerminalServices-LocalSessionManager/Operational` | Default configuration. | Event `23: Remote Desktop Services: Session logoff succeeded` |
| `Microsoft-Windows-TerminalServices-LocalSessionManager/Operational` | Default configuration. | Event `25: Remote Desktop Services: Session reconnection succeeded` <br> Events with a source network address set to `LOCAL` can sometimes be generated for console, non `Remote Desktop` logins. |
| `Microsoft-Windows-RemoteDesktopServices-RdpCoreTS/Operational` | Introduced in `Windows Server 2012`. <br><br> Default configuration. | Event `131: The server accepted a new TCP connection from client <IP>`. <br><br> Only indicates a network access to the `Remote Desktop` service. <br><br> For the aforementioned events, a `Source Network Address` of `::%16777216` could indicate that a `ngrok` tunnel was used to make `Remote Desktop` access. |
