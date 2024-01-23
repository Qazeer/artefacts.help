---
title: ETW - PowerShell remoting - Destination host
summary: 'Destination host of a PowerShell remoting / WinRM access.\n\nMain events:\n\nChannel: Microsoft-Windows-Windows Remote Management/Operational.\nEvent ID 91: "Creating WSMan shell on server with ResourceUri: <X>".'
keywords: PowerShell remoting, WinRM, WSMan, ServerRemoteHost, 91
tags:
  - windows_etw
  - windows_powershell_activity
  - windows_lateral_movement
  - windows_lateral_movement_dst
  - windows_powershell_remoting
  - windows_powershell_remoting_dst
location: 'Channels:\n\nMicrosoft-Windows-Windows Remote Management/Operational.\nEvent: 91.\n\nWindows PowerShell.\nEvents: 400, 403, 600.\nWith the HostName field set to "ServerRemoteHost".'
last_updated: 2024-01-22
sidebar: sidebar
permalink: windows_etw_powershell_remoting_dst_host.html
folder: windows
---

### Overview

| Channel | Conditions | Events |
|---------|------------|--------|
| Channel: <br> `Microsoft-Windows-Windows Remote Management/Operational` <br><br> EVTX file: <br> `Microsoft-Windows-WinRM\Operational.evtx` | Default configuration (starting with PowerShell 2.0). | Event `91: Creating WSMan shell on server with ResourceUri: <X>`. <br><br> Indicates that a remote `WinRM` session was opened. <br><br> Information of interest: <br> - Domain and username of the user that opened the session. <br> - Does NOT include the source host. |
| `Windows PowerShell` | Default configuration (starting with PowerShell 2.0). | Event `400: Engine state is changed from None to Available`. <br><br> Logged at the start of any local or remote PowerShell activity, with the `Hostname` field set to `ServerRemoteHost` for `WinRM` session. <br><br> The `RunaspaceId` identify the PowerShell activity and can be linked to the session termination (`EID 403`). <br><br> This event provides no information on the source host or on the user that performed the access. |
| `Windows PowerShell` | Default configuration (starting with PowerShell 2.0). | Event `403: Engine state is changed from Available to Stopped`. <br><br> Logged at the end of any local or remote PowerShell activity, with the `Hostname` field set to `ServerRemoteHost` for `WinRM` session. <br><br> The event contains the same level of information as the `EID 400` event. |
| `Windows PowerShell` | Default configuration (starting with PowerShell 2.0). | Event `600: Provider "<PROVIDER_NAME>" is Started`. <br><br> Logs the start and stop of a PowerShell provider, with the `HostName` field set to `ServerRemoteHost` for `WinRM` session. <br><br> Additionally, the provider `WSMan` ("Provider WSMan Is Started") is loaded in case of a `WinRM` session. |

More information on the PowerShell code executed can be available if
`Module Logging` and / or `Script Block Logging` are enabled. For more
information, refer to the [PowerShell activity page](./etw_powershell.md).

### References

  - [Attack and Defense Around PowerShell Event Logging](https://nsfocusglobal.com/Attack-and-Defense-Around-PowerShell-Event-Logging/)
