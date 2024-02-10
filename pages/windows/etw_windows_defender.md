---
title: ETW - Windows Defender
summary: 'Windows Defender activity, such as malware detection or configuration change.\n\nMain events:\n\nChannel: Microsoft-Windows-Windows Defender/Operational.\nEvent ID 1116: "The antimalware engine found malware or other potentially unwanted software".\nEvent ID 5001: "Real-time protection is disabled".\nEvent ID 5007: "The antimalware platform configuration changed".'
keywords: 'Windows Defender, Defender, '
tags:
  - windows_etw
  - windows_defense_evasion
location: 'Channel: Microsoft-Windows-Windows Defender/Operational.\nEvents: 1006, 1007, 1008, 1009, 1010, 1011, 1012, 1013, 1015, 1116, 1117, 1118, 1119, 1121, 1122, 5001, 5007, 5010, 5012, 5013.'
last_updated: 2024-01-29
sidebar: sidebar
permalink: windows_etw_windows_defender.html
folder: windows
---

### Overview

`Windows Defender` generates, among other telemetry, event logs that can be of
great use during incident response investigations. Even if another security
product is installed on the system, `Windows Defender` may still be generating
events related to malware detections or suspicious users behavior.

### Windows Defender malware detection events

| Channel | Conditions | Events |
|---------|------------|--------|
| `Microsoft-Windows-Windows Defender/Operational` | Default configuration. | Events `1006` and `1116`: `The antimalware engine found malware or other potentially unwanted software`. <br><br> Logged whenever a malware is detected by `Windows Defender`. <br><br> Suspicious behaviors, such as the dump of the `LSASS` process memory by the `Taskmgr` or export of the `SAM` / `SYSTEM` / `SECURITY` registry hives using the `reg` utility, can also generate events `1116` (category `Behavior:*`). <br><br> Information of interest: <br> - The file path of the file that triggered the detection. For behavioral detections, the file path field can instead store information on the process, such as the process command line or `Process ID (PID)` and start time. <br> - The threat name (such as `Backdoor:JS/Chopper.VH!MSR` or `Trojan:PowerShell/ReverseShell.SA`), category, and severity. <br> - The eventual process and domain and username of the user associated with the detection. <br> - The action taken by `Windows Defender`, such as putting the file in quarantine. |
| `Microsoft-Windows-Windows Defender/Operational` | Default configuration. | Event `1015: The antimalware platform detected suspicious behavior`. <br><br> Logged whenever suspicious behavior is detected by `Windows Defender`. Suspicious behaviors may generate events `1116` instead of events `1015`. <br><br> Information of interest: similar level of information to events `1006` / `1116`. |
| `Microsoft-Windows-Windows Defender/Operational` | Default configuration. | Events `1007` and `1117`: `The antimalware platform performed an action to protect your system from malware or other potentially unwanted software`. <br> Event `1008` and `1118`: `The antimalware platform attempted to perform an action to protect your system from malware or other potentially unwanted software, but the action failed`. <br> Event `1119: The antimalware platform encountered a critical error when trying to take action on malware or other potentially unwanted software`. <br><br> Logged whenever `Windows Defender` takes an action following a detection, such as putting the malicious file in quarantine. The action may however fail, resulting in error events `1008` / `1118` or `1119`. <br><br> Information of interest: similar level of information to events `1006` / `1116`. |
| `Microsoft-Windows-Windows Defender/Operational` | Default configuration. | Event `1009: The antimalware platform restored an item from quarantine`. <br> Event `1010: The antimalware platform couldn't restore an item from quarantine`. <br> Event `1011: The antimalware platform deleted an item from quarantine`. <br> `1012: The antimalware platform couldn't delete an item from quarantine`. <br><br> Logged on quarantine lifecycle actions taken by `Windows Defender`: restoring a file from quarantine or deleting a quarantined file. The operation may fail, resulting in associated error events `1010` or `1012`. <br><br> Information of interest: <br> - The original file path of the file in quarantine. <br> - The threat name, category, and severity. |
| `Microsoft-Windows-Windows Defender/Operational` | Require `ASR` rules to be configured on audit / block mode. | Event `1121: Windows Defender Exploit Guard has blocked an operation that is not allowed by your IT administrator`. <br> Event `1122: Windows Defender Exploit Guard audited an operation that is not allowed by your IT administrator`. <br><br> Logged whenever an [`ASR rule`](https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/attack-surface-reduction-rules-reference) got triggered, either in block or audit mode. `ASR rules` may however [generate many false-positives](https://blog.palantir.com/microsoft-defender-attack-surface-reduction-recommendations-a5c7d41c3cf8). <br><br> Information of interest: <br> - The triggered `ASR rule` `GUID`. <br> - The process name, file, and user associated with the trigger. |

### Windows Defender configuration change events

| Channel | Conditions | Events |
|---------|------------|--------|
| `Microsoft-Windows-Windows Defender/Operational` | Default configuration. | Event `5001: Real-time protection is disabled`. <br><br> Indicates that the real-time protection of `Windows Defender` was disabled. <br><br> The event does not indicate which user performed the action. |
| `Microsoft-Windows-Windows Defender/Operational` | Default configuration. | Event `5007: The antimalware platform configuration changed`. <br><br> Indicates a change of the `Windows Defender` configuration, with one configuration change per event. The previous and new value of the configuration item, as stored in the registry, are logged. <br><br> For instance: <br><br> `Old value: HKLM\SOFTWARE\Microsoft\Windows Defender\Features\TamperProtection = 0x5` <br> `New value: HKLM\SOFTWARE\Microsoft\Windows Defender\Features\TamperProtection = 0x4` <br> Indicates that the tamper protection of `Windows Defender` was disabled. <br><br> `Old value: HKLM\SOFTWARE\Microsoft\Windows Defender\SpyNet\SubmitSamplesConsent = 0x1` <br> `New value: HKLM\SOFTWARE\Microsoft\Windows Defender\SpyNet\SubmitSamplesConsent = 0x0` <br> Indicates that automatic samples submission was turned off. <br><br> `New value: HKLM\SOFTWARE\Microsoft\Windows Defender\Exclusions\Paths\C:\ = 0x0` <br> Indicates that the `C:\` folder was added to `Windows Defender` folder exclusion list. <br><br> `New value: HKLM\SOFTWARE\Microsoft\Windows Defender\Exclusions\Processes\cmd.exe = 0x0` <br> Indicates that `cmd.exe` processes were added to the `Windows Defender` process exclusion list. <br><br> The event does not indicate which user performed the action. |
| `Microsoft-Windows-Windows Defender/Operational` | Default configuration. | Event `5010: Scanning for malware and other potentially unwanted software is disabled` (`MALWAREPROTECTION_ANTISPYWARE_DISABLED`). <br> Event `5012: Scanning for viruses is disabled` (`MALWAREPROTECTION_ANTIVIRUS_DISABLED`). <br><br> Indicates that the `Windows Defender` feature associated with the event was disabled. |
| `Microsoft-Windows-Windows Defender/Operational` | Default configuration. | Event `5013: Tamper protection blocked a change to Microsoft Defender Antivirus` (`MALWAREPROTECTION_SCAN_CANCELLED`). <br><br> Indicates that a change to the `Windows Defender` configuration was blocked by the tamper protection mechanism. |
| `Microsoft-Windows-Windows Defender/Operational` | Default configuration. | Events `1013: Antivirus Microsoft Defender has removed history of malware and other potentially unwanted software`. <br><br> Logged whenever the `Windows Defender` Protection history (as displayed in the "Virus & threat protection" tab) is cleared. The Protection history is displayed to end-users and notably includes detections and quarantined files of the last 30 days. `Windows Defender` protection history records are automatically deleted after that 30 days period, so this event occurs legitimately. <br><br> Manually clearing the `Microsoft-Windows-Windows Defender%4Operational.evtx` file, or other `Windows Defender` log files, do not generate an event `1013`. <br><br> The event include two timestamps: <br> - The event timestamp, that indicates when the protection history was cleared. <br> - The `Timestamp` field (in the event's `EventData` section), that represents the time of occurrence of the record that was automatically removed. |

### References

  - [Microsoft - Review event logs and error codes to troubleshoot issues with Microsoft Defender Antivirus](https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/troubleshoot-microsoft-defender-antivirus?view=o365-worldwide)

  - [Palantir - Chad D. - Microsoft Defender Attack Surface Reduction Recommendations](https://blog.palantir.com/microsoft-defender-attack-surface-reduction-recommendations-a5c7d41c3cf8)

  - [Microsoft Antimalware has removed history of malware and other potentially unwanted software 1013 - Kosh Vorlon](https://web.archive.org/web/20160727113019/https://answers.microsoft.com/en-us/protect/forum/mse-protect_scanning/microsoft-antimalware-has-removed-history-of/f15af6c9-01a9-4065-8c6c-3f2bdc7de45e)
