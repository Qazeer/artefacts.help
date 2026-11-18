---
title: ETW - Integrity
summary: 'The Windows event logs can be tampered with in a number of ways, potentially impacting their integrity.\n\nTwo categories of Windows event log tampering can be distinguished:\n - Tampering with the Event Log service, to avoid the generation of new events.\n - Tampering with the existing Windows events, to delete traces of past activities. The log file/channel can be deleted altogether, or events can be deleted or tampered with individually.\n\nMain events:\n\nChannel: Security.\nEvent ID 1102: "The audit log was cleared".\n\nChannel: System.\nEvent ID 104: "The <CHANNEL> log file was cleared".\nEvent ID 7040: "The start type of the Windows Event Log service was changed from auto start to disabled".'
keywords: 'Event Log, EventRecordID, Tampering, wevtutil, EventCleaner, Clear-Eventlog, DeleteRecordofFile, mimikatz, wevtsvc.dll, ActualProcessEvent, danderspritz, EvtxECmd, 1100, 1102, 104, 7001, 7040'
tags:
  - windows_etw
  - windows_defense_evasion
location: 'Channel: Security.\nEvents: 1100, 1102.\n\nChannel: System.\nEvents: 104, 7001, 7040.'
last_updated: 2024-01-26
sidebar: sidebar
permalink: windows_etw_integrity.html
folder: windows
---

### Overview

The Windows event logs can be tampered with in a number of ways, potentially
impacting their integrity.

Two categories of Windows event log tampering can be distinguished:

  - **Tampering with the `Event Log` service**, to avoid the generation of
    new events (including events related to the deletion of events).

  - **Tampering with the existing Windows events**, to delete traces of past
    activities. The log file/channel can be deleted altogether, or events can
    be deleted or tampered with individually.

| Category | Method | Description | Detection(s) |
|----------|--------|-------------|--------------|
| `Event Log` service tampering. | Stopping the `Event Log` service. | The `Event Log` service can be stopped or disabled, to prevent further logs from being generated. The `Event Log` service often cannot be stopped directly, requiring the service to be first disabled and the system then restarted. | Events `Security` `1100` and `System` `7001`, `7040`. |
| `Event Log` service tampering. | Killing or suspending the `Event Log` service process's threads. | The threads of the `Event Log` service process can be killed or suspended to tamper with the service. The `Event Log` service is loaded by an instance of the `Service Host` (`svchost.exe`) process, usually under a few threads. On more recent versions of the Windows operating system, killing the threads is not sufficient, and the threads must be suspended instead. <br><br> Even if the threads are suspended, event(s) associated with the deletion may still be generated if the clearing is done using utilities that leverage the Windows APIs. To delete or tamper with log files directly, without using the Windows API, the handle acquired by the `Event Log` service process on the targeted `EVTX` files must first be closed. While the `Event Log` service threads are suspended and their handle(s) on the target `EVTX` file are closed, the log file can be deleted altogether (through the filesystem), or individual events can be deleted/tampered with, without generating deletion event(s). | Execution of specific tooling (with specific IoCs, such as opening handles on the `Event Log` service process and `EVTX` files, suspending remote threads, etc.). |
| `Event Log` service tampering. | Patching of the `Event Log` service. | The `Event Log` service process can be patched, as implemented in `mimikatz`. The `Channel::ActualProcessEvent` function, from the `Windows Event Service` (`wevtsvc.dll`) library, can be patched to always return before writing the events to their associated `EVTX` log files. <br><br> While the process is patched, new events won't be written to log files (including event(s) related to the deletion of events). | Execution of specific tooling (with specific IoCs, such as in-memory patching of the `Event Log` service process). |
| Event tampering. | Clearing the log file/channel. | The log files/channels can be cleared using, for example, the `Event Viewer` client, the `wevtutil` utility, or the `Clear-EventLog` PowerShell cmdlet. <br><br> Event(s) associated with the clearing will be generated if the `Event Log` service process is running normally. | Events `Security` `1102` or `System` `104`. |
| Event tampering. | Deleting or tampering an event individually. | The events can be manipulated individually, to delete an event, modify its content, or unreference the event (as implemented in the [Equation Group's DanderSpritz framework](https://danderspritz.com/)). <br><br> Manipulating individual events requires the `Event Log` service to be stopped or in a suspended, non-functional state. <br><br> Three header checksum fields ensure the integrity of an event: [the `File Header Checksum`, `Chunk Header Checksum`, and the `Event Record Checksum`](https://svch0st.medium.com/event-log-tampering-part-2-manipulating-individual-event-logs-3de37f7e3a85). Depending on the manipulation conducted, all three checksums may need to be recalculated. <br><br> As stated, events can be unreferenced by increasing the size of the previous `Event Record` (by the size of the target log) to "cover" the target record. `EventRecordID` may also be redefined to be linear, and a number of headers will need to be recalculated. <br><br> The [DeleteRecordofFile.cpp](https://github.com/3gstudent/Eventlogedit-evtx--Evolution/blob/master/DeleteRecordofFile.cpp) proof-of-concept can be used to delete an event and restore the `EventRecordID` and checksums. | Depending on how the events were deleted/tampered with: missing time ranges, non-sequential [`EventRecordID`](#eventrecordid). <br><br> Unreferenced events can be detected, and retrieved, by `EvtxECmd` or [`danderspritz-evtx`](https://github.com/fox-it/danderspritz-evtx). |

The [EventCleaner](https://github.com/QAX-A-Team/EventCleaner) tool can be used
to automate the process of suspending the `Event Log` service threads,
closing the handle on the `Security.evtx` log file, and deleting individual
events.

#### EventRecordID

The `EventRecordID` field is the record number assigned to the event when it was
logged. The `EventRecordID` grows sequentially for a given log channel/file.

Non-sequential `EventRecordID` values can thus be an indicator of individual
event deletion. However, non-sequential `EventRecordID` values may also occur
under normal circumstances.

### Windows events details

| Channel | Conditions | Events |
|---------|------------|--------|
| `Security` | Default configuration. | Event `1102: The audit log was cleared`. <br><br> Generated upon the deletion of events in the `Security` logs. <br><br> However, the absence of event `1102` should not be taken as a sign of the integrity of the `Security` events, as the generation of this event can be bypassed. For instance, the threads of the `Event Log` service (hosted by `svchost.exe`) can be suspended to prevent event generation while the threads are suspended (even though all events will be written upon resumption of the threads). <br><br> Information of interest: <br> - The domain, username, and `SID` of the user that cleared the `Security` log. <br> - The `Logon ID` of the session associated with the log clearing. |
| `System` | Default configuration. | Event `104: The <CHANNEL> log file was cleared`. <br><br> Generated upon the deletion of events from event logs (other than the `Security` channel). <br><br> Information of interest: <br> - Domain and username of the user that cleared the log. <br> - The name of the log channel cleared. |
| `Security` | Default configuration. | Event `1100: The event logging service has shut down`. <br><br> Generated upon the shutdown of the `Event Log` service. <br><br> The event does not seem to indicate the user that stopped the service. |
| `System` | Default configuration. | Event `7040: The start type of the Windows Event Log service was changed from auto start to disabled`. <br><br> Indicates that the `Event Log` service was disabled. <br><br> Information of interest: <br> - Domain and username of the user that disabled the Windows service. |
| `System` | Default configuration. | Event `7001: The <SERVICE_NAME> service depends on the Windows Event Log service which failed to start because of the following error: The service cannot be started, either because it is disabled or because it has no enabled devices associated with it`. <br><br> Indicates that a service that depends on the `Event Log` service could not start because the `Event Log` service is not running. Can be an indicator of the `Event Log` service being disabled after a system reboot. |

### References

  - [svch0st - Event Log Tampering Part 1: Disrupting the EventLog Service](https://svch0st.medium.com/event-log-tampering-part-1-disrupting-the-eventlog-service-8d4b7d67335c)

  - [svch0st - Event Log Tampering Part 2: Manipulating Individual Event Logs](https://svch0st.medium.com/event-log-tampering-part-2-manipulating-individual-event-logs-3de37f7e3a85)

  - [svch0st - Event Log Tampering Part 3: Combining Techniques](https://svch0st.medium.com/event-log-tampering-part-3-combining-techniques-ce6ead21ca49)
