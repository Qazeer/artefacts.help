---
title: Basic system information
summary: 'Various information about the local system are stored as text files: computer hostname, system timezone, mounted filesystems, DNS nameservers, etc.'
keywords: etc, hostname, timezone, fstab, DNS, nameservers, resolv.conf, openresolv, systemd-resolved, hosts, environment
tags:
  - linux_system_information
default_location: '/etc/hostname\n /etc/timezone\n /etc/fstab\n /etc/resolv.conf\n /etc/hosts\n /etc/environment'
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
| `/etc/timezone` <br><br> `/etc/adjtime` <br><br> `/etc/localtime` | Contains the timezone of the system. |

### Mounted filesystems

| File(s) | Description |
|---------|-------------|
| Configuration: <br> `/etc/fstab` <br><br> Mount logs (such as `Mounting` operation / keyword): <br> `/var/log/dmesg` | Contains information on the filesystems, their mountpoints, and mount options. Notably include the partition types (ext3 / ext4, etc.) of mounted filesystems. |

### DNS nameservers

| File(s) | Description |
|---------|-------------|
| `/etc/resolv.conf` | Contains the DNS nameservers and can be managed a number of ways: <br> - As a static file, with hardcoded DNS nameservers. <br> - By the `DHCP` daemon / client directly. <br> - By a middleman component between the network configuration services and the `/etc/resolv` file. They are several implementations of this intermediary: `openresolv`'s `resolvconf` package, Debian's `resolvconf` package, `systemd`'s `systemd-resolved` service, etc. <br><br> The `systemd-resolved` daemon is configured through the `/etc/systemd/resolved.conf` file. This file contains the daemon parameters, DNS nameservers, fallback servers, and other DNS resolver configuration. The `openresolv` package stores the same information in the `/etc/resolvconf.conf` file. <br><br> The `resolv.conf` contain DNS nameserver entries, specified by the `nameserver` directive (such as `nameserver 1.1.1.1`), and can contain `domain` or `search` entries. A `domain` entry specifies which default domain name to append to names that do not end with a `.`. A `search` entry defines the list of domains to search when resolving a name. |

### Static domain name resolutions (hosts)

| File(s) | Description |
|---------|-------------|
| `/etc/hosts` | Contains static domain name resolution(s) to map hostnames to IP addresses with out relying on `DNS` queries. Entries in the `/etc/hosts` take priority over `DNS` resolution. |

### System-wide environment variables

| File(s) | Description |
|---------|-------------|
| `/etc/environment` <br> `/etc/security/environ` | Contains, on some distributions, the system-wide environment variables, defined as key-value pairs. <br><br> The `/etc/environment` is notably linked to the `Pluggable Authentication Modules (PAM)` component. When a `PAM-aware` program (such as `su` or `sudo`) is executed, the program loads `PAM`, which in turn searches under `/etc/pam.d` for a configuration file with the same name as the launched program (such as `/etc/pam.d/su`). If the `pam_env` module is specified (i.e `session required pam_env.so readenv=1`), the environnement variables are loaded from the `/etc/environment`. |

### References

  - [openresolv](https://roy.marples.name/projects/openresolv)

  - [IBM - resolv.conf File Format for TCP/IP](https://www.ibm.com/docs/en/aix/7.3.0?topic=formats-resolvconf-file-format-tcpip)

  - [chicoree.fr - Environment (/etc/environment)](https://www.chicoree.fr/w/Environment_(/etc/environment))
