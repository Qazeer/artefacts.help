---
title: Application Compatibility Cache / Shimcache
summary: 'Application compatibility feature that aim to maintain support of existing software to new versions of the Windows operating system.\n\nA Shimcache entry is created whenever a program is executed from a specific path. However, starting from Windows Vista and Windows Server 2008, entries may also be created for files in a directory that is accessed interactively.\n\nStores up to 1024 entries starting from the Windows Vista and Windows Server 2008 operating systems.\n\nInformation of interest: file full path, LastModifiedTime ($Standard_Information) timestamp of the file at the time of execution, the cache entry position (insertion position in the Shimcache), and from Windows Vista / Windows Server 2008 up to Windows 8.1 / Windows Server 2012 R2, an (undocumented) execution flag.'
keywords: Shimcache, Application Compatibility Cache, shimming
tags:
  - windows_program_execution
  - windows_file_knowledge
location: 'SYSTEM registry hive.\n\nRegistry keys:\n\n>= Windows Server 2003 and Windows XP 64-bit:\nHKLM\SYSTEM\CurrentControlSet\Control\Session Manager\AppCompatCache\AppCompatCache\n\nWindows XP 32-bit:\nHKLM\SYSTEM\CurrentControlSet\Control\Session Manager\AppCompatibility\AppCompatCache'
last_updated: 2024-01-01
sidebar: sidebar
permalink: windows_shimcache.html
folder: windows
---

### Overview

The `Application Compatibility Cache`, also known as `Shimcache`, was
introduced in `Windows XP` as part of the `Application Compatibility
Infrastructure (Shim Infrastructure)` feature. The `Shim Infrastructure` is
designed to identify application compatibility issues and maintain support of
existing software to new versions of the `Windows` operating system. As stated
in the Microsoft documentation, the `Shim Infrastructure` "implements a form of
application programming interface (API) hooking" in order to redirect API calls
made by an application to an alternative library containing stub functions,
known as the `Shim`. The process of making an application compatible to a new
version of Windows through `Shims` is referred to as "`shimming`".

As a part of this framework, the `Application Compatibility Database`
references the applications that have known `shimming` solutions. Upon
execution of an application, the `Shim Engine` will query this database to
determine whether the applications require `shimming`. The `Shimcache` contains
metadata about the files that have been subject to such lookup, for
optimizing and improve the speed of eventual later lookups.

A `Shimcache` entry is created whenever a program is executed from a specific
path. However, starting from the `Windows Vista` and `Windows Server 2008`
operating systems, **entries may also be created for files in a directory that
is accessed interactively**. Indeed, browsing a directory using `explorer.exe`
will generate `Shimcache` entries for the executables stored within the
directory (if the executable was visible in the `Windows Explorer` windows).

**`Shimcache` entries are only written to the registry upon shutdown of the
system. The `Shimcache` entries generated since the last system boot are
thus only stored in memory.**

While the `Shimcache` entry is not removed upon deletion of the associated
file, `Shimcache` entries may be overwritten and information lost as the oldest
entries are replaced by new data. A maximum of 96 `Shimcache` entries are
stored in `Windows XP` / `Windows Server 2003` and up to 1024 entries starting
can be stored starting from the `Windows Vista` and `Windows Server 2008`
operating systems. `Shimcache` entries roll over depending on the level of
activity on the system, with historical data that can extend up to a few months
on Windows server with minimal user activity.

### Information of interest

Each `Shimcache` entries contain the following information, varying depending
on the version of the Windows operating system in use:

  - The associated **file full path**.

  - On `Windows 2003 and XP 64-bit` and older, **the file size**.

  - The **`LastModifiedTime` (`$Standard_Information`) timestamp of the file at
    the time of execution**, which **does not necessarily reflect the execution
    time**. Indeed, `Shimcache` entries are not directly associated with an
    insert / executed timestamp.
    [Some executables](https://github.com/WithSecureLabs/chainsaw/blob/master/analysis/shimcache_patterns.txt),
    such as `PsExesvc`, have their `LastModifiedTime` that corresponds to their
    timestamp of execution (as the binary are automatically downloaded /
    uploaded and executed).

  - The cache entry position, as a numerical value starting from 0, which
    represents the insertion position in the `Shimcache`.
    **The lower the value, generally the more recently the program was
    shimmed.** However, `Shimcache` entries can sometimes be updated in place,
    with out generating a new entry with a lower cache entry position.

  - From `Windows Vista` / `Windows Server 2008` up to `Windows 8.1` /
    `Windows Server 2012 R2`, the (undocumented) `Insert Flag` flag which, when
    set, seems to indicate that the entry was executed. This flag is no
    longer present starting from Windows 10 / Windows Server 2016, and thus a
    `Shimcache` entry does not necessarily reflect an execution** (as entries
    may also be created for files in a directory that is accessed
    interactively).

  - On `Windows XP 32-bit`, the file `Last Update Time` timestamp.

### Tool(s)

#### Entries stored on disk

[`AppCompatCacheParser`](https://github.com/EricZimmerman/AppCompatCacheParser)
tool (`KAPE` associated module `AppCompatCacheParser`) and the
[`ShimCacheParser.py`](https://github.com/mandiant/ShimCacheParser) Python
script can be used to parse `Shimcache` entries.

By default, both tools will parse all the `ControlSet` found in the `SYSTEM`
hive.

```bash
# Parses the live system Registry.
AppCompatCacheParser.exe --csv <OUTPUT_FOLDER>

python ShimCacheParser.py --local -o <OUTPUT_FILE>

# Parses the specified SYSTEM hive.
# --nl: option to force the parsing of the hive even if the even is in a "dirty" state and no transaction logs are available.
AppCompatCacheParser.exe [--nl] -f <SYSTEM_HIVE_FILE> --csv <OUTPUT_FOLDER>

python ShimCacheParser.py [--hive <SYSTEM_HIVE_FILE> | --reg <EXPORTED_SYSTEM_FILE>] -o <OUTPUT_FILE>
```

[Chainsaw](https://github.com/WithSecureLabs/chainsaw) can be used to draw a
timestamp based timeline of `Shimcache` entries using
[novel techniques](https://labs.withsecure.com/tools/chainsaw-analyse-shimcache).

```bash
# tspair: Enable near timestamp pair detection between shimcache and amcache for finding additional insertion timestamps for shimcache entries.

chainsaw analyse shimcache <SYSTEM_HIVE> --regexfile ./analysis/shimcache_patterns.txt [--amcache <AMCACHE_HIVE> --tspair] --output <OUTPUT_CSV>
```

#### Entries only present in memory

The `Volatility2`'s `shimcache` plugin can be used to extract the `Shimcache`
entries living in memory (generated since the last system boot).

```bash
vol.py -f <MEMORY_IMAGE> --profile=<PROFILE> shimcache
```

### References

  - [Microsoft - Application Compatibility Database](https://docs.microsoft.com/en-us/windows/win32/devnotes/application-compatibility-database)

  - [Mandiant - TIMOTHY PARISI - Caching Out: The Value of Shimcache for Investigators](https://www.fireeye.com/blog/threat-research/2015/06/caching_out_the_val.html)

  - [FireEye - Shimcache whitepaper](https://www.fireeye.com/content/dam/fireeye-www/services/freeware/shimcache-whitepaper.pdf)

  - [Alex Ionescu - Secrets of the Application Compatilibity Database (SDB) – Part 1](http://www.alex-ionescu.com/?p=39)

  - [LIFARS - Amcache and Shimcache Forensics](https://lifars.com/wp-content/uploads/2017/03/Technical_tool_Amcache_Shimcache.pdf)

  - [WithSecure - Markus Tuominen & Mehmet Mert Surmeli - Unleashing the Power of Shimcache with Chainsaw](https://labs.withsecure.com/tools/chainsaw-analyse-shimcache)
