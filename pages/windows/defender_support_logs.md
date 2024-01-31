---
title: Windows Defender - Support logs
summary: 'Windows Defender stores on disk a number of plain-text log files.\n\nAmong these log files, the Microsoft Protection Log (MPLog) log includes a number of event types related to past Windows Defender scanning activity and detections.\n\nThe MPLog can notably be a source of historical information on:\n - Program  and suspicious command line executions.\n - Files existence and access.\n - Windows Defender configuration state, detections, and other telemetry.'
keywords:
tags:
  - windows_program_execution
  - windows_file_knowledge
  - windows_files_and_folders_access
  - windows_defense_evasion
location: 'Log files, and notably "MPLog-YYMMDD-hhmmss.log", under:\n\n<SYSTEMDRIVE>\ProgramData\Microsoft\Windows Defender\Support'
last_updated: 2024-01-31
sidebar: sidebar
permalink: windows_defender_support_logs.html
folder: windows
---

### Overview

`Windows Defender` stores on disk a number of plain-text log files under
`%SystemDrive%\ProgramData\Microsoft\Windows Defender\Support`, including:

| File | Description |
|------|-------------|
| `MPLog-YYMMDD-hhmmss.log` | The `Microsoft Protection Log` (`MPLog`) log includes a number of event types related to past `Windows Defender` scanning activity and detections, notably to permit the troubleshooting of performance issues related to real-time protection. <br><br> The `MPLog` can be a source of historical information on: <br> - Program  and suspicious command line executions. <br> - Files existence and access. <br> - `Windows Defender` configuration state and detections. |
| `MPScanSkip-YYMMDD-hhmmss.log` | The `Microsoft Scan Skip` (`MPScanSkip`) log stores information on `Windows Defender` scans that were skipped or aborted. A scan can be aborted for a number of reason, including reaching the timeout limit. <br><br> The `MPScanSkip` log can be a source of historical information on files that were not, or partially, scanned by `Windows Defender`. |

### Microsoft Protection Log (MPLog)

The `MPLog` log include a number of different event types, that can be of
interest for incident response investigations.

#### Estimated performance impact events

{% include note.html content="Estimated performance impact events can be used: <br> - As an evidence of process execution. <br> - To determine the (minimun) number of files accessed by a given process. <br> - To retrieve the path of a (single) file accessed by a given process." %}

The "Estimated performance impact" events track the performance impact of
`Windows Defender` scans and can be a source of historical information on
program executions.

Format:

```
<TIMESTAMP_UTC> ProcessImageName: <PROCESS_NAME>, Pid: <PROCESS_PID>, TotalTime: <SCAN_TIME>, Count: <FILE_COUNT>, MaxTime: <MAX_FILE_SCAN_TIME>, MaxTimeFile: <MAX_FILE_SCAN_PATH>, EstimatedImpact: <ESTIMATED_IMPACT>%
```

Example:

```
2022-11-23T09:10:55.472Z ProcessImageName: svchost.exe, Pid: 7836, TotalTime: 6070, Count: 24, MaxTime: 828, MaxTimeFile: \Device\HarddiskVolume1\Downloads\binary.exe, EstimatedImpact: 100%
```

Information of interest available, as stated in the
[Microsoft documentation](https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/troubleshoot-performance-issues?view=o365-worldwide):

| Field | Description | Example event |
|-------|-------------|---------------|
| ProcessImageName | The process's image name. | `svchost.exe` |
| Pid | The process's `Process ID (PID)`. | 7836 |
| TotalTime | The total time `Windows Defender` spent scanning the files accessed by the process. | 6070 |
| Count | The total number of files accessed by the process that were scanned by `Windows Defender`. <br><br> As some files can be excluded from scanning, the total number of files accessed by the process may be higher. | 24 |
| MaxTime | The longest scan time recorded, in milliseconds. | 828 |
| MaxTimeFile | The path of the file for which the longest scan time (of MaxTime duration) was recorded. | \Device\HarddiskVolume1\Downloads\binary.exe |
| EstimatedImpact | "The percentage of time spent in scans for files accessed by this process out of the period in which this process experienced scan activity" | 100% |

#### Real-time detections events

{% include note.html content="Detection-related events can be used to retrieve information on malware detections by `Windows Defender` (in complement to [`Microsoft-Windows-Windows Defender/Operational` ETW events](./etw_windows_defender.md#windows-defender-malware-detection-events)). The information available notably include the filepath of the file that raised the detection, and the threat type and name of the detection." %}

Various events are emitted by `Windows Defender` following a detection. These
events can be used to retrieve information on the detection, such as the
malicious file path, acceding process, and remediation action taken by
`Windows Defender`.

Examples:

```
# "DETECTIONEVENT MPSOURCE_REALTIME" events.
2022-12-15T11:24:18.854Z DETECTIONEVENT MPSOURCE_REALTIME HackTool:MSIL/Mimikatz!MSR file:C:\Users\<USERNAME>\Videos\binary.exe;

# "DETECTION_ADD" events.
2022-12-15T11:24:18.854Z DETECTION_ADD#1 HackTool:MSIL/Mimikatz!MSR file:C:\Users\<USERNAME>\Videos\binary.exe PropBag [length: 0, data: (null)]

# "[RTP] [Mini-filter] Blocked file" events.
2022-12-15T11:24:29.026Z [RTP] [Mini-filter] Blocked file(#74): \Device\HarddiskVolume2\Users\<USERNAME>\Videos\binary.exe. Process: \Device\HarddiskVolume2\Windows\explorer.exe, Status: 0x0, [...]

# "DETECTION_CLEANEVENT MPSOURCE_REALTIME" events.
2022-12-15T11:25:01.549Z DETECTION_CLEANEVENT MPSOURCE_REALTIME MP_THREAT_ACTION_QUARANTINE 0x80508033 HackTool:MSIL/Mimikatz!MSR file:C:\Users\<USERNAME>\Videos\binary.exe;

# "Resource Scan" events.
Begin Resource Scan
Scan ID:{C343E826-1234-1234-1234-12345678999}
Scan Source:6
Start Time:12-15-2022 11:24:33
End Time:12-15-2022 11:24:45
Explicit resource to scan
Resource Schema:file
Resource Path:C:\Users\<USERNAME>\Videos\binary.exe
Result Count:1
Threat Name:HackTool:MSIL/Mimikatz!MSR
ID:2147805916
Severity:4
Number of Resources:1
Resource Schema:file
Resource Path:C:\Users\<USERNAME>\Videos\binary.exe
Extended Info - SigSeq:00001667a6e4976b
Extended Info - SigSha:3de04872ea8ce2ceea7d0787abe1234567899876
End Scan
************************************************************

# "threat actions" events.
Beginning threat actions
Start time:12-15-2022 11:27:46
Threat Name:HackTool:MSIL/Mimikatz!MSR
Threat ID:2147805916
Action:quarantine
Resource action complete:Quarantine
Path:\\?\C:\Users\<USERNAME>\Videos\binary.exe
File to act on SHA1:343051CC1B3F33201D076478EA9BADC796951423
File owner:<DOMAIN>\<USERNAME>
File cleaned/removed successfully
File Name:C:\Users\<USERNAME>\Videos\binary.exe
Action remove successful on file:\\?\C:\Users\<USERNAME>\Videos\binary.exe
Resource action complete:Removal
Finished threat actions
[...]
End time:12-15-2022 11:27:46
```

#### Suspicious command line events

{% include note.html content="'Engine:command line reported' events can be used to track suspicious and potentially malicious command lines (as detected by `Windows Defender`)." %}

`Windows Defender` records events on detection of suspicious and potentially
malicious command line executions. Two level of criticality are indicated by
`Windows Defender`: `lowfi` and `threat`.

Format:

```
<TIMESTAMP_UTC> Engine:command line reported as <lowfi | threat>: <COMMAND_LINE>
```

Example:

```
2022-12-14T10:55:21.580Z Engine:command line reported as lowfi: C:\Windows\System32\reg.exe(reg add HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential /t REG_DWORD /d 1 /f)
```

#### "BM telemetry" events

{% include note.html content="'BM telemetry' events can be used to track potentially malicious files and activity (as detected by the `Behavior Monitoring` component), notably recording the suspicious process image path, parent process, and (sometimes) loaded `DLLs`." %}

The `Windows Defender` `Behavior Monitoring` component generates telemetry /
events on suspicious files and activity. The feature, and associated event
type, are not well documented.

Information of interest available:

| Field | Description |
|-------|-------------|
| ImagePath | The process's image path. |
| ProcessID | The process's `Process ID (PID)`. |
| ProcessCreationTime | The creation time of the process, in `Windows NT Time` format. |
| Modules | Sometimes, the `Dynamic Link Libraries (DLL)` loaded by the process. |
| Parents | The parent process's image path of the process. Additional image paths may sometimes be referenced, with out a clear link between the processes. For example: `Parents: C:\Program Files (x86)\XXX\userland-binary.exe:3052:2,Registry:104:2,C:\Windows\System32\lsass.exe:700:2` |

Examples:

```
BEGIN BM telemetry
GUID:{12345678-ACEA-905D-0234-123456789876}
SignatureID:119519209161905
SigSha:9dd373682d5f42cfff1504fe09e860ed9e16d7c3
ThreatLevel:0
ProcessID:7968
ProcessCreationTime:133474951288037348
SessionID:0
CreationTime:12-19-2023 22:32:09
ImagePath:C:\Windows\SysWOW64\WindowsPowerShell\v1.0\powershell.exe
Taint Info:Friendly: Y; Reason: ; Modules: C:\Windows\assembly\NativeImages_v4.0.30319_32\Microsoft.Pb378ec07#\36f0282d3041ebc3f52968b6d1cb281d\Microsoft.PowerShell.ConsoleHost.ni.dll:25,[...]; Parents: C:\Program Files (x86)\Mesh Agent\MeshAgent.exe:2504:3,
Operations:None
END BM telemetry

BEGIN BM telemetry
GUID:{12345678-1F83-D721-D5E3-123456789876}
SignatureID:50246999395008
SigSha:37b77ab291baf8b386fd48ac9fb8f92077e7f4a4
ThreatLevel:0
ProcessID:6972
ProcessCreationTime:133474494856736096
SessionID:0
CreationTime:12-19-2023 09:51:29
ImagePath:C:\Windows\PSEXESVC.exe
Taint Info:Friendly: Y; Reason: ; Modules: ; Parents: C:\Windows\System32\services.exe:692:1,
Operations:None
END BM telemetry
```

#### "Filter caching disabled" events

{% include note.html content="'Filter caching disabled' events can be used to track potentially malicious files." %}

`Windows Defender` seems to disable its "filter caching" on potentially
suspicious files, generating an associated "Filter caching disabled" event.
This event can be used to track (some) files that `Windows Defender` identified
as suspicious (with out necessarily raising a detection alert). For instance,
"Filter caching disabled" events are raised on `PsExec` binary, some port
scanning utilities (such as `netscan`), or some remote monitoring and
management products. The feature, and associated event type, are not
documented.

Format:

```
2022-05-17T09:22:32.600Z Filter caching disabled for <FILE_PATH> (runtime MpDisableCaching from 0x0)
```

Examples:

```
2022-05-17T09:22:32.600Z Filter caching disabled for \Device\HarddiskVolume2\Users\<USERNAME>\Documents\1\PsExec.exe (runtime MpDisableCaching from 0x0)
2022-05-17T09:41:48.226Z Filter caching disabled for \Device\HarddiskVolume2\Users\<USENRAME>\Documents\1\netscan.exe (runtime MpDisableCaching from 0x0)
```

#### EMS Detection events

{% include note.html content="'Filter caching disabled' events can be used to track detection of malicious content in memory." %}

`Windows Defender` seems to periodically scan the memory of processes,
generating `Engine:EMS scan for process` events. If malicious content is
detected in memory, an `EMS detection` event may be recorded.

Format:

```
<TIMESTAMP_UTC> Engine:EMS detection: <THREAT_NAME>, sigseq=<SIGSEQ>, pid=<PROCESS_ID>
```

Example:

```
2023-11-24T06:19:53.270Z Engine:EMS detection: HackTool:MSIL/Mimikatz!MSR, sigseq=0x0000123456789B73, pid=1254
```

#### "Issuing SDN query" events

{% include note.html content="'Issuing SDN query' events can be used as an evidence of file existence, along with the file's `SHA1` and `SHA256` hashes." %}

`Windows Defender` seems to make "Cloud" query on suspicious files, generating
"Issuing SDN query" events. Other events may however be generated instead (as
shown below) depending on the `Windows Defender` version. This event is likely
to be dependant on the activation of "Cloud-delivered protection"
(`SubmitSamplesConsent` set to `0x1`). The feature, and associated event type,
are not documented.

Format:

```
<TIMESTAMP_UTC> SDN:Issuing SDN query for <FILE_PATH> (<FILE_PATH>) (sha1=<SHA1>, sha2=<SHA256>)
```

Examples:

```
# SND query event.
2023-07-12T11:16:34.869Z SDN:Issuing SDN query for \Device\HarddiskVolume4\inetpub\wwwroot\file.asp (\Device\HarddiskVolume4\inetpub\wwwroot\file.asp) (sha1=12345678900b1a36ee0e7f932386ca1234567890, sha2=1234567899876543215059e9780f802a2f75b432b0d87a123456789987654321)

# Cloud query events with out file information.
2023-12-18T18:41:02.439Z [Cloud] Engine is requesting config to do cloud query [regular network].
2023-12-18T18:41:02.470Z [Cloud] SubmitReport(CMpSpyDssContext), ShouldSendEvenOnPaidNetworks: 1
2023-12-18T18:41:02.470Z [Cloud] Start of cloud request. Passive mode: 0
2023-12-18T18:41:02.470Z [Cloud] Queued cloud request.
2023-12-18T18:41:02.470Z [Cloud] MpEngineCloudRequest(). hr = 0
2023-12-18T18:41:02.470Z [Cloud] Dequeued cloud request.
2023-12-18T18:41:02.470Z [Cloud] RpcSpynetQueueGenerateReport(). hr = 0
2023-12-18T18:41:02.861Z SDN:SDN query completed: 00000000
```

#### "Setting original file name" events

{% include note.html content="'Setting original file name' events can be used to track renamed files." %}

`Windows Defender` records events that indicate the "original" filename of
renamed files. The "original" filename seems to be retrieved from the file
`VERSIONINFO` header's `OriginalFilename` or `ProductName` field (if the
`OriginalFilename` field is not specified). This event can thus be an indicator
of file whose filename does not match their `OriginalFilename` / `ProductName`,
such as system utilities (such as `cmd.exe`) renamed by threat actors for
defense evasion purposes.

Format:

```
<TIMESTAMP_UTC> Engine:Setting original file name "<ORIGINAL_FILENAME>" for "<FILE_PATH>", hr=0x0
```

Example:

```
2022-10-24T17:56:18.140Z Engine:Setting original file name "psexec.c" for "c:\users\<USERNAME>\videos\binary.exe", hr=0x0
```

#### RTP Perf Log

{% include note.html content="'RTP Perf Log' events can be used to determine the `Windows Defender` scan exclusion(s) at the time of the event." %}

The 'RTP Perf Log' events reference the scan exclusion(s) configured for
`Windows Defender` (and other settings and parameters) at the time of the
event. Exclusions can be configured on process names / paths, folders, or file
extensions.

```
****************************RTP Perf Log***************************
RTP Start:N/A
Last Perf:(null)
First RTP Scan:N/A
Plugin States:  AV:2  AS:2  RTP:2  OA:2  BM:2
Process Exclusions:
  C:\Microsoft System Center 2012\xxx\SQL\MSRS10_50.MSDPM2012\Reporting Services\ReportServer\bin\ReportingServicesService.exe
Path Exclusions:
  C:\Windows\Security\Database\*.chk
  %windidr%\SoftwareDistribution\Datastore\*.*
Ext Exclusions:
  .DBF
  .NDF
  .RAR
  .XML
  .IDX
  .BAK
  .PDF
  .BKP
[...]
```

In addition to 'RTP Perf Log' events, other `Windows Defender` real-time
protection events may reference the configured scan exclusions.
"[RTP] [Exclusion]" or
"[RTP] [Mini-filter] volume <XXX> excluded from scanning due to path exclusion"
events also reference the folders excluded from a scan.

```
2024-01-26T22:46:35.043Z [RTP] [Exclusion] T:\ is discarded due to error 0x80070002
2024-01-26T22:46:35.043Z [RTP] [Mini-filter] volume \Device\HarddiskVolume34 excluded from scanning due to path exclusion
```

### Tool(s)

The [`mplog_parser`](https://github.com/Intrinsec/mplog_parser) Python script
can be used to parse `MPLog` files into multiple CSV files (one file per event
type parsed).

`mplog_parser` parses:

  - [Estimated performance impact events](#estimated-performance-impact-events).

  - [Real-time detections events](#real-time-detections-events).

  - [Suspicious command line events](#suspicious-command-line-events).

  - ['BM telemetry' events](#bm-telemetry-events).

  - ["Setting original file name" events](#setting-original-file-name-events).

  - ["RTP Perf Log" events](#rtp-perf-log), that reference the configured scan
    exclusion(s).

```bash
mplog_parser -d "<INPUT_FOLDER>" -o "<OUTPUT_FOLDER>"
```

### References

  - [Crowdstrike - James Lovato - Mind the MPLog: Leveraging Microsoft Protection Logging for Forensic Investigations](https://crowdstrike.com/blog/how-to-use-microsoft-protection-logging-for-forensic-investigations/)

  - [Microsoft - Troubleshoot performance issues related to real-time protection](https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/troubleshoot-performance-issues?view=o365-worldwide)

  - [INTRINSEC - Hunting attackers using Microsoft Protection Logs (MPLogs)!](https://www.intrinsec.com/hunt-mplogs/)
