---
title: ETW - Authentication - Destination host
summary: 'Destination host of a local or remote access.\n\nMain events:\n\nEvent ID 4624: "An account was successfully logged on".\n\nEvent ID 4625: "An account failed to log on".\n\nEvent ID 4672: "Special privileges assigned to new logon".'
keywords: Security, Logon, 4624, 4625, 4672, 4634, 4647, 4649, 4778, 4779, 4800, 4801, 4802, 4803, 5378, 5632, 5633, LogonType, ImpersonationLevel
tags:
  - windows_etw
  - windows_lateral_movement
  - windows_lateral_movement_dst
location: 'Channel: Security.\nEvents: 4624, 4625, 4672, 4634, 4647, 4649, 4778, 4779, 4800, 4801, 4802, 4803, 5378.'
last_updated: 2024-01-02
sidebar: sidebar
permalink: windows_etw_authentication_dst_host.html
folder: windows
---

### Overview

| Channel | Conditions | Events |
|---------|------------|--------|
| `Security` | Default configuration. | [Event `4624: An account was successfully logged on`.](#security-event-id-4624) <br><br> Legacy: <br> Events `528: Successful Logon` and `540: Successful Network Logon`. |
| `Security` | Default configuration. | Event `4625: An account failed to log on`. <br><br> Legacy: <br> Events `529`, `530`, `531`, `532`, `533`, `534`, `535`, `536`, `537`, and `539`. |
| `Security` | Default configuration. <br><br> Only logged on for logon with elevated privileges. | [Event `4672: Special privileges assigned to new logon`.](#security-event-id-4672) <br><br> Legacy: <br> Events `576: Special privileges assigned to new logon`. |
| `Security` | Default configuration. | Event `4634: An account was logged off`. <br><br> Legacy: <br> Events `538: User Logoff`. |
| `Security` | Default configuration. <br><br> Only logged on for `Interactive` and `RemoteInteractive` logons. | Event `4647: User initiated logoff`. <br><br> Legacy: <br> Events `551: User initiated logoff`. |
| `Security` | Requires `Audit Other Logon/Logoff Events`. | Event `4649: A replay attack was detected`. <br><br> Event `4778: A session was reconnected to a Window Station`. <br><br> Event `4779: A session was disconnected from a Window Station`. <br><br> Event `4800: The workstation was locked`. <br><br> Event `4801: The workstation was unlocked`. <br><br> Event `4802: The screen saver was invoked`. <br><br> Event `4803: The screen saver was dismissed`. <br><br> Event `5378: The requested credentials delegation was disallowed by policy`. <br><br> Event `5632: A request was made to authenticate to a wireless network`. <br><br> Event `5633: A request was made to authenticate to a wired network`. |

### Security Event ID 4624

Location: destination machine `Security.evtx`.<br>
Event ID: `4624: An account was successfully logged on`.

Privileged logon will generate an additional `Security` event: `4672: Special
privileges assigned to new logon`.

The `4624` event yields information such as:
  - The SID `SubjectUserSid`, account name `SubjectUserName`, and domain
    `SubjectDomainName` of the user logging in.
  - the source machine hostname `WorkstationName`, IP `IpAddress` and port
    `IpPort` if the event corresponds to remote login (otherwise the three
    aforementioned fields are set to `-`).
  - The authentication protocol in the `AuthenticationPackageName` field
    (`NTLM`, `Kerberos` or `Negotiate `) used for the logging. If the logon is
    made through the `NTLM` protocol, the `LmPackageName` field precisely
    identify the `NTLM` version in use (`LM`, `NTLM V1`, `NTLM V2`).
  - The logon type in the `LogonType` field (detailed below).
  - The privileges level in the `ElevatedToken` field. If set to `%%1842`
    (`Yes`), the session the event represents runs in an elevated context. The
    event can be correlated with the `Security` event `EID: 4672` to precisely
    identify the privilege tokens of the session.
  - The impersonation level of the event in the `ImpersonationLevel` field
    (detailed below).
  - the `LogonID` field identifying the logon session, which can be correlated
    with various other `Security` events.

#### LogonType

The `LogonType` field provides information on how the logging was established:

| Logon Type | Description |
|------------|-------------|
| 2          | Interactive logon. <br><br> *Logon type generated for on screen login at the keyboard as well as some remote access with specific tools. <br> Note that access made using `PsExec` with a user specified using the `-u` option will result in an interactive logon.* |
| 3          | Network logon (share access, etc.). <br><br> *Logon type generated for access over the network (access to `SMB` share, `PsExec`, `WMI` / `WinRM`, etc.).* |
| 4          | Batch logon (scheduled task) |
| 5          | Service logon (service startup) |
| 7          | Unlock (on screen unlocking) |
| 8          | NetworkCleartext authentication (usually HTTP basic authentication) |
| 9          | NewCredentials authentication (does not seem to be in use) |
| 10         | RemoteInteractive authentication (Terminal Services, Remote Desktop or Remote Assistance) |
| 11         | CachedInteractive authentication (logging using cached credentials when a domain controller cannot be reached) |

**Interactive logons (`Logon type 2` and `Logon type 10`) will result in the
storing of the given users secrets (`NTLM` hash or `Kerberos` tickets) in
`LSASS` memory.** Knowing which users logged on interactively on a system can
help determine which accounts could be compromised following the takeover of a
system by an attacker.

#### ImpersonationLevel

The `ImpersonationLevel` field may take the following values:

| Flag | Correspondence | Description |
|------|----------------|-------------|
| `-` | `SecurityAnonymous` | The server process cannot obtain security information about the client. |
| `%%1832` | `Identification` | The server process can obtain information about the client but cannot impersonate the client and thus the client has no privileges. |
| `%%1833 ` | `Impersonation` | The server process can obtain information and impersonate the client's security context on the local system. |
| `%%1840 ` | `Delegation` | The server process can impersonate the client's security context on remote systems. |

### Security Event ID 4672

Location: destination machine `Security.evtx`.<br>
Event ID: `4672: Special privileges assigned to new logon`.

This event occurs whenever an account is assigned one, or more, of the
following privileges:

  - SeTcbPrivilege
  - SeBackupPrivilege
  - SeCreateTokenPrivilege
  - SeDebugPrivilege
  - SeEnableDelegationPrivilege
  - SeAuditPrivilege
  - SeImpersonatePrivilege
  - SeLoadDriverPrivilege
  - SeSecurityPrivilege
  - SeSystemEnvironmentPrivilege
  - SeAssignPrimaryTokenPrivilege
  - SeRestorePrivilege
  - SeTakeOwnershipPrivilege

The `SubjectLogonId` field can be correlated with the `Security` event
`EID: 4624` in order to retrieve more information on the logon session.
