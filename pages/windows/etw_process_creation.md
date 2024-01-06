---
title: ETW - Process creation
summary: 'Process creation event.\n\nRequires "Audit Process Creation" to be enabled and ProcessCreationIncludeCmdLine_Enabled to be enabled for the command line to be logged.\n\nEvents:\n\n Event ID 4688: "A new process has been created".\n\nEvent ID 4689: "Process Termination: Success and Failure".'
keywords: 'Audit Process Creation, ProcessCreationIncludeCmdLine_Enabled, 4688, A new process has been created, 4689, TokenElevationType, NewProcessName, ParentProcessName, SubjectLogonId'
tags:
  - windows_etw
  - windows_program_execution
location: 'Channel: Security.\nEvents: 4688, 4689.'
last_updated: 2024-01-06
sidebar: sidebar
permalink: windows_etw_process_creation.html
folder: windows
---

### Overview

| Channel | Conditions | Events |
|---------|------------|--------|
| `Security` | Requires `Audit Process Creation` to be enabled. <br><br> Requires `ProcessCreationIncludeCmdLine_Enabled` to be enabled for the command line to be logged. | Event `4688: A new process has been created`. <br><br> Logged upon the creation of every new process on the system. |
| `Security` | Requires `Audit Process Creation` to be enabled. | Event `4689: Process Termination: Success and Failure` <br><br> Logged upon the termination of the process. |

### Fields of interest

#### Process and parent process names

The `NewProcessName` field stores the full path of the process's executable and
the `ProcessId` field the `Process ID (PID)` of the process. The
`ParentProcessName` field logs the parent process's executable full path and
can be used to identity suspicious processes activity, such as `outlook.exe` or
`iexplorer.exe` starting `cmd.exe` or `powershell.exe` processes.

#### Associated user and logon session

This event includes the SID `SubjectUserSid`, account name `SubjectUserName`,
and domain `SubjectDomainName` of the user creating the process. Additionally,
the `SubjectLogonId` field can be used to correlate the process creation with
the logon session ([event ID `4624`](./etw_authentication_dst_host.md)).

#### TokenElevationType

The `TokenElevationType` field represent the privileges of the process and can
take the following values:

| Flag | Correspondence | Description |
|------|----------------|-------------|
| `%%1936` | `TokenElevationTypeDefault` | The process is started with a full token with no privileges removed or groups disabled. A full token is only used if `User Account Control (UAC)` is disabled or if the user starting the process is the built-in `Administrator` (`RID: 500`), `NT AUTHORITY\SYSTEM` or service account. |
| `%%1937` | `TokenElevationTypeFull` | The process is started with an elevated token with no privileges removed or groups disabled. An elevated token is used when `User Account Control (UAC)` is enabled and the user chooses to start the program in a elevated security context (`Run as administrator` for example). |
| `%%1938` | `TokenElevationTypeLimited` | The process is started with limited privileges, and privileged tokens such as `SeImpersonatePrivilege`, `SeDebugPrivilege`, etc. are removed from the process security context. |

#### ProcessCommandLine

If the `ProcessCreationIncludeCmdLine_Enabled` audit policy is enabled, the
command line specified at the process creation will be logged in the
`ProcessCommandLine` field.
