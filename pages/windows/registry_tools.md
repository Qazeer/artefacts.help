---
title: Registry - Tools
summary: 'Tools for processing the Windows Registry, including: RegistryExplorer, RECmd, and RegRipper.'
keywords: 'Registry tools, RegistryExplorer, RECmd, RegistryPlugins, RECmd_AllBatchFiles, RegRipper'
tags:
  - windows_registry
last_updated: 2024-01-03
sidebar: sidebar
permalink: windows_registry_tools.html
folder: windows
---

### RegistryExplorer/RECmd

[`RegistryExplorer`](https://www.sans.org/tools/registry-explorer/) and
[`RECmd`](https://github.com/EricZimmerman/RECmd) are tools developed by Eric
Zimmerman to analyze the Windows registry. `RegistryExplorer` is a graphical
utility to visualize registry hives, while `RECmd` is command-line tool to
parse registry hives to CSV or JSON outputs.

`RegistryExplorer` can process registry hive files or directly access the
registry of the current live system. Additionally, both the `RegistryExplorer`
/ `RECmd` utilities can apply transaction log files, for example
`ntuser.dat.LOG1`, to identify and recover deleted keys/values.

`RegistryExplorer` implements a number of
[bookmarks](https://github.com/EricZimmerman/RegistryExplorerBookmarks) of
well-known key to facilitate analysis.

`RECmd` can search for strings or regex in key and values names, value
data, and value slack space, in the specified registry hives directory
(processed recursively).

`RegistryExplorer` and `RECmd` can use
[EricZimmerman's RegistryPlugins](https://github.com/EricZimmerman/RegistryPlugins)
to access and parse specific keys. For instance, the `UserAssist` plugin
automatically decode the program names from their `ROT13` encoded format.

```bash
# Uses the given plugin to parse the specified hive or the hives in the specified directory.
# Registry plugin example: \BatchExamples\RegistryASEPs.reb

RECmd.exe [-f <HIVE_FILE> | -d <NTFS_VOLUME | FOLDER_CONTAINING_REGISTRY_HIVES>] --bn <REGISTRY_PLUGIN> --csv <OUTPUT_FOLDER>
```

A number of `KAPE` module can execute `RECmd` with various plugins. The
`RECmd_AllBatchFiles` compound module execute all available `RECmd` modules.

### RegRipper

[RegRipper](https://github.com/keydet89/RegRipper3.0).

### References

  - [AboutDFIR - Registry Explorer/RECmd](https://aboutdfir.com/toolsandartifacts/windows/registry-explorer-recmd/)
