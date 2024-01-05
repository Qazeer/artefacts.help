---
title: Registry - User Assist
summary: 'The UserAssist registry key references GUI program executions, and, starting from Windows 7, shortcut executions.\n\nInformation of interest: full path of the executed program / shortcut (encoded in ROT13), sometimes the timestamp of the last execution, an unreliable run counter and focus count and time.'
keywords:
tags:
  - windows_program_execution
  - windows_registry
location: 'File: <SYSTEMDRIVE>:\Users\<USERNAME>\NTUSER.dat\n\nRegistry key:\nHKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\<GUID>\Count\n\nWindows Xp:\n{75048700-EF1F-11D0-9888-006097DEACF9} (GUI program execution).\n\nStarting from Windows 7:\n{CEBFF5CD-ACE2-4F4F-9178-9926F41749EA} (GUI program execution).\n{F4E57C4B-2036-45F0-A9AB-443BCFE33D9F} (shortcut execution).'
last_updated: 2024-01-05
sidebar: sidebar
permalink: windows_registry_userassist.html
folder: windows
---

### Overview

The purpose of the `UserAssist` registry key is not officially documented.

The registry key references **execution of programs with a graphical
interface**, installed or from a portable executable, and, starting from
`Windows 7`, shortcuts execution.

### Information of interest

One or two main registry subkeys can be found depending on the Windows OS
version:

  - On `Windows Xp`: `{75048700-EF1F-11D0-9888-006097DEACF9}` linked to
    execution of executable files.

  - Starting from `Windows 7`:
    - `{CEBFF5CD-ACE2-4F4F-9178-9926F41749EA}` linked to execution of
      executable files.

    - `{F4E57C4B-2036-45F0-A9AB-443BCFE33D9F}` linked to execution of shortcut
      files.

Each execution is associated with an entry that contains the following notable
information:

  - Full path of the executed program / shortcut (as the value name, encoded
    in `ROT13`).

  - Sometimes, the timestamp of the last execution (in the binary value data).

  - An unreliable run counter and focus count and time (in the binary value
    data).

### References

  - [Defence Institute of Advanced Technology (DU), India - Bhupendra Singh and Upasna Singh - Program Execution Analysis using UserAssist Key in Modern Windows](https://www.scitepress.org/papers/2017/64167/64167.pdf)
