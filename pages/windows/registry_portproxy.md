---
title: Registry - PortProxy
summary: 'The PortProxy registry key stores the port forwards configured on the local system using the netsh built-in utility.\n\nInformation of interest: the local and
remote IP address:port of each port forward.'
keywords: PortProxy, port forward, forwarding, netsh, v4tov4, v4tov6, v6tov4, v6tov6
tags:
  - windows_registry
  - windows_lateral_movement
location: 'File: <SYSTEMROOT>\System32\config\SYSTEM\n\nRegistry key:\nHKLM\SYSTEM\CurrentControlSet\Services\PortProxy\*\n\nIPv4 endpoint to IPv4 endpoint: v4tov4\tcp subkey.'
last_updated: 2024-01-21
sidebar: sidebar
permalink: windows_registry_portproxy.html
folder: windows
---

### Overview

The `PortProxy` registry key stores the port forwards configured on the local
system using the `netsh` built-in utility.

### Information of interest

The port forwards configured are stored in subkeys under `PortProxy` depending
on their parameters:

  - A first level subkey is based on the IP address source and destination
    versions (`IPv4` or `IPv6`). The following values are supported by `netsh`:
    `v4tov4`, `v4tov6`, `v6tov4`, and `v6tov6`.

  - A second level subkey is based on the `transport layer` protocol of
    the port forward. Only `TCP` is currently supported by netsh.

Each port forward configuration is then stored as a value under the second
level subkey, with the value's name referencing the local endpoint and the
value's data storing the remote endpoint.

Example for a port forward that redirect connections received on localhost port
`TCP` `4444` to `10.2.3.4:1234`:

```
PortProxy
  v4tov4
    tcp
      Value name: 127.0.0.1/4444
      Value data: 10.2.3.4/1234
```

### References

  - [Mandiant - DAVID PANY, STEVE MILLER, DANIELLE DESFOSSES - Bypassing Network Restrictions Through RDP Tunneling](https://www.mandiant.com/resources/blog/bypassing-network-restrictions-through-rdp-tunneling)

  - [DFIR notes - Port Proxy detection](https://www.dfirnotes.net/portproxy_detection/)
