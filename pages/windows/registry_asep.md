---
title: Registry - Auto-Start Extensibility Points
summary: 'A number of registry keys, known as Auto-Start Extensibility Points (ASEP) registry keys, are run whenever the system is booted or a specific user logs in.\n\nThe ASEP keys under HKLM are run every time the system is started, while the ASEP keys under HKCU are only executed when the user associated with the keys logs onto the system.\n\nWhile a subset of ASEP registry keys are
leveraged by threat actors, hundreds of keys may be used to execute a program at boot or following a user logging.'
keywords: ASEP, Auto-Start Extensibility Points, Run, RunOnce, Winlogon, Startup, Appinit_Dlls, Userinit, Autoruns, AutorunsC, RECmd, RegistryASEPs, T1547.001
tags:
  - windows_registry
  - windows_local_persistence
location: 'HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run\nHKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run\n\nHKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce\nHKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce\n\nHKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\Shell\nHKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\Shell\n\nHKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\nHKCU\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\n\nHKLM\SOFTWARE\Policies\Microsoft\Windows\System\Scripts\Startup\nHKCU\SOFTWARE\Policies\Microsoft\Windows\System\Scripts\Startup\n\nHKLM\SOFTWARE\Policies\Microsoft\Windows\System\Scripts\Logon\nHKCU\SOFTWARE\Policies\Microsoft\Windows\System\Scripts\Logon\n\n...'
last_updated: 2024-01-22
sidebar: sidebar
permalink: windows_registry_asep.html
folder: windows
---

### Overview

A number of registry keys, known as `Auto-Start Extensibility Points (ASEP)`
registry keys, are run whenever the system is booted or a specific user logs
in.

The `ASEP` keys under `HKEY_LOCAL_MACHINE (HKLM)` are run every time the system
is started, while the `ASEP` keys under `HKEY_CURRENT_USER (HKCU)` are
only executed when the user associated with the keys logs onto the system. The
programs started through an `ASEP` key in a user's `HKCU` will execute under
the context and privileges of the user.

While a subset of common and well-known `ASEP` registry keys are
[leveraged by threat actors](https://attack.mitre.org/techniques/T1547/001/),
hundreds of keys may be used to execute a program at boot or following a user
logging.

### Common ASEPs

```
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnceEx
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunServices
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunServicesOnce
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\Shell
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\Notify
HKLM\SOFTWARE\Policies\Microsoft\Windows\System\Scripts\Startup
HKLM\SOFTWARE\Policies\Microsoft\Windows\System\Scripts\Logon
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\Shell
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\Userinit
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\Taskman
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Windows\Appinit_Dlls
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\SharedTaskScheduler
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer\Run
HKLM\SOFTWARE\Microsoft\Active Setup\Installed Components
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\ShellServiceObjectDelayLoad
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Browser Helper Objects
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\ShellExecuteHooks
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Shell Extensions\Approved
HKLM\SOFTWARE\Microsoft\Internet Explorer\Toolbar
HKLM\System\CurrentControlSet\Services

HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnceEx
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\RunServices
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\RunServicesOnce
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\Shell
HKCU\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon
HKCU\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\Notify
HKCU\SOFTWARE\Policies\Microsoft\Windows\System\Scripts\Startup
HKCU\SOFTWARE\Policies\Microsoft\Windows\System\Scripts\Logon
HKCU\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\Shell
HKCU\SOFTWARE\Microsoft\Active Setup\Installed Components
HKCU\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Windows\Load
HKCU\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Windows\Run
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\ShellServiceObjectDelayLoad
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer\Run
```

### Tool(s)

The most common `ASEP` keys can be automatically checked using the
[`SysInternals` `Autoruns`](https://learn.microsoft.com/fr-fr/sysinternals/)
(GUI) and `AutorunsC` (CLI) utilities.

The [`RECmd`](https://github.com/EricZimmerman/RECmd) utility can also be used
to parse the registry and extract a predefined list of `ASEP` registry keys
using the
[`RegistryASEPs.reb`](https://github.com/EricZimmerman/RECmd/blob/master/BatchExamples/RegistryASEPs.reb)
plugin. A list of nearly 500 `ASEPs` registry keys and 400 values are
referenced by the plugin.

```bash
RECmd.exe -d "<NTFS_VOLUME | FOLDER_CONTAINING_REGISTRY_HIVES>" --bn ".\BatchExamples\RegistryASEPs.reb" --csv "<OUTPUT_FOLDER>"
```

### References

  - [MITRE ATT&CK - T1547.001- Boot or Logon Autostart Execution: Registry Run Keys / Startup Folder](https://attack.mitre.org/techniques/T1547/001/)

  - [RegistryASEPs.reb](https://github.com/EricZimmerman/RECmd/blob/master/BatchExamples/RegistryASEPs.reb)
