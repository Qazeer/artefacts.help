---
title: ETW - WMI events
summary: 'For WMI activity.\n\nTracking process execution is the only way to natively detect lateral movement leveraging WMI. With out "Audit process tracking" enabled to log process creation event 4688 (or a dedicated product tracking process creation, such as Sysmon or an EDR), lateral movement over WMI cannot be reliably investigated.\n\nMain events:\n\nChannel: Security.\nEvent ID 4688: "A new process has been created", to track WMI process execution (wmic.exe and WmiPrvSE.exe notably).\n\nChannel: Microsoft-Windows-WMI-Activity/Operational.\n Event ID 5861 for permanent WMI Event subscription creation.'
keywords: WMI, wmic, node, WmiPrvSE, WMI Event subscription, Event subscription, 4688
tags:
  - windows_etw
  - windows_wmi
  - windows_local_persistence
  - windows_lateral_movement
location: 'Channels:\n\nSecurity.\nEvent: 4688.\n\nMicrosoft-Windows-WMI-Activity/Operational.\nEvents: 5858, 5859, 5860, 5861.'
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
| `Microsoft-Windows-WMI-Activity/Operational` | Introduced in `Windows Sever 2012R2`. <br><br> Default configuration. | Event `5858: Operation_ClientFailure`. |
| `Microsoft-Windows-WMI-Activity/Operational` | Introduced in `Windows Sever 2012R2`. <br><br> Default configuration. | Event `5859: Operation_EssStarted:`. |
| `Microsoft-Windows-WMI-Activity/Operational` | Introduced in `Windows Sever 2012R2`. <br><br> Default configuration. | Event `5860: Operation_TemporaryEssStarted`. |
| `Microsoft-Windows-WMI-Activity/Operational` | Introduced in `Windows Sever 2012R2`. <br><br> Default configuration. | Event `5861: Operation_ESStoConsumerBinding`. |

### References

  - [Network Security Ninja - WMI Forensics](https://netsecninja.github.io/dfir-notes/wmi-forensics/)

  - [SANS - Chad Tilbury - Investigating WMI Attacks](https://www.youtube.com/watch?v=aBQ1vEjK6v4)
