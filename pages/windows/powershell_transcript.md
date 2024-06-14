---
title: PowerShell Transcript
summary: 'PowerShell Transcript is a mechanism to record a PowerShell console session. The full console input and, depending on the transcript configuration, stdout and stderr streams are logged to a text file.\n\n This logging mechanism, disabled by default, is the only Windows built-in feature to gain extended visibility on PowerShell console interactions, and in particular to the results of PowerShell commands entered in a console. Contrary to Script Block Logging however, PowerShell Transcript does not include content of scripts executed and only the commands as they are entered in the PowerShell console.\n\n PowerShell Transcript can be enabled using the Start-Transcript cmdlet, by GPO, or directly in the registry.'
keywords: PowerShell, Transcripts
tags:
  - windows_program_execution
  - windows_powershell_activity
default_location: 'By default:\n\n<USERPROFILE>\Documents\PowerShell_transcript.*.txt\n\nHowever, alternative path, on the local filesystem or
on a remote server, can be specified.'
last_updated: 2024-06-14
sidebar: sidebar
permalink: windows_powershell_transcript.html
folder: windows
---

### Overview

`PowerShell Transcript` is a mechanism to record a `PowerShell` console
session. The full console input and, depending on the transcript configuration,
`stdout` and `stderr` streams are logged to a text file. This logging
mechanism, disabled by default, is the only Windows built-in feature to gain
extended visibility on `PowerShell` console interactions, and in particular to
the results of `PowerShell` commands entered in a console.

Contrary to `Script Block Logging` however, `PowerShell Transcript` does not
include content of scripts executed and only the commands as they are entered /
typed in the `PowerShell` console (and thus potentially obfuscated).

`PowerShell Transcript` is available from `PowerShell 2.0` and onward, with
a different default file naming convention starting with `PowerShell 5.0`:

  - Before `PowerShell 5.0`: `PowerShell_transcript.YYYYMMDDHHmmss.txt`, with
    the timestamp being in the system local timezone.

  - Starting with `PowerShell 5.0`:
    `PowerShell_transcript.<HOSTNAME>.<8_RANDOM_CHAR>.YYYYMMDDHHmmss.txt`
    (with the timestamp also in the system local timezone).

By default, `PowerShell Transcript` logs will be located directly under
a user's `Documents` folder (`<USERPROFILE>\Documents`), with one file per
"transcripted" console session. An alternative path, on the local filesystem or
on a remote server, can be specified. A number of known paths are referenced in
the [`KAPE` target `PowerShellTranscripts`](https://github.com/EricZimmerman/KapeFiles/blob/master/Targets/Windows/PowerShellTranscripts.tkape).

#### Enabling PowerShell Transcript

`PowerShell Transcript` can be enabled in a number of ways. By default, only
the timestamps of the `PowerShell` session start and end are logged and an
additional configuration is required to log the timestamp of each command
execution.

  - For the current `PowerShell` console session, using the `Start-Transcript`
    cmdlet. The `IncludeInvocationHeader` switch toggles the logging of command
    execution timestamps. This cmdlet can be added to a user
    `PowerShell profile` to be automatically executed upon new `PowerShell`
    session. The `PowerShell profile` can however be trivially bypassed by
    launching a session with out loading the profile
    (`powershell.exe –NoProfile`).

  - Through `Group Policy`: Computer configuration -> Polices -> Administrative
    Templates -> Windows Components -> Windows PowerShell ->
    "Turn on PowerShell Transcription". As stated in the `GPO` setting, this is
    equivalent of calling the `Start-Transcript` cmdlet for every `PowerShell`
    session. The "Include invocation headers" checkbox determine, similarly to
    the `IncludeInvocationHeader` switch, wether execution timestamps should be
    logged for each command.

  - Directly through the registry, by setting the values,
    under `HKLM\SOFTWARE\Policies\Microsoft\Windows\PowerShell\Transcription`,
    `EnableTranscripting` to `0x1` and `EnableInvocationHeader` to `0x1` (to
    enable commands timestamps). While supporting the `Start-Transcript`
    cmdlet, `PowerShell core` does not however appear to take into account
    these values.

### Information of interest

Each `PowerShell Transcript` is associated with a `PowerShell` session and
include:

  - The user associated with the session.

  - The timestamps of the session start and finish (once the session is
    closed).

  - The `Process ID (PID)` of the `PowerShell` process linked to the session.

  - The `PowerShell` edition (`desktop`, `core`) and version used.

  - Each commands entered / typed in the session, their working directory (as
    the `PowerSehll` prompt is logged as well), and their respective `stdout`
    and `stderr` outputs.

  - [If enabled](#enabling-powershell-transcript), the timestamp of each
    command execution, in `YYYYMMDDHHmmss` format and in the system local
    timezone.

Example of a `PowerShell Transcript` log file (with command execution
timestamps) `PowerShell_transcript.HOSTNAME.AZCz+5lm.20240615001946.txt`:

```
**********************
Windows PowerShell transcript start
Start time: 20240615001946
Username: HOSTNAME\USERNAME
RunAs User: HOSTNAME\USERNAME
Configuration Name:
Machine: HOSTNAME (Microsoft Windows NT 10.0.22631.0)
Host Application: C:\WINDOWS\System32\WindowsPowerShell\v1.0\powershell.exe
Process ID: 26208
PSVersion: 5.1.22621.2506
PSEdition: Desktop
PSCompatibleVersions: 1.0, 2.0, 3.0, 4.0, 5.0, 5.1.22621.2506
BuildVersion: 10.0.22621.2506
CLRVersion: 4.0.30319.42000
WSManStackVersion: 3.0
PSRemotingProtocolVersion: 2.3
SerializationVersion: 1.1.0.1
**********************
**********************
Command start time: 20240615001949
**********************
PS C:\Users\USERNAME> whoami
HOSTNAME\USERNAME
**********************
Command start time: 20240615002105
**********************
PS C:\Users\USERNAME> exit
**********************
Windows PowerShell transcript end
End time: 20240615002105
**********************
```

### References

  - [Microsoft - Start-Transcript](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.host/start-transcript)

  - [IT-CONNECT - PowerShell : activer la journalisation des commandes et scripts](https://www.it-connect.fr/powershell-activer-la-journalisation-des-commandes-et-scripts/)

  - [STIG Viewer - PowerShell Transcription must be enabled on Windows 10](https://www.stigviewer.com/stig/windows_10/2021-03-10/finding/V-230220)
