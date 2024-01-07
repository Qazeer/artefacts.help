---
title: Registry - Services
summary: 'The Services registry key hold the configuration information of the installed Windows services.\n\nInformation of interest, for each service: service name and display name, image or DLL path, service type, service start mode, and eventual Windows privileges required.\n\nThe timestamp of a service creation, or last configuration update, can be deduced from the last write timestamp of its registry key.'
keywords: 'Services, installed services, windows service, svchost'
tags:
  - windows_local_persistence
  - windows_registry
location: 'File: <SYSTEMROOT>\System32\config\SYSTEM\n\nRegistry keys: HKLM\SYSTEM\CurrentControlSet\Services\<SERVICE_NAME>'
last_updated: 2024-01-07
sidebar: sidebar
permalink: windows_registry_services.html
folder: windows
---

### Overview

The `Services` registry key **hold the configuration information of the
installed Windows services**: name, display name, image path, start mode,
service type, required privileges if any, etc.

### Information of interest

**Each service configuration is defined in a dedicated subkey** under
`Services`, **identified by the service name**. The last write timestamp of the
service sub key indicates the service creation or last modification timestamp.

For each service, the following notable information is available (under the
service name root key):

  - Service name and display name.

  - Service image path.

    Services implemented as `Dynamic Link Library (DLL)` will usually have
    their image path set to
    `%SystemRoot%\system32\svchost.exe -k <SERVICE_HOST_GROUP>`,
    with the `-k` flag defining the `Service Host Groups` of the service.
    As stated in the Microsoft documentation, the `Service Host`
    (`svchost.exe`) is a shared-service process that serves as a shell for
    loading services from `DLL` files. Services are organized into related
    `Service Host Groups`, and each group runs inside a different instance of
    the `Service Host` process.
    The list of services defined in a `Service Host Group` is set in the
    `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Svchost` registry key.

    Two other argument may be specified to the `Service Host` process:
      - `-s <SERVICE_NAME>`: to only load the specified service from the
        given `Service Host Group`.
      - `-p`: [to enforce different policies: `DynamicCodePolicy`,
        `BinarySignaturePolicy` and `ExtensionPolicy`](https://pusha.be/index.php/2020/05/07/exploration-of-svchost-exe-p-flag/).

  - The service type:
    - `0x1`: Kernel driver.
    - `0x2` / `0x8`: File system driver.
    - `0x10`: Standard Windows service that runs in a process by itself.
    - `0x20`: Windows service that can share a process with other services.
    - `0x50`: "USER_OWN_PROCESS TEMPLATE".
    - `0x60`: "USER_SHARE_PROCESS TEMPLATE".
    - `0x110`: Similar to `0x10`, but can interact with users.
    - `0x120`: Similar to `0x20`, but can interact with users.

 - The service start mode:
   - `0x0`: "Boot Start"
   - `0x01`: "System Start"
   - `0x02`: "Auto Start"
   - `0x03`: "Manual"
   - `0x04`: "Disabled"

  - The Windows specific privileges required by the service
    (`SeImpersonatePrivilege`, `SeDebugPrivilege`, etc.). No specific
    privileges may also be set, for example if the service runs as
    `NT AUTHORITY\SYSTEM`.

### References

  - [Microsoft - Service host grouping in Windows 10](https://learn.microsoft.com/en-us/windows/application-management/svchost-service-refactoring)

  - [Nasreddine Bencherchali - Demystifying the “SVCHOST.EXE” Process and Its Command Line Options](https://nasbench.medium.com/demystifying-the-svchost-exe-process-and-its-command-line-options-508e9114e747)

  - [Superuser - What command line options are available to svchost.exe?](https://superuser.com/questions/391864/what-command-line-options-are-available-to-svchost-exe)

  - [pusha.be - med - Exploration of svchost.exe /P flag](https://pusha.be/index.php/2020/05/07/exploration-of-svchost-exe-p-flag/)
