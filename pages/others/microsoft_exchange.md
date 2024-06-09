---
title: Microsoft Exchange
summary: 'Microsoft Exchange is a complex mail server ecosystem, running exclusively on the Windows operating system. Microsoft Exchange logs telemetry and events in various text-based log files (20+) and EVTX event logs (40+).\n\nThe following sources of logs can be of forensics interest to investigate the compromise of a Microsoft Exchange server or account:\n - Exchange IIS logs, that contain information on the HTTP requests made to the various Exchange HTTP web services.\n - EVTX event logs, of Exchange ETW channels (such as the "MSExchange Management" channel), and of other providers depending on the Exchange server Windows logging configuration.\n - 20+ text-based log files, each associated with a given Exchange service.'
keywords: Mail, email, mailserver, Microsoft Exchange, Exchange, IIS, MSExchange Management, PowerShell, webshell, W3SVC1, W3SVC2, u_exYYmmdd, c-ip
tags:
  - mailservers
default_location: 'Exchange IIS logs:\n %SystemDrive%\inetpub\logs\LogFiles\W3SVC1 and W3SVC2 folders.\n\n ETW channels:\n\n MSExchange Management, for usage of cmdlets from the ExchangePowerShell module (that interact with the Exchange Web Services (EWS) API).\n\n Microsoft-Windows-Windows Defender/Operational, events 1006 / 1116 and 1007 / 1117, for detections of suspicious behavior related to Exchange.\n\n Security event 4688, if "Audit Process Creation" is enabled, to identity suspicious process spawned by the Exchange IIS process.'
last_updated: 2024-06-09
sidebar: sidebar
permalink: microsoft_exchange.html
folder: others
---

### Overview

`Microsoft Exchange` is a complex mail server ecosystem, running exclusively on
the `Windows` operating system. `Microsoft Exchange` logs telemetry and
events in various text-based log files (20+) and `EVTX` event logs (40+).

The following sources of logs can be of forensics interest to investigate the
compromise of a `Microsoft Exchange` server or account:

  - `Exchange IIS` logs, that contain information on the `HTTP` requests made
    to the various `Exchange HTTP` web services.

  - `EVTX` event logs, of `Exchange` `ETW` channels (such as the
    `MSExchange Management` channel), and of other providers depending on the
    `Exchange` server `Windows` logging configuration (process creation events,
    `Windows Defender` detections, ...).

  - 20+ text-based log files, each associated with a given `Exchange` service.

Additionally, the `Exchange` web folders should be:

  - Reviewed for files created or modified during the attack time-frame (using
    artefacts such as the `MFT` or `UsrJnrl`)

  - Scanned for webshells presence (using `yara` and rules from
    [`yara-forge`](https://yarahq.github.io/) for instance).

### Exchange IIS logs

The `Exchange IIS` logs contain information on the `HTTP` requests made to the
various `Exchange HTTP` web services (`owa`, `ecp`, `EWS`, `Autodiscover`,
`ActivSync`, etc.). The `IIS` logs can contain indicator of vulnerability
exploitation (such as `ProxyLogon`), webshells access, as well as evidence of
data exfiltration (through threat actor created files, such as compressed
archives, exposed by the `Exchange IIS` web server).

The `Exchange IIS` log files are stored under the
`%SystemDrive%\inetpub\logs\LogFiles\W3SVC1` and `W3SVC2` folders (as
`Exchange` defines two `IIS` sites, "Default Web Site" and "Exchange Back
End").

As with any `IIS` logs, the available fields will depend on the `IIS` logging
configuration, with various log formats supported and a selection of possible
fields to track. By default, both `Exchange IIS` sites have the same logging
configuration and log `HTTP` requests to text-based log files `u_exYYmmdd`, in
`W3C` format, and with the same fields selected.

| Field | Description | Example |
|-------|-------------|---------|
| `date` | Date of the request, `YYYY-mm-dd` format. | 2023-12-25 |
| `time` | Time of the request, `HH:MM:SS` format in `UTC`. | 23:20:54 |
| `s-ip` | The IP address of the `Exchange` server. | 10.10.130.5 |
| `cs-method` | The `HTTP` method (usually `GET` or `POST`) of the request. | GET |
| `cs-uri-stem` | The `URI` requested, with the query string excluded. | /owa/auth/aKdMadOeT.aspx |
| `cs-uri-query` | The query string, if any, of the request. | cmd=whoami |
| `s-port` | The `Exchange` port the request was received on. | 443 |
| `cs-username` | For authenticated users, the username of the user that performed the request. Otherwise, for non authenticated requests, an hyphen. | - |
| `c-ip` | The client IP address. | 1.2.3.4 |
| `cs(User-Agent)` | The client `HTTP` `User-Agent` header. | Mozilla/5.0+Mozilla/5.0 (Windows NT 10.0; Win64; x64)[...] |
| `cs(Referer)` | The client `HTTP` `Referer` header, containing the address from which the resource has been requested (last visited page). | - |
| `sc-status` | The response `HTTP` status code. | 200 |
| `sc-substatus` | The response [`HTTP` sub status code](https://learn.microsoft.com/en-us/troubleshoot/developer/webapps/iis/www-administration-management/http-status-code). | 0 |
| `sc-win32-status` | The Windows status code, `0` for success and non `0` if an error occurred. The error message associated with an error code can be retrieved using `certutil.exe -error <ERROR_CODE>`. | 0 |
| `time-taken` | The time taken by the request, in milliseconds. | 142 |

Example of an `HTTP` `GET` request made to a threat actor uploaded webshell:

```
# Fields: date time s-ip cs-method cs-uri-stem cs-uri-query s-port cs-username c-ip cs(User-Agent) cs(Referer) sc-status sc-substatus sc-win32-status time-taken

2023-12-25 11:54:21 10.10.130.5 GET /owa/auth/aKdMadOeT.aspx cmd=whoami 443 - 1.2.3.4 Mozilla/5.0+Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/522.36 (KHTML, like Gecko) Chrome/121.0.0.0 Safari/512.34 - 200 0 0 142
```

### EVTX based events

| Channel | Conditions | Events |
|---------|------------|--------|
| `MSExchange Management` | Default configuration. | The `MSExchange CmdletLogs` provider logs, to the `MSExchange Management` channel, usage of cmdlets from the `ExchangePowerShell` module. The cmdlets of the `ExchangePowerShell` module can be used to perform a number of actions through the `Exchange Web Services (EWS)` API, such as managing the Exchange server and its mailboxes, searching and exporting mailboxes content, setting email forward rules, etc. <br><br> Event `1: Cmdlet suceeded. Cmdlet <CMDLET>, parameters <COMMAND_LINE_PARAMETERS>`. <br><br> Event `6: Cmdlet failed. Cmdlet <CMDLET>, parameters <COMMAND_LINE_PARAMETERS>`, with the error details in the raw event data. <br><br> Information of interest: <br> - The PowerShell `cmdlet` used, with the associated parameters. <br><br> Among other, the executions of the following `ExchangePowerShell` cmdlets should be reviewed: <br><br> *Reconnaissance* <br> - [`Get-ExchangeServer`](https://learn.microsoft.com/en-us/powershell/module/exchange/get-exchangeserver) <br> - [`Get-Mailbox`](https://learn.microsoft.com/en-us/powershell/module/exchange/get-mailbox) <br> - [`Get-Recipient`](https://learn.microsoft.com/en-us/powershell/module/exchange/get-recipient) <br> - [`Get-MailboxPermission`](https://learn.microsoft.com/en-us/powershell/module/exchange/get-mailboxpermission) <br> - [`Get-MailboxRegionalConfiguration`](https://learn.microsoft.com/en-us/powershell/module/exchange/get-mailboxregionalconfiguration) <br> - [`Get-InboxRule`](https://learn.microsoft.com/en-us/powershell/module/exchange/get-inboxrule) <br><br> *Mailbox search for data access* <br> - [`Search-Mailbox`](https://learn.microsoft.com/en-us/powershell/module/exchange/search-mailbox) <br> - [`New-MailboxSearch`](https://learn.microsoft.com/en-us/powershell/module/exchange/new-mailboxsearch) <br> - [`New-MailboxExportRequest`](https://learn.microsoft.com/en-us/powershell/module/exchange/new-mailboxexportrequest) <br> - [`Remove-MailboxExportRequest`](https://learn.microsoft.com/en-us/powershell/module/exchange/remove-mailboxexportrequest) <br><br> *Configuration of email forwarding for persistent data exfiltration (through direct email forwarding, inbox rules, or transport rules)* <br> - [`Set-Mailbox`](https://learn.microsoft.com/en-us/powershell/module/exchange/set-mailbox), with `ForwardingSmtpAddress` to forward emails or `HiddenFromAddressListsEnabled` to hide the mailbox from the `Global Address List` <br> - [`Set-InboxRule`](https://learn.microsoft.com/en-us/powershell/module/exchange/set-inboxrule) <br> - [`New-InboxRule`](https://learn.microsoft.com/en-us/powershell/module/exchange/new-inboxrule) <br> - [`Enable-InboxRule`](https://learn.microsoft.com/en-us/powershell/module/exchange/enable-inboxrule) <br> - [`New-TransportRule`](https://learn.microsoft.com/en-us/powershell/module/exchange/new-transportrule) <br><br> *Configuration of mailbox delegations for persistent access (to access a mailbox / mailbox's folder or to send emails from the mailbox)* <br> - [`Add-MailboxPermission`](https://learn.microsoft.com/en-us/powershell/module/exchange/add-mailboxpermission) <br> - [`Add-MailboxFolderPermission`](https://learn.microsoft.com/en-us/powershell/module/exchange/add-mailboxfolderpermission) <br> - [`Add-RecipientPermission`](https://learn.microsoft.com/en-us/powershell/module/exchange/add-recipientpermission) <br> - [`Set-TransportRule`](https://learn.microsoft.com/en-us/powershell/module/exchange/set-transportrule) <br><br> *Administration of the Exchange server* <br> - [`Set-RoleGroup`](https://learn.microsoft.com/fr-fr/powershell/module/exchange/set-rolegroup) <br> - [`New-ManagementRoleAssignment`](https://learn.microsoft.com/en-us/powershell/module/exchange/new-managementroleassignment), including the `Mailbox Import Export` role to export mailbox content <br> - [`New-ExchangeCertificate`](https://learn.microsoft.com/en-us/powershell/module/exchange/new-exchangecertificate) |
| `Microsoft-Windows-Windows Defender/Operational` | Requires `Windows Defender` to be running. | `Windows Defender` integrates a number of detections for suspicious `Exchange` behavior: `Exchange IIS` process spwaning `PowerShell` or `cmd` (`IISExchgSpawnCMD.A`), exploitation of the `ProxyLogon` vulnerability (`Exploit:Win32/CVE-2021-31207.B` / `Behavior:Win32/SuspExchgSession.E`), etc. <br><br> Standard [events `1006` / `1116` and `1007` / `1117`](./../windows/etw_windows_defender.md#windows-defender-malware-detection-events) will be logged for detections of suspicious behavior related to `Exchange`. |
| `Security` | Requires `Audit Process Creation` to be enabled. <br><br> Requires `ProcessCreationIncludeCmdLine_Enabled` to be enabled for the command line to be logged. | [Event `4688: A new process has been created`](./../windows/etw_process_creation.md). <br><br> Can be used to identity suspicious process (`NewProcessName`), such as `powershell.exe` or `cmd.exe`, spawned (`ParentProcessName`) by the `Exchange IIS` process `w3wp.exe`. |

### Text based Exchange logs

### Webshells hunting

### References

  - [InverseCos - Hunting for APT Abuse of Exchange](https://www.inversecos.com/2022/07/hunting-for-apt-abuse-of-exchange.html)

  - [m365internals - Hunting In On-Premises Exchange Server Logs](https://m365internals.com/2022/10/07/hunting-in-on-premises-exchange-server-logs/)

  - [Microsoft - Configure Logging in IIS](https://learn.microsoft.com/en-us/iis/manage/provisioning-and-managing-iis/configure-logging-in-iis)
