---
title: ETW - AppLocker
summary: 'AppLocker is a native Windows feature introduced in Windows 7 Enterprise, designed to replace "Software Restriction Policies (SRP)". AppLocker enables the restriction and control of executable files that users can run.\n\n AppLocker can control the process creation for executables, Dynamic Link Library, scripts (PowerShell, DOS, VBS, etc.), MSI, and packaged apps.\n\n AppLocker operates on the principle of whitelisting, where files are only allowed to execute or run if they are explicitly allowed by defined rules. Rule sets can be enforced (blocking the execution of files associated with specific types) or configured in audit mode (only generating ETW events).'
keywords: 'AppLocker, PowerShell, EXE, DLL, EXE and DLL, Microsoft-Windows-AppLocker/EXE and DLL, MSI, Script, PowerShell, MSI and Script, Microsoft-Windows-AppLocker/MSI and Script, 8002, 8003, 8004, 8005, 8006, 8007'
tags:
  - windows_etw
  - windows_program_execution
  - windows_powershell_activity
location: 'Channel: Microsoft-Windows-AppLocker/EXE and DLL.\n Events: 8002, 8003, 8004.\n\n Channel: Microsoft-Windows-AppLocker/MSI and Script.\n Events: 8005, 8006, 8007.\n\n Channel: Microsoft-Windows-AppLocker/Packaged app-Execution.\n Events: 8020, 8021, 8022.\n\n Channel: Microsoft-Windows-AppLocker/Packaged app-Deployment.\n Events: 8023, 8024, 8025.\n\n'
last_updated: 2025-08-02
sidebar: sidebar
permalink: windows_etw_applocker.html
folder: windows
---

### Overview

AppLocker is a native Windows feature introduced in Windows 7 Enterprise, 
designed to replace `Software Restriction Policies (SRP)`. AppLocker enables
the restriction and control of executable files that users can run.

AppLocker can control the process creation for the following file types,
controlled through 5 sets of rules:

  - `Executables Rules` for executables (`.exe` and `.dom` files).

  - `Scripts Rules` for scripts (`.ps1`, `.bat`, `.cmd`, `.vbs` and `.js`
    files).

  - `Windows Installer Rules` for Windows Installers (`.msi`, `.msp` and
    `.mst` files).

  - `Packaged Apps Rules` for Windows packaged applications (`.appx` files).

  - `Shared Libraries and Controls Rules` for `Dynamic Link Library` (`.dll`
    and `.ocx` files).

AppLocker operates on the principle of whitelisting, where files are only
allowed to execute or run if they are explicitly allowed by defined rules.
Rule sets can be enforced (blocking the execution of files associated with
specific types) or configured in audit mode (only generating ETW events).

A computer can implement one or more AppLocker rules that are defined
locally (in `Local Security Policy`) or centrally via one or more `Group Policy
Object (GPO)`. The effective rules that are actually implemented on the
computer is the sum of all rules defined in Local and Group policies.

Note: for AppLocker to be activated, the `AppIDSvc` service must be enabled and
running.

#### Default AppLocker rules

AppLocker provides different default rules for each files type category:

  - **Executables**
  
    The members of the local administrators group (SID `S-1-5-32-544`) can
    execute any binaries, while the other users can only execute binaries from
    the `%PROGRAMFILES%` and `%WINDIR%` folders.

  - **Scripts**

  Similarly, the default scripts rules allow the members of the local
  Administrators group (SID `S-1-5-32-544`) to execute any scripts, while the
  other users can only execute scripts from the `%PROGRAMFILES%` and `%WINDIR%`
  folders.

  - **Windows Installer**

  The default Windows Installers rules allow the members of the local
  Administrators group (SID `S-1-5-32-544`) to execute any Windows Installer
  files, while other users may only execute Windows Installer files that are
  digitally signed, by any authority, or from the `%WINDIR%\Installer\` folder.

  - **Packaged Apps**

  By default, any user (`Everyone`) can execute digitally signed, by any
  authority, packaged apps.

  - **Shared Libraries and Controls**

  The `Dynamic Link Libraries (DLL)` rules must be enforced through advanced
  configuration, as they can affect system performance. In a basic AppLocker
  configuration, DLL rules may not be enforced.
  
  If enforced, the default DLL rules work in the same fashion as the
  executables and scripts rules. The members of the local administrators group
  (SID `S-1-5-32-544`) can load any DLL, while the other users can only load
  DLLs from the `%PROGRAMFILES%` and `%WINDIR%` folders.

#### Extract AppLocker configuration

The effectively applied AppLocker rules can be retrieved using the
`Get-AppLockerPolicy` PowerShell cmdlet.

An AppLocker rule is defined for an user or group, identified by the
`UserOrGroupSid` attribute, and one or more conditions, which can be a
filesystem paths, publishers for digitally signed files, or `Authenticode`
hashes.

```bash
Get-WinEvent -LogName "Microsoft-Windows-AppLocker/EXE and DLL"

Get-AppLockerPolicy -Effective | Select-Object -ExpandProperty RuleCollections
```

### AppLocker ETW events

The AppLocker channels contain information about executions affected by
AppLocker rules. By default no events are generated, as AppLocker events are
linked to the state of the AppLocker security mechanism on the system.

The events in the AppLocker channels share a common set of logged properties:

  - Policy name and information about the rule that matched: rule identifier,
    rule name, and rule content in `SDDL` notation.

  - Execution context of the process that triggered the event: `SID` of the
    user, session `LogonId`, and process `PID`.

  - For Executables (executables and `DLLs`), scripts, and Windows installers
    rules, information on the file:
     - Filepath.
     - [`Authenticode`](https://web.archive.org/web/20250503053059/https://reversea.me/index.php/authenticode-i-understanding-windows-authenticode/)
       file hash.
     - `Fully Qualified Binary Name` (`FQBN`) (a string constructed as
       `Company\Product Suite\Product, Version` for signed Windows binary
       files).

  - For Packaged Apps: the package app name.


| Channel | Conditions | Events |
|---------|------------|--------|
| Channel: <br> `Microsoft-Windows-AppLocker/EXE and DLL` <br><br> EVTX file: <br> `Microsoft-Windows-AppLocker%4EXE and DLL.evtx` | Requires a matching AppLocker rule (from `Executables Rules` or `DLL Rules`) that allowed the execution. | Event `8002: <PE_FILE> was allowed to run`. <br><br> If the [default AppLocker rules](#default-applocker-rules) are enabled, this event is logged for every executions by members of the local administrators group and for executions from the `%PROGRAMFILES%` and `%WINDIR%` folders for every other users. Other execution attempts would raise events `8004` (if AppLocker `Executables Rules` are enforced) or `8003`. |
| Channel: <br> `Microsoft-Windows-AppLocker/EXE and DLL` <br><br> EVTX file: <br> `Microsoft-Windows-AppLocker%4EXE and DLL.evtx` | Requires a matching AppLocker rule (from `Executables Rules` or `DLL Rules`) that would have blocked the execution if the rule set was in `enforce` mode. | Event `8003: <PE_FILE> was allowed to run but would have been prevented from running if the AppLocker policy were enforced`. <br><br> Logged when an executable or `DLL` file matching a rule was allowed to execute but would have been blocked by an AppLocker rule if the rule was enforced. |
| Channel: <br> `Microsoft-Windows-AppLocker/EXE and DLL` <br><br> EVTX file: <br> `Microsoft-Windows-AppLocker%4EXE and DLL.evtx` | Requires a matching AppLocker rule (from `Executables Rules` or `DLL Rules`) that blocked the execution. | Event `8004: <PE_FILE> was prevented from running`. <br><br> Logged when an executable or `DLL` was prevented from executing by a matching enforced AppLocker rule. |
| Channel: <br> `Microsoft-Windows-AppLocker/MSI and Script` <br><br> EVTX file: <br> `Microsoft-Windows-AppLocker%4MSI and Script.evtx` | Requires a matching AppLocker rule (from `Scripts Rules` or `Windows Installer Rules`) that allowed the execution. | Event `8006: <SCRIPT_FILE | MSI FILE> was allowed to run`. <br><br> Similarly to `EXE and DLL` events, logged for every scripts / MSI executions by members of the local administrators group and for scripts / MSI executions from the `%PROGRAMFILES%` and `%WINDIR%` folders for every other users. Other execution attempts would raise events `8007` (if AppLocker `Scripts Rules` or `Windows Installer Rules` rules are enforced) or `8006`. |
| Channel: <br> `Microsoft-Windows-AppLocker/MSI and Script` <br><br> EVTX file: <br> `Microsoft-Windows-AppLocker%4MSI and Script.evtx` | Requires a matching AppLocker rule (from `Scripts Rules` or `Windows Installer Rules`) that would have blocked the execution if the rule set was in `enforce` mode. | Event `8007: <SCRIPT_FILE | MSI FILE> was allowed to run but would have been prevented from running if the AppLocker policy were enforced`. |
| Channel: <br> `Microsoft-Windows-AppLocker/MSI and Script` <br><br> EVTX file: <br> `Microsoft-Windows-AppLocker%4MSI and Script.evtx` | Requires a matching AppLocker rule (from `Scripts Rules` or `Windows Installer Rules`) that blocked the execution. | Event `8008: <SCRIPT_FILE | MSI FILE> was prevented from running`. |
| Channel: <br> `Microsoft-Windows-AppLocker/Packaged app-Execution` <br><br> EVTX file: <br> `Microsoft-Windows-AppLocker%4Packaged app-Execution.evtx` | Introduced in `Windows Server 2012` and `Windows 8`. <br><br> Requires a matching AppLocker rule (from `Packaged app Rules`) that allowed the app execution. | Event `8020: <PACKAGED_APP> was allowed to run`. |
| Channel: <br> `Microsoft-Windows-AppLocker/Packaged app-Execution` <br><br> EVTX file: <br> `Microsoft-Windows-AppLocker%4Packaged app-Execution.evtx` | Introduced in `Windows Server 2012` and `Windows 8`. <br><br> Requires a matching AppLocker rule (from `Packaged app Rules`) that would have blocked the app execution if the rule set was in `enforce` mode. | Event `8021: <PACKAGED_APP> was allowed to run but would have been prevented from running if the AppLocker policy were enforced`. |
| Channel: <br> `Microsoft-Windows-AppLocker/Packaged app-Execution` <br><br> EVTX file: <br> `Microsoft-Windows-AppLocker%4Packaged app-Execution.evtx` | Introduced in `Windows Server 2012` and `Windows 8`. <br><br> Requires a matching AppLocker rule (from `Packaged app Rules`) that blocked the app execution. | Event `8022: <PACKAGED_APP> was prevented from running`. |
| Channel: <br> `Microsoft-Windows-AppLocker/Packaged app-Deployment` <br><br> EVTX file: <br> `Microsoft-Windows-AppLocker%4Packaged app-Deployment.evtx` | Introduced in `Windows Server 2012` and `Windows 8`. <br><br> Requires a matching AppLocker rule (from `Packaged app Rules`) that allowed the app installation. | Event `8023: <PACKAGED_APP> was allowed to be installed`. |
| Channel: <br> `Microsoft-Windows-AppLocker/Packaged app-Deployment` <br><br> EVTX file: <br> `Microsoft-Windows-AppLocker%Packaged app-Deployment.evtx` | Introduced in `Windows Server 2012` and `Windows 8`. <br><br> Requires a matching AppLocker rule (from `Packaged app Rules`) that that would have blocked the app installation if the rule set was in `enforce` mode. | Event `8024: <PACKAGED_APP> was allowed to run but would have been prevented from running if the AppLocker policy were enforced`. |
| Channel: <br> `Microsoft-Windows-AppLocker/Packaged app-Deployment` <br><br> EVTX file: <br> `Microsoft-Windows-AppLocker%4Packaged app-Deployment.evtx` | Introduced in `Windows Server 2012` and `Windows 8`. <br><br> Requires a matching AppLocker rule (from `Packaged app Rules`) that blocked the app installation. | Event `8025: <PACKAGED_APP> was prevented from running`. |


### References

  - [Microsoft - Monitor app usage with AppLocker](https://web.archive.org/web/20250709125429/https://learn.microsoft.com/en-us/windows/security/application-security/application-control/app-control-for-business/applocker/monitor-application-usage-with-applocker)

  - [Microsoft - Using Event Viewer with AppLocker](https://web.archive.org/web/20250709125418/https://learn.microsoft.com/en-us/windows/security/application-security/application-control/app-control-for-business/applocker/using-event-viewer-with-applocker)

  - [Splunk -  Michael Haag - Deploy, Test, Monitor: Mastering Microsoft AppLocker (Part 2)](https://web.archive.org/web/20250710111801/https://www.splunk.com/en_us/blog/security/deploy-test-monitor-mastering-microsoft-applocker-part-2.html)