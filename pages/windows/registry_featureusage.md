---
title: Registry - FeatureUsage
summary: 'Introduced in Windows 10 version 1903, the FeatureUsage registry key is linked to the Windows Task, storing a number of metrics related to the Task bar usage.\n\nInformation of interest: program full path and run counter of the associated taskbar operation (brought to focus, right-clicked, icon updated, etc.).\n\nNo timestamp of execution / occurrence is available.'
keywords: 'FeatureUsage, AppSwitched, ShowJumpView, AppBadgeUpdated, AppLaunch, TrayButtonClicked'
tags:
  - windows_registry
  - windows_program_execution
location: 'File: <SYSTEMDRIVE>:\Users\<USERNAME>\NTUSER.dat\n\nRegistry subkeys under:\nHKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\FeatureUsage\n\nAppSwitched, ShowJumpView, AppBadgeUpdated, AppLaunch, and TrayButtonClicked'
last_updated: 2024-01-06
sidebar: sidebar
permalink: windows_registry_featureusage.html
folder: windows
---

### Overview

Introduced in `Windows 10`'s version 1903, the `FeatureUsage` registry key is
linked to the Windows Task, storing a number of metrics related to the Task bar
usage.

### Information of interest

The `FeatureUsage` registry key has a number of subkeys, holding metrics
related to different taskbar operations. A program is added to a subkey upon
occurrence of the operation associated with the subkey (switch to focus, jump
menu opened, etc.). Each further occurrence of the operation for the given
program increments the run count of the program entry.

Each entry, stored as a value under an operation subkey, is composed of the
**program full path** (as the value name) and an **operation run count** (as
the value data). **No timestamp of execution / occurrence is available**.

The following operation subkeys are populated:

  - `AppSwitched`: applications brought to focus (application left-clicked on
    the taskbar).

  - `ShowJumpView`: applications whose jump menu was opened (application
    right-clicked on the taskbar).

  - `AppBadgeUpdated`: applications that had their task bar icon updated
    (for example for notifications).

  - `AppLaunch`: applications pinned on the taskbar that have been executed.

  - `TrayButtonClicked`: clicks on the default taskbar buttons (such as the
    start button).

### References

  - [CrowdStrike - Jai Minton - Employing FeatureUsage for Windows 10 Taskbar Forensics](https://www.crowdstrike.com/blog/how-to-employ-featureusage-for-windows-10-taskbar-forensics/)
