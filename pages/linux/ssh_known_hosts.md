---
title: SSH known_hosts
summary: 'When an SSH connection is being established, the server sends its public host key to the client and proves knowledge of the associated private host key. The OpenSSH client automatically stores the public host key of hosts that were previously accessed. These keys, called "known host keys", are stored in the known_hosts files.\n\n The known_hosts files are plaintext files, with (generally) one known host information per line with the following information: the hostname(s) / IP address(es), the public host key type, the base64-encoded public host key, and an optional comment.\n\n The remote hosts hostname and IP address can be either stored in clear-text or hashed. If the host information is hashed, it is possible to test wether a specific host is present in the known_hosts file, or to brute force the hashes to try to recover the associated hostname / IP addresses.'
keywords: SSH
tags:
  - linux_lateral_movement
  - linux_lateral_movement_src
  - linux_ssh
  - linux_ssh_src
default_location: 'System-wide:\n /etc/ssh/known_hosts\n\n User scoped:\n ~/.ssh/known_hosts'
last_updated: 2024-08-24
sidebar: sidebar
permalink: linux_ssh_known_hosts.html
folder: linux
---

### Overview

When an `SSH` connection is being established, the server sends its public host
key to the client and proves knowledge of the associated private host key. The
host keys are generally generated upon the installation of the `SSH` service
and are used for server-side authentication of the `SSH` server.

The `OpenSSH` client stores the public host key of hosts that were previously
accessed. These keys, called "known host keys", are stored in the `known_hosts`
text-based files. The `known_hosts` file can be system-wide
(`/etc/ssh/known_hosts`) or user scoped (`~/.ssh/known_hosts`).

The system-wide `known_hosts` file can be optionally manually set by a system
administrator, while the user scoped `known_hosts` file is automatically
populated by the `SSH` client. The client indeed add an entry to the per-user
file upon the first access to a given `SSH` server and host key pair, following
the user validation of the server authenticity (through its public host key
fingerprint).

The `ssh-keyscan(1)` utility to can be used to collect the public `SSH` host
key of the specified host to manually add it to the `known_hosts` files:

```bash
# To add the REMOTE_HOST public key to the known_hosts system-wide.

ssh-keyscan -H -t rsa <REMOTE_HOST> >> /etc/ssh/ssh_known_hosts
```

If the legitimacy of the public host key was validated upon the first
connection, the `known_hosts` files can be used to establish the trust with the
remote server in subsequent access in order to protect against
man-in-the-middle network attacks. If the public host key of a known host
changes, the `SSH` client will display an alert.

*The `known_hosts` is one of the few endpoint disk artefacts to identify
outgoing `SSH` connections.*

### Information of interest

The `known_hosts` files are plaintext files, with (generally) one known host
information per line.

Each line contains the following fields: the hostname(s), the public host key
type (`ssh-rsa`, `ssh-ed25519`, `ecdsa-sha2-nistp256`, ...), the base64-encoded
public host key, and an optional comment. The fields are separated by spaces.
An older format stores the public host key's bits, exponent, and modulus
(separated by spaces) instead of the base64-encoded public host key.

The hostname(s) is a single pattern or a comma-separated list of patterns. A
pattern can be a hostname or an IP address, with `*` and `?` acting as
wildcards.

Example of `known_hosts` entries:

```bash
# Entry referencing the known host through "server_hostname" and its IP address 10.0.0.10.
server_hostname,10.0.0.10 ssh-rsa AAAA12[...]KXIOz

# Entry with a (single) hashed known host.
|1|9dM[...]/NZ4U= ssh-rsa AAAAB3Nza[...]D02jEM=
```

#### Hashed known hosts

The remote hosts hostname and IP address can be either stored in clear-text or
hashed if `HashKnownHosts` is set to "yes" in the `SSH` client `ssh_config`
configuration file. Only one hashed hostname may appear on a single line. The
lines containing an hashed hostname start with `|1|`.

Even if the hosts information is hashed, the following command can be used to
check whether the specified hostname is present in the given known hosts file:

```bash
ssh-keygen -l -f <KNOWN_HOST_FILE> -F <HOSTNAME>
```

Additionally, `John the Ripper` can be used to brute force known hosts files,
using for instance the private IP address ranges:

```bash
known_host_file=<KNOWN_HOST_FILE>

john --format=known_hosts $known_host_file

nmap -sL -Pn -n 192.168.0.0/16 172.16.0.0/12 10.0.0.0/8 | grep '^Nmap scan report for' | cut -d ' ' -f 5 > IP_list.txt

john --wordlist=IP_list.txt --format=known_hosts $known_host_file
```

### References

  - [Linux man page - sshd(8)](https://linux.die.net/man/8/sshd)

  - [ssh.com - What are SSH Host Keys?](https://www.ssh.com/academy/ssh/host-key)

  - [Linux handbook - Everything You Important You Should Know About the known_hosts file in Linux](https://linuxhandbook.com/known-hosts-file/)

  - [IBM - ssh_known_hosts file format](https://www.ibm.com/docs/en/zos/3.1.0?topic=daemon-ssh-known-hosts-file-format)
