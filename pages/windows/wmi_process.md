---
title: WMI - Process
summary: 'The following process are related to WMI activity:\n\n- wmic.exe: client command line utility to interact with WMI (locally or on a remote computer). The PowerShell Invoke-WmiMethod cmdlet can be used as an alternative to wmic.\n\n- WmiPrvSE.exe: WMI Provider Host program that is executed to run WMI commands. If a program is executed through WMI, it will be spawned as a child of the wmiprvse.exe process.'
keywords: WMI, wmic, node, Invoke-WmiMethod, Win32_Process, WmiPrvSE, scrcons
tags:
  - windows_wmi
  - windows_lateral_movement
  - windows_lateral_movement_dst
last_updated: 2024-01-24
sidebar: sidebar
permalink: windows_wmi_process.html
folder: windows
---

The following process are related to `WMI` activity:

  - `wmic.exe`: client command line utility to interact with `WMI` (locally or
    on a remote computer). The `/node` parameter can be used to specify a remote
    computer and the `process call create  "<COMMAND>"` command to create a
    process to execute the specified command.

    PowerShell `Invoke-WmiMethod` cmdlet can be used as an alternative to
    `wmic` to execute `WMI` query locally or on a remote computer.

    For example, to spawn a process on a remote computer using the
    `Win32_Process` `WMI` class:

    ```powershell
    wmic /node:<REMOTE_HOST> /user:<USERNAME> /password:<PASSWORD> process call create "<COMMAND>"

    Invoke-WmiMethod -Computer <REMOTE_HOST> [-Credential <PS_CREDENTIALS>] -Class Win32_Process -Name create -Argument "<COMMAND>"
    ```

  - `WmiPrvSE.exe`: `WMI Provider Host` program that is executed to run `WMI`
    commands. If a program is executed through `WMI`, it will be spawned as a
    child of the `wmiprvse.exe` process.

    Suspicious child process of `WmiPrvSE.exe` (such as `powershell.exe` or
    `cmd.exe`) can be an indicator of lateral movement over `WMI` or
    persistence through a `WMI Event Subscription`.

  - `scrcons.exe`: `WMI Standard Event Consumer` process that spawn for
    `ActiveScriptEventConsumer` execution.

As `WMI` can be used legitimately in the environment, the execution of a `WMI`
related program may not necessarily be an indicator of malicious activity.
