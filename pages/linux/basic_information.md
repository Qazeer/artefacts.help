---
title: Basic system information
summary: 'A variety of information about the local system is stored as text files: computer hostname, system timezone, mounted filesystems, DNS nameservers, etc.'
keywords: etc, hostname, timezone, localtime, adjtime, fstab, DNS, nameservers, resolv.conf, openresolv, systemd-resolved, hosts, environment
tags:
  - linux_system_information
default_location: '/etc/hostname\n /etc/localtime\n /etc/timezone\n /etc/fstab\n /etc/resolv.conf\n /etc/hosts\n /etc/environment'
last_updated: 2025-06-24
sidebar: sidebar
permalink: linux_basic_information.html
folder: linux
---

### Hostname

| File(s) | Description |
|---------|-------------|
| `/etc/hostname` | Contains the hostname of the system. |

### Timezone

| File(s) | Description |
|---------|-------------|
| `/etc/localtime` <br><br> `/etc/timezone` <br><br> `/etc/adjtime` | `/etc/localtime`: configures the timezone of the system. Symbolic link to the binary timezone data file (`tzfile` format) of the configured timezone, under `/usr/share/zoneinfo/`. The timezone name can thus be determined from the link target (such as `/usr/share/zoneinfo/Europe/Paris`). <br><br> `/etc/timezone`: legacy file of `Debian`-based distributions that contains the name of the timezone of the system (such as `Europe/Paris`). The file is no longer created by the `tzdata` package since version `2024b-5` (and by the `Debian 13 (Trixie)` installer), and is removed upon upgrade to version `2025c-2` or later. <br><br> `/etc/adjtime`: does not contain the timezone, but records whether the hardware clock (`RTC`) is set to `UTC` or to the local time (`UTC` or `LOCAL` on the third line of the file). |

### Mounted filesystems

| File(s) | Description |
|---------|-------------|
| Configuration: <br> `/etc/fstab` <br><br> Mount logs (such as `Mounting` operation / keyword): <br> `/var/log/dmesg` | Contains information on the filesystems, their mount points, and mount options. Notably includes the partition types (ext3 / ext4, etc.) of mounted filesystems. |

### DNS nameservers

| File(s) | Description |
|---------|-------------|
| `/etc/resolv.conf` | Contains the DNS nameservers and can be managed in a number of ways: <br> - As a static file, with hardcoded DNS nameservers. <br> - By the `DHCP` daemon / client directly. <br> - By a middleman component between the network configuration services and the `/etc/resolv.conf` file. There are several implementations of this intermediary: `openresolv`'s `resolvconf` package, Debian's `resolvconf` package, `systemd`'s `systemd-resolved` service, etc. <br><br> The `systemd-resolved` daemon is configured through the `/etc/systemd/resolved.conf` file. This file contains the daemon parameters, DNS nameservers, fallback servers, and other DNS resolver configuration. The `openresolv` package stores the same information in the `/etc/resolvconf.conf` file. <br><br> The `resolv.conf` file contains DNS nameserver entries, specified by the `nameserver` directive (such as `nameserver 1.1.1.1`), and can contain `domain` or `search` entries. A `domain` entry specifies which default domain name to append to names that do not end with a `.`. A `search` entry defines the list of domains to search when resolving a name. |

### Static domain name resolutions (hosts)

| File(s) | Description |
|---------|-------------|
| `/etc/hosts` | Contains static domain name resolution(s) to map hostnames to IP addresses without relying on `DNS` queries. Entries in the `/etc/hosts` file take priority over `DNS` resolution. |

### System-wide environment variables

| File(s) | Description |
|---------|-------------|
| `/etc/environment` <br> `/etc/security/environ` | Contains, on some distributions, the system-wide environment variables, defined as key-value pairs. <br><br> The `/etc/environment` file is notably linked to the `Pluggable Authentication Modules (PAM)` component. When a `PAM-aware` program (such as `su` or `sudo`) is executed, the program loads `PAM`, which in turn searches under `/etc/pam.d` for a configuration file with the same name as the launched program (such as `/etc/pam.d/su`). If the `pam_env` module is specified (for example, `session required pam_env.so readenv=1`), the environment variables are loaded from the `/etc/environment` file. |

### References

  - [openresolv](https://roy.marples.name/projects/openresolv)

  - [IBM - resolv.conf File Format for TCP/IP](https://www.ibm.com/docs/en/aix/7.3.0?topic=formats-resolvconf-file-format-tcpip)

  - [chicoree.fr - Environment (/etc/environment)](https://www.chicoree.fr/w/Environment_(/etc/environment))
