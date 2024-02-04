---
title: User and group identifiers
summary: 'Each process is associated with a set of user and group identifiers: uid / gid, euid / egid, ruid / rgid, suid / sgid, and fsuid / fsgid.'
keywords:
tags:
  - linux_system_information
last_updated: 2024-02-04
sidebar: sidebar
permalink: linux_user_and_group_identifiers.html
folder: linux
---

### Overview

Each Linux process has its own credentials / identity, which include various
user and group identifiers.

| Identifier | Description |
|------------|-------------|
| `uid` <br><br> `gid` | `user identifier` and `group identifier`. <br><br> The `uid` and `gid` are uniquely linked, respectively, to a specific  user and group. The association between the `uid` / `gid` and textual usernames / group names is defined in the `/etc/passwd` and `/etc/group` files. <br><br> In addition to being used as identifiers in process credentials, the `uid` and `gid` are also stored in the `inodes` of the file system. |
| `euid` <br><br> `egid` | `effective user identifier` and `effective group identifier`. <br><br> The `euid` and `egid` of the process are used for most access checks and as the user / group owner for files created by that process. <br><br> The `euid` and `egid` should generally be equal, respectively, to the `uid` and `gid` of the user associated with the process. <br><br> Executing a `SUID` or `SGID` binary (i.e. a binary with its `SUID` / `SGID` bit set) will result in the new process `euid` / `egid` to be set to the user / group owner of the `SUID` / `SGID` file. <br><br> `euid` != `uid` can thus be an indication of privilege escalation through a `SUID` binary. |
| `ruid` <br><br> `rgid` | `real user identifier` and `real group identifier`. <br><br> The `ruid` and `rgid` are inherited from the parent process and represent the "real" identifiers of the process. Only root can change the `ruid` or `rgid` of a process. <br><br> If executing a `SUID` or `SGID` binary, the `ruid` and `rgid` of the new process will remain inherited from the invoking process. <br><br> If calling a command through `sudo` (a `SUID` binary owned by root): <br> - The `sudo` process itself will run with the `euid` of root and the `ruid` of the invoking process. <br> - The command executed through `sudo` will run with `euid`, `egid`, `ruid`, and `rgid` set to root, as `sudo` sets the `ruid` and `rgid` to 0. |
| `suid` <br><br> `sgid` | `saved user identifier` and `saved group identifier`. <br><br> The `suid` and `sgid` are used as a way to allow a privileged process to temporarily downgrade privileges and revert back to full privileges. Setting `euid` from a privileged value (typically 0 / root) to a lower privilege value (typically != 0 / root), will result in the `suid` of the process to be set to root. As unprivileged process may set their `euid` from their `suid` (and `uid` or `ruid` as well) value, the unprivileged process can revert back to full privilege. <br><br> Executing a `SUID` or `SGID` binary will result in the new process `suid` / `sgid` to be set to the user / group owner of the `SUID` / `SGID` file. |
| `fsuid` <br><br> `fsgid` | `filesystem user identifier` and `filesystem group identifier`. <br><br> The `fsuid` and `fsgid` are used specifically for filesystem access checks, and should, unless set otherwise, be equal to respectively `euid` and `egid`. <br><br> `fsuid` and `fsgid` mostly remain for retro-compatibility reason as they are no longer necessary since `Linux Kernel 2.0`. |

### Administrative operations and resulting identifiers

```bash
# standard user (uid / gid 1000) logged-in.
ps -o uid,euid,ruid,suid,fsuid,egid,rgid,sgid
    UID  EUID  RUID  SUID FSUID  EGID  RGID  SGID
   1000  1000  1000  1000  1000  1000  1000  1000

# root user logged-in.
ps -o uid,euid,ruid,suid,fsuid,egid,rgid,sgid
    UID  EUID  RUID  SUID FSUID  EGID  RGID  SGID
      0     0     0     0     0     0     0     0

# SUID (user owner "root", uid 0) ps, executed as user with uid 1000.
# sudo cp /usr/bin/ps /tmp/ps & sudo chown root /tmp/ps & sudo chmod u+s /tmp/ps
# /tmp/ps -rwsr-xr-x root user
/tmp/ps -o uid,euid,ruid,suid,fsuid,egid,rgid,sgid
    UID  EUID  RUID  SUID FSUID  EGID  RGID  SGID
      0     0  1000     0     0  1000  1000  1000

# SUID (user owner "test", uid 1001) ps, executed as user with uid 1000.
# sudo cp /usr/bin/ps /tmp/ps & sudo chown test /tmp/ps & sudo chmod u+s /tmp/ps
# /tmp/ps -rwsr-xr-x test user
/tmp/ps -o uid,euid,ruid,suid,fsuid,egid,rgid,sgid
    UID  EUID  RUID  SUID FSUID  EGID  RGID  SGID
   1001  1001  1000  1001  1001  1000  1000  1000

# SGID (group owner "root", gid 0) ps, executed as user with uid 1000.
# sudo cp /usr/bin/ps /tmp/ps & sudo chown user:root /tmp/ps & sudo chmod g+s /tmp/ps
# /tmp/ps -rwxr-sr-x user root
ps -o uid,euid,ruid,suid,fsuid,egid,rgid,sgid
    UID  EUID  RUID  SUID FSUID  EGID  RGID  SGID
   1000  1000  1000  1000  1000     0  1000     0

# SGID (group owner "test", gid 1001) ps, executed as user with uid 1000.
# sudo cp /usr/bin/ps /tmp/ps & sudo chown user:test /tmp/ps & sudo chmod g+s /tmp/ps
# /tmp/ps -rwxr-sr-x user test
ps -o uid,euid,ruid,suid,fsuid,egid,rgid,sgid
    UID  EUID  RUID  SUID FSUID  EGID  RGID  SGID
   1000  1000  1000  1000  1000  1001  1000  1001

# sudo, executed as user with uid 1000.
sudo ps -o cmd,uid,euid,ruid,suid,fsuid,egid,rgid,sgid
    CMD                           UID  EUID  RUID  SUID FSUID  EGID  RGID  SGID
    sudo ps -o cmd,uid,euid,rui     0     0  1000     0     0     0     0     0
    ps -o cmd,uid,euid,ruid,sui     0     0     0     0     0     0     0     0

# su (root).
su root
ps -o uid,euid,ruid,suid,fsuid,egid,rgid,sgid
    UID  EUID  RUID  SUID FSUID  EGID  RGID  SGID
      0     0     0     0     0     0     0     0
```

### References

  - [Wikipedia - User identifier](https://en.wikipedia.org/wiki/User_identifier)

  - [stackoverflow - Difference between Real User ID, Effective User ID and Saved User ID - Answer Flow & Rotem jackoby](https://stackoverflow.com/questions/32455684/difference-between-real-user-id-effective-user-id-and-saved-user-id)
