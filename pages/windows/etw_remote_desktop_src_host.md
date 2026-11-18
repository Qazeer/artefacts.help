---
title: ETW - Remote Desktop - Source host
summary: 'Source host initiating Remote Desktop access.\n\nMain events:\n\nChannel: Microsoft-Windows-TerminalServices-RDPClient/Operational.\nEvent ID 1024: "RDP ClientActiveX is trying to connect to the server (<HOSTNAME>)".\nEvent ID 1102: "The client has initiated a multi-transport connection to the server <IP>".\nEvent ID 1029: "Base64(SHA256(UserName)) is = <HASH>".'
keywords: RDP, Remote Desktop, Terminal Services, RDPClient, ClientActiveX
tags:
  - windows_etw
  - windows_lateral_movement
  - windows_lateral_movement_src
  - windows_remote_desktop
  - windows_remote_desktop_src
location: 'Channel:\n\nMicrosoft-Windows-TerminalServices-RDPClient/Operational.\nEvents: 1024, 1029, 1102.'
last_updated: 2024-01-08
sidebar: sidebar
permalink: windows_etw_remote_desktop_src_host.html
folder: windows
---

### Overview

| Channel | Conditions | Events |
|---------|------------|--------|
| `Microsoft-Windows-TerminalServices-RDPClient/Operational` | Default configuration. | Event `1024: RDP ClientActiveX is trying to connect to the server (<HOSTNAME>)`. |
| `Microsoft-Windows-TerminalServices-RDPClient/Operational` | Default configuration. | Event `1102: The client has initiated a multi-transport connection to the server <IP>`. |
| `Microsoft-Windows-TerminalServices-RDPClient/Operational` | Default configuration. | Event `1029: Base64(SHA256(UserName)) is = <HASH>`. <br><br> [This `CyberChef` formula](https://gchq.github.io/CyberChef/#recipe=Decode_text('UTF-8%20(65001)')Encode_text('UTF-16LE%20(1200)')SHA2('256',64,160)From_Hex('Space')To_Base64('A-Za-z0-9%2B/%3D')&input=QWRtaW5pc3RyYXRvcg) can be used to compute the hash. |

For each event of the `Microsoft-Windows-TerminalServices-RDPClient/Operational`
channel, the domain and `SID` of the user initiating the `Remote Desktop` connection are
logged.
