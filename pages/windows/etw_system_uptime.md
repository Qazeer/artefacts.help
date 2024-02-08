---
title: ETW - System uptime
summary: 'System boot and shutdown events, to determine the time ranges that the system was turned on.\n\nMain events:\n\nEvent 1074: "The process <PROCESS_EXE> has initiated the xxx of computer <HOSTNAME> on behalf of user <USERNAME> for the following reason: <SHUTDOWN_REASON_TEXT>".\n\nEvent 12: "The operating system started at system time <TIME>".\n\nEvent 13: "The operating system is shutting down at system time <TIME>".\n\nEvent 41: "The system has rebooted without cleanly shutting down first".'
keywords: 'system uptime, User32, Microsoft-Windows-Kernel-General, Microsoft-Windows-Kernel-Power, Microsoft-Windows-Power-Troubleshooter, EventLog, TurnedOnTimesView'
tags:
  - windows_etw
  - windows_system_information
location: 'File: System.evtx.\n\nChannels:\n\nUser32.\nEvents: 1074.\n\nMicrosoft-Windows-Kernel-General.\nEvents: 12, 13.\n\nMicrosoft-Windows-Kernel-Power.\nEvents: 41, 42, 109.\n\nMicrosoft-Windows-Power-Troubleshooter.\nEvents: 1.\n\nEventLog.\nEvents: 6013, 6005, 6006.'
last_updated: 2024-01-06
sidebar: sidebar
permalink: windows_etw_system_uptime.html
folder: windows
---

### Overview

| Channel | Events |
|---------|--------|
| Channel: <br> `User32`. <br><br> File: `System.evtx`. | Event `1074: The process <PROCESS_EXE> has initiated the xxx of computer <HOSTNAME> on behalf of user <USERNAME> for the following reason: <SHUTDOWN_REASON_TEXT>` |
| Channel: <br> `Microsoft-Windows-Kernel-General`. <br><br> File: `System.evtx`. | Event `12: The operating system started at system time <TIME>` |
| Channel: <br> `Microsoft-Windows-Kernel-General`. <br><br> File: `System.evtx`. | Event `13: The operating system is shutting down at system time <TIME>` |
| Channel: <br> `Microsoft-Windows-Kernel-Power`. <br><br> File: `System.evtx`. | Event `41: The system has rebooted without cleanly shutting down first. This error could be caused if the system stopped responding, crashed, or lost power unexpectedly` |
| Channel: <br> `Microsoft-Windows-Kernel-Power`. <br><br> File: `System.evtx`. | Event `42: The system is entering sleep` |
| Channel: <br> `Microsoft-Windows-Kernel-Power`. <br><br> File: `System.evtx`. | Event `109: The kernel power manager has initiated a shutdown transition. Shutdown Reason: <SHUTDOWN_REASON_INT>` |
| Channel: <br> `Microsoft-Windows-Power-Troubleshooter`. <br><br> File: `System.evtx`. | Event `1: The system has resumed from sleep` |
| Channel: <br> `EventLog`. <br><br> File: `System.evtx`. | Event: `6013: The system uptime is <INT> seconds.` |
| Channel: <br> `EventLog`. <br><br> File: `System.evtx`. | Event `6005: The Event log service was started` |
| Channel: <br> `EventLog`. <br><br> File: `System.evtx`. | Event `6006: The Event log service was stopped` |

### Tool(s)

The
[`TurnedOnTimesView`](https://www.nirsoft.net/utils/computer_turned_on_times.html)
utility can be used to parse `System.evtx` files and determine the time ranges
that a system was turned on (by looking as a set of the aforementioned events).

### References

  - [Nir Sofer - TurnedOnTimesView README](https://www.nirsoft.net/utils/computer_turned_on_times.html)
