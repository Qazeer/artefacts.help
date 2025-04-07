---
title: ETW - Network shares activity and access
summary: 'For access and operations on network shares configured on the local system, and access to files and folders hosted on network shares.\n\nBy default no events are generated, as network share auditing requires "Audit File Share" (share access and lifecycle) and/or "Audit Detailed File Share" (hosted files and folders access) to be enabled. Enabling network share auditing may however generate an overwhelming amount of events.\n\nMain events:\n\nChannel: Security.\nEvent ID 5140: "A network share object was accessed".\nEvent ID 5145: "A network share object was checked to see whether client can be granted desired access".'
keywords: 'shares, network share, network drive, Audit File Share, Audit Detailed File Share, 5140, 5142, 5143, 5144, 5145'
tags:
  - windows_etw
  - windows_lateral_movement
  - windows_lateral_movement_dst
location: 'Channel: Security.\nEvents: 5140, 5142, 5143, 5144, 5145.'
last_updated: 2024-01-13
sidebar: sidebar
permalink: windows_etw_network_shares.html
folder: windows
---

### Overview

For access and operations on network shares configured on the local system, and
access to files and folders hosted on network shares.

By default no events are generated, as network share auditing requires advanced
auditing policies to be enabled:

  - `Audit File Share`, for share access and lifecycle.

  - `Audit Detailed File Share`, for access to hosted files and folders access.

Enabling network share auditing may however generate an overwhelming amount of
events.

| Channel | Conditions | Events |
|---------|------------|--------|
| `Security` | Requires `Audit File Share` to be enabled. | Events related to network shares: creation, deletion, modification, and access attempts of network shares. **Do not track access to folders and files hosted on network shares**. <br> As there are no `System Access Control Lists` (`SACLs`) for shares, access to all shares on the system are audited.<br><br> Event `5140: A network share object was accessed`. <br> Generated every time a network share is accessed, but only once per session (upon first access attempt). <br> `Object Type` is always `File` for this event. <br><br> Event `5142: A network share object was added`. <br><br> Event `5143: A network share object was modified`. <br><br> Event `5144: A network share object was deleted`. <br><br> All events include information about the account that performed the operation: username, domain, and `SID` as well as the `Logon ID` associated with the logon. <br> Events `5140` also include network information: source IP address and port. |
| `Security` | Requires `Audit Detailed File Share` to be enabled. | Event related to access to folders and files hosted on network shares. The event is generated upon every access to a network shared file or folder (successful or not). <br> Failure events are generated only when access is denied at the file share level, not a the file/folder level. **The event may thus not indicate that the access to the shared file or folder was successful**. <br><br> Event `5145: A network share object was checked to see whether client can be granted desired access`. <br> Includes information about the account that performed the operation: username, domain, and `SID`, the `Logon ID` associated with the logon, and the source IP address and port. |
