---
title: ETW - Remote Desktop - Remote Desktop Gateway
summary: 'For Remote Desktop access through a Remote Desktop Gateway (Windows server role that implements Remote Desktop Protocol (RDP) over HTTPS.\n\nMain events:\n\nChannel: Microsoft-Windows-TerminalServices-Gateway/Operational.\nEvent ID 200: "<DOMAIN>\<USERNAME> on client computer <SOURCE_IP> met resource authorization policy [...] to access the TS Gateway server".\nEvent ID 302: "<DOMAIN>\<USERNAME> on client computer <SOURCE_IP> connected to <REMOTE_HOST_FQDN>".\nEvent 303: "<DOMAIN>\<USERNAME> on client computer <SOURCE_IP> disconnected from <REMOTE_HOST_FQDN>. Before <DOMAIN>\<USERNAME> disconnected, the client transferred <BYTES_SENT> bytes and received <BYRES_RECEIVED> bytes. The client session duration was <SESSION_DURATION> seconds".'
keywords: 'RDP, Remote Desktop, Terminal Services, Remote Desktop Gateway, Microsoft-Windows-TerminalServices-Gateway'
tags:
  - windows_etw
  - windows_lateral_movement
  - windows_remote_desktop
  - windows_remote_desktop_dst
location: 'Channels:\n\nMicrosoft-Windows-TerminalServices-Gateway/Operational.\nEvents: 200, 300, 302, 303, 308, 312, 313.'
last_updated: 2024-01-11
sidebar: sidebar
permalink: windows_etw_remote_desktop_rdg_host.html
folder: windows
---

### Overview

`Remote Desktop Gateway` is a Windows server role that implements the
`Remote Desktop Protocol (RDP)` over HTTPS to establish an encrypted
connection. The gateway can be Internet facing to allow remote users to access
internal network resources through `RDP`.

| Channel | Conditions | Events |
|---------|------------|--------|
| `Microsoft-Windows-TerminalServices-Gateway/Operational` | Default configuration | Event `200: <DOMAIN>\<USERNAME> on client computer <SOURCE_IP> met resource authorization policy requirements and was therefore authorized to access the TS Gateway server`. <br><br> Information of interest: domain, username, and source IP of the user connecting to the `Remote Desktop Gateway` host, and authentication protocol used (such as `NTLM`). |
| `Microsoft-Windows-TerminalServices-Gateway/Operational` | Default configuration | Event `300: <DOMAIN>\<USERNAME> on client computer <SOURCE_IP> met resource authorization policy requirements and was therefore authorized to connect to <REMOTE_HOST_FQDN>`. <br><br> Information of interest: domain, username, and source IP of the user connecting to the `Remote Desktop Gateway` host and remote host (the user is connecting to through the `Remote Desktop Gateway`). |
| `Microsoft-Windows-TerminalServices-Gateway/Operational` | Default configuration | Event `302: <DOMAIN>\<USERNAME> on client computer <SOURCE_IP> connected to <REMOTE_HOST_FQDN>`. <br><br> Information of interest: domain, username, and source IP of the user connecting to the `Remote Desktop Gateway` host and remote host (the user is connecting to through the `Remote Desktop Gateway`). |
| `Microsoft-Windows-TerminalServices-Gateway/Operational` | Default configuration | Event `303: <DOMAIN>\<USERNAME> on client computer <SOURCE_IP> disconnected from <REMOTE_HOST_FQDN>.` <br><br> Additionnal payloads: `Before <DOMAIN>\<USERNAME> disconnected, the client transferred <BYTES_SENT> bytes and received <BYRES_RECEIVED> bytes. The client session duration was <SESSION_DURATION> seconds`. <br><br> Information of interest: domain, username, and source IP of the user connecting to the `Remote Desktop Gateway` host and remote host (the user is connecting to through the `Remote Desktop Gateway`). Metrics about the `Remote Desktop` session: bytes sent and received, and duration in seconds. |
| `Microsoft-Windows-TerminalServices-Gateway/Operational` | Default configuration | Event `312: <USERNAME>@<DOMAIN> on client computer <SOURCE_IP>:<SOURCE_PORT> has initiated an outbound connection that has yet to be authenticated`. <br><br> Information of interest: domain, username, and source IP of the user connecting to the `Remote Desktop Gateway` host. |
| `Microsoft-Windows-TerminalServices-Gateway/Operational` | Default configuration | Event `313: "<DOMAIN>\<USERNAME> on client computer <SOURCE_IP> has initiated an inbound connection that has yet to be authenticated`. <br><br> Information of interest: domain, username, and source IP of the user connecting to the `Remote Desktop Gateway` host. le|

### References

  - [Remote Desktop Gateway](https://v2cloud.com/tutorials/remote-desktop-gateway)

  - [EVTX Maps - Eric Zimmerman & Andrew Rathbun](https://github.com/EricZimmerman/evtx/tree/master/evtx/Maps)
