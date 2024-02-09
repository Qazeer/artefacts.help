---
title: ETW - Windows Firewall
summary: 'Windows Firewall activity, such as configuration changes and rules creation, modification, or deletion.\n\nMain events:\n\nChannel: Microsoft-Windows-Windows Firewall With Advanced Security/Firewall.\nEvent ID 2003: "A Windows Defender Firewall setting in the <Domain | Private | Public> profile has changed".\nEvents 2004, 2071, and 2097 (depending on the Windows operating system version): "A rule has been added to the Windows Defender Firewall exception list".\nEvents 2005 and 2099 (depending on the Windows operating system version): "A rule has been modified in the Windows Defender Firewall exception list".'
keywords: 'Firewall, Windows Firewall, Domain, Private, Public, rule, block, allow, 2003, 2004, 2071, 2097, 2005, 2099, 2006, 2033, 2052'
tags:
  - windows_etw
  - windows_system_information
  - windows_lateral_movement
  - windows_defense_evasion
location: 'Channel: Microsoft-Windows-Windows Firewall With Advanced Security/Firewall.\nEvents: 2002, 2003, 2004, 2005, 2006, 2033, 2052, 2071, 2097, 2099.\n\nChannel: Security (events not enabled by default).\nEvents: 4946, 4947, 4948, 4950.'
last_updated: 2024-02-09
sidebar: sidebar
permalink: windows_etw_windows_firewall.html
folder: windows
---

### Overview

The `Windows Firewall` (officially called the `Microsoft Defender Firewall` in
Windows 10 version 2004 and later) is a host-based firewall, builtin in the
Windows operating systems.

One of three profiles is activated automatically for each network interface:

  - `Domain` profile: used when a Active Directory joined endpoint is connected
    to the domain network. The `Domain` profile is the least restrictive.

  - `Private` profile: applied to a network adapter when it is connected to a
    network that is identified by the user as a private network. Windows
    enables network discovery features, allows file sharing and other network
    features.

  - `Public` profile: applied to a network adapter by default, or if specified
    so by the user. The `Public` profile is the most restrictive.

By default, The `Windows Firewall` blocks inbound connections and allows
outbound connections that do not match a rule.

### Windows Firewall configuration change events

| Channel | Conditions | Events |
|---------|------------|--------|
| `Microsoft-Windows-Windows Firewall With Advanced Security/Firewall` | Default configuration. | Event `2003: A Windows Defender Firewall setting in the <Domain | Private | Public> profile has changed`. <br><br> Logged whenever the settings of a given `Windows Firewall` profile are changed. <br><br> Information of interest: <br> - The impacted profile. <br> - The setting updated and the new value set. <br> - The `SID` of the user and the process that modified the profile. <br><br> For instance: <br><br> `A Windows Defender Firewall setting in the Domain profile has changed` <br> `New Setting:` <br> `Type: Enable Windows Defender Firewall` <br> `Value:	No` <br> Indicates that the `Windows Firewall` was turned off for the `Domain` profile. <br><br> `A Windows Defender Firewall setting in the Private profile has changed` <br> `New Setting:` <br> `Type: Default Inbound Action` <br> `Value: Allow` <br> Indicates that the default action for inbound connection for the `Private` profile is now to accept the connections. |
| `Microsoft-Windows-Windows Firewall With Advanced Security/Firewall` | Default configuration. | `2002: A Windows Defender Firewall setting has changed`. <br><br> Logged whenever a  `Windows Firewall` setting is changed, excluding updates on a profile's settings (that are logged through event `2003`). <br><br> Information of interest: <br> - The setting updated and the new value set, in the same format as event `2003`. <br> - The `SID` of the user and the process that changed the setting. |
| `Security` | Requires `Audit MPSSVC Rule-Level Policy Change` to be enabled. | Event `4950: A Windows Firewall setting has changed`. <br><br> Logged whenever the  `Windows Firewall` settings are changed, including updates on a profile's settings. <br><br> Information of interest: <br> - The impacted profile. <br> - The setting updated and the new value set, in the same format as event `2003`. |

### Windows Firewall rules activity events

| Channel | Conditions | Events |
|---------|------------|--------|
| `Microsoft-Windows-Windows Firewall With Advanced Security/Firewall` | Default configuration. | Events `2004`, `2071`, and `2097` (depending on the Windows operating system version): `A rule has been added to the Windows Defender Firewall exception list`. <br><br> Logged whenever a new rule is configured for the `Windows Firewall`. <br><br> Information of interest: <br> The rule name and identifier. <br> - The rule parameters: <br> > Impacted `Windows Firewall` profile(s) (`Public`, `Private`, `Domain`). <br> > Origin and direction (local or remote). <br> > Action (allow or deny traffic). <br> > Network protocol (any, `TCP`, `UDP`, `ICMP`, ...). <br> > Eventual impacted application path. <br> > Eventual local / remote IP address(es) and port(s). <br> > ... <br> - The `SID` of the user and the process that created the rule. |
| `Microsoft-Windows-Windows Firewall With Advanced Security/Firewall` | Default configuration. | Events `2005` and `2099` (depending on the Windows operating system version): `A rule has been modified in the Windows Defender Firewall exception list`. <br><br> Logged whenever a `Windows Firewall` rule is modified. <br><br> Holds the same exact information of interest as rule creation events `2004`, `2071`, and `2097`. All the parameters of the modified rule are logged, whether they were updated or not. The modified value is not highlighted and the previous values are not logged. |
| `Microsoft-Windows-Windows Firewall With Advanced Security/Firewall` | Default configuration. | Events `2006`, `2033` and `2052` (depending on the Windows operating system version): `A rule has been deleted in the Windows Firewall exception list`. <br><br> Logged whenever a `Windows Firewall` rule is deleted. <br><br> Information of interest: <br> - Rule name and identifier. <br> - The `SID` of the user and the process that deleted the rule. |
| `Security` | Requires `Audit MPSSVC Rule-Level Policy Change` to be enabled. | `4946: A change has been made to Windows Firewall exception list. A rule was added`. <br> `4947: A change has been made to Windows Firewall exception list. A rule was modified`. <br> `4948: A change has been made to Windows Firewall exception list. A rule was deleted`. <br><br> Logged whenever a `Windows Firewall` rule is created / modified / deleted. <br><br> Information of interest: <br> - The `Windows Firewall` profile(s) the rule is applied to. <br> - The rule name and identifier. <br><br> The information provided on the impacted rule is thus minimal and do not include rule parameters. |

### References

  - [Wikipedia - Windows Firewall](https://en.wikipedia.org/wiki/Windows_Firewall)

  - [NSA - Event-Forwarding-Guidance](https://github.com/nsacyber/Event-Forwarding-Guidance/tree/master/Events)

  - [Audit MPSSVC Rule-Level Policy Change](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/audit-mpssvc-rule-level-policy-change)
