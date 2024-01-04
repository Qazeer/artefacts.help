---
title: System Information
summary: ''
keywords:
tags:
  - XXX
last_updated: 2024-01-05
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
