---
title: ETW - Registry Auto-Start Extensibility Points
summary: 'Events are generated for tasks executed through the Run and RunOnce registry keys. Additionally, events can be generated for modification of registry keys, but requires non-default audit settings and the configuration of SACL on the registry keys to audit.\n\nMain events:\n\nChannel: Microsoft-Windows-Shell-Core/Operational.\nEvent ID 9707: "Started execution of command <COMMAND>".\nEvent ID 9708: "Finished execution of command <COMMAND> (PID <PROCESS_ID>)".\n\nChannel: Security.\nEvent ID 4657: "A registry value was modified".\nRequires non-default audit settings and the configuration of SACL on the registy keys to audit.'
keywords: Run, RunOnce, ASEP, registry modification
tags:
  - windows_etw
  - windows_program_execution
  - windows_local_persistence
location: 'Channels:\n\nMicrosoft-Windows-Shell-Core/Operational.\nEvents: 9705, 9707, 9708.\n\nSecurity.\nEvent: 4657.'
last_updated: 2024-01-22
sidebar: sidebar
permalink: windows_etw_registry_asep.html
folder: windows
---

### Overview

A number of registry keys, known as `Auto-Start Extensibility Points (ASEP)`
registry keys, are run whenever the system is booted or a specific user logs
in.

Events are generated for tasks executed through the `Run` / `RunOnce` registry
keys, in the `Microsoft-Windows-Shell-Core/Operational` channel. Additionally,
events can be generated for modification of registry keys, but requires
non-default audit settings and the configuration of
`System Access Control List` (`SACL`) on the registry keys to audit.

For more information on registry `ASEP` keys, refer to the
[Registry - Auto-Start Extensibility Points page](./registry_asep.md).

### Registry ASEP execution events

| Channel | Conditions | Events |
|---------|------------|--------|
| `Microsoft-Windows-Shell-Core/Operational` | Default configuration. <br><br> Introduced in `Windows 10` and `Windows Server 2016`. | Event `9707: Started execution of command '<COMMAND>'`. <br><br> Logged whenever a task is executed through the `Run` / `RunOnce` registry keys. <br><br> Information of interest: <br> - Domain and username of the user for whom the task executed. <br> - Command line of the task (with only the program filename and not its full path, even if the full path is specified in the registry). |
| `Microsoft-Windows-Shell-Core/Operational` | Default configuration. <br><br> Introduced in `Windows 10` and `Windows Server 2016`. | Event `9708: Finished execution of command '<COMMAND>' (PID <PROCESS_ID>)`. <br><br> Logged whenever a task executed through the `Run` / `RunOnce` registry keys finishes execution. <br><br> Information of interest: <br> - Domain and username of the user for whom the task executed. <br> - Command line of the task (with only the program filename and not its full path, even if the full path is specified in the registry). <br> - `Process ID` of the task process. |
| `Microsoft-Windows-Shell-Core/Operational` | Default configuration. <br><br> Introduced in `Windows 10` and `Windows Server 2016`. | Event `9705: Started enumeration of commands for registry key '<Software\Microsoft\Windows\CurrentVersion\Run | Software\Microsoft\Windows\CurrentVersion\RunOnce>'`. <br><br> Logged whenever the system enumerates the configured `Run` or `RunOnce` registry key's tasks, before their execution. <br><br> Information of interest: <br> - Domain and username of the user for whom the enumeration was performed. |

### Registry ASEP defintion events

| Channel | Conditions | Events |
|---------|------------|--------|
| `Security` | Requires: <br><br> - `Audit: Force audit policy subcategory settings` to be enabled. <br><br> - `Audit object access` set to `Success(, Failure)`. <br><br> - The `System Access Control List` (`SACL`) on the `ASEP` registry keys to define audit on the rights `Create Subkey`, `Set Value`, `Create Link`, `Write DAC`, and `Delete` for the user conducting the action (possibly through identity / group membership, such as, for example, `Everyone`). <br><br> **-> very likely not logged.** | Event `4657: A registry value was modified`. <br><br> Logged whenever a user modify a registry key for which the audit policy is set to audit usage of the `Set Value` rights (by the user). <br><br> Information of interest: <br> - Domain, username, and `LogonID` of the user conducting the modification. <br> - The registry key modified and the new value defined. |

### References

  - [Nasreddine Bencherchali - Finding Forensic Goodness In Obscure Windows Event Logs](https://nasbench.medium.com/finding-forensic-goodness-in-obscure-windows-event-logs-60e978ea45a3)
