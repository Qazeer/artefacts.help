---
title: utmp, wtmp and btmp
summary: 'The utmp, wtmp and btmp files track successful and failed logins on the system. They are are notably maintained by the login(1), init(1), sshd(8), and some versions of the getty(8) programs.\n\n The utmp entry format is specified in the "utmp.h" header. This format is used by the utmp, wtmp, and btmp logs. The username and the remote hosts hostname or IP address are notably recorded for remote logins.\n\n tmp login records are not stored in clear-text and must be parsed with adequate utilities, such as utmpdump or Dissect.'
keywords: utmp, wtmp, btmp
tags:
  - linux_lateral_movement
  - linux_lateral_movement_dst
default_location: 'Linux:\n /var/run/utmp\n /var/log/wtmp\n /var/log/btmp\n\n Solaris:\n (deprecated) /var/adm/utmp\n /var/adm/utmpx\n (deprecated) /var/adm/wtmp\n /var/adm/wtmpx\n\n FreeBSD 9.0:\n /var/run/utx.active (utmp equivalent)\n /var/log/utx.log (wtmp equivalent)'
last_updated: 2024-08-18
sidebar: sidebar
permalink: linux_utmp_wtmp_btmp.html
folder: linux
---

### Overview

The `utmp`, `wtmp` and `btmp` files track successful and failed logins on the
system. They are are notably maintained by the `login(1)`, `init(1)`,
`sshd(8)`, and some versions of the `getty(8)` programs. However, none of these
programs creates the `wtmp` and `btmp` files, so if the files are deleted,
record-keeping is effectively turned off.

The following `*tmp` login record files are commonly defined on Linux
distributions:

  - `utmp`/`utmpx` (under `/var/run/`): currently logged users. As
    `/var/run/` (often a symlink to `/run/`) is a `tmpfs` filesystem in RAM,
    the `utmp`/`utmpx` files do not persist across system shutdowns.

    The `utmpx` and `utmp` files are read by the `who(1)` utility to list the
    user(s) currently logged-in on the system.

  - `wtmp`/`wtmpx` (under `/var/log/`): all current and past logins, with
    additional details on system shutdown and reboots, etc. All entries are
    thus not necessarily related to user authentication.

  - `btmp`/`btmpx` (under `/var/log/`): all bad/failed login attempts.

The `*tmpx` files are extended database files that supersede the `*tmp` files
on some distributions.

### Information of interest

#### utmp record format

The `utmp` entry format is specified in the `utmp.h` (`/usr/include/`) or
`bits/utmp.h` header. This format is used by the `utmp`, `wtmp`, and `btmp`
logs.

The fields available depend on the `utmp` implementation for the given system
and may vary between distributions and `utmp.h` versions/implementations.

| Field | Description |
|-------|-------------|
| `ut_type` | The type of login associated with the record. <br><br> The following type values are defined: <br>  - `EMPTY` (0): No valid user accounting information. <br> - `RUN_LVL` (1): The system's runlevel. <br> - `BOOT_TIME` (2): Time of system boot. <br> - `NEW_TIME` (3): Time after system clock changed. <br> - `OLD_TIME` (4): Time when system clock changed. <br> - `INIT_PROCESS` (5): Process spawned by the init process. <br> - `LOGIN_PROCESS` (6): Session leader of a logged in user. <br> - `USER_PROCESS` (7): Normal process. <br> - `DEAD_PROCESS` (8): Terminated process. <br><br> The `login(1)` program creates records of `USER_PROCESS` type after a user has successful authenticated, populating the `ut_host` and `ut_addr` fields. Terminal emulators may also directly create `USER_PROCESS` records. |
| `ut_pid` | The `process ID` of the process associated with the record. |
| `ut_line` & `ut_id` | The name of the (native `tty` or pseudo `pts`) terminal device associated with the record (such as `tty1` or `pts/3`). <br> `ut_line` is set to `~` and `ut_id` to `~~` for records related to system shutdown or reboot. |
| `ut_name`/`ut_user` | The username or technical information associated with the record. <br><br> The username is set to: <br> - `shutdown` for system shutdown <br> - `reboot` for system reboot. <br> - `LOGIN` for `LOGIN_PROCESS` records. <br> - `runlevel` for `RUN_LVL` record related to `init(1)`. <br> - The username of the user associated with the login for `USER_PROCESS` records. |
| `ut_host` | The source host's hostname or IP address set for `USER_PROCESS` records linked to a remote login (set to `:0` for local logins) or the kernel version for `RUN_LVL` records. |
| `ut_addr` | The source host's IP address set for `USER_PROCESS` records linked to a remote login (or set to `0.0.0.0` otherwise). |
| `ut_time`/`ut_tv` | The timestamp of the record, with up to microseconds precision depending on the system architecture (`32-bit` or `64-bit`) and `utmp.h` versions/implementation. |

### Tool(s)

`*tmp` login records are not stored in clear-text and must be parsed with
adequate utilities:

  - The `utmpdump` utility can be used to parse `*tmp` logs into an ascii
    table.

    ```bash
    utmputmpdump [-o <OUTPUT_FILE>] <INPUT_FILE>

    # Output format:
    # [ut_type] [ut_pid] [ut_id] [ut_user] [ut_line] [ut_host] [ut_addr] [ut_time]
    ```

  - The `target-query` tool, part of the
    [`dissect` Python framework](https://docs.dissect.tools/en/latest/index.html),
    can be used to parse `wtmp` and `btmp` logs in `CSV` or `JSON` outputs.

    ```bash
    # To parse a single wtmp or btmp file, the files must be under <TARGET>/var/log/

    target-query -f <wtmp | btmp> <TARGET> | rdump <--csv | --json | --jsonlines>
    ```

### References

  - [Linux manual page - utmp(5)](https://man7.org/linux/man-pages/man5/utmp.5.html)

  - [Wikipedia - Utmp](https://en.wikipedia.org/wiki/Utmp)

  - [IBM AIX - utmp.h File](https://www.ibm.com/docs/en/aix/7.3?topic=files-utmph-file)
