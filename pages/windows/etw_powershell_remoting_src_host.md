---
title: ETW - PowerShell remoting - Source host
summary: 'Source host initiating a PowerShell remoting / WinRM access.\n\nMain events:\n\nChannel: Microsoft-Windows-Windows Remote Management/Operational.\nEvent ID 6: "Creating WSMan Session. The connection string is: <REMOTE_HOST>/wsman?PSVersion=XXX".'
keywords:
tags:
  - windows_etw
  - windows_powershell_activity
  - windows_lateral_movement
  - windows_lateral_movement_src
  - windows_powershell_remoting
  - windows_powershell_remoting_src
location: 'Channel:\n\nMicrosoft-Windows-Windows Remote Management/Operational.\nEvents: 2, 4, 6, 8, 12, 15, 16, 30, 31, 33, 80, 162, 166.'
last_updated: 2024-01-22
sidebar: sidebar
permalink: windows_etw_powershell_remoting_src_host.html
folder: windows
---

### Overview

| Channel | Conditions | Events |
|---------|------------|--------|
| Channel: <br> `Microsoft-Windows-Windows Remote Management/Operational` <br><br> EVTX file: <br> `Microsoft-Windows-WinRM\Operational.evtx` | Default configuration (starting with PowerShell 2.0). | Event `6: Creating WSMan Session. The connection string is: <IP_ADDRESS | HOSTNAME>/wsman?PSVersion=XXX`. <br><br> Logged whenever an attempt, successful or not, was made to open a `WinRM` session. This event does NOT necessarily indicates that a `WinRM` session was opened on the remote host. <br><br> Information of interest: <br> - Domain and username of the currently logged-in user on the source host (not necessarily the credentials that were used to attempt to open the `WinRM` session on the remote host). <br> - The remote host IP address or hostname. |
| Channel: <br> `Microsoft-Windows-Windows Remote Management/Operational` <br><br> EVTX file: <br> `Microsoft-Windows-WinRM\Operational.evtx` | Default configuration (starting with PowerShell 2.0). | Multiple events are generated during the life-cycle of a `WinRM session`: <br><br> Event `2: Initializing WSMan API`. <br><br> Event `4: Deinitializing WSMan API`. <br><br> Event `8: Closing WSMan Session`. <br><br> Event `15: Closing WSMan command`. <br><br> Event `16: Closing WSMan shell`. <br><br> Event `30: Deinitialization of WSMan API completed successfuly`. <br><br> Event `31: WSMan Create Session operation completed successfuly`. <br><br> Event `33: Closing WSMan Session completed successfully`. <br><br> These events do not provide information on the session, the remote host, or the remote user. Only the domain and username of the currently logged-in user on the source host are specified. |
| Channel: <br> `Microsoft-Windows-Windows Remote Management/Operational` <br><br> EVTX file: <br> `Microsoft-Windows-WinRM\Operational.evtx` | Default configuration (starting with PowerShell 2.0). | Event `162: Authenticating the user failed. The credentials didn't work`. <br><br> Indicates that an attempt to open a `WinRM` session failed because the provided credentials were invalid. <br><br> Information of interest: <br> - Domain and username of the currently logged-in user on the source host. <br> - The event usually occurs in very close proximity (i.e. same second) to an event `6`, allowing correlation and deduction of the remote host. |
| Channel: <br> `Microsoft-Windows-Windows Remote Management/Operational` <br><br> EVTX file: <br> `Microsoft-Windows-WinRM\Operational.evtx` | Default configuration (starting with PowerShell 2.0). | Event `12: WSMan shell creation failed, error code 2150858980`. <br><br> Indicates that an attempt to open a `WinRM` session failed because the authentication schema is not `Kerberos`, the transport protocol is not `HTTPS`, or the remote host is not added in the `TrustedHosts` registry key on the source host. <br><br> This error usually occurs when authentication is conducted in `NTLM` (whenever the remote host IP address is specified instead of an hostname) or [for nested / double hop remote access](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/kerberos-double-hop-problem). <br><br> Information of interest: <br> - Domain and username of the currently logged-in user on the source host. <br> - The event usually occurs in very close proximity (i.e. same second) to an event `6`, allowing correlation and deduction of the remote host. |
| Channel: <br> `Microsoft-Windows-Windows Remote Management/Operational` <br><br> EVTX file: <br> `Microsoft-Windows-WinRM\Operational.evtx` | Default configuration (**PowerShell 2.0 only**). | Event `166: The chosen authentication mechanism is <AUTHENTICATION_SCHEMA>`. <br><br> Indicates the authentication schema, such as `Negotiate`, used to authenticate the `WinRM` session. <br><br> Information of interest: <br> - Domain and username of the currently logged-in user on the source host. <br> - The event usually occurs in very close proximity (i.e. same second) to an event `6`, allowing correlation and deduction of the remote host. |
| Channel: <br> `Microsoft-Windows-Windows Remote Management/Operational` <br><br> EVTX file: <br> `Microsoft-Windows-WinRM\Operational.evtx` | Default configuration (**PowerShell 2.0 only**). | Event `80: Sending the request for operation DeleteShell to destination machine and port <IP_ADDRESS | HOSTNAME>:<5985 | 5986>`. <br><br> Information of interest: <br> - Domain and username of the currently logged-in user on the source host. |

More information on the PowerShell code executed can be available if
`Module Logging` and / or `Script Block Logging` are enabled. For more
information, refer to the [PowerShell activity page](./etw_powershell.md).
