---
title: Microsoft Exchange
summary: 'Microsoft Exchange is a complex mail server ecosystem, running exclusively on the Windows operating system. Microsoft Exchange logs telemetry and events in various text-based log files (100+) and EVTX event logs (40+).\n\nThe following sources of logs can be of forensics interest to investigate the compromise of a Microsoft Exchange server or account:\n - Exchange IIS logs, that contain information on the HTTP requests made to the various Exchange HTTP web services.\n - EVTX event logs, of Exchange ETW channels (such as the "MSExchange Management" channel), and of other providers depending on the Exchange server Windows logging configuration.\n - 20+ text-based log files, each associated with a given Exchange service.'
keywords: Mail, email, mailserver, Microsoft Exchange, Exchange, IIS, Outlook, EWS, ECP, OWA, ActiveSync, MAPI, Message Tracking, Autodiscover, OAB, RpcHttp, SMTP, MSExchange Management, MSExchange CmdletLogs, ExchangePowerShell, webshell, W3SVC1, W3SVC2, u_exYYmmdd, c-ip, CVE-2021-26855, 26858, 27065, 26857, ProxyLogon, Get-ExchangeServer, Get-Mailbox, Get-Recipient, Get-MailboxPermission, Get-MailboxRegionalConfiguration, Get-InboxRule, Search-Mailbox, New-MailboxSearch, New-MailboxExportRequest, Remove-MailboxExportRequest, Set-Mailbox, Set-InboxRule, New-InboxRule, Enable-InboxRule, New-TransportRule, Add-MailboxPermission, Add-MailboxFolderPermission, Add-RecipientPermission, Set-TransportRule, New-ManagementRoleAssignment, New-ExchangeCertificate
tags:
  - mailservers
location: 'Exchange IIS logs:\n %SystemDrive%\inetpub\logs\LogFiles\W3SVC1 and W3SVC2 folders.\n\n ETW channels:\n MSExchange Management, for usage of cmdlets from the ExchangePowerShell module (that interact with the Exchange Web Services API).\n Microsoft-Windows-Windows Defender/Operational, events 1006 / 1116 and 1007 / 1117, for detections of suspicious behavior related to Exchange.\n Security event 4688, if "Audit Process Creation" is enabled, to identity suspicious process spawned by the Exchange IIS process.\n\nExchange components (EWS, ECP, OWA, ExchangePowerShell, ActiveSync, MAPI, etc.) text-based logs:\n %SystemDrive%\Program Files\Microsoft\Exchange Server\V15\Logging and TransportRoles folders.'
last_updated: 2024-06-09
sidebar: sidebar
permalink: microsoft_exchange.html
folder: windows
---

### Overview

`Microsoft Exchange` is a complex mail server ecosystem, running exclusively on
the `Windows` operating system. `Microsoft Exchange` logs telemetry and
events in various text-based log files (100+) and `EVTX` event logs (40+).

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
    artefacts such as the [`MFT`](./../windows/ntfs_mft.md) or
    [`UsrJnrl`](./../windows/ntfs_usnjrnl.md))

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

### Exchange components text-based logs

#### Overview

`Exchange` default install folder: `%SystemDrive%\Program Files\Microsoft\Exchange Server\V15\`.

| Component | Path | Description |
|-----------|------|-------------|
| `Exchange AgentLog` | \<EXCHANGE_FOLDER\>\TransportRoles\Logs\FrontEnd\AgentLog\AgentLog\*.log | `AgentLog` logs contain the actions performed on a message by specific [`Exchange` anti-spam agents](https://learn.microsoft.com/en-us/exchange/anti-spam-agent-logging-exchange-2013-help). |
| `Exchange Autodiscover HttpProxy` | \<EXCHANGE_FOLDER\>\Logging\HttpProxy\Autodiscover\HttpProxy_\*.log | `Autodiscover` logs contain client interactions to the [`Autodiscover` service](https://learn.microsoft.com/en-us/exchange/architecture/client-access/autodiscover), which is used by clients to retrieve information on the `Exchange` infrastructure. `Autodiscover` is typically used to find the `EWS` endpoint `URL` for `Exchange Web Services (EWS)` clients. |
| `Exchange CosmosQueue` | \<EXCHANGE_FOLDER\>\Logging\CosmosQueue\audit\*.LOG | `CosmosQueue` logs contain some administration actions performed, such as `New-Mailbox` or `Set-Mailbox`, with the account that performed the operation. |
| `Exchange ActiveSync (EAS) HttpProxy` | \<EXCHANGE_FOLDER\>\Logging\HttpProxy\Eas\HttpProxy_\*.log | `Exchange ActiveSync` logs contain client interactions through the [`Exchange ActiveSync`](https://learn.microsoft.com/en-us/exchange/clients/exchange-activesync/exchange-activesync) protocol. `Exchange ActiveSync` is an `Exchange` synchronization protocol design to let mobile devices access their mailbox items (such as email messages, meetings, and contacts). |
| `Exchange Control Panel (ECP)` | \<EXCHANGE_FOLDER\>\Logging\ECP\Activity\ECPActivity_\*.LOG <br><br> \<EXCHANGE_FOLDER\>\Logging\ECP\Server\ECPServer\*.LOG <br><br> \<EXCHANGE_FOLDER\>\Logging\HttpProxy\Ecp\HttpProxy_\*.log | `Exchange Control Panel (ECP)` logs contain client interactions with the web-based management console [`Exchange Control Panel (ECP)` interface](https://learn.microsoft.com/en-us/exchange/architecture/client-access/exchange-admin-center?view=exchserver-2019). The `ECP`, accessible at `https://<EXCHANGE_SERVER>/ecp`, can be used to administer `Exchange` components (management of mailboxes, creation of policies to manage mail traffic, etc.). |
| `Exchange Web Services (EWS)` API | \<EXCHANGE_FOLDER\>\Logging\Ews\Ews_\*.log <br><br> \<EXCHANGE_FOLDER\>\Logging\HttpProxy\Ews\HttpProxy_\*.log | `Exchange Web Services` logs contain client interactions with the [`EWS` API](https://learn.microsoft.com/en-us/exchange/client-developer/exchange-web-services/ews-applications-and-the-exchange-architecture), which provides access to mailbox items (such as email messages, meetings, and contacts). The `EWS` API is notably used by the cmdlets of the `ExchangePowerShell` module. |
| `Exchange Messaging Application Programming Interface (MAPI)` | \<EXCHANGE_FOLDER\>\Logging\HttpProxy\Mapi\HttpProxy_\*.log | [`Exchange Messaging Application Programming Interface (MAPI)`](https://learn.microsoft.com/en-us/exchange/clients/mapi-over-http/mapi-over-http) over `HTTP` `HttpProxy` logs. The `MAPI` transport protocol is notably used by the `Outlook` client to access and synchronize locally mailboxes data. |
| `Exchange Message Tracking` | \<EXCHANGE_FOLDER\>\TransportRoles\Logs\MessageTracking\MSG\*.LOG | [`Exchange Message Tracking` logs](https://learn.microsoft.com/en-us/Exchange/mail-flow/transport-logs/message-tracking) contain detailed information about the history of each email message as it travels through an `Exchange` server, including the emails sender and recipient addresses and subject. |
| `Exchange Offline Address Book (OAB)` | \<EXCHANGE_FOLDER\>\Logging\HttpProxy\Oab\HttpProxy_\*.log | `Exchange Offline Address Book (OAB)` logs contain access to the `OAB` feature, which allow `Outlook` users to cache the contents of the `Global Address List (GAL)` (to access it even when no longer connected to an `Exchange` server). |
| `Exchange Outlook Web Access (OWA)` | \<EXCHANGE_FOLDER\>\Logging\HttpProxy\Owa\HttpProxy_\*.log | `Exchange Outlook Web Access (OWA)` logs contain clients interactions with the `Outlook Web Access (OWA)` webmail interface, that can be used by end-users to access and manage their mailbox (reading, sending, and deletion of emails, creation of inbox rules, etc.). |
| `PowerShell CmdletInfra` and `HttpProxy` | \<EXCHANGE_FOLDER\>\Logging\CmdletInfra\LocalPowerShell\Cmdlet\\* <br><br> \<EXCHANGE_FOLDER\>\Logging\CmdletInfra\Powershell-Proxy\Cmdlet\\* <br><br> \<EXCHANGE_FOLDER\>\Logging\HttpProxy\PowerShell\HttpProxy_\*.log | `PowerShell CmdletInfra` and `HttpProxy` logs contain the `ExchangePowerShell` cmdlets executed, and can be an alternative to the `MSExchange Management.evtx` event logs. <br><br> Information of interest: <br> - The PowerShell `cmdlet` used, with the associated parameters. <br> - The user that executed the cmdlet. |
| `Exchange RPC over HTTP` | \<EXCHANGE_FOLDER\>\Logging\HttpProxy\RpcHttp\HttpProxy_\*.log | `Exchange RPC over HTTP` log contains requests over the `RpcHttp` transport protocol, which is / was notably used by the `Outlook` client for mailboxes access. The `RpcHttp` transport protocol has been replaced by the `MAPI over HTTP` protocol. |
| `Exchange SmtpReceive` <br><br> `Exchange SmtpSend` | \<EXCHANGE_FOLDER\>\TransportRoles\Logs\FrontEnd\ProtocolLog\SmtpReceive\RECV\*.LOG <br><br> \<EXCHANGE_FOLDER\>\TransportRoles\Logs\FrontEnd\ProtocolLog\SmtpSend\SEND\*.LOG | `Exchange SMTP Receive` and `Exchange SMTP Send Protocol` protocols logs contain, respectively, the inbound and outbound `SMTP` requests. |

#### Examples of (some) Exchange components logs

{% include note.html content="Exchange AgentLog logs" %}

Example log filename: `AgentLog20220913-1.LOG`.

```
#Software: Microsoft Exchange Server
#Version: 15.0.0.0
#Log-type: Agent Log
#Date: 2022-09-13T14:38:59.177Z
#Fields: Timestamp,SessionId,LocalEndpoint,RemoteEndpoint,EnteredOrgFromIP,MessageId,P1FromAddress,P2FromAddresses,Recipient,NumRecipients,Agent,Event,Action,SmtpResponse,Reason,ReasonData,Diagnostics,NetworkMsgID,TenantID,Directionality
2022-09-13T14:38:59.177Z,1234DADADA795F2,10.10.10.10,95.11.22.33:46904,95.11.22.33,,user.name@domain.com,user.name@domain.com;,user.name2@domain2.com,1,Inbound Trust Agent,OnEndOfHeaders,ModifyHeaders,,,,Cross premises headers filtered,12345678-3232-2323-1111-12345679987,,Incoming
2022-09-13T14:51:57.936Z,1234DADADA79623,10.10.10.10,95.11.22.33:56804,95.11.22.33,,user.name@domain.com,user.name@domain.com;,user.name2@domain2.com,1,Inbound Trust Agent,OnEndOfHeaders,ModifyHeaders,,,,Cross premises headers filtered,12345678-3232-2323-1111-98765465465,,Incoming
```

{% include note.html content="Exchange Autodiscover HttpProxy logs" %}

Example log filename: `Autodiscover\HttpProxy_2023010919-1.LOG`.

```
#Software: Microsoft Exchange Server
#Version: 15.01.2375.007
#Log-type: HttpProxy Logs
#Date: 2023-01-09T19:00:00.130Z
#Fields: DateTime,RequestId,MajorVersion,MinorVersion,BuildVersion,RevisionVersion,ClientRequestId,Protocol,UrlHost,UrlStem,ProtocolAction,AuthenticationType,IsAuthenticated,AuthenticatedUser,Organization,AnchorMailbox,UserAgent,ClientIpAddress,ServerHostName,HttpStatus,BackEndStatus,ErrorCode,Method,ProxyAction,TargetServer,TargetServerVersion,RoutingType,RoutingHint,BackEndCookie,ServerLocatorHost,ServerLocatorLatency,RequestBytes,ResponseBytes,TargetOutstandingRequests,AuthModulePerfContext,HttpPipelineLatency,CalculateTargetBackEndLatency,GlsLatencyBreakup,TotalGlsLatency,AccountForestLatencyBreakup,TotalAccountForestLatency,ResourceForestLatencyBreakup,TotalResourceForestLatency,ADLatency,SharedCacheLatencyBreakup,TotalSharedCacheLatency,ActivityContextLifeTime,ModuleToHandlerSwitchingLatency,ClientReqStreamLatency,BackendReqInitLatency,BackendReqStreamLatency,BackendProcessingLatency,BackendRespInitLatency,BackendRespStreamLatency,ClientRespStreamLatency,KerberosAuthHeaderLatency,HandlerCompletionLatency,RequestHandlerLatency,HandlerToModuleSwitchingLatency,ProxyTime,CoreLatency,RoutingLatency,HttpProxyOverhead,TotalRequestTime,RouteRefresherLatency,UrlQuery,BackEndGenericInfo,GenericInfo,GenericErrors,EdgeTraceId,DatabaseGuid,UserADObjectGuid,PartitionEndpointLookupLatency,RoutingStatus
2023-01-09T19:00:00.130Z,5e58f5e5-1ddc-48d3-82c6-0199bc121640,15,1,2375,7,,Autodiscover,autodiscover.domain.com,/autodiscover/autodiscover.xml,,NTLM,true,DOMAIN\username,,Sid~S-1-5-21-123456789-9876543211-123456798-4343,AppleExchangeWebServices/818.120.2 accountsd/113,95.1.2.3,SERVER-HOSTNAME,200,200,,POST,Proxy,SERVER-HOSTNAME.domain.test,15.01.2375.000,IntraForest,WindowsIdentity,Database~fb7adba2-51f4-47e3-b170-fd39f85aaf35~~2023-02-08T18:59:59~domain.test~1,,,336,327,,,0,0,,0,,0,,0,0,,0,13,0,1,0,0,11,0,0,0,0,0,13,0,11,1,1,1,13,,,,BeginRequest=2023-01-09T19:00:00.116Z;CorrelationID=<empty>;ProxyState-Run=None;RandomBE=SERVER-HOSTNAME.domain.test~1942063431;FEAuth=BEVersion-1942063431;BeginGetRequestStream=2023-01-09T19:00:00.117Z;OnRequestStreamReady=2023-01-09T19:00:00.118Z;BeginGetResponse=2023-01-09T19:00:00.118Z;OnResponseReady=2023-01-09T19:00:00.129Z;EndGetResponse=2023-01-09T19:00:00.129Z;ProxyState-Complete=ProxyResponseData;SharedCacheGuard=0;EndRequest=2023-01-09T19:00:00.130Z;,,,|RoutingDB:fb7adba2-51f4-47e3-b170-fd39f85aaf35|RoutingDB:fb7adba2-51f4-47e3-b170-fd39f85aaf35,,,CafeV1
```

{% include note.html content="Exchange CosmosQueue logs" %}

Example log filename: `audit20230111-1.LOG`.

```
#Software: Microsoft Exchange
#Version: 15.01.2375.007
#Log-type: audit
#Date: 2023-01-11T15:56:33.189Z
#Fields: Timestamp,Server,TenantId,RecordType,Data,UserKey,RecordId,Operation,Workload,ResultStatus,Version,Scope
2023-01-11T15:56:33.189Z,SERVER-HOSTNAME,00000000-0000-0000-0000-000000000000,ExchangeAdmin,"{""CreationTime"":""2023-01-11T16:56:11"",""Id"":""12345678-1234-1234-123456789987654"",""Operation"":""Set-Mailbox"",""OrganizationId"":""00000000-0000-0000-0000-000000000000"",""RecordType"":1,""ResultStatus"":""True"",""UserKey"":""AUTORITE NT\\Système (MSExchangeHMHost)"",""UserType"":3,""Version"":1,""Workload"":""Exchange"",""ObjectId"":""DOMAIN.TEST\/Microsoft Exchange System Objects\/Monitoring Mailboxes\/HealthMailboxXXX"",""UserId"":""AUTORITE NT\\Système (MSExchangeHMHost)"",""ExternalAccess"":false,""OrganizationName"":""First Org"",""OriginatingServer"":""SERVER-HOSTNAME (15.01.2375.007)"",""Parameters"":[{""Name"":""Identity"",""Value"":""DOMAIN.TEST\/Microsoft Exchange System Objects\/Monitoring Mailboxes\/HealthMailboxXXX""},{""Name"":""Password"",""Value"":""<Secure Information Omitted>""}]}",AUTORITE NT\Système (MSExchangeHMHost),12345678-1234-1234-123456789987654,Set-Mailbox,Exchange,True,1,Online
```

{% include note.html content="Exchange ActiveSync (EAS) HttpProxy logs" %}

Example log filename: `Eas\HttpProxy_2023011116-1.LOG`.

```
#Software: Microsoft Exchange Server
#Version: 15.01.2375.007
#Log-type: HttpProxy Logs
#Date: 2023-01-11T16:00:11.478Z
#Fields: DateTime,RequestId,MajorVersion,MinorVersion,BuildVersion,RevisionVersion,ClientRequestId,Protocol,UrlHost,UrlStem,ProtocolAction,AuthenticationType,IsAuthenticated,AuthenticatedUser,Organization,AnchorMailbox,UserAgent,ClientIpAddress,ServerHostName,HttpStatus,BackEndStatus,ErrorCode,Method,ProxyAction,TargetServer,TargetServerVersion,RoutingType,RoutingHint,BackEndCookie,ServerLocatorHost,ServerLocatorLatency,RequestBytes,ResponseBytes,TargetOutstandingRequests,AuthModulePerfContext,HttpPipelineLatency,CalculateTargetBackEndLatency,GlsLatencyBreakup,TotalGlsLatency,AccountForestLatencyBreakup,TotalAccountForestLatency,ResourceForestLatencyBreakup,TotalResourceForestLatency,ADLatency,SharedCacheLatencyBreakup,TotalSharedCacheLatency,ActivityContextLifeTime,ModuleToHandlerSwitchingLatency,ClientReqStreamLatency,BackendReqInitLatency,BackendReqStreamLatency,BackendProcessingLatency,BackendRespInitLatency,BackendRespStreamLatency,ClientRespStreamLatency,KerberosAuthHeaderLatency,HandlerCompletionLatency,RequestHandlerLatency,HandlerToModuleSwitchingLatency,ProxyTime,CoreLatency,RoutingLatency,HttpProxyOverhead,TotalRequestTime,RouteRefresherLatency,UrlQuery,BackEndGenericInfo,GenericInfo,GenericErrors,EdgeTraceId,DatabaseGuid,UserADObjectGuid,PartitionEndpointLookupLatency,RoutingStatus
2023-01-11T16:06:06.997Z,823d6c00-01f1-4067-9002-18ff1cf2e5d9,15,1,2375,7,,Eas,webmail.domain.com,/Microsoft-Server-ActiveSync/default.eas,,Basic,true,DOMAIN\username,,Sid~S-1-5-21-123456789-9876543211-123456798-4343,Outlook-iOS-Android/1.0,20.10.20.30,SERVER-HOSTNAME,403,403,,OPTIONS,Proxy,SERVER-HOSTNAME.domain.test,15.01.2375.000,IntraForest,WindowsIdentity-NoDatabase,,,,0,5278,,,30,7,,0,2;,2,,0,2,,0,19068,1,,,,19027,0,0,1,0,0,19037,0,19028,10,10,40,19068,,?Cmd=Options&User=domain.test%5Cusername&DeviceId=1234567987987987987987987987987&DeviceType=Outlook,,BeginRequest=2023-01-11T16:05:47.929Z;CorrelationID=<empty>;ProxyState-Run=None;AccountForestGuard_domain.test=1;FEAuth=BEVersion-1942063431;RoutingEntry=DatabaseGuid:fb7adba2-51f4-47e3-b170-fd39f85aaf35%40domain.test%40domain.test Server:SERVER-HOSTNAME.domain.test+1942063431@638070367506148886;NewConnection=fe80::1234:5678:62ef:245e%3&0;BeginGetResponse=2023-01-11T16:05:47.970Z;OnResponseReady=2023-01-11T16:06:06.996Z;EndGetResponse=2023-01-11T16:06:06.996Z;ProxyState-Complete=ProxyResponseData;SharedCacheGuard=0;EndRequest=2023-01-11T16:06:06.997Z;I32:ATE.C[DC-HOSTNAME.domain.test]=2;F:ATE.AL[DC-HOSTNAME.domain.test]=0.5;I32:ADS.C[DC-HOSTNAME]=2;F:ADS.AL[DC-HOSTNAME]=1.700816,WebExceptionStatus=ProtocolError;ResponseStatusCode=403;WebException=System.Net.WebException: Le serveur distant a retourné une erreur : (403) Interdit.    à System.Net.HttpWebRequest.EndGetResponse(IAsyncResult asyncResult)    à Microsoft.Exchange.HttpProxy.ProxyRequestHandler.<>c__DisplayClass197_0.<OnResponseReady>b__0();,,|RoutingDB:fb7adba2-51f4-47e3-b170-fd39f85aaf35,,,CafeV1
```

{% include note.html content="Exchange Control Panel (ECP)` Activity logs" %}

Example log filename: `ECPActivity_11500_20230111-1.LOG`.

```
#Software: Microsoft Exchange Server
#Version: 15.0.0.0
#Log-type: ECP Activity Context Log
#Date: 2023-01-10T00:02:11.539Z
#Fields: TimeStamp,ServerName,EventId,EventData
2023-01-10T00:52:12.021Z,SERVER-HOSTNAME,Request,S:PSA=<PII>HealthMailboxXXX@domain.com</PII>;S:URL=https://localhost:444/ecp/RulesEditor/InboxRules.svc/GetList?msExchEcpCanary=CANARY-fk60n05Eauw.;S:Bld=15.1.2375.7;S:ActID=12345678-1234-1234-123456789987654;Dbl:MAPI.T[SERVER-HOSTNAME.44445678-1234-1234-123456789987654]=3;Dbl:BudgUse.T[]=12.0008001327515;I32:ADS.C[DC-HOSTNAME]=1;F:ADS.AL[DC-HOSTNAME]=1.692039;I32:ADR.C[DC-HOSTNAME]=2;F:ADR.AL[DC-HOSTNAME]=0.5463775;Dbl:EXR.T[SERVER-HOSTNAME.44445678-1234-1234-123456789987654]=1;I32:ROP.C[SERVER-HOSTNAME.44445678-1234-1234-123456789987654]=13654639;I32:MAPI.C[SERVER-HOSTNAME.44445678-1234-1234-123456789987654]=9;I32:RPC.C[SERVER-HOSTNAME.44445678-1234-1234-123456789987654]=4;I32:ATE.C[DC-HOSTNAME.DOMAIN.TEST]=2;F:ATE.AL[DC-HOSTNAME.DOMAIN.TEST]=0;Dbl:RPC.T[SERVER-HOSTNAME.44445678-1234-1234-123456789987654]=3;Dbl:VCGS.T[SERVER-HOSTNAME]=0;I32:MB.C[SERVER-HOSTNAME.44445678-1234-1234-123456789987654]=4;F:MB.AL[SERVER-HOSTNAME.44445678-1234-1234-123456789987654]=0.75;I32:VCGS.C[SERVER-HOSTNAME]=1;S:WLM.Bal=2.147484E+09;Dbl:WLM.TS=16
2023-01-10T00:47:12.020Z,SERVER-HOSTNAME,Request,S:URL=https://localhost:444/ecp/logoff.aspx;S:Bld=15.1.2375.7;S:ActID=555555678-1234-1234-123456789987654;Dbl:WLM.TS=0
```

{% include note.html content="Exchange Control Panel (ECP)` Server logs" %}

Example log filename: `ECPServer20230111-1.LOG`.

```
#Software: Microsoft Exchange Server
#Version: 15.0.0.0
#Log-type: ECP Server Log
#Date: 2023-01-11T00:02:33.938Z
#Fields: TimeStamp,ServerName,EventId,EventData
2023-01-11T00:02:33.966Z,SERVER-HOSTNAME,ECP.Request,S:TIME=13;S:SID=666655678-1234-1234-123456789987654;S:CMD=Get-InboxRule;S:REQID=;S:URL=/ecp/RulesEditor/InboxRules.svc/GetList?msExchEcpCanary=CANARY;S:REFERRER=;S:EX=;S:ACTID=775555678-1234-1234-123456789987654;S:RS=0;S:BLD=15.1.2375.7;S:TNAME=<null>;S:TID=;S:USID=88555678-1234-1234-123456789987654;S:EDOID=;S:ACID=
```

{% include note.html content="Exchange Control Panel (ECP) HttpProxy logs" %}

Example log filename: `Ecp\HttpProxy_2023010611-1.LOG`.

```
#Software: Microsoft Exchange Server
#Version: 15.01.2375.007
#Log-type: HttpProxy Logs
#Date: 2023-01-06T11:00:05.327Z
#Fields: DateTime,RequestId,MajorVersion,MinorVersion,BuildVersion,RevisionVersion,ClientRequestId,Protocol,UrlHost,UrlStem,ProtocolAction,AuthenticationType,IsAuthenticated,AuthenticatedUser,Organization,AnchorMailbox,UserAgent,ClientIpAddress,ServerHostName,HttpStatus,BackEndStatus,ErrorCode,Method,ProxyAction,TargetServer,TargetServerVersion,RoutingType,RoutingHint,BackEndCookie,ServerLocatorHost,ServerLocatorLatency,RequestBytes,ResponseBytes,TargetOutstandingRequests,AuthModulePerfContext,HttpPipelineLatency,CalculateTargetBackEndLatency,GlsLatencyBreakup,TotalGlsLatency,AccountForestLatencyBreakup,TotalAccountForestLatency,ResourceForestLatencyBreakup,TotalResourceForestLatency,ADLatency,SharedCacheLatencyBreakup,TotalSharedCacheLatency,ActivityContextLifeTime,ModuleToHandlerSwitchingLatency,ClientReqStreamLatency,BackendReqInitLatency,BackendReqStreamLatency,BackendProcessingLatency,BackendRespInitLatency,BackendRespStreamLatency,ClientRespStreamLatency,KerberosAuthHeaderLatency,HandlerCompletionLatency,RequestHandlerLatency,HandlerToModuleSwitchingLatency,ProxyTime,CoreLatency,RoutingLatency,HttpProxyOverhead,TotalRequestTime,RouteRefresherLatency,UrlQuery,BackEndGenericInfo,GenericInfo,GenericErrors,EdgeTraceId,DatabaseGuid,UserADObjectGuid,PartitionEndpointLookupLatency,RoutingStatus
2023-01-06T11:47:32.435Z,29d1a846-41f6-4da6-9dbf-dd8a9151ce59,15,1,2375,7,,Ecp,95.1.2.3,/ecp/Current/exporttool/microsoft.exchange.ediscovery.exporttool.application,,FBA,false,,,ServerVersion~Version 15.0 (Build 0.0),Mozilla/5.0 zgrab/0.x,95.5.6.8,SERVER-HOSTNAME,200,200,,GET,Proxy,SERVER-HOSTNAME.domain.test,15.01.2375.000,IntraForest,EDiscoveryExportTool-ServerVersion,,,,0,10112,,,0,6,,0,,0,,0,0,,0,29,0,,,,21,1,0,0,,0,29,0,23,7,8,8,29,,,,BeginRequest=2023-01-06T11:47:32.406Z;CorrelationID=<empty>;ProxyState-Run=None;FEAuth=BEVersion-1942063431;Krb=UA;CT=An;NewConnection=95.1.2.3&0;BeginGetResponse=2023-01-06T11:47:32.413Z;OnResponseReady=2023-01-06T11:47:32.435Z;EndGetResponse=2023-01-06T11:47:32.435Z;ProxyState-Complete=ProxyResponseData;SharedCacheGuard=0;EndRequest=2023-01-06T11:47:32.435Z;,,,,,,CafeV1
```

{% include note.html content="Exchange Web Services (EWS) logs" %}

Example log filename: `Ews_2023010616-1.LOG`.

```
#Software: Microsoft Exchange Server
#Version: 15.01.2375.007
#Log-type: EWS Logs
#Date: 2023-01-06T16:00:06.386Z
#Fields: DateTime,RequestId,MajorVersion,MinorVersion,BuildVersion,RevisionVersion,Ring,ClientRequestId,AuthenticationType,IsAuthenticated,AuthenticatedUser,Organization,UserAgent,VersionInfo,ClientIpAddress,ServerHostName,FrontEndServer,SoapAction,HttpStatus,RequestSize,ResponseSize,ErrorCode,ImpersonatedUser,ProxyAsUser,ActAsUser,Cookie,CorrelationGuid,PrimaryOrProxyServer,TaskType,RemoteBackendCount,LocalMailboxCount,RemoteMailboxCount,LocalIdCount,RemoteIdCount,BeginBudgetConnections,EndBudgetConnections,BeginBudgetHangingConnections,EndBudgetHangingConnections,BeginBudgetAD,EndBudgetAD,BeginBudgetCAS,EndBudgetCAS,BeginBudgetRPC,EndBudgetRPC,BeginBudgetFindCount,EndBudgetFindCount,BeginBudgetSubscriptions,EndBudgetSubscriptions,MDBResource,MDBHealth,MDBHistoricalLoad,ThrottlingPolicy,ThrottlingDelay,ThrottlingRequestType,TotalDCRequestCount,TotalDCRequestLatency,TotalMBXRequestCount,TotalMBXRequestLatency,RecipientLookupLatency,ExchangePrincipalLatency,HttpPipelineLatency,CheckAccessCoreLatency,AuthModuleLatency,CallContextInitLatency,PreExecutionLatency,CoreExecutionLatency,TotalRequestTime,DetailedExchangePrincipalLatency,ClientStatistics,GenericInfo,AuthenticationErrors,GenericErrors,Puid,StartTime,ProcessId,TimeInGC,StartTotalMemory,EndTotalMemory,StartGCCounts,EndGCCounts,TokenBasedThrottlingPolicy,BudgetKey,CoinsCharged,CoinsChargedMethod,SidBudgetInfo,AppBudgetInfo,TenantBudgetInfo,ResourceAccessed,ResourceHealthBasedThreshold,ThrottledBy,BackoffHint,WorkClassification
2023-01-06T16:00:06.410Z,5399c497-d85f-4914-bc22-1ff088e33d72,15,1,2375,7,Unknown,,Logon,true,HealthMailboxXXX@domain.test,DOMAIN.TEST,Ews_AM_Probe/Local (ExchangeServicesClient/15.01.2375.007),Target=None;Req=Exchange2010_SP1/Exchange2010_SP1;,::1,SERVER_HOSTNAME,,GetFolder,200,666,,,HealthMailboxXXX@domain.test,,,92e4ad319de84a06a5baf5b123456789,03a7c763-3ad2-4fac-8313-2b63d63630b3,PrimaryServer,LocalTask,0,1,0,1,0,,,,,,,,,,,,,,,,,,,,[C],0,0,1,0,10,,1,10,,0,14,1,17,,MessageId_0=12345678-9876-451c-ada2-1e987689112c;ResponseTime_0=20;SoapAction_0=GetFolder;,SKU=Unknown;App_BeginReq_Start=0;App_BeginReq_End=0;GetHandler_Start=0;RequestHandler=Wcf;GetHandler_End=0;BackEndAuthenticator=WindowsAuthenticator;TotalBERehydrationModuleLatency=0;CSCWTI=0;ADIdentityCache=Miss;CSCWTI=0;cpn=RUM_ABR/RUM_ABRC/ABR/APAR/EWS_CE/EWS_CEC/APSRH/APRHE/RUM_AER/RUM_AERC/AER/AERC/;cpv=0/0/0/0/15/16/17/17/17/17/17/17/;MailboxTypeCacheSize=21;S:cmn=ID_CDFMS.T;S:AspDispatchLatency.BeginRequest=0;S:ADRS.InclI=1;S:cmv=0;S:AspDispatchLatency.EndRequest=0;S:ADRS.Check=00;S:ServiceTaskMetadata.WatsonReportCount=0;S:WLM.Bal=299999;S:ServiceTaskMetadata.ServiceCommandBegin=15;S:ServiceTaskMetadata.ServiceCommandEnd=16;S:ActivityStandardMetadata.Component=Ews;S:WLM.BT=Ews;S:EwsMetadata.ParticipantResolveLatency=0;S:EwsMetadata.HttpHandlerGetterLatency=0;Dbl:WLM.TS=17;Dbl:MAPI.T[DOMAIN.TEST.fb7adba2-51f4-47e3-b170-fd39f85aaf35]=0;Dbl:BudgUse.T[]=2.00119996070862;I32:ADS.C[DOMAIN.TEST]=1;F:ADS.AL[DOMAIN.TEST]=5.46475;Dbl:CCpu.T[CMD]=0;I32:ROP.C[SERVER-HOSTNAME.fb7adba2-51f4-47e3-b170-fd39f85aaf35]=2898759;I32:MAPI.C[SERVER-HOSTNAME.fb7adba2-51f4-47e3-b170-fd39f85aaf35]=2;I32:RPC.C[SERVER-HOSTNAME.fb7adba2-51f4-47e3-b170-fd39f85aaf35]=1;I32:ATE.C[DOMAIN.TEST.domain.test]=1;F:ATE.AL[DOMAIN.TEST.domain.test]=1;I32:MB.C[SERVER-HOSTNAME.fb7adba2-51f4-47e3-b170-fd39f85aaf35]=1;F:MB.AL[SERVER-HOSTNAME.fb7adba2-51f4-47e3-b170-fd39f85aaf35]=0,,,,2023-01-06T16:00:06.387Z,9716,,137332048,139253608,4038_450_53,4038_450_53,,,,,,,,,,,,
```

{% include note.html content="Exchange Web Services (EWS) HttpProxy logs" %}

Example log filename: `Ews\HttpProxy_2023010601-1.LOG`.

```
#Software: Microsoft Exchange Server
#Version: 15.01.2375.007
#Log-type: HttpProxy Logs
#Date: 2023-01-06T01:00:05.345Z
#Fields: DateTime,RequestId,MajorVersion,MinorVersion,BuildVersion,RevisionVersion,ClientRequestId,Protocol,UrlHost,UrlStem,ProtocolAction,AuthenticationType,IsAuthenticated,AuthenticatedUser,Organization,AnchorMailbox,UserAgent,ClientIpAddress,ServerHostName,HttpStatus,BackEndStatus,ErrorCode,Method,ProxyAction,TargetServer,TargetServerVersion,RoutingType,RoutingHint,BackEndCookie,ServerLocatorHost,ServerLocatorLatency,RequestBytes,ResponseBytes,TargetOutstandingRequests,AuthModulePerfContext,HttpPipelineLatency,CalculateTargetBackEndLatency,GlsLatencyBreakup,TotalGlsLatency,AccountForestLatencyBreakup,TotalAccountForestLatency,ResourceForestLatencyBreakup,TotalResourceForestLatency,ADLatency,SharedCacheLatencyBreakup,TotalSharedCacheLatency,ActivityContextLifeTime,ModuleToHandlerSwitchingLatency,ClientReqStreamLatency,BackendReqInitLatency,BackendReqStreamLatency,BackendProcessingLatency,BackendRespInitLatency,BackendRespStreamLatency,ClientRespStreamLatency,KerberosAuthHeaderLatency,HandlerCompletionLatency,RequestHandlerLatency,HandlerToModuleSwitchingLatency,ProxyTime,CoreLatency,RoutingLatency,HttpProxyOverhead,TotalRequestTime,RouteRefresherLatency,UrlQuery,BackEndGenericInfo,GenericInfo,GenericErrors,EdgeTraceId,DatabaseGuid,UserADObjectGuid,PartitionEndpointLookupLatency,RoutingStatus
2023-01-06T01:00:05.344Z,3656c386-6be1-4301-865a-27c1f99599bd,15,1,2375,7,,Ews,localhost,/ews/,,Negotiate,true,DOMAIN\HealthMailboxXXX,,Sid~S-1-5-21-123456789-9876543211-123456798-1142,AMProbe/Local/ClientAccess,127.0.0.1,SERVER_HOSTNAME,200,,,GET,Proxy,server_hostname.domain.test,15.01.2375.000,IntraForest,WindowsIdentity,,,,0,,,,0,1,,0,,0,,0,0,,0,1,0,,,,,,,,,0,1,0,,1,,1,1,,,,BeginRequest=2023-01-06T01:00:05.343Z;CorrelationID=<empty>;ProxyState-Run=None;FEAuth=BEVersion-1942063431;ProxyState-Complete=PrepareServerRequest;SharedCacheGuard=0;EndRequest=2023-01-06T01:00:05.344Z;,,,|RoutingDB:fb7adba2-51f4-47e3-b170-fd39f85aaf35,,,CafeV1
```

{% include note.html content="Exchange Messaging Application Programming Interface (MAPI) HttpProxy logs" %}

Example log filename: `Mapi\HttpProxy_2023010601-1.LOG`.

```
#Software: Microsoft Exchange Server
#Version: 15.01.2375.007
#Log-type: HttpProxy Logs
#Date: 2023-01-06T01:00:04.124Z
#Fields: DateTime,RequestId,MajorVersion,MinorVersion,BuildVersion,RevisionVersion,ClientRequestId,Protocol,UrlHost,UrlStem,ProtocolAction,AuthenticationType,IsAuthenticated,AuthenticatedUser,Organization,AnchorMailbox,UserAgent,ClientIpAddress,ServerHostName,HttpStatus,BackEndStatus,ErrorCode,Method,ProxyAction,TargetServer,TargetServerVersion,RoutingType,RoutingHint,BackEndCookie,ServerLocatorHost,ServerLocatorLatency,RequestBytes,ResponseBytes,TargetOutstandingRequests,AuthModulePerfContext,HttpPipelineLatency,CalculateTargetBackEndLatency,GlsLatencyBreakup,TotalGlsLatency,AccountForestLatencyBreakup,TotalAccountForestLatency,ResourceForestLatencyBreakup,TotalResourceForestLatency,ADLatency,SharedCacheLatencyBreakup,TotalSharedCacheLatency,ActivityContextLifeTime,ModuleToHandlerSwitchingLatency,ClientReqStreamLatency,BackendReqInitLatency,BackendReqStreamLatency,BackendProcessingLatency,BackendRespInitLatency,BackendRespStreamLatency,ClientRespStreamLatency,KerberosAuthHeaderLatency,HandlerCompletionLatency,RequestHandlerLatency,HandlerToModuleSwitchingLatency,ProxyTime,CoreLatency,RoutingLatency,HttpProxyOverhead,TotalRequestTime,RouteRefresherLatency,UrlQuery,BackEndGenericInfo,GenericInfo,GenericErrors,EdgeTraceId,DatabaseGuid,UserADObjectGuid,PartitionEndpointLookupLatency,RoutingStatus
2023-01-06T01:00:04.124Z,3fe8b64c-9d09-495d-9123-c7759ae9bbfb,15,1,2375,7,R:1b7dabb2-cd5b-4ce0-a65d-b25915e598cf:1;RT:Connect;CI:97c0f33d-7e97-4147-9742-ee1e9a3cc2f4:1;CID:82392570-79d9-402a-8c7c-fbf14bcde8fc,Mapi,SERVER_HOSTNAME.domain.test,/mapi/emsmdb/,,Negotiate,true,DOMAIN\HealthMailboxXXX,domain.test,MailboxGuid~1234567-4444-1234-4567-123456789987,MapiHttpClient,fe80::1234:5678:123abc:245e%3,SERVER_HOSTNAME,200,200,,POST,Proxy,SERVER_HOSTNAME.domain.test,15.01.2375.000,IntraForest,MailboxGuidWithDomain,,,,371,443,,,0,1,,0,,0,,0,0,,0,16,0,0,1,1,11,0,0,0,0,0,16,0,12,3,4,4,16,,?mailboxId=1234567-4444-1234-4567-123456789987@domain.test,,BeginRequest=2023-01-06T01:00:04.107Z;CorrelationID=<empty>;ProxyState-Run=None;FEAuth=BEVersion-1942063431;RoutingEntry=DatabaseGuid:fb7adba2-51f4-47e3-b170-fd39f85aaf35%40domain.test%40domain.test Server:SERVER_HOSTNAME.domain.test+1942063431@638070367506148886;NewConnection=fe80::1234:5678:123abc:245e%3&0;BeginGetRequestStream=2023-01-06T01:00:04.110Z;OnRequestStreamReady=2023-01-06T01:00:04.111Z;BeginGetResponse=2023-01-06T01:00:04.111Z;OnResponseReady=2023-01-06T01:00:04.123Z;EndGetResponse=2023-01-06T01:00:04.123Z;ProxyState-Complete=ProxyResponseData;SharedCacheGuard=0;EndRequest=2023-01-06T01:00:04.124Z;,,,|RoutingDB:fb7adba2-51f4-47e3-b170-fd39f85aaf35,,,CafeV1
2023-01-06T01:00:04.141Z,9b241890-db8d-44c5-a50e-ccdc753cab92,15,1,2375,7,R:1b7dabb2-cd5b-4ce0-a65d-b25915e598cf:2;RT:Execute;CI:97c0f33d-7e97-4147-9742-ee1e9a3cc2f4:1;CID:45534129-271f-4423-bae6-e038600d6c36,Mapi,SERVER_HOSTNAME.domain.test,/mapi/emsmdb/,,Negotiate,true,DOMAIN\HealthMailboxXXX,domain.test,MailboxGuid~1234567-4444-1234-4567-123456789987,MapiHttpClient,10.128.53.11,SERVER_HOSTNAME,200,200,,POST,Proxy,SERVER_HOSTNAME.domain.test,15.01.2375.000,IntraForest,MailboxGuidWithDomain,Database~fb7adba2-51f4-47e3-b170-fd39f85aaf35~~2023-02-05T01:00:04,,,273,295,,,0,0,,0,,0,,0,0,,0,12,0,0,1,0,9,0,0,0,0,0,11,1,9,2,3,3,12,,?mailboxId=1234567-4444-1234-4567-123456789987@domain.test,,BeginRequest=2023-01-06T01:00:04.129Z;CorrelationID=<empty>;ProxyState-Run=None;FEAuth=BEVersion-1942063431;BeginGetRequestStream=2023-01-06T01:00:04.131Z;OnRequestStreamReady=2023-01-06T01:00:04.131Z;BeginGetResponse=2023-01-06T01:00:04.132Z;OnResponseReady=2023-01-06T01:00:04.141Z;EndGetResponse=2023-01-06T01:00:04.141Z;ProxyState-Complete=ProxyResponseData;SharedCacheGuard=0;EndRequest=2023-01-06T01:00:04.141Z;,,,|RoutingDB:fb7adba2-51f4-47e3-b170-fd39f85aaf35,,,CafeV1
2023-01-06T01:00:04.152Z,7119d1a3-0b37-4261-a966-39f54264937e,15,1,2375,7,R:1b7dabb2-cd5b-4ce0-a65d-b25915e598cf:3;RT:Disconnect;CI:97c0f33d-7e97-4147-9742-ee1e9a3cc2f4:1;CID:50d53854-a8f7-439e-b5ea-018f74812d10,Mapi,SERVER_HOSTNAME.domain.test,/mapi/emsmdb/,,Negotiate,true,DOMAIN\HealthMailboxXXX,domain.test,MailboxGuid~1234567-4444-1234-4567-123456789987,MapiHttpClient,fe80::1234:5678:123abc:245e%3,SERVER_HOSTNAME,200,200,,POST,Proxy,SERVER_HOSTNAME.domain.test,15.01.2375.000,IntraForest,MailboxGuidWithDomain,Database~fb7adba2-51f4-47e3-b170-fd39f85aaf35~~2023-02-05T01:00:04,,,4,94,,,0,0,,0,,0,,0,0,,0,6,0,0,1,0,3,0,0,1,0,0,6,0,4,1,2,2,6,,?mailboxId=1234567-4444-1234-4567-123456789987@domain.test,,BeginRequest=2023-01-06T01:00:04.146Z;CorrelationID=<empty>;ProxyState-Run=None;FEAuth=BEVersion-1942063431;BeginGetRequestStream=2023-01-06T01:00:04.148Z;OnRequestStreamReady=2023-01-06T01:00:04.148Z;BeginGetResponse=2023-01-06T01:00:04.149Z;OnResponseReady=2023-01-06T01:00:04.152Z;EndGetResponse=2023-01-06T01:00:04.152Z;ProxyState-Complete=ProxyResponseData;SharedCacheGuard=0;EndRequest=2023-01-06T01:00:04.152Z;,,,|RoutingDB:fb7adba2-51f4-47e3-b170-fd39f85aaf35,,,CafeV1
```

{% include note.html content="Exchange Message Tracking logs" %}

Example log filename: `MSGTRKMS2022091915-1.LOG`.

```
#Software: Microsoft Exchange Server
#Version: 15.01.2375.007
#Log-type: Message Tracking Log
#Date: 2022-09-19T15:02:29.339Z
#Fields: date-time,client-ip,client-hostname,server-ip,server-hostname,source-context,connector-id,source,event-id,internal-message-id,message-id,network-message-id,recipient-address,recipient-status,total-bytes,recipient-count,related-recipient-address,reference,message-subject,sender-address,return-path,message-info,directionality,tenant-id,original-client-ip,original-server-ip,custom-data,transport-traffic-type,log-id,schema-version
2022-09-19T15:02:29.339Z,fe80::1234:5678:4321:1010%3,SERVER-HOSTNAME,,,"MDB:12345678-3232-2323-1111-12345679666, Mailbox:33345678-3232-2323-1111-08da9fd00918, Event:63779422, MessageClass:IPM.Note, CreationTime:2022-09-19T15:02:29.211Z, ClientType:MOMT, SubmissionAssistant:MailboxTransportSubmissionEmailAssistant",,STOREDRIVER,NOTIFYMAPI,,,,,,,,,,,user.name1@domain.com,,2022-09-19T15:02:29.211Z;LSRV=XXX-MAIL.DOMAIN.TEST:TOTAL-SUB=0.128|SA=0.128|MTSS-PEN=0.000,,,,,S:ItemEntryId=00-XXX-00,,12345678-3232-2323-1111-12345679988,15.01.2375.007
2022-09-19T15:02:29.450Z,fe80::1234:5678:4321:1010,SERVER-HOSTNAME.DOMAIN.TEST,1234:5678:4321:1010%3,SERVER-HOSTNAME,"MDB:12345678-3232-2323-1111-12345679987, Mailbox:33345678-3232-2323-1111-08da9fd00918, Event:63779422, MessageClass:IPM.Note, CreationTime:2022-09-19T15:02:29.211Z, ClientType:MOMT, SubmissionAssistant:MailboxTransportSubmissionEmailAssistant, ServerMdbConnectionId:08DA9806C5B670C1",,STOREDRIVER,RECEIVE,438,<XXX@domain.com>,6a638d20-1e63-493a-80a2-08da9a4ff8da,user.name4@gmail.com,To,3750222,1,,,MAIL SUBJECT,user.name1@domain.com,user.name1@domain.com,04I: ,Originating,,95.11.22.33,fe80::1234:4545:3214:1010e%3,S:MailboxDatabaseGuid=12345678-3232-2323-1111-12345679987;S:ItemEntryId=00-00-XXX-00;S:DeliveryPriority=Normal;S:AccountForest=DOMAIN.TEST,Email,12345678-3232-2323-1111-12345679987,15.01.2375.007
```

{% include note.html content="Exchange Outlook Web Access (OWA) HttpProxy logs" %}

Example log filename: `Owa\HttpProxy_2023010602-1.LOG`.

```
#Software: Microsoft Exchange Server
#Version: 15.01.2375.007
#Log-type: HttpProxy Logs
#Date: 2023-01-06T02:00:17.456Z
#Fields: DateTime,RequestId,MajorVersion,MinorVersion,BuildVersion,RevisionVersion,ClientRequestId,Protocol,UrlHost,UrlStem,ProtocolAction,AuthenticationType,IsAuthenticated,AuthenticatedUser,Organization,AnchorMailbox,UserAgent,ClientIpAddress,ServerHostName,HttpStatus,BackEndStatus,ErrorCode,Method,ProxyAction,TargetServer,TargetServerVersion,RoutingType,RoutingHint,BackEndCookie,ServerLocatorHost,ServerLocatorLatency,RequestBytes,ResponseBytes,TargetOutstandingRequests,AuthModulePerfContext,HttpPipelineLatency,CalculateTargetBackEndLatency,GlsLatencyBreakup,TotalGlsLatency,AccountForestLatencyBreakup,TotalAccountForestLatency,ResourceForestLatencyBreakup,TotalResourceForestLatency,ADLatency,SharedCacheLatencyBreakup,TotalSharedCacheLatency,ActivityContextLifeTime,ModuleToHandlerSwitchingLatency,ClientReqStreamLatency,BackendReqInitLatency,BackendReqStreamLatency,BackendProcessingLatency,BackendRespInitLatency,BackendRespStreamLatency,ClientRespStreamLatency,KerberosAuthHeaderLatency,HandlerCompletionLatency,RequestHandlerLatency,HandlerToModuleSwitchingLatency,ProxyTime,CoreLatency,RoutingLatency,HttpProxyOverhead,TotalRequestTime,RouteRefresherLatency,UrlQuery,BackEndGenericInfo,GenericInfo,GenericErrors,EdgeTraceId,DatabaseGuid,UserADObjectGuid,PartitionEndpointLookupLatency,RoutingStatus
2023-01-06T02:20:32.841Z,47a25adf-57a3-4538-ba98-4bafdfee8095,15,1,2375,7,,Owa,95.1.2.3,/owa/admin.cfg,,FBA,false,,,,Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML  like Gecko) Chrome/62.0.3202.62 Safari/537.36,95.5.6.7,SERVER_HOSTNAME,302,,,GET,,,,,,,,,0,,,,,,,,,,,,,,,0,,,,,,,,,,,,,,0,,0,0,,,,BeginRequest=2023-01-06T02:20:32.840Z;CorrelationID=<empty>;SharedCacheGuard=0;NoCookies=302 - GET/E14AuthPost;EndRequest=2023-01-06T02:20:32.841Z;,,,,,,
```

{% include note.html content="PowerShell CmdletInfra (Powershell-Proxy) logs" %}

Example log filename: `Rps_Cmdlet_2023011116-1.LOG`.

```
#Software: Microsoft Exchange Server
#Version: 15.01.2375.007
#Log-type: Rps Cmdlet Logs
#Date: 2023-01-11T01:01:19.898Z
#Fields: DateTime,StartTime,RequestId,ClientRequestId,MajorVersion,MinorVersion,BuildVersion,RevisionVersion,ServerHostName,ProcessId,ProcessName,ThreadId,CultureInfo,Organization,AuthenticatedUser,ExecutingUserSid,EffectiveOrganization,UserServicePlan,IsAdmin,ClientApplication,Cmdlet,Parameters,CmdletUniqueId,UserBudgetOnStart,ContributeToFailFast,RunspaceSettingsCreationHint,ADViewEntireForest,ADRecipientViewRoot,ADConfigurationDomainControllers,ADPreferredGlobalCatalogs,ADPreferredDomainControllers,ADUserConfigurationDomainController,ADUserPreferredGlobalCatalog,ADuserPreferredDomainControllers,ThrottlingInfo,DelayInfo,ThrottlingDelay,IsOutputObjectRedacted,CmdletProxyStage,CmdletProxyRemoteServer,CmdletProxyRemoteServerVersion,CmdletProxyMethod,ProxiedObjectCount,CmdletProxyLatency,OutputObjectCount,ParameterBinding,BeginProcessing,ProcessRecord,EndProcessing,StopProcessing,BizLogic,PowerShellLatency,UserInteractionLatency,ProvisioningLayerLatency,ActivityContextLifeTime,TotalTime,ErrorType,ExecutionResult,CacheHitCount,CacheMissCount,GenericLatency,GenericInfo,GenericErrors,ObjectGuid,ExternalDirectoryOrganizationId,ExternalDirectoryObjectId,NonPiiParameters
2023-01-11T01:01:19.898Z,2023-01-11T01:01:19.882Z,7b91be61-3d94-406e-b00c-3d80e3e5f115,,15,1,2375,7,domain-MAIL,15756,w3wp#MSExchangePowerShellAppPool,85,,,HealthMailboxXXX,S-1-5-21-123456789-9876543211-123456798-4343,,,False,ActiveMonitor,Get-Mailbox,"-ResultSize ""1""",d61f0cba-945f-4857-8c62-91321f16b7b4,,,GCRandomly,False,domain.test,DC-HOSTNAME.domain.test,DC-HOSTNAME.domain.test,DC-HOSTNAME.domain.test,,,,,,,,,,,,0,,,0,6,8,0,,9,,,6,22681600,15,,Success,,,Dispose=1/11/2023 12:58:24 AM;LatencyMissed=latencyBreakDowns.Count is Zero;GetApplicationPrivateData=11/01/2023 00:59:22;WinRMDataReceiver.LoadItemsFromNamedPipe=0;WinRMDataReceiver.Ctor=0;ExchangeRunspaceConfiguration=37;ExchangeRunspaceConfiguration.C=24;PowerShellThrottlingPolicyUpdater=0;InitialSessionStateBuilder=19;GetInitialSessionStateCore=37;GetInitialSessionState=11/01/2023 00:59:22;TaskModuleLatency=4;TaskModuleLatency.C=68;ProvisioningLayerLatency.C=9;BizLogic.C=15;Cmd=15;Cmd.ADC=3;Cmd.AD=4;Cmd.ATEC=9;Cmd.ATE=1;,PSSenderInfo=domain\healthmailboxXXX;ADServerSettingsInEnd=ADViewEntireForest:False WriteShadowProperties:False WriteOriginatingChangeTimestamp:False ADRecipientViewRoot:domain.test ADConfigurationDomainControllers:DC-HOSTNAME.domain.test ADPreferredGlobalCatalogs:DC-HOSTNAME.domain.test ADPreferredDomainControllers:DC-HOSTNAME.domain.test ADUserPreferredDomainControllers: ;,,,,,"-ResultSize ""1"""
```

{% include note.html content="PowerShell HttpProxy logs" %}

Example log filename: `PowerShell\HttpProxy_2023011116-1.LOG`.

```
#Software: Microsoft Exchange Server
#Version: 15.01.2375.007
#Log-type: HttpProxy Logs
#Date: 2023-01-11T16:00:32.466Z
#Fields: DateTime,RequestId,MajorVersion,MinorVersion,BuildVersion,RevisionVersion,ClientRequestId,Protocol,UrlHost,UrlStem,ProtocolAction,AuthenticationType,IsAuthenticated,AuthenticatedUser,Organization,AnchorMailbox,UserAgent,ClientIpAddress,ServerHostName,HttpStatus,BackEndStatus,ErrorCode,Method,ProxyAction,TargetServer,TargetServerVersion,RoutingType,RoutingHint,BackEndCookie,ServerLocatorHost,ServerLocatorLatency,RequestBytes,ResponseBytes,TargetOutstandingRequests,AuthModulePerfContext,HttpPipelineLatency,CalculateTargetBackEndLatency,GlsLatencyBreakup,TotalGlsLatency,AccountForestLatencyBreakup,TotalAccountForestLatency,ResourceForestLatencyBreakup,TotalResourceForestLatency,ADLatency,SharedCacheLatencyBreakup,TotalSharedCacheLatency,ActivityContextLifeTime,ModuleToHandlerSwitchingLatency,ClientReqStreamLatency,BackendReqInitLatency,BackendReqStreamLatency,BackendProcessingLatency,BackendRespInitLatency,BackendRespStreamLatency,ClientRespStreamLatency,KerberosAuthHeaderLatency,HandlerCompletionLatency,RequestHandlerLatency,HandlerToModuleSwitchingLatency,ProxyTime,CoreLatency,RoutingLatency,HttpProxyOverhead,TotalRequestTime,RouteRefresherLatency,UrlQuery,BackEndGenericInfo,GenericInfo,GenericErrors,EdgeTraceId,DatabaseGuid,UserADObjectGuid,PartitionEndpointLookupLatency,RoutingStatus
2023-01-11T16:01:44.205Z,b78ed0f5-21b2-4cae-aad9-c8e86bdd5482,15,1,2375,7,,PowerShell,server_hostname.domain.test,/powershell,New-PSSession:Receive,Kerberos,true,DOMAIN\HealthMailboxXXX,,ServerVersion~Version 15.1 (Build 2374.0),Microsoft WinRM Client,95.1.2.3,server_hostname,200,200,,POST,Proxy,server_hostname.domain.test,15.01.2375.000,IntraForest,ExchClientVer-ServerCookie,Server~server_hostname.domain.test~1942063431~2023-01-11T16:11:44,,,2014,934,,,0,0,,0,,0,,0,0,,0,13,0,0,1,0,7,0,0,0,0,0,13,0,8,5,6,6,13,,?clientApplication=ActiveMonitor;PSVersion=5.1.14393.5582&sessionID=Version_15.1_(Build_2374.0)=NjduieGsoKEjcuiAHDiczeiIDIZOFDOPqstdGTkJyek4HOxsvNz8nMy8zOgc3PzczSz87Szs6rzsnFzs7Fy8s=,,BeginRequest=2023-01-11T16:01:44.192Z;CorrelationID=<empty>;ProxyState-Run=None;FEAuth=BEVersion-1942063431;SessionId=D5FFDBEB-C73B-4C9B-93BB-5E2AA5824904;ShellId=2C72B54F-AD1D-4778-A3F0-64514659F016;BeginGetRequestStream=2023-01-11T16:01:44.197Z;OnRequestStreamReady=2023-01-11T16:01:44.197Z;BeginGetResponse=2023-01-11T16:01:44.197Z;OnResponseReady=2023-01-11T16:01:44.205Z;EndGetResponse=2023-01-11T16:01:44.205Z;ProxyState-Complete=ProxyResponseData;OnEndRequest.ContentType=application/soap+xml charset UTF-8;SharedCacheGuard=0;EndRequest=2023-01-11T16:01:44.205Z;,,,,,,CafeV1
2023-01-11T16:01:44.238Z,6b5578b6-e7d5-4cbf-9ac5-221b8c9fee5b,15,1,2375,7,,PowerShell,server_hostname.domain.test,/powershell,Get-Mailbox:Command,Kerberos,true,DOMAIN\HealthMailboxXXX,,ServerVersion~Version 15.1 (Build 2374.0),Microsoft WinRM Client,95.1.2.3,server_hostname,200,200,,POST,Proxy,server_hostname.domain.test,15.01.2375.000,IntraForest,ExchClientVer-ServerCookie,Server~server_hostname.domain.test~1942063431~2023-01-11T16:11:44,,,5127,847,,,0,0,,0,,0,,0,0,,0,24,0,0,1,0,18,0,0,0,0,0,24,0,19,5,6,6,24,,?clientApplication=ActiveMonitor;PSVersion=5.1.14393.5582&sessionID=Version_15.1_(Build_2374.0)=NjduieGsoKEjcuiAHDiczeiIDIZOFDOPqstdGTkJyek4HOxsvNz8nMy8zOgc3PzczSz87Szs6rzsnFzs7Fy8s=,,BeginRequest=2023-01-11T16:01:44.214Z;CorrelationID=<empty>;ProxyState-Run=None;FEAuth=BEVersion-1942063431;CommandId=1DFAA9CA-3A2A-461F-B94C-40AA8915416A;SessionId=D5FFDBEB-C73B-4C9B-93BB-5E2AA5824904;ShellId=2C72B54F-AD1D-4778-A3F0-64514659F016;NewConnection=10.128.53.11&0;BeginGetRequestStream=2023-01-11T16:01:44.218Z;OnRequestStreamReady=2023-01-11T16:01:44.219Z;BeginGetResponse=2023-01-11T16:01:44.220Z;OnResponseReady=2023-01-11T16:01:44.237Z;EndGetResponse=2023-01-11T16:01:44.237Z;ProxyState-Complete=ProxyResponseData;OnEndRequest.ContentType=application/soap+xml charset UTF-8;SharedCacheGuard=0;EndRequest=2023-01-11T16:01:44.238Z;,,,,,,CafeV1
```

{% include note.html content="RpcHttp HttpProxy logs" %}

Example log filename: `RpcHttp\HttpProxy_2023010708-1.LOG`.

```
#Software: Microsoft Exchange Server
#Version: 15.01.2375.007
#Log-type: HttpProxy Logs
#Date: 2023-01-11T14:00:25.732Z
#Fields: DateTime,RequestId,MajorVersion,MinorVersion,BuildVersion,RevisionVersion,ClientRequestId,Protocol,UrlHost,UrlStem,ProtocolAction,AuthenticationType,IsAuthenticated,AuthenticatedUser,Organization,AnchorMailbox,UserAgent,ClientIpAddress,ServerHostName,HttpStatus,BackEndStatus,ErrorCode,Method,ProxyAction,TargetServer,TargetServerVersion,RoutingType,RoutingHint,BackEndCookie,ServerLocatorHost,ServerLocatorLatency,RequestBytes,ResponseBytes,TargetOutstandingRequests,AuthModulePerfContext,HttpPipelineLatency,CalculateTargetBackEndLatency,GlsLatencyBreakup,TotalGlsLatency,AccountForestLatencyBreakup,TotalAccountForestLatency,ResourceForestLatencyBreakup,TotalResourceForestLatency,ADLatency,SharedCacheLatencyBreakup,TotalSharedCacheLatency,ActivityContextLifeTime,ModuleToHandlerSwitchingLatency,ClientReqStreamLatency,BackendReqInitLatency,BackendReqStreamLatency,BackendProcessingLatency,BackendRespInitLatency,BackendRespStreamLatency,ClientRespStreamLatency,KerberosAuthHeaderLatency,HandlerCompletionLatency,RequestHandlerLatency,HandlerToModuleSwitchingLatency,ProxyTime,CoreLatency,RoutingLatency,HttpProxyOverhead,TotalRequestTime,RouteRefresherLatency,UrlQuery,BackEndGenericInfo,GenericInfo,GenericErrors,EdgeTraceId,DatabaseGuid,UserADObjectGuid,PartitionEndpointLookupLatency,RoutingStatus
2023-01-07T08:39:45.725Z,12345678-9876-1234-1234-d8c5ea0edcd3,15,1,2375,7,,RpcHttp,95.1.2.3,/rpc,,NTLM,false,,,,Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML  like Gecko) Chrome/107.0.0.0 Safari/537.36,95.5.6.7,SERVER-HOSTNAME,401,,,GET,,,,,,,,,0,,,,,,,,,,,,,,,0,,,,,,,,,,,,,,0,,0,0,,,,BeginRequest=2023-01-07T08:39:45.724Z;CorrelationID=<empty>;SharedCacheGuard=0;EndRequest=2023-01-07T08:39:45.725Z;,,,,,,
```

{% include note.html content="SmtpReceive logs" %}

Example log filename: `RECV2023011116-1.LOG`.

```
#Software: Microsoft Exchange Server
#Version: 15.0.0.0
#Log-type: SMTP Receive Protocol Log
#Date: 2023-01-11T16:00:04.113Z
#Fields: date-time,connector-id,session-id,sequence-number,local-endpoint,remote-endpoint,event,data,context
2023-01-11T15:59:51.433Z,SERVER-HOSTNAME\Default Frontend SERVER-HOSTNAME,123453EC62278461,1,10.10.10.10:25,95.10.20.33:56026,>,"220 SERVER-HOSTNAME.DOMAIN.TEST Microsoft ESMTP MAIL Service ready at Wed, 11 Jan 2023 16:59:51 +0100",
2023-01-11T15:59:53.909Z,SERVER-HOSTNAME\Default Frontend SERVER-HOSTNAME,123453EC62278461,2,10.10.10.10:25,95.10.20.33:56026,<,EHLO [95.10.20.33],
2023-01-11T15:59:53.910Z,SERVER-HOSTNAME\Default Frontend SERVER-HOSTNAME,123453EC62278461,3,10.10.10.10:25,95.10.20.33:56026,>,250  SERVER-HOSTNAME.DOMAIN.TEST Hello [95.10.20.33] SIZE 37748736 PIPELINING DSN ENHANCEDSTATUSCODES STARTTLS X-ANONYMOUSTLS AUTH NTLM X-EXPS GSSAPI NTLM 8BITMIME BINARYMIME CHUNKING XRDST,
2023-01-11T15:59:56.232Z,SERVER-HOSTNAME\Default Frontend SERVER-HOSTNAME,123453EC62278461,4,10.10.10.10:25,95.10.20.33:56026,<,STARTTLS,
2023-01-11T15:59:56.232Z,SERVER-HOSTNAME\Default Frontend SERVER-HOSTNAME,123453EC62278461,5,10.10.10.10:25,95.10.20.33:56026,>,220 2.0.0 SMTP server ready,
2023-01-11T15:59:56.232Z,SERVER-HOSTNAME\Default Frontend SERVER-HOSTNAME,123453EC62278461,6,10.10.10.10:25,95.10.20.33:56026,*, CN=SERVER-HOSTNAME CN=SERVER-HOSTNAME 12DEABC6565646512346ABDC13456894 12DEABC6565646512346ABDC1345689413589876 2020-01-17T15:15:33.000Z 2025-01-17T15:15:33.000Z SERVER-HOSTNAME;SERVER-HOSTNAME.DOMAIN.TEST,Sending certificate Subject Issuer name Serial number Thumbprint Not before Not after Subject alternate names
2023-01-11T16:00:01.010Z,SERVER-HOSTNAME\Default Frontend SERVER-HOSTNAME,123453EC62278461,7,10.10.10.10:25,95.10.20.33:56026,*,,"TLS protocol SP_PROT_TLS1_2_SERVER negotiation succeeded using bulk encryption algorithm CALG_AES_128 with strength 128 bits, MAC hash algorithm CALG_SHA_256 with strength 0 bits and key exchange algorithm CALG_ECDH_EPHEM with strength 256 bits"
2023-01-11T16:00:02.105Z,SERVER-HOSTNAME\Default Frontend SERVER-HOSTNAME,123453EC62278461,8,10.10.10.10:25,95.10.20.33:56026,<,EHLO [95.10.20.33],
2023-01-11T16:00:02.106Z,SERVER-HOSTNAME\Default Frontend SERVER-HOSTNAME,123453EC62278461,9,10.10.10.10:25,95.10.20.33:56026,*,,Client certificate chain validation status: 'EmptyCertificate'
2023-01-11T16:00:02.106Z,SERVER-HOSTNAME\Default Frontend SERVER-HOSTNAME,123453EC62278461,10,10.10.10.10:25,95.10.20.33:56026,*,,TlsDomainCapabilities='None'; Status='NoRemoteCertificate'
2023-01-11T16:00:02.106Z,SERVER-HOSTNAME\Default Frontend SERVER-HOSTNAME,123453EC62278461,11,10.10.10.10:25,95.10.20.33:56026,>,250  SERVER-HOSTNAME.DOMAIN.TEST Hello [95.10.20.33] SIZE 37748736 PIPELINING DSN ENHANCEDSTATUSCODES AUTH NTLM LOGIN X-EXPS GSSAPI NTLM 8BITMIME BINARYMIME CHUNKING XRDST,
2023-01-11T16:00:03.115Z,SERVER-HOSTNAME\Default Frontend SERVER-HOSTNAME,123453EC62278461,12,10.10.10.10:25,95.10.20.33:56026,<,AUTH LOGIN,
2023-01-11T16:00:06.195Z,SERVER-HOSTNAME\Default Frontend SERVER-HOSTNAME,123453EC62278461,14,10.10.10.10:25,95.10.20.33:56026,>,334 <authentication response>,
2023-01-11T16:00:10.644Z,SERVER-HOSTNAME\Default Frontend SERVER-HOSTNAME,123453EC62278461,15,10.10.10.10:25,95.10.20.33:56026,*,,Inbound AUTH LOGIN failed because of LogonDenied
2023-01-11T16:00:10.644Z,SERVER-HOSTNAME\Default Frontend SERVER-HOSTNAME,123453EC62278461,15,10.10.10.10:25,95.10.20.33:56026,*,,Inbound AUTH LOGIN failed because of LogonDenied
2023-01-11T16:00:10.644Z,SERVER-HOSTNAME\Default Frontend SERVER-HOSTNAME,123453EC62278461,15,10.10.10.10:25,95.10.20.33:56026,*,,User Name: username@domain.com
```

### References

  - [InverseCos - Hunting for APT Abuse of Exchange](https://www.inversecos.com/2022/07/hunting-for-apt-abuse-of-exchange.html)

  - [m365internals - Hunting In On-Premises Exchange Server Logs](https://m365internals.com/2022/10/07/hunting-in-on-premises-exchange-server-logs/)

  - [Microsoft - Configure Logging in IIS](https://learn.microsoft.com/en-us/iis/manage/provisioning-and-managing-iis/configure-logging-in-iis)

  - [Cyber Polygon - Hunting Down MS Exchange Attacks. Part 1. ProxyLogon (CVE-2021-26855, 26858, 27065, 26857)](https://cyberpolygon.com/materials/okhota-na-ataki-ms-exchange-chast-1-proxylogon/)

  - [Microsoft - HAFNIUM targeting Exchange Servers with 0-day exploits](https://www.microsoft.com/en-us/security/blog/2021/03/02/hafnium-targeting-exchange-servers/)
