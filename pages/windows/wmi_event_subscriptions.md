---
title: WMI - Event Subscription
summary: 'WMI Event Subscriptions can be used to maintain persistence on a Windows system, with permanent event subscriptions persisting across system reboots.\n\nPermanent event subscriptions are composed of: an "event filter" (event that will trigger the consumer), an "event consumer" (that will perform an action, such as executing a command), and a "filter to consumer binding".\n\nEvent subscriptions are written to disk in the "OBJECTS.DATA" file that notably contains the event filters and event consumers.'
keywords: WMI, Event Subscription, event filter, event consumer, ActiveScriptEventConsumer, CommandLineEventConsumer, LogFileEventConsumer, NtEventLogEventConsumer, SMTPEventConsumer, FilterToConsumerBinding, OBJECTS.DATA, CIM, INDEX.BTR, Autoruns, Get-WMIObject, subscription, PyWMIPersistenceFinder, python-cim
tags:
  - windows_wmi
  - windows_local_persistence
location: 'WMI repository files under <SYSTEMROOT>\System32\wbem\Repository\:\n- OBJECTS.DATA\n- INDEX.BTR\n- MAPPING<1-3>.MAP'
last_updated: 2024-01-24
sidebar: sidebar
permalink: windows_wmi_event_subscription.html
folder: windows
---

### Overview

`Windows Management Instrumentation (WMI)` allows, through
`Event Subscription`, to maintain persistence on a Windows system. Permanent
`WMI` event subscriptions can be configured to persist across system reboots.

Permanent event subscriptions are composed of:

  - An `event filter`, which is the event of interest that will trigger the
    consumer. Such event can be, for example, a logon success or system
    startup.

  - An `event consumer`, which is the action to perform upon trigger of the
    event filter.

    Five Consumer classes are available:

      - The `ActiveScriptEventConsumer` class that run arbitrary `VBScript`
        or `JScript` code.

      - The `CommandLineEventConsumer` class that run an arbitrary system
        command.

      - The `LogFileEventConsumer` class that write an arbitrary string to a
        text-based log file.

      - The `NtEventLogEventConsumer` class that write an arbitrary Windows `ETW`
        event.

      - The `SMTPEventConsumer` class that send an email.

  - A `filter to consumer binding` (`FilterToConsumerBinding`) which is the
    registration mechanism binding an event filter to an event consumer.

### WMI repository files

The persistent `WMI Event Subscription` are written to disk in the
(undocumented) `WMI` repository files under
`%SystemRoot%\System32\wbem\Repository\` or
`%SystemRoot%\System32\wbem\Repository\FS\`:

  - `OBJECTS.DATA`: contains the `CIM objects` with, among other things, the
    event subscriptions data (event consumer, filter, and filter to consumer
    binding).

  - `INDEX.BTR`: paged file in B-tree structure, "used to efficiently lookup
    CIM entities in the objects.data file".

  - `MAPPING<1-3>.MAP`: correlate / map pages from `OBJECTS.DATA` and
    `INDEX.BTR`.

All three files are required to properly conduct forensics analysis on WMI
persistence.

### Tool(s)

#### Live forensics

The [`SysInternals` `Autoruns`](https://learn.microsoft.com/fr-fr/sysinternals/)
(GUI) and `AutorunsC` (CLI) utilities can be used to detect (and delete)
`WMI`-related persistence.

The `WMI` event subscriptions can also be enumerated with the PowerShell cmdlet
`Get-WMIObject`:

```powershell
ForEach ($NameSpace in "root\subscription","root\default") { Get-WMIObject -Namespace $Namespace -Query "SELECT * FROM __EventFilter" }

ForEach ($NameSpace in "root\subscription","root\default") { Get-WMIObject -Namespace $Namespace -Query "SELECT * FROM __EventConsumer" }

ForEach ($NameSpace in "root\subscription","root\default") { Get-WMIObject -Namespace $Namespace -Query "SELECT * FROM __FilterToConsumerBinding" }
```

#### WMI repository files parsing

`WMI Event Subscription` data can be extracted from `OBJECTS.DATA` files using
the [`PyWMIPersistenceFinder`](https://github.com/davidpany/WMI_Forensics)
Python script (that rely on regexes to extract the data):

```bash
PyWMIPersistenceFinder.py "<OBJECTS.DATA_FILE>"
```

If a deeper analysis is required, for example if a consumer reference other
`WMI` objects, [`python-cim`](https://github.com/mandiant/flare-wmi) can be
leveraged to extract data from the `WMI` repository:

```bash
python3 samples/dump_class_layout.py win7 "<WMI_REPOSITORY_FOLDER>" "<ROOT\cimv2 | WMI_NAMESPACE>" "<WMI_CLASS_NAME>"
```

### References

  - [FireEye - William Ballenthin, Matt Graeber, Claudiu Teodorescu - WINDOWS MANAGEMENT INSTRUMENTATION (WMI) OFFENSE, DEFENSE, AND FORENSICS](https://www.mandiant.com/sites/default/files/2021-09/wp-windows-management-instrumentation.pdf)

  - [Network Security Ninja - WMI Forensics](https://netsecninja.github.io/dfir-notes/wmi-forensics/)

  - [SANS - Chad Tilbury - Investigating WMI Attacks](https://www.youtube.com/watch?v=aBQ1vEjK6v4)
