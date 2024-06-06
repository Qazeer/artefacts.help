---
title: Hiberfil.sys
summary: 'The hiberfil.sys file is linked to the hibernation, hybrid sleep, and Fast Boot (Windows 8) / Fast Startup (Windows 10) features. Those features are mostly in use on Windows laptops / desktops and are generally not available by default on Windows virtual machines.\n\nAs the hiberfil.sys file is shared by three features, the file can be in various states with its content being either a partial or a full memory snapshot. The partial memory snapshot, linked to the Fast Boot / Fast Startup feature, contains the Windows kernel and session 0 processes memory, notably including the MFT and registry hives.\n\nThe hiberfil.sys file is zeroed out after a system boot starting from Windows 8 / 8.1 and must thus be retrieved from a powered off system.\n\nThe structure of the hiberfil.sys file has evolved starting with Windows 8. Both the old and new formats can be analyzed using volatility, by first converting the hibernation file to a raw file using volatilty3 windows.hibernation.Dump plugin.'
keywords: hibernation, hiberfil.sys, memory, sleep, fast boot, fast startup, ClearPageFileAtShutdown, volatility, hibernation recon, hibernation.Dump
tags:
  - windows_misc
location: '<SYSTEMDRIVE>:\hiberfil.sys'
last_updated: 2024-03-13
sidebar: sidebar
permalink: windows_hiberfilsys.html
folder: windows
---

### Overview

The `hiberfil.sys` file is linked to the hibernation, hybrid sleep, and
`Fast Boot` (`Windows 8`) / `Fast Startup` (`Windows 10`) features. Those
features are mostly in use on Windows laptops / desktops and are generally not
available by default on Windows virtual machines (and require the hibernation
feature to be implemented at the hypervisor level).

As the `hiberfil.sys` file is shared by three different (but similar) features,
the file can be in different states:

  - `Hybernation`: full main memory snapshot, created following a user
    triggered hibernation.

  - `Hybrid sleep`: full main memory snapshot, combination of the sleep and
    hibernation states. The main memory is written to the `hiberfil.sys` file,
    then the system enters a sleep mode. If power is lost during sleep, the
    system uses the `hiberfil.sys` file to boot and restore the system state.

    Available since `Windows Vista`,
    [`hybrid sleep` is on by default for desktop systems but off by default on laptops](https://devblogs.microsoft.com/oldnewthing/20110510-00/?p=10703)
    and requires the support of hibernation (and is thus not generally
    available on virtual machines).

  - `Fast Boot` / `Fast Startup`: partial memory snapshot, that contains the
    memory of the Windows kernel and of the `session 0` processes (background
    system services notably). `Fast startup` is a type of shutdown that uses an
    hibernation file to speed up the subsequent boot, with user(s) being logged
    off before the hibernation file is created. In this state, the
    `hiberfil.sys` file will notably contain `MFT` file and `INDX` records, and
    registry hives
    ([`SYSTEM` only after `Windows 10 Build 17134`](https://arsenalrecon.com/products/hibernation-recon/faqs)).

    `Fast Boot` / `Fast Startup` is enabled by default, but requires support of
    hibernation (and is thus not generally available on virtual machines).

The `hiberfil.sys` file is zeroed out after a system boot starting from
`Windows 8 / 8.1`, and may also be zeroed out on system shutdown if the
`ClearPageFileAtShutdown` registry setting is enabled (set to `0x1`). As such,
the `hiberfil.sys` file must be retrieved from a powered off system.

The structure of the `hiberfil.sys` file have evolved starting with
`Windows 8`, with notable changes in the compression methods used. There is
thus currently two possible formats:

  - The "old" format, starting from `Windows XP` to `Windows 7`.

  - The "new" format, starting from `Windows 8` to `Windows 11`.

### Tool(s)

Both `hiberfil.sys` file formats can be processed with
[Hibernation recon](https://arsenalrecon.com/downloads)
and ([more recently](https://www.forensicxlab.com/posts/hibernation/))
`volatility2` / `volatility3` to convert the hibernation file to a raw file.

Once converted, the resulting image can be analyzed as a standard memory
image (with potentially less information however) using tools such as
`volatility` and `MemProcFS`.

```bash
# volatility2 for hibernation files in the old format.

# Prints basic information about the hibernation file.
volatility -f <HIBERNATION_FILE> --profile=<PROFIL> hibinfo

# Converts the hibernation file to a raw file.
volatility -f <HIBERNATION_FILE> --profile=<PROFIL> imagecopy -O <OUTPUT>

# volatility3 for hibernation files in the new format.

# Prints basic information about the hibernation file.
volatility3 -f <HIBERNATION_FILE> windows.hibernation.Info

# Converts the hibernation file to a raw file.
# The version to specify depends on the Windows version targeted (Windows 8/8.1 to Windows 11 23H2).
# Possible values can be checked using windows.hibernation.Dump -h.
volatility3 -f <HIBERNATION_FILE> windows.hibernation.Dump --version <VERSION>
```

### References

  - [FORENSICXLAB - Volatility3: Modern Windows Hibernation file analysis](https://www.forensicxlab.com/posts/hibernation/)

  - [ArsenalRecon - Hibernation Recon FAQs](https://arsenalrecon.com/products/hibernation-recon/faqs)
