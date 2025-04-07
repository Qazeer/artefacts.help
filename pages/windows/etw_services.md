---
title: ETW - Windows Services
summary: 'For local Windows services creation and operations.\n\nMain events:\n\nChannel: System.\nEvent ID 7045: "A service was installed in the system".\nEvent ID 7036: "The <SERVICE_NAME> service entered the <running/stopped> state".\n\nChannel: Security.\nEvent ID 4697: "A service was installed in the system" (not enabled by default).'
keywords: Windows service, services, installed, running, stopped, 7045, 7036, 7035, 7000, 7023, 7031, 7034, 7040, 7030, 4697
tags:
  - windows_etw
  - windows_lateral_movement
  - windows_lateral_movement_dst
  - windows_local_persistence
location: 'Channel: System.\nEvents: 7045, 7036, 7035, 7000, 7023, 7031, 7034, 7040, 7030.\n\nChannel: Security.\nEvent: 4697.'
last_updated: 2024-01-19
sidebar: sidebar
permalink: windows_etw_services.html
folder: windows
---

### Overview

Windows services are programs that operate in the background and conform to the
interface rules and protocols of the `Service Control Manager (SCM)` (the
component responsible for managing Windows services). Services can be
implemented as binaries or `Dynamic Link Libraries (DLL)`. The services
implemented as `DLL` are loaded and executed by an instance of the
`Service Host` (`svchost.exe`) process.

Refer to the [registry Windows services page](./registry_services.md) for more
information on Windows services.

### Services Windows events

| Channel | Conditions | Events |
|---------|------------|--------|
| `System` <br><br> Provider: `Service Control Manager`. | Default configuration. | Event `7045: A service was installed in the system`. <br><br> Logged whenever a Windows service is created on the machine. <br><br> Information of interest: <br> - Domain and username of the user that installed the service (often the `NT AUTHORITY\SYSTEM` account). <br> - Name, binary path, type, start type and executing account of the created service. |
| `Security` | Introduced in `Windows Server 2016` and `Windows 10`. <br><br> Requires: <br> `Audit: Force audit policy subcategory settings` to be enabled. <br> And `Other Object Access Events` set to `Success(, Failure)`. | Event `4697: A service was installed in the system`. <br><br> Logged whenever a Windows service is created on the machine. <br><br> Information of interest: <br> - Domain, username and Logon ID of the user that installed the service (often the `NT AUTHORITY\SYSTEM` account). <br> - Name, binary path, type, start type and executing account of the created service. <br><br> Legacy (`Windows Server 2003` and `Windows XP`): <br> Event `601: Attempt to install service`. |
| `System` <br><br> Provider: `Service Control Manager`. | Default configuration. | Event `7036: The <SERVICE_NAME> service entered the <running/stopped> state`. <br><br> Logged whenever a Windows service is effectively stated/stopped. <br><br> Information of interest: the name of the service impacted and the account used to instruct the service to start/stop. |
| `System` <br><br> Provider: `Service Control Manager`. | Logged only on <= `Windows XP` and `Windows Server 2003`. | Event `7035: The <SERVICE_NAME> service was successfully sent a <start/stop> control`. <br><br> Logged whenever a Windows service is instructed to start/stop. <br><br> Information of interest: the name of the service impacted. The account used to instruct the service to start/stop does not appear to be consistently logged. |
| `System` <br><br> Provider: `Service Control Manager`. | Default configuration. | Event `7000: The <SERVICE_NAME> service failed to start due to the following error: <ERROR>`. <br><br> Logged whenever a Windows service fails to start due to an error (such as an invalid / not found binary path, a driver that was blocked from running, etc.). <br><br> Information of interest: the name of the service. |
| `System` <br><br> Provider: `Service Control Manager`. | Default configuration. | Event `7023: The <SERVICE_NAME> service failed terminated with the following error: <ERROR>`. <br><br> Logged whenever a Windows service terminate with an error. <br><br> Information of interest: the name of the service. |
| `System` <br><br> Provider: `Service Control Manager`. | Default configuration. | Event `7031: The <SERVICE_NAME> service terminated unexpectedly. It has done this <NUMBER_OF_FAILURES> time(s). The following corrective action will be taken in <SECONDS> milliseconds. <ACTION>`. <br><br> Logged whenever a Windows service terminate unexpectedly. The usual corrective action is to restart the service ("The following corrective action will be taken in 1000 milliseconds: Restart the service"). <br><br> Information of interest: the name of the service. |
| `System` <br><br> Provider: `Service Control Manager`. | Default configuration. | Event `7034: The <SERVICE_NAME> service terminated unexpectedly. It has done this <NUMBER_OF_FAILURES> time(s)`. <br><br> Logged whenever a Windows service terminate unexpectedly. <br><br> Information of interest: the name of the service. |
| `System` <br><br> Provider: `Service Control Manager`. | Default configuration. | Event `7040: The start type of the <SERVICE_NAME> service was changed from demand <OLD_START_TYPE> to <NEW_START_TYPE>`. <br><br> Logged whenever there is a change to a service start type. <br><br> Information of interest: the name of the service impacted and the account that modified the service start type. |
| `System` <br><br> Provider: `Service Control Manager`. | Introduced in `Windows Vista` and `Windows Server 2008`. | Event `7030: The <SERVICE_NAME> service is marked as an interactive service`. <br><br> Logged whenever a service is configured as an interactive service, which is not supported since `Windows Vista` and `Windows Server 2008 (du to security risks posed by interactive services). <br><br> Information of interest: the name of the service. |

### References

  - [Tony Phipps - SIEM - Notable-Event-IDs](https://github.com/TonyPhipps/SIEM/blob/master/Notable-Event-IDs.md#system-events)
