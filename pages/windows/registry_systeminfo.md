---
title: Registry - System Information
summary: 'Various information about the local system as stored in the registry: computer hostname and domain, local users, network interfaces, system timezone, exposed network shares, firewall status and rules, SID of users that have interactively logged-in, installed applications, etc.'
keywords: 'HKLM, SYSTEM, SAM, ComputerName, CurrentVersion, Policy, ProfileList, Users, TimeZoneInformation, Select, Interfaces, NetworkList, FirewallPolicy, App Paths, Uninstall'
tags:
  - windows_registry
  - windows_system_information
location: 'HKLM\SYSTEM - ComputerName\n\nHKLM\SOFTWARE - CurrentVersion\n\nHKLM\SECURITY - Policy\n\nHKLM\SOFTWARE - ProfileList\n\nHKLM\SAM - Users\n\nHKLM\SYSTEM - TimeZoneInformation\n\nHKLM\SYSTEM - Select\n\nHKLM\SYSTEM - Interfaces\n\nHKLM\SYSTEM - NetworkList\n\nHKLM\SYSTEM - LanmanServer\Shares\n\nHKLM\SYSTEM - FirewallPolicy\n\nHKLM\SOFTWARE & NTUSER - App Paths\n\nHKLM\SOFTWARE & NTUSER - Uninstall'
last_updated: 2024-01-06
sidebar: sidebar
permalink: windows_registry_systeminfo.html
folder: windows
---

### ComputerName

| Hive | Description | Location |
| `HKLM\SYSTEM` | Name of the computer. | File: `%SystemRoot%\System32\config\SYSTEM` <br><br> Registry key: <br> `HKLM\SYSTEM\CurrentControlSet\Control\ComputerName\ComputerName` |

### CurrentVersion

| Hive | Description | Location |
| `HKLM\SOFTWARE` | Version and Service pack number of the Windows operating system. | File: `%SystemRoot%\System32\config\SOFTWARE` <br><br> Registry key: <br> `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion` (`ProductName` value) |

### Security - Policy

| Hive | Description | Location |
| `HKLM\SECURITY` | Basic information on the system: <br><br> - Computer name and `SID`. <br><br> - Computer's domain and domain `SID` (for domain-joined hosts). | File: `%SystemRoot%\System32\config\SECURITY` <br><br> Registry keys under `HKLM\SECURITY\Policy`: <br><br> - `PolAcDmN`: computer name <br><br> - `PolAcDmS`: computer `SID` <br><br> - `PolDnDDN`: computer's domain name <br><br> - `PolPrDmS`: computer's domain `SID` |

### ProfileList

| Hive | Description | Location |
| `HKLM\SOFTWARE` | `SID` to user profile folder correspondence for both local and domain accounts that have interactively logged on the system. Each account is referenced by a dedicated subkey under `ProfileList`, named after the user `SID`. <br><br> The user profile folder is referenced in the `ProfileImagePath` value under the per-user sunkey, and can be used to determine the account username. <br><br> The last write timestamp of each key indicates when the associated user last logged on the system. | File: `%SystemRoot%\System32\config\SOFTWARE` <br><br> Registry key: `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList` |

### SAM - Local users

| Hive | Description | Location |
| `HKLM\SAM` | Local users and groups definition. <br><br> The `SAM` database contains the local users attributes (username, `RID`, Last Password Change, group memberships), as well as their `LM` and `NTLM` hashes. <br><br> The `SysKey`, also referred to as the `BootKey`, stored in the `HKLM\SYSTEM` registry hive is necessary to decrypt the secrets in the `SAM` database. The `Impacket`'s `secretsdump.py` Python script can be used to extract the credentials from the `HKLM\SAM` (and `HKLM\SECURITY`) hives: <br> `secretsdump.py -sam <SAM> -system <SYSTEM> [-security <SECURITY>] LOCAL`. | File: `%SystemRoot%\System32\config\SAM` <br><br> Registry keys: <br> `SAM\Domains\Account\Users` |

### TimeZoneInformation

| Hive | Description | Location |
| `HKLM\SYSTEM`| System time zone information. | File: `%SystemRoot%\System32\config\SYSTEM` <br><br> Registry key: `HKLM\System\CurrentControlSet\Control\TimeZoneInformation` |

### Select

| Hive | Description | Location |
| `HKLM\SYSTEM` | `ControlSet` information for the `CurrentControlSet`, `ControlSet002`, ... registry keys: <br><br> - Current `ControlSet` pointed by the `CurrentControlSet` key. <br><br> - Last known good `ControlSet`. | File: `%SystemRoot%\System32\config\SYSTEM` <br><br> Registry key: `HKLM\SYSTEM\Select` |

### Network interfaces (Interfaces)

| Hive | Description | Location |
| `HKLM\SYSTEM` | Basic information about network interfaces (interface name, associated IP address, default gateway, and DHCP lease and eventual domain). <br><br> Additional network information is available in the `NetworkList` registry key. | File: `%SystemRoot%\System32\config\SYSTEM` <br><br> Registry keys: `HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\Interfaces\*` |

### Network interfaces (NetworkList)

| Hive | Description | Location |
| `HKLM\SYSTEM` | Basic network historical information (network name and type, first and last connection, etc.) | File: `%SystemRoot%\System32\config\SYSTEM` <br><br> Registry key: `HKLM\Microsoft\Windows NT\CurrentVersion\NetworkList\Profiles\` |

### LanmanServer\Shares

| Hive | Description | Location |
| `HKLM\SYSTEM` | Network SMB shares hosted by the system. <br><br> Each network share is associated with a `REG_MULTI_SZ` value. <br><br> The value is named from the network share name. The share name is also defined in the `ShareName` field of the registry value's data. <br><br> The share path on disk is defined in the `Path` field of the registry value's data. | File: `%SystemRoot%\System32\config\SYSTEM` <br><br> Registry key: `HKLM\SYSTEM\CurrentControlSet\Services\LanmanServer\Shares` |

### FirewallPolicy

| Hive | Description | Location |
| `HKLM\SYSTEM` | Windows local Firewall profiles (Public, Private, and Domain) status and configured rules. | File: `%SystemRoot%\System32\config\SYSTEM` <br><br> Registry key: `HKLM\SYSTEM\CurrentControlSet\Services\SharedAccess\Parameters\FirewallPolicy\*` |

### Installed applications (App Paths)

| Hive | Description | Location |
| `HKLM\SOFTWARE` <br><br> and <br><br> `NTUSER` | Applications installed on the system, on a system-wide or per-user basis. <br><br> The entries are mainly used by the Windows operating system for two purposes: <br><br> - Mapping an application file name to its executable full path. <br><br> - Pre-pending information to the `PATH` environment variable on a per-application and per-process basis. <br><br> Applications installed system-wide have their information written in the `HKLM\SOFTWARE` registry hive, while applications installed per user have their information written in the user `NTUSER` hive. <br><br> For each installed application the following notable information is available: <br><br> - File name and full file path of the application executable. <br><br> - Timestamp of installation. | For system-wide applications: <br><br> File: <br> `%SystemRoot%\System32\config\SOFTWARE` <br><br> Registry key: `Microsoft\Windows\CurrentVersion\App Paths` <br><br><br> For per-user applications: <br><br> File: `%SystemDrive%:\Users\<USERNAME>\NTUSER.dat` <br><br> Registry key: `HKCU\Software\Microsoft\Windows\CurrentVersion\App Paths` |

### Installed applications (Uninstall)

| Hive | Description | Location |
| `HKLM\SOFTWARE` <br><br> and <br><br> `NTUSER` | Applications installed on the system, on a system-wide or per-user basis, as displayed in the "Add or remove programs" of the Windows Control Panel / Settings. <br><br> Applications installed system-wide have their information written in the `HKLM\SOFTWARE` registry hive, while applications installed per user have their information written in the user `NTUSER` hive. <br><br> Each application installation data is defined in a dedicated subkey under `Uninstall`, identified by the application name. <br><br> For each installed application the following notable information is available: <br><br> - The application name. <br><br> - The application installation location, display icon (often based directly on the application main executable, thus giving the full path of the application main program), full path of the uninstaller. <br><br> - The date of the installation. The last write timestamp of the registry key can also be an indicator of when the application was installed (with better precision). <br><br> - The size of the applicationn. <br><br> - Various metadata on the application (provided by the application installer itself): version, publisher, ... | For system-wide applications: <br><br> File: `%SystemRoot%\System32\config\SOFTWARE` <br><br> Registry key: `Microsoft\Windows\CurrentVersion\Uninstall` <br><br><br> For per-user applications: <br><br> File: `%SystemDrive%:\Users\<USERNAME>\NTUSER.dat` <br><br> Registry key: `HKCU\Software\Microsoft\Windows\CurrentVersion\Uninstall` |
