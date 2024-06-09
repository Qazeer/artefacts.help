---
title: Microsoft Exchange
summary: 'Microsoft Exchange is a complex mail server ecosystem, running exclusively on the Windows operating system. Microsoft Exchange logs telemetry and events in various text-based log files (20+) and EVTX event logs (40+).\n\nThe following sources of logs can be of forensics interest to investigate the compromise of a Microsoft Exchange server or account:\n - Exchange IIS logs, that contain information on the HTTP requests made to the various Exchange HTTP web services.\n - EVTX event logs, of Exchange ETW channels (such as the "MSExchange Management" channel), and of other providers depending on the Exchange server Windows logging configuration.\n - 20+ text-based log files, each associated with a given Exchange service.'
keywords: Mail, email, mailserver, Microsoft Exchange, Exchange, IIS, MSExchange Management, PowerShell, webshell, W3SVC1, W3SVC2, u_exYYmmdd, c-ip
tags:
  - mailservers
default_location: 'Exchange IIS logs:\n %SystemDrive%\inetpub\logs\LogFiles\W3SVC1 and W3SVC2 folders.'
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

### Text based Exchange logs

### Webshells hunting

### References

  - [InverseCos - Hunting for APT Abuse of Exchange](https://www.inversecos.com/2022/07/hunting-for-apt-abuse-of-exchange.html)

  - [m365internals - Hunting In On-Premises Exchange Server Logs](https://m365internals.com/2022/10/07/hunting-in-on-premises-exchange-server-logs/)

  - [Microsoft - Configure Logging in IIS](https://learn.microsoft.com/en-us/iis/manage/provisioning-and-managing-iis/configure-logging-in-iis)
