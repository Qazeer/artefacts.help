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

### HKLM\SYSTEM - ComputerName

| Hive | Description | Location |
| `HKLM\SYSTEM` | Name of the computer. | File: `%SystemRoot%\System32\config\SYSTEM`<br><br>Registry key: `HKLM\SYSTEM\CurrentControlSet\Control\ComputerName\ComputerName` |

### HKLM\SOFTWARE - CurrentVersion

| Hive | Description | Location |
| `HKLM\SOFTWARE` | Version and Service pack number of the Windows operating system. | File: `%SystemRoot%\System32\config\SOFTWARE`<br><br>Registry key: `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion` (`ProductName` value) |
