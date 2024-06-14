---
title: PowerShell ConsoleHost_history
summary: 'Starting with PowerShell v5 on Windows 10, the commands entered in a PowerShell console will be logged by the PSReadline module to a user-scoped ConsoleHost_history.txt file.\n\nBy default, only the last 4096 commands are stored.\n\nInformation of interest: command entered, with no associated timestamps (or any additional metadata). The last entered command execution timestamp can be deduced from the last write timestamp of the ConsoleHost_history file itself.'
keywords: PowerShell, ConsoleHost_history, PSReadline
tags:
  - windows_program_execution
  - windows_powershell_activity
location: 'By default:\n\n<APPDATA>\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt\n\ni.e\n<SYSTEMDRIVE>:\Users\<USERNAME>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt.'
last_updated: 2024-01-06
sidebar: sidebar
permalink: windows_powershell_consolehost_history.html
folder: windows
---

### Overview

Starting with `PowerShell v5` on `Windows 10`, the commands entered in a
PowerShell console will be logged by the `PSReadline` module to a user-scoped
`ConsoleHost_history.txt` file.

Console-less PowerShell sessions, such as the content of PowerShell script or
commands execution through the `PowerShell ISE`, will not be logged in this
file.

By default, the `ConsoleHost_history.txt` file will be located under
`$env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`.

Bypassing `PSReadline` logging is however easy, as it simply requires to unload
the `PSReadline` module (for instance with the `Remove-Module PSReadline` in an
existing PowerShell session).

### Information of interest

The `ConsoleHost_history.txt` file contains the commands entered, with one
command per line and no associated timestamps (or any additional metadata). The
last entered command execution timestamp can however be deduced using the last
write timestamp of the `ConsoleHost_history` file itself.

By default, only the last 4096 commands are stored.

### Tool(s)

The [`ConvertPSHistoryTo-CSV.ps1`](https://gist.github.com/Qazeer/a0c1c14bb1eae233c1147d1d9dfb3e93)
PowerShell script (`KAPE` associated module
`PowerShell_ConvertPSHistoryTo-CSV`) can be used to recursively process the
specified directory to aggregate the `ConsoleHost_history.txt` files in a
single CSV output (grouping commands by user).
