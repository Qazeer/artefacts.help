---
title: .NET CLR UsageLogs
summary: 'Following the execution (or in-memory injection) of a .NET assembly, the Common Language Runtime (CLR) creates a Usage Log file whose named is based on the name of the executed assembly.\n\nInformation of interest: the filename of the log file match the name of the assembly / binary executed.\nThe file creation timestamp corresponds to the first time the associated assembly was executed and the file last modification timestamp corresponds to the last execution time of the assembly.'
keywords:
tags:
  - windows_program_execution
location: '<SYSTEMDRIVE>:\Users\<USERNAME>\AppData\Local\Microsoft\CLR_v<VERSION>\<BINARY_NAME>.exe.log\n<SYSTEMDRIVE>:\Users\<USERNAME>\AppData\Local\Microsoft\CLR_v<VERSION>\UsageLogs\<BINARY_NAME>.exe.log\n\n<SYSTEMROOT>\System32\config\systemprofile\AppData\Local\Microsoft\CLR_<VERSION>\<BINARY_NAME>.exe.log\n<SYSTEMROOT>\System32\config\systemprofile\AppData\Local\Microsoft\CLR_<VERSION>\UsageLogs\<BINARY_NAME>.exe.log'
last_updated: 2024-01-06
sidebar: sidebar
permalink: windows_net_clr_usagelogs.html
folder: windows
---

### Overview

**Following the execution (or in-memory injection) of a `.NET` assembly**, the
`Common Language Runtime (CLR)` creates **a `Usage Log` file whose named is
based on the name of the executed assembly**.

The file is written just prior the assembly execution terminate, and will thus
not be written if the process does not gracefully exit.

### Information of interest

**The filename of the log file match the name of the assembly / binary
executed.**

The **file creation timestamp corresponds to the first time the associated
assembly was executed** and the **file last modification timestamp corresponds
to the last execution time** of the assembly.

The content of the file itself does not appear to hold forensics value.

### Tool(s)

The [`ConvertUsageLogsTo-CSV.ps1`](https://gist.github.com/Qazeer/6c655627962f034aa2b6e92594770ee2)
PowerShell script (`KAPE` associated module
`PowerShell_ConvertUsageLogsTo-CSV`) can be used to recursively process the
specified directory to aggregate the `Usage Log` file names in a single CSV
output (grouping filename by user).

### References

  - [BOHOPS - INVESTIGATING .NET CLR USAGE LOG TAMPERING TECHNIQUES FOR EDR EVASION](https://bohops.com/2021/03/16/investigating-net-clr-usage-log-tampering-techniques-for-edr-evasion/)

  - Menasec - Interesting DFIR traces of .NET CLR - DOWN
