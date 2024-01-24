---
title: ETW - WMI events
summary: 'For WMI activity.\n\nTracking process execution is the only way to natively detect lateral movement leveraging WMI. With out "Audit process tracking" enabled to log process creation event 4688 (or a dedicated product tracking process creation, such as Sysmon or an EDR), lateral movement over WMI cannot be reliably investigated.\n\nMain events:\n\nChannel: Security.\nEvent ID 4688: "A new process has been created", to track WMI process execution (wmic.exe and WmiPrvSE.exe notably).\n\nChannel: Microsoft-Windows-WMI-Activity/Operational.\nEvent ID 5860 for temporary WMI Event subscription creation.\nEvent ID 5861 for permanent WMI Event subscription creation.'
keywords: WMI, wmic, node, WmiPrvSE, WMI Event subscription, Event subscription, 4688
tags:
  - windows_etw
  - windows_wmi
  - windows_lateral_movement
  - windows_local_persistence
location: 'Channels:\n\nSecurity.\nEvent: 4688.\n\nMicrosoft-Windows-WMI-Activity/Operational.\nEvents: 5857, 5858, 5859, 5860, 5861.'
last_updated: 2024-01-24
sidebar: sidebar
permalink: windows_etw_wmi.html
folder: windows
---

### Process execution

Tracking process execution is the only way to natively detect lateral movement
leveraging `WMI`. With out `Audit process tracking` enabled to log process
creation event `4688` (or a dedicated product tracking process creation, such
as `Sysmon` or an `EDR`), lateral movement over `WMI` cannot be reliably
detected or investigated.

| Channel | Conditions | Events |
|---------|------------|--------|
| `Security` | Requires `Audit Process Creation` to be enabled. <br><br> Requires `ProcessCreationIncludeCmdLine_Enabled` to be enabled for the command line to be logged. | Event `4688: A new process has been created`. <br><br> Refer to the [ETW - Process creation page](./etw_process_creation.md) for more general information on the event. <br><br> The event can be used to track the execution of [programs related to `WMI`](./wmi_process.md). For instance, the execution of `wmic [...] /node:<REMOTE_HOST>` can be an indicator of outgoing lateral movement using `WMI` on the source host, and process with `WmiPrvSE.exe` as a parent-process can be an indicator of incoming lateral movement using `WMI` on the destination host. |

### WMI Event subscription

| Channel | Conditions | Events |
|---------|------------|--------|
| `Microsoft-Windows-WMI-Activity/Operational` | Introduced in `Windows Sever 2012R2`. <br><br> Default configuration. | Event `5858: Operation_ClientFailure`. <br><br> Logged for error in a `WMI` operation, such as, for example, a `WMI` query ([`IWbemServices::ExecQuery`](https://learn.microsoft.com/en-us/windows/win32/api/wbemcli/nf-wbemcli-iwbemservices-execquery) or [IWbemServices::ExecNotificationQuery](https://learn.microsoft.com/en-us/windows/win32/api/wbemcli/nf-wbemcli-iwbemservices-execnotificationquery)), an object retrieval ([IWbemServices::GetObject](https://learn.microsoft.com/en-us/windows/win32/api/wbemcli/nf-wbemcli-iwbemservices-getobject)), or a `CIM`-object exported method execution ([IWbemServices::ExecMethod](https://learn.microsoft.com/en-us/windows/win32/api/wbemcli/nf-wbemcli-iwbemservices-execmethod)). <br><br> Information of interest: <br> - Domain and username of the user that conducted the `WMI` operation. <br> - The result code of the failed operation. <br> - The `Process ID (PID)` of the process that conducted the operation. <br> - Details on the operation that raised the error. <br><br>   For `WMI` query, the query string is included. <br> For example: `Start IWbemServices::ExecQuery - ROOT\CIMV2 : SELECT * FROM Win32_ComputerSystemProduct`. <br><br> For method execution, the `WMI` namespace, the class, and the method called are included, in the following format <br> `IWbemServices::ExecMethod - <NAMESPACE> : <CLASS>::<METHOD>`. <br> For example: `IWbemServices::ExecMethod - ROOT\Microsoft\Windows\Smb : MSFT_SmbShare::FireShareChangeEvent`. |
| `Microsoft-Windows-WMI-Activity/Operational` | Introduced in `Windows Sever 2012R2`. <br><br> Default configuration. | Event `5857: <PROVIDER_NAME> provider started with result code 0x0. HostProcess = wmiprvse.exe; ProcessID = <PID>; ProviderPath = <PROVIDER_DLL_PATH>`. <br><br> Logged whenever a provider is loaded by `WMI`. <br><br> `WMI` uses providers to access system components (that it then exposes as classes). As providers are loaded under `NT AUTHORITY\SYSTEM`, they can be a way to maintain elevated persistence on the system. <br><br> Information of interest: <br> - Provider name and `DLL` full path. <br> - The `Process ID (PID)` of the `wmiprvse.exe` process that loaded the provided. |
| `Microsoft-Windows-WMI-Activity/Operational` | Introduced in `Windows Sever 2012R2`. <br><br> Default configuration. | Event `5860: Operation_TemporaryEssStarted`. <br><br> Logged whenever a [**temporary** `WMI Event Subscription`](./wmi_event_subscriptions.md) is configured. <br><br> Information of interest: <br> - Domain and username of the user executing the temporary subscription. <br> - `Process ID (PID)` of the process under which the subscription is executed. <br> - `WMI` namespace and query of the temporary subscription. |
| `Microsoft-Windows-WMI-Activity/Operational` | Introduced in `Windows Sever 2012R2`. <br><br> Default configuration. | Event `5861: Operation_ESStoConsumerBinding`. <br><br> Logged whenever a [**permanent** `WMI Event Subscription`](./wmi_event_subscriptions.md) is configured (more precisely when a [`filter to consumer binding`](./wmi_event_subscriptions.md) is configured). <br><br> Information of interest: <br> - Details on the [`event filter`](./wmi_event_subscriptions.md): name, creator `SID`, `WMI` namespace, `WMI` query language and query. <br> - Details on the [`event consumer`](./wmi_event_subscriptions.md): name, type of consumer (such as `CommandLineEventConsumer` for program execution), creator `SID`, and consumer parameters (such as the program path and command line for `CommandLineEventConsumer` consumer). |
| `Microsoft-Windows-WMI-Activity/Operational` | Introduced in `Windows Sever 2012R2`. <br><br> Default configuration. | Event `5859: Operation_EssStarted`. <br><br> Appears to be logged in relation with some permanent `WMI Event Subscriptions`. <br><br> Information of interest: <br> - `WMI` query assocatied with a permanent subscription. |

### References

  - [darkoperator - Carlos PEREZ - Basics of Tracking WMI Activity](https://www.darkoperator.com/blog/2017/10/14/basics-of-tracking-wmi-activity)

  - [Network Security Ninja - WMI Forensics](https://netsecninja.github.io/dfir-notes/wmi-forensics/)

  - [SANS - Chad Tilbury - Investigating WMI Attacks](https://www.youtube.com/watch?v=aBQ1vEjK6v4)
