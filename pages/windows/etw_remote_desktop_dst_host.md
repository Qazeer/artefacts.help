---
title: ETW - Remote Desktop - Destination host
summary: 'Destination host of a Remote Desktop access.\n\nMain events:\n\nChannel: Security.\nEvent ID 4624: "An account was successfully logged on", with LogonType 10.
\n\nChannel:\nMicrosoft-Windows-TerminalServices-RemoteConnectionManager/Operational.\nEvent ID 1149: "Remote Desktop Services: User authentication succeeded".\n\nChannel:\nMicrosoft-Windows-TerminalServices-LocalSessionManager/Operational.\nEvent ID 21: "Remote Desktop Services: Session logon succeeded".\nEvent ID 23: "Remote Desktop Services: Session logoff succeeded".\nEvent ID 24: "Remote Desktop Services: Session has been disconnected".\nEvent ID 25: "Remote Desktop Services: Session reconnection succeeded".'
keywords: 'RDP, Remote Desktop, Terminal Services, RemoteConnectionManager, LocalSessionManager, RdpCoreTS, LogonType 10, 1149, 21, 22, 23, 24, 25, 131, 1158, ::%16777216, %16777216, 16777216'
tags:
  - windows_etw
  - windows_lateral_movement
  - windows_lateral_movement_dst
  - windows_remote_desktop
  - windows_remote_desktop_dst
location: 'Channels:\n\nSecurity\nEvent: 4624 (LogonType 10).\n\nMicrosoft-Windows-TerminalServices-RemoteConnectionManager/Operational\nEvent: 1149.\n\nMicrosoft-Windows-TerminalServices-LocalSessionManager/Operational\nEvents: 21, 22, 23, 24, 25.\n\nMicrosoft-Windows-TerminalServices-RemoteConnectionManager/Admin\nEvent: 1158.\n\nMicrosoft-Windows-RemoteDesktopServices-RdpCoreTS/Operational\nEvent: 131.'
last_updated: 2025-07-13
sidebar: sidebar
permalink: windows_etw_remote_desktop_dst_host.html
folder: windows
---

### Overview

| Channel | Conditions | Events |
|---------|------------|--------|
| `Security` | Default configuration. <br><br> Also logged for other logon types (`Network`, `Console`, `Batch`, `Service`, ...). | Event [`4624: An account was successfully logged on`](./etw_authentication_dst_host.md#security-event-id-4624), with `LogonType` `10`. |
| `Microsoft-Windows-TerminalServices-RemoteConnectionManager/Operational` | Default configuration. | Event `1149: Remote Desktop Services: User authentication succeeded`. <br><br> Access to the Windows login screen, not necessarily a successful session opening. This event is however only generated upon successful authentication if `Network Level Authentication (NLA)` is required. |
| `Microsoft-Windows-TerminalServices-LocalSessionManager/Operational` | Default configuration. | Event `21: Remote Desktop Services: Session logon succeeded`. <br><br> Information of interest: user logging in, source network address, and RDP session ID. |
| `Microsoft-Windows-TerminalServices-LocalSessionManager/Operational` | Default configuration. | Event `22: Remote Desktop Services: Shell start notification received`. <br><br> Information of interest: user logging in, source network address, and RDP session ID. |
| `Microsoft-Windows-TerminalServices-LocalSessionManager/Operational` | Default configuration. | Event `23: Remote Desktop Services: Session logoff succeeded`. <br><br> Information of interest: user logging in and RDP session ID. |
| `Microsoft-Windows-TerminalServices-LocalSessionManager/Operational` | Default configuration. | Event `24: Remote Desktop Services: Session has been disconnected`. <br><br> If no `event 23` (`session logoff succeeded`) is associated with this event, the session was simply "disconnected". In case of a session disconnect (triggered by closing the RDP client window or using the "Disconnect" sign-out option), running processes and opened windows are preserved. <br><br> Information of interest: user logging in, source network address, and RDP session ID. |
| `Microsoft-Windows-TerminalServices-LocalSessionManager/Operational` | Default configuration. | Event `25: Remote Desktop Services: Session reconnection succeeded`. <br><br> Logged upon reconnection of a disconnected session. Could indicate that a malicious actor got access to running processes / programs state started legitimately by a user. <br><br> Information of interest: user logging in, source network address, and RDP session ID. <br><br> Events with a source network address set to `LOCAL` can sometimes be generated for console, non `Remote Desktop` logins. |
| `Microsoft-Windows-RemoteDesktopServices-RdpCoreTS/Operational` | Introduced in `Windows Server 2012`. <br><br> Default configuration. | Event `131: The server accepted a new TCP connection from client <IP>`. <br><br> Only indicates a network access to the `Remote Desktop` service. <br><br> Information of interest: source network address. |
| `Microsoft-Windows-TerminalServices-RemoteConnectionManager/Admin` | Default configuration. | Event `1158: Remote Desktop Services accepted a connection from IP address <IP>`. <br><br> Only indicates a network access to the Remote Desktop service. <br><br> Information of interest: source network address. |

#### ngrok tunnel - 16777216

A `Source Network Address` of `::%16777216` in the `Microsoft-Windows-TerminalServices-LocalSessionManager` and `Microsoft-Windows-TerminalServices-RemoteConnectionManager` events could
indicate that a `ngrok` tunnel was used to make `Remote Desktop` access.

### Tool(s)

The [`LogParser`'s KAPE module `LogParser_RDPUsageEvents`](./windows_etw_tools#cli-logparser)
can be used to parse EVTX files and extract the aforementioned `Remote Desktop`
events into a CSV timeline.

### References

  - [13Cubed -  RDP Event Log Forensics](https://www.youtube.com/watch?v=OlENso8_u7s)

  - [13Cubed - RDP Authentication vs. Authorization](https://www.youtube.com/watch?v=OlENso8_u7s)

  - [DFIR on the Mountain - RDP Event Log DFIR](https://dfironthemountain.wordpress.com/2019/02/15/rdp-event-log-dfir/)

  - [Stephan Berger @malmoeb - Tweet 16777216](https://x.com/malmoeb/status/1519710302820089857)
