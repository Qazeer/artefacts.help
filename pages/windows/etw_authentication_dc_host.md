---
title: ETW - Authentication - Active Directory Domain Services (Domain Controllers)
summary: 'For authentication attempts from a source host to an Active Directory domain-joined destination host (which is not a Domain Controller).\n\nMain events:\n\nEvent ID 4624: "An account was successfully logged on", with LogonType 3 (only for a remote interactive logon on a domain-joined destination host).\n\nEvent ID 4776 "The domain controller attempted to validate the credentials for an account", for NTLM authentication.\n\nEvent 4768: "A Kerberos authentication ticket (TGT) was requested" and 4769: "A Kerberos service ticket was requested", for Kerberos tickets request and usage.\n\nEvent 4771: "Kerberos pre-authentication failed", for authentication failures over Kerberos.'
keywords: Security, Authentication, Active Directory, Domain Services, Domain Controllers, domain-joined, NTLM, 4776, Kerberos, TGT, TGS, ST, 4768, 4769, 4771, pre-authentication
tags:
  - windows_etw
  - windows_lateral_movement
location: 'Channel: Security.\nEvents: 4776, 4768, 4769, 4771, 4624, 4625.'
last_updated: 2024-01-02
sidebar: sidebar
permalink: windows_etw_authentication_dc_host.html
folder: windows
---

### Overview

| Channel | Conditions | Events |
|---------|------------|--------|
| `Security` | Default configuration. <br><br> For `NTLM` successful or failed authentication attempts. | Event `4776`: `The domain controller attempted to validate the credentials for an account`. <br><br> *If the `Result Code` field is not equal to `0x0` the authentication failed. The event is associated with the computer from which the logon attempt originated and does not identify the target service. This event is also logged for non Domain Controllers, on the target computer, for logon attempts with local `SAM` accounts.* |
| `Security` | Default configuration. <br><br> For `Kerberos` ticket request and usage. | If the user has not already retrieved a `TGT` during the session opening on the source host: <br><br> Event `4768`: `A Kerberos authentication ticket (TGT) was requested`. <br><br> *If the `Result Code` field is not equal to `0x0` the request failed (but not for a failed authentication).* <br><br> Event `4769`: `A Kerberos service ticket was requested`. <br><br> *The `ServiceName` and `ServiceSid` fields indicate the service the `service ticket` is requested for. However, for lateral movement, the service and service `SID` are often set to the destination machine account, with no information on the actual service targeted (`RPC`, `CIFS`, etc.).* |
| `Security` | Default configuration. <br><br> For `Kerberos` failed authentication attempts. | Event `4771`: `Kerberos pre-authentication failed`. <br><br> *For authentication failures over Kerberos (in case of password spraying or bruteforce over Kerberos for instance).* |
| `Security` | Default configuration. <br><br> As part of an interactive session opening. | Event `4624: An account was successfully logged on`, with `LogonType` `3` (for the user opening the session on the remote destination). |
| `Security` | Default configuration. | Event `4625: An account failed to log on`. <br><br> *For authentication failures over NTLM.* |

### Specificities of Domain Controller: authentication versus interactive session opening

**As the Domain Controllers only handle the authentication, and will not open a
login session in this scenario, no `4624` or `4625` events will be logged.**

**However, for a remote interactive logon on a domain-joined destination host,
a `4624` event of LogonType `3` (and `4768` + `4769` events) will be logged on
a Domain Controller** (potentially different than the one that processed the
authentication from the source host) originating from the destination host (as
part of the interactive session opening process).
