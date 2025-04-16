---
title: ETW - Windows Scheduled Tasks
summary: 'For local Windows Scheduled Tasks creation and operations.\n\nMain events:\n\nChannel: Microsoft-Windows-TaskScheduler/Operational (channel not enabled by default).\nEvent ID 106: "User "<ACCOUNT>" registered Task Scheduler task "\<TASK_NAME>"".\nEvent ID 140: "User "<ACCOUNT>" updated Task Scheduler task "<TASK_NAME>"".\nEvent ID 200: "Task Scheduler launched action "<EXECUTABLE>" in instance "<INSTANCE_GUID>" of task "<TASK_NAME>"".\n\nChannel: Security (events not enabled by default).\nEvent ID 4698: "A scheduled task was created".\nEvent ID 4702: "A scheduled task was updated".'
keywords: TaskScheduler, Windows Scheduled Tasks, Scheduled Tasks, Task, Microsoft-Windows-TaskScheduler/Operational, 100, 102, 103, 106, 107, 108, 110, 118, 119, 129, 140, 141, 200, 201
tags:
  - windows_etw
  - windows_lateral_movement
  - windows_lateral_movement_dst
  - windows_local_persistence
  - windows_local_persistence_user
location: 'Channel: Microsoft-Windows-TaskScheduler/Operational (channel not enabled by default).\nEvents: 100, 102, 103, 106, 107, 108, 110, 118, 119, 129, 140, 141, 200, 201.\n\nChannel: Security (events not enabled by default).\nEvents: 4698, 4699, 4700, 4701, 4702.'
last_updated: 2024-01-19
sidebar: sidebar
permalink: windows_etw_scheduled_task.html
folder: windows
---

### Overview

Scheduled tasks are used to automatically perform a task on the system whenever
the criteria associated to the scheduled task occurs. The scheduled tasks can
either be run at a defined time, on repeat at set intervals, or when a specific
event occurs, such as the system boot.

A single scheduled task can be associated with one or multiple trigger(s) and
one or multiple action(s). A single task can thus execute multiple distinct
executables.

Refer to the [registry Scheduled Tasks page](./registry_scheduled_tasks.md) for
more information on the components that constitute scheduled tasks.

### Scheduled Tasks Windows events

| Channel | Conditions | Events |
|---------|------------|--------|
| `Microsoft-Windows-TaskScheduler/Operational` | Introduced in `Windows 7` and `Windows 2008`. <br><br> Requires task history to be enabled (non-default). | Event `106: User "<DOMAIN | WORKGROUP>\<USERNAME>" registered Task Scheduler task "\<TASK_NAME>"`. <br><br> Logged whenever a scheduled task is registered. <br><br> Information of interest: <br> - The registered task name. <br> - The domain and username of the user that registered the task. |
| `Microsoft-Windows-TaskScheduler/Operational` | Introduced in `Windows 7` and `Windows 2008`. <br><br> Requires task history to be enabled (non-default). | Event `140: User "<DOMAIN | WORKGROUP>\<USERNAME>" updated Task Scheduler task "<TASK_NAME>"`. <br><br> Logged whenever a scheduled task is updated. <br><br> Information of interest: <br> - The modified task name. <br> - The domain and username of the user that modified the task. <br><br> **The properties modified are not logged.** |
| `Microsoft-Windows-TaskScheduler/Operational` | Introduced in `Windows 7` and `Windows 2008`. <br><br> Requires task history to be enabled (non-default). | Event `107: Task triggered on scheduler`. <br> Event `108: Task triggered on event`. <br> Event `110: Task triggered by user`. <br> Event `118: Task triggered by computer startup`. <br> Event `119: Task triggered on logon`. <br><br> Event payload for each event: `Task Scheduler launched "<INSTANCE_GUID>" instance of task "<TASK_NAME>" due to [...]`. <br><br> Logged whenever a scheduled task is started due to the criteria associated with the event (schedule, event, system startup, logon, or manual trigger). <br><br> Information of interest: <br> - The launched task name. <br> - The execution instance `GUID`. <br> - The task execution reason. |
| `Microsoft-Windows-TaskScheduler/Operational` | Introduced in `Windows 7` and `Windows 2008`. <br><br> Requires task history to be enabled (non-default). | Event `100: Task Scheduler started  <INSTANCE_GUID>" instance of the "<TASK_NAME>" task for user "<EXECUTING_ACCOUNT>"`. <br><br> Logged whenever a scheduled task is executed. <br><br> Information of interest: <br> - The launched task name. <br> - The execution instance `GUID` and the account running the task. |
| `Microsoft-Windows-TaskScheduler/Operational` | Introduced in `Windows 7` and `Windows 2008`. <br><br> Requires task history to be enabled (non-default). | Event `129: Task Scheduler launch task "<TASK_NAME>", instance "<EXECUTABLE>" with process ID <PID>`. <br><br> Logged whenever a scheduled task or a scheduled task's action is executed. <br><br> Information of interest: <br> - The launched task name and the launched action's executable full path. <br> - The execution instance `GUID` and associated `process identifier` (`PID`). |
| `Microsoft-Windows-TaskScheduler/Operational` | Introduced in `Windows 7` and `Windows 2008`. <br><br> Requires task history to be enabled (non-default). | Event `200: Task Scheduler launched action "<EXECUTABLE>" in instance "<INSTANCE_GUID>" of task "<TASK_NAME>"`. <br><br> Logged whenever a scheduled task's action is executed. A single scheduled task can define one or multiple action(s). <br><br> Information of interest: <br> - The launched task name and the launched action's executable full path. <br> - The execution instance `GUID`. <br><br> This event can be used to correlate a task name with its / one of its executable. |
| `Microsoft-Windows-TaskScheduler/Operational` | Introduced in `Windows 7` and `Windows 2008`. <br><br> Requires task history to be enabled (non-default). | Event `103: Task Scheduler failed to start instance "<INSTANCE_GUID>" instance of the "<TASK_NAME>" task for user "<EXECUTING_ACCOUNT>". Additional Data: Error Value: <ERROR_CODE>`. <br><br> Information of interest: <br> - The launched task name. <br> - The execution instance `GUID` and the account running the task. <br> - The error code associated with the start failure. |
| `Microsoft-Windows-TaskScheduler/Operational` | Introduced in `Windows 7` and `Windows 2008`. <br><br> Requires task history to be enabled (non-default). | Event `201: Task Scheduler successfully completed task "<TASK_NAME>", instance "<INSTANCE_GUID>" , action "<EXECUTABLE>" with return code <INT>"`. <br><br> Logged whenever a scheduled task's action finished its execution. <br><br> Information of interest: <br> - The launched task name and the finished action's executable full path. <br> - The execution instance `GUID`. <br> - The execution return code. <br><br> Similarly to event `200`, this event can be used to correlate a task name with its / one of its executable. |
| `Microsoft-Windows-TaskScheduler/Operational` | Introduced in `Windows 7` and `Windows 2008`. <br><br> Requires task history to be enabled (non-default). | Event `102: Task Scheduler successfully finished "<INSTANCE_GUID>" instance of the "<TASK_NAME>" task for user "<EXECUTING_ACCOUNT>"`. <br><br> Logged whenever a scheduled task action finished its execution. <br><br> Information of interest: <br> - The finished task name. <br> - The finished execution instance `GUID` and the account that ran the task. |
| `Microsoft-Windows-TaskScheduler/Operational` | Introduced in `Windows 7` and `Windows 2008`. <br><br> Requires task history to be enabled (non-default). | Event `141: User "<DOMAIN | WORKGROUP>\<USERNAME>" deleted Task Scheduler task "<TASK_NAME>"`. <br><br> Logged whenever a scheduled task is deleted. <br><br> Information of interest: <br> - The deleted task name. <br> - The domain and username of the user that deleted the task. |
| `Security` | `Audit: Force audit policy subcategory settings` to be enabled. <br><br> And `Other Object Access Events` set to `Success(, Failure)`. | Event `4698: A scheduled task was created`. <br><br> Event `4699: A scheduled task was deleted`. <br><br> Event `4700: A scheduled task was enabled`. <br><br> Event `4701: A scheduled task was disabled`. <br><br> Event `4702: A scheduled task was updated`. <br><br> Logged whenever the operation associated with the event (creation, deletion, enabling, disabling, modification) is performed on a scheduled task. <br><br> Each event holds the same following information of interest: <br> - Domain, username and `Logon ID` of the user that performed the action. <br> - The scheduled task full parameters: task name, registration timestamp, action(s) (including the associated command(s)), trigger(s), running user and privileges, etc. <br><br> Legacy: <br> (Only) event `602: Scheduled Task created`. |


### References

  - [Tony Phipps - SIEM - Notable-Event-IDs](https://github.com/TonyPhipps/SIEM/blob/master/Notable-Event-IDs.md#system-events)

  - [4698(S): A scheduled task was created](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4698)

  - [4699(S): A scheduled task was deleted](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4699)

  - [4700(S): A scheduled task was enabled](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4700)

  - [4701(S): A scheduled task was disabled](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4701)

  - [4702(S): A scheduled task was updated](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4702)
