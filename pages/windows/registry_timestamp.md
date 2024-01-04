---
title: Registry - Timestamp and timestomping
summary: 'The last write / modified timestamp of a registry key is the only generic timestamp available regarding registry keys and correspond to the last time a write operation occurred on the key.\n\nThere is indeed no last write / modified timestamp for registry value.\n\nSimilarly to MFT MACB timestamp, the last write / modified timestamp of a registry key can be timestomped, which is something hard to detect without dedicated monitoring tools.'
keywords: 'last write registry key, timestomping, NtSetInformationKey'
tags:
  - windows_registry
  - windows_defense_evasion
last_updated: 2024-01-04
sidebar: sidebar
permalink: windows_registry_timestamp.html
folder: windows
---

### Registry last write timestamps

The **last write / modified timestamp of a registry key correspond to the last
time a write operation occurred on the key**. Multiple types of write operation
may trigger an update of the last write / modified timestamp of the key:

  - Addition / modification / deletion of one (or multiple) values under the
    key.

  - Addition / deletion of a sub-key under the key.

  - Change in the security descriptor (including `Access Control List (ACL)`)
    of the key.

**The last write / modified timestamp of a registry key is the only generic
timestamp available regarding registry keys**. There is indeed no last write /
modified timestamp for registry value.

### Registry timestomping

Similarly to `MFT` `MACB` timestamp, **the last write / modified timestamp of a
registry key can be timestomped**. This can be achieved using the Windows
`NtSetInformationKey` API.

**Such timestomping can be hard to detect** without a dedicated monitoring
product (such as an EDR), [as no associated event logs are generated and
timestamps discrepancies can only occur for registry keys with subkeys](https://www.inversecos.com/2022/04/malicious-registry-timestamp.html).

### References

  - [InverseCos - Malicious Registry Timestamp Manipulation Technique: Detecting Registry Timestomping](https://www.inversecos.com/2022/04/malicious-registry-timestamp.html)
