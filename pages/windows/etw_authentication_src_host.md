---
title: ETW - Authentication - Source host
summary: 'Source host of a remote access.\n\nMain events:\n\nEvent ID 4648: "A logon was attempted using explicit credentials".\n\nEvent ID 4624: "An account was successfully logged on", with LogonType 9 (can be a sign of runas NetOnly usage or Pass-The-Hash attack).'
keywords: Security, Logon, RunAs, explicit credentials, 4648, 4624, Logon Type 9, LogonType 9, Seclogo, LogonProcess, Pass-The-Hash
tags:
  - windows_etw
  - windows_lateral_movement
  - windows_lateral_movement_src
location: 'Channel: Security.\nEvents: 4648, 4624 (LogonType 9).'
last_updated: 2025-04-16
sidebar: sidebar
permalink: windows_etw_authentication_src_host.html
folder: windows
---

### Overview

| Channel | Conditions | Events |
|---------|------------|--------|
| `Security` | Default configuration. <br><br> Only logged whenever alternate credentials are used. | [Event `4648: A logon was attempted using explicit credentials`](#security-event-id-4648). <br><br> Legacy: <br> Events `552: Logon attempt using explicit credentials`. |
| `Security` | Default configuration. <br><br> Only logged for `runas /NetOnly` (and similar) process execution. | Event [`4624: An account was successfully logged on`](./etw_authentication_dst_host.md#security-event-id-4624), with `LogonType` `9` and the specified alternate credentials as `Network Account Domain` and `Network Account Name`. <br><br> Using the [`runas`](https://learn.microsoft.com/fr-fr/windows/win32/com/runas) command with the `/NetOnly` switch will generate an `4624` event with a `LogonType 9`, a `Seclogo` `LogonProcess`, and `svchost.exe` as the originating process. With the `/NetOnly` switch, Windows will not try to validate the specified credentials. The new process will run as the currently logged on user for local access, but any network connections to other computers will be made using the user account specified. This is due to the fact that, following a `runas /NetOnly`, Windows will create a new [`Logon Session`](https://learn.microsoft.com/en-us/windows/win32/secauthn/lsa-logon-sessions) with the provided credentials but will copy the current [`access token`](https://learn.microsoft.com/en-us/windows/win32/secauthz/access-tokens) for the new process. Access to local resources will leverage that `access token`, while network connections will be made using the credentials associated with the `Logon Session`. Beyond `runas /NetOnly`, this `LogonType 9`, `Seclogo` `LogonProcess`, and `svchost.exe` originating process combination can be an indicator that a `Pass-The-Hash` attack was conducted. <br><br> Without the `/NetOnly` switch, `runas` will generate `4624` (`LogonType 2`) and `4648` events in close proximity. |

### Security Event ID 4648

Windows Security Log Event ID
`4648: A logon was attempted using explicit credentials`.

Includes information about the target server:
`Target Server Name` (hostname or IP) and `Additional Information` of the
service requested.

The `TargetServerName` and `TargetInfo` fields can reference information about
the remote server and service (such as `TargetInfo` set to `TERMSRV/<HOSTNAME>`
for outgoing `RDP`).

### References

  - [Fortra - Raphael Mudge - Windows Access Tokens and Alternate Credentials](https://www.cobaltstrike.com/blog/windows-access-tokens-and-alternate-credentials)
  
  - [Microsoft - LSA Logon Sessions](https://learn.microsoft.com/en-us/windows/win32/secauthn/lsa-logon-sessions)

  - [Microsoft - Access Tokens](https://learn.microsoft.com/en-us/windows/win32/secauthz/access-tokens)
