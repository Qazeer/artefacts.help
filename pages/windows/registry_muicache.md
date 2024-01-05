---
title: Registry - MUICache
summary: 'The Multilanguage User Interface (MUI) is a feature to allow applications to have a single executable for multiple languages.\n\nThe MUICache registry key references GUI program executions only.\n\nInformation of interest: executable full path, executable PE FileDescription attribute (that references the original filename, allowing to identify renamed files), the executable PE CompanyName attribute.\n\nThe MUICache does not provide a timestamp of execution.'
keywords:
tags:
  - windows_program_execution
  - windows_registry
location: 'File:\n<SYSTEMDRIVE>:\Users\<USERNAME>\AppData\Local\Microsoft\Windows\UsrClass.dat\n\nRegistry keys:\nHKCU\Software\Classes\Local Settings\Software\Microsoft\Windows\Shell\MUICache\nHKCU\Local Settings\MuiCache'
last_updated: 2024-01-05
sidebar: sidebar
permalink: windows_registry_muicache.html
folder: windows
---

### Overview

The `Multilanguage User Interface (MUI)` is a feature to allow applications to
have a single executable for multiple languages. `MUI` files can be created,
one per supported language, to switch the application display language.

The registry key references execution of **programs with a graphical
interface**, installed or from a portable executable.

### Information of interest

Each execution is associated with two values under the `MUICache` registry key,
both starting with the executable full path.

The values data reference information retrieved from the executable's
`Version` information from its resources section (`.rsrc`):

  - `<PE_FULL_PATH>.FriendlyAppName`: references the executable
    `FileDescription`.
    **This can be used to identify renamed executable, as the original filename
    is likely going to be referenced by the `FileDescription` attribute**.

  - `<PE_FULL_PATH>.ApplicationCompany`: references the executable
    `CompanyName`.

**The `MUICache` does not provide a timestamp of execution**, and the last
write timestamp of the key cannot be used to infer the timestamp of execution
(as the entries are stored directly as registry values).

### References

  - [13Cubed - Let's Talk About MUICache](https://www.youtube.com/watch?v=ea2nvxN878s)
