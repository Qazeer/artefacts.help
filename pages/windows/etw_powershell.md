---
title: ETW - PowerShell activity
summary: 'Windows PowerShell version 2.0, and prior versions, provide few useful audit settings, thereby limiting the availability of evidence (such as a command history).\n\nStarting with PowerShell v5, PowerShell logging was enhanced, with the notable addition of Script Block Logging, that record full contents of PowerShell code executed (both original and deobfuscated code). While Script Block Logging is not fully enabled by default, it will record events for code containing suspicious keywords (from a Microsoft pre-defined list).'
keywords: 'PowerShell, Script Block Logging, Windows PowerShell.evtx, Microsoft-Windows-PowerShell\Operational.evtx, Microsoft-Windows-PowerShell\Analytic.etl, Microsoft-Windows-WinRM\Operational.evtx, Microsoft-Windows-WinRM\Analytic.etl, AppLocker, PSDecode, 400, 403, 500, 501, 600, 800, 4100, 4103 4104, SuspiciousContentChecker'
tags:
  - windows_etw
  - windows_program_execution
location: 'Channels:\n\nWindows PowerShell.\n Events: 400, 403, 500, 501, 600, 800.\n\nMicrosoft-Windows-PowerShell\Operational.\nEvents: 4100, 4103, 4104, 40961, 40962, 53504.\n\nMicrosoft-Windows-AppLocker\MSI and Script.\nEvents: 8005, 8006.'
last_updated: 2024-01-06
sidebar: sidebar
permalink: windows_etw_powershell.html
folder: windows
---

### Overview

Windows PowerShell version 2.0, and prior versions, provide few useful audit
settings, thereby limiting the availability of evidence (such as a command
history). Starting with `PowerShell v5`, [PowerShell logging was
enhanced](https://devblogs.microsoft.com/powershell/powershell-the-blue-team/)
with the notable addition of `Script Block Logging`, that record full contents
of PowerShell code executed. While `Script Block Logging` is not fully enabled
by default, it will record events for code containing suspicious keywords (from
a Microsoft pre-defined list).

Upon executing any PowerShell command or script, either locally or using PS
remoting, Windows may write events to the following files:

  - `Windows PowerShell.evtx`

  - `Microsoft-Windows-PowerShell\Operational.evtx`

  - `Microsoft-Windows-PowerShell\Analytic.etl` (non default).

As PowerShell implements its remoting functionality through the `Windows Remote
Management (WinRM)` service, remote PowerShell activity may induce events in
the following files:

  - `Microsoft-Windows-WinRM\Operational.evtx`

  - `Microsoft-Windows-WinRM\Analytic.etl` (non default).

If enabled, `AppLocker` will record PowerShell activity in the
`Microsoft-Windows-AppLocker\MSI and Script` hive.

Additionally, starting with `PowerShell v5` on `Windows 10`, the commands
entered in a PowerShell console will be logged by the `PSReadline` module to an
user-scoped [`ConsoleHost_history.txt`](./consolehost_history.md) file.

### PowerShell suspicious keywords and deobfuscation

The following keywords can be a sign of suspicious activity:
  - `-Enc` / `-e`
  - `-nop` / `bypass`
  - `IEX` / `Invoke-Expression`
  - `ICM` / `Invoke-command`
  - `Net.WebClient` / `io.`
  - `DownloadString` / `DownloadFile`
  - `&` / `|`
  - `//` / `http` / `ftp` / `cifs` / `smb` / ...
  - `join` / `nioj` / `replace` / `ecalper` / `-f` / `CHAR` / `RAHC` / `STRING`
    / `GNIRTS` / `marshal` / `convert` / `env` / `{` / `}` (obfuscation
    detection)

While the occurrence of these keywords may entail malicious activities, their
absence is not a formal proof of lack of malicious PowerShell activity as
PowerShell code can be deeply obfuscated.

The `PSDecode` PowerShell script can be used to deobfuscate malicious
PowerShell scripts that have several layers of encodings.

```bash
# Source: https://github.com/R3MRUM/PSDecode.

Import-Module PSDecode.psm1

PSDecode <ENCODED_POWERSHELL_FILE>
```

### PowerShell Windows events

| Channel  | Event ID | Conditions | Description |
|----------|----------|------------|-------------|
| `Windows PowerShell` | 400 | PowerShell 2.0. | `Engine state is changed from None to Available`. <br><br> Logged on the start of any local or remote PowerShell activity (execution of powershell.exe). <br><br> The `HostApplication` field record the binary path at the origin of the powershell activity and contain the commandline arguments provided to powershell.exe. <br> If the `Hostname` field is equal to : <br> - `ConsoleHost`, the event concern a local activity <br> - `ServerRemoteHost`,  the event occured du to PowerShell remoting activity. <br> The `RunaspaceId` identify the PowerShell activity and can be linked to the session termination (`EID 403`). Note that however this event cannot be strictly correlated to a logon session. |
| `Windows PowerShell` | 403 | PowerShell 2.0. | `Engine state is changed from Available to Stopped`. <br><br> Logged at the end of any local or remote PowerShell activity (execution of powershell.exe) and contains the same level of information as the `EID 400` events. <br><br> The `RunaspaceId` identify the PowerShell activity and can be linked to the session start (`EID 400`). Note that however this event cannot be strictly correlated to a logon session. |
| `Windows PowerShell` | 500 | PowerShell 2.0. <br><br> Requires `$LogCommandLifeCycleEvent` to be set to true (non default). | `Command "<COMMAND>" is Started`. <br><br> Logged whenever a PowerShell command is executed, but can be bypassed by starting PowerShell using the `-NoProfile` / `-nop` flag. |
| `Windows PowerShell` | 501 | PowerShell 2.0. <br><br> Requires `$LogCommandLifeCycleEvent` to be set to true (non default). | `Command "<COMMAND>" is Stopped`. <br><br> Logged whenever a PowerShell command finish its execution, but can be bypassed by starting PowerShell using the `-NoProfile` / `-nop` flag. |
| `Windows PowerShell` | 600 | PowerShell 2.0. | `Provider "<PROVIDER_NAME>" is Started`. <br><br> Logs the start and stop of PowerShell providers. <br><br> Similarly to the events `EID 400` and `EID 403`, this event include the `HostApplication` field. <br> If the provider is `WSMan` ("Provider WSMan Is Started"), the event, logged on both the client and remote systems, indicate the use of PS remoting. <br><br> If the PowerShell activity relies on built-in alias, such as `IEX`, an event will be generated for the `Alias` provider. |
| `Windows PowerShell` | 800 | PowerShell 3.0. | `Pipeline execution details for command line`. <br><br> Inconsistently logged. <br><br> Similarly to the events EID 400 and EID 403, this event include the `HostApplication` field and present the advantage of logging, in the `UserId` field, the user account executing PowerShell.  |
| `Microsoft-Windows-PowerShell\Operational` | 4100 | PowerShell 5.0. | `Error message [...]`. <br><br> Logged whenever an error occurs in a PowerShell activity. <br><br> Includes an `HostApplication` field, the `<DOMAIN>\<USER>` executing PowerShell in the `User` field, and may include the script path of the executed script in the `ScriptName` field. |
| `Microsoft-Windows-PowerShell\Operational` | 4103 | PowerShell 3.0. <br><br> Requires PowerShell `Module Logging` to be enabled (`EnableModuleLogging` registry key set to 1). | `Module Logging`. <br><br> Logged upon the execution of functions in the module(s) set to be logged. <br><br> If the module names, configured in the `Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging\ModuleNames` registry key, is set to `*`, the activity of the members of all modules are logged. <br> Provides verbose information on the PowerShell activity and, in addition to user information, events may yields the parameters and output of the executed PowerShell cmdlets. |
| `Microsoft-Windows-PowerShell\Operational` | 4104 | PowerShell 5.0. <br><br> Requires PowerShell `Script Block Logging` to be enabled to generate all events. <br><br> By default (without a specific policy), events will however be logged for potentially-malicious commands execution. The [following ressource](https://gist.github.com/nasbench/50cd0b64bedacabccecc9149c15228da) lists the suspicious strings used by PowerShell `SuspiciousContentChecker` function to trigger an event creation. | `Script block logging`: `Creating Scriptblock text [...]`. <br><br> Logged upon the execution of PowerShell scripts and cmdlets. <br><br> If the `Path` field is empty, the command was executed interactively through the PowerShell console. <br> Includes, in the `ScriptBlockText` field, the script block (content of the PowerShell script or cmdlet and the commandline) being executed. The original code will be logged as well as deobfuscated code (such as the encoded or encrypted code, decoded / decrypted at runtime). <br><br> If the script block code exceed the maximum length, the code will be fragmented into multiple events. <br><br> This event provides valuable information but may be bypassed by malicious actors by starting PowerShell 2.0 (`powershell.exe -version 2.0`). |
| `Microsoft-Windows-PowerShell\Operational` | 40961 <br> 40962 | PowerShell 3.0. | `PowerShell console is starting up` (`EID 40961`) followed by `PowerShell console is ready for user input` (`EID 40962`). <br><br> Logged upon the start of a PowerShell activity (execution of powershell.exe). <br> Includes the `<DOMAIN>\<USER>` executing PowerShell in the `User` field. |
| `Microsoft-Windows-PowerShell\Operational` | 53504 | PowerShell 3.0. | `Windows PowerShell has started an IPC listening thread on process: <PID> in `AppDomain`: <DOMAIN>`. <br><br> Indicates that a PowerShell `AppDomain` was started. <br><br> Usually logged upon the start of the PowerShell console, in between events `EID: 40961` and `EID: 40962`. |
| `Microsoft-Windows-AppLocker\MSI and Script` | 8005 | Require `AppLocker` to be enabled and running in `Audit only` mode. | `<SCRIPT_PATH> was allowed to run`. <br><br> Logged upon the execution of a local PowerShell script. |
| `Microsoft-Windows-AppLocker\MSI and Script` | 8006 | Require `AppLocker` to be enabled and running in `Audit only` mode. | `<SCRIPT_PATH> was allowed to run but would have been prevented from running if the AppLocker policy were enforced`. <br><br> Logged upon the execution of a local PowerShell script. |

Additionally, `Security` event ID
[`4688: A new process has been created`](./etw_process_creation.md), that
require `Audit Process Creation` to be enabled, can `powershell.exe` execution.

### References

  - [Microsoft - PowerShell ♥ the Blue Team](https://devblogs.microsoft.com/powershell/powershell-the-blue-team/)

  - [Black Hat USA 2014 - Mandiant - Ryan Kazanciyan, Matt Hastings - Investigating PowerShell Attacks](https://www.blackhat.com/docs/us-14/materials/us-14-Kazanciyan-Investigating-Powershell-Attacks.pdf)

  - [Mandiant - Matthew Dunwoody - Greater Visibility Through PowerShell Logging](https://www.mandiant.com/resources/blog/greater-visibility)

  - [JPCERT - Detecting Lateral Movement through Tracking Event Logs](https://www.jpcert.or.jp/english/pub/sr/20170612ac-ir_research_en.pdf)

  - [JPCERT - Invoke-Mimikatz (PowerSploit)](http://jpcertcc.github.io/ToolAnalysisResultSheet/details/PowerSploit_Invoke-Mimikatz.htm)

  - [EVENTSENTRY BLOG - ingmar.koecher - From PowerShell to P0W3rH3LL – Auditing PowerShell](https://www.eventsentry.com/blog/2018/01/powershell-p0wrh11-securing-powershell.html)

  - [](https://www.powershellmagazine.com/2014/07/16/investigating-powershell-attacks/)

  - [NSFocus - Adeline Zhang - Attack and Defense Around PowerShell Event Logging](https://nsfocusglobal.com/Attack-and-Defense-Around-PowerShell-Event-Logging)

  - [MalwareArchaeology.com - WINDOWS POWERSHELL LOGGING CHEAT SHEET - Win 7/Win 2008 or later](https://static1.squarespace.com/static/552092d5e4b0661088167e5c/t/59c1814829f18782e24f1fe2/1505853768977/Windows+PowerShell+Logging+Cheat+Sheet+ver+Sept+2017+v2.1.pdf)

  - [JPCERT - WinRM](https://jpcertcc.github.io/ToolAnalysisResultSheet/details/WinRM.htm)
