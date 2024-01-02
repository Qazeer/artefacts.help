---
title: ETW - Authentication - Source host
summary: 'Source host of a remote access.\n\nMain events:\n\nEvent ID 4648: "A logon was attempted using explicit credentials".\n\nEvent ID 4624: "An account was successfully logged on", with LogonType 9.'
keywords: Security, Logon, RunAs, explicit credentials, 4648, 4624, Logon Type 9, LogonType 9
tags:
  - windows_etw
  - windows_lateral_movement
location: 'Channel: Security'
last_updated: 2024-01-02
sidebar: sidebar
permalink: windows_etw_authentication_src_host.html
folder: windows
---

### Overview

| Channel | Conditions | Events |
|---------|------------|--------|
| `Security` | Default configuration. <br><br> Only logged whenever alternate credentials are used. | [Event `4648: A logon was attempted using explicit credentials`](#security-event-id-4648). <br><br> Legacy: <br> Events `552: Logon attempt using explicit credentials`. |
| `Security` | Default configuration. <br><br> Only logged for `runas /NetOnly` (and similar) process execution. | Event `4624: An account was successfully logged on`, with `LogonType` `9` and the specified alternate credentials as `Network Account Domain` and `Network Account Name`. |

### Security Event ID 4648

Windows Security Log Event ID
`4648: A logon was attempted using explicit credentials`.

Includes information about the target server:
`Target Server Name` (hostname or IP) and `Additional Information` of the
service requested.

The `TargetServerName` and `TargetInfo` fields can reference information about
the remote server and service (such as `TargetInfo` set to `TERMSRV/<HOSTNAME>`
for outgoing `RDP`).
