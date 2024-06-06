---
title: Linux Audit framework (audit logs)
summary: 'The Linux Audit system is an non-default auditing and logging framework that can be configured to log multiple type of operations, such as authentication successes or failures, process executions, file accesses, user commands executed in a TTY, etc.\n\nThe Linux Audit framework implements kernel-mode hooks to monitor user-mode processes and generate audit telemetry. The auditd daemon is the main user-mode component of the Linux Audit framework, that receives audit messages sent by the kernel and other components (such as pam).\n\nThe Linux Audit system operates on rules, that define what records will be captured in the audit logs. If no rules are defined, which is the case by default, only distribution specific records and telemetry from other components may be logged to audit log file by the auditd daemon.\n\nDepending on the rule configured, multiple events can be generated for the same operation. An event can be split in multiple records, with each record of the same event sharing the same timestamp and same unique identifier.\n\nEach record is associated with a given type: USER_AUTH, USER_LOGIN, EXECVE, SYSCALL, OPENAT, PROCTITLE, USER_CMD, TTY, SOCKADDR, etc.'
keywords: Linux Audit, audit, auditd, audit rules, auditctl, aureport, ausearch, audit.log, auditd.conf, audit.rules, auid, USER_AUTH, USER_LOGIN, EXECVE, SYSCALL, OPENAT, PROCTITLE, USER_CMD, TTY, SOCKADDR, msg, dissect, target-query, sigma, Zircolite, ChopChopGo
tags:
  - linux_logging_frameworks
  - linux_program_execution
  - linux_files_and_folders_access
  - linux_file_knowledge
  - linux_lateral_movement
  - linux_lateral_movement_dst
default_location: 'auditd daemon configuration:\n/etc/audit/auditd.conf\n\nAudit rules:\n/etc/audit/audit.rules\n/etc/audit/rules.d/\n\nAudit logs:\n/var/log/audit.log*\n/var/log/audit/audit.log.*.gz'
last_updated: 2024-02-03
sidebar: sidebar
permalink: linux_audit_framework.html
folder: linux
---

### Overview

The `Linux Audit` system is an non-default auditing and logging framework that
can be configured to log multiple type of operations, such as authentication
successes or failures, process executions, file accesses, user commands
executed in a TTY, etc.

The `Linux Audit` framework implements kernel-mode hooks to monitor user-mode
processes and generate audit telemetry. The `auditd` daemon is the main
user-mode component of the `Linux Audit` framework. It receives audit messages
sent by the kernel (via a `netlink socket`) and is responsible for storing the
messages on the file system. Launched at operating system startup, the `auditd`
daemon is configured through the configuration file `/etc/audit/auditd.conf`,
and connects to the socket to receive events from the kernel. The `auditd`
daemon can also received events from other system components, such as `pam` or
`sshd`.

#### Audit rules

The `Linux Audit` system operates on rules, that define what records will be
captured in the audit logs. Audit rules are defined in the
`/etc/audit/audit.rules` file or in the `/etc/audit/rules.d/` directory. One
rule is defined on a single line, as it was arguments to the `auditctl`
utility.

As the events generated are dependent on the audit rules configured, it is
recommended to first review the rules defined on the system before analyzing
the audit logs. By default, no audit rules are configured, apart from eventual
control rules often specific to the Linux distribution used. If no rules are
configured, other components (`pam`, `sshd`, etc.) may still be sending events
to the `auditd` daemon.

There are three types of audit rules:

  - Control rules: to modify the `Linux Audit` system behavior and its
    configuration.

  - File or watch rules: to audit access to files or directories.

    The watch rule format is:
    `-w <FILE | FOLDER> -p <PERMISSION> -k <KEYNAME>`.

    The access type to be audited, specified using the `-p` option, can be:

      - `r`: read of the file.

      - `w`: write to the file.

      - `x`: execute the file.

      - `a`: change in the file's attribute.

    Example:

    ```bash
    # Generates events on modification to the "sudoers" file or files under the "sudoers.d" directory.
    -w /etc/sudoers -p wa -k sudoers_modification
    -w /etc/sudoers.d/ -p wa -k sudoers_modification
    ```

  - System call rules: to audit system calls (by any or the specified program).

    The system call rule format is:
    `-a <ACTION>,<LIST> -S <SYSCALL> -F <FILTER_FIELD>=<FILTER_VALUE> [-F <FILTER_FIELD>=<FILTER_VALUE>] -k <KEY_NAME>`

    Example:

    ```bash
    # Generates events on successful "connect" syscall from "/bin/bash".
    -a always,exit -F arch=b64 -F exe=/bin/bash -F success=1 -S connect -k "remote_shell"
    ```

The [Neo23x0 auditd rules set](https://github.com/Neo23x0/auditd/tree/master)
can be used as a baseline, which may require fine tuning depending on the
environnement and applications running on each system.

### Information of interest

#### auditd events versus records

An `auditd` event can be split in multiple records. Each record of the same
event shares the same timestamp (in the `epoch` format) and same unique
identifier. Each record is associated with a [given type](#record-types).
Records can be sometimes separated by hundreds other unrelated records.

The records are formatted as series of `FIELD = VALUE` pairs separated by a
space. The available fields for `auditd` records, and their description, can be
found in the
[Linux Audit Documentation](https://github.com/linux-audit/audit-documentation/blob/main/specs/fields/field-dictionary.csv).

Multiple events can be generated for a single operation, with each event
potentially comporting multiple records. For example, the `cat /etc/passwd`
command entered by a user in a `TTY` shell may generate:

  - An event for the execution of `cat`.

    Example rule: `-w /usr/bin/cat -p x -k cat_exec`.

  - An event for the access to `/etc/passwd`.

    Example rule: `-w /etc/passwd -p warx -k etc_passwd`

    This event would be split in the following records:

    ```
    type=SYSCALL msg=audit(1364481363.243:24287): arch=c000003e syscall=2 success=no exit=-13 a0=7fffd19c5592 a1=0 a2=7fffd19c4b50 a3=a items=1 ppid=2686 pid=3538 auid=1000 uid=1000 gid=1000 euid=1000 suid=1000 fsuid=1000 egid=1000 sgid=1000 fsgid=1000 tty=pts0 ses=1 comm="cat" exe="/bin/cat" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key="etc_passwd"
    type=CWD msg=audit(1364481363.243:24287):  cwd="/home/"
    type=PATH msg=audit(1364481363.243:24287): item=0 name="/etc/passwd" inode=409248 dev=fd:00 mode=0100600 ouid=0 ogid=0 rdev=00:00 obj=system_u:object_r:etc_t:s0  objtype=NORMAL cap_fp=none cap_fi=none cap_fe=0 cap_fver=0
    type=PROCTITLE msg=audit(1364481363.243:24287) : proctitle=636174002F6574632F706173737764
    ```

  - An event for a command being entered in a `TTY` shell.

#### Record types

The `type` field contains the type of the record:

| Operation | Field type(s) |
|-----------|---------------|
| User authentication and access | `USER_AUTH` <br> `USER_LOGIN` <br> `USER_LOGIN_SUCCESS` <br> `USER_LOGIN_FAILED` <br> `USER_AUTH_SUCCESS` <br> `USER_AUTH_FAILED` <br> `USER_START_SUCCESS` <br> `USER_START_FAILED` <br> `SESSION_TERMINATED` |
| Process execution | `EXECVE` <br> `SYSCALL` |
| Filesystem access | `PATH` <br> *For relative or absolute file access.* <br><br> `CWD` <br> *Current working directory, useful to reconstruct full path if a relative path has been recorded in `PATH` records.* <br><br> `OPENAT` |
| Commands entered in a `TTY` console | `TTY` |
| Commands entered by users | `USER_CMD` |
| Full command-line of process | `PROCTITLE` <br> *The associated `proctitle` field MAY be encoded in hexadecimal.* |
| Network socket connections | `SOCKADDR` <br> *The associated `saddr` field contains IP and port information, and can be interpreted directly at event generation (if `log_format = ENRICHED` is set), with `ausearch -i`, or [simple scripting](https://gist.github.com/Qazeer/3aaa6be263380483d68159cae6f33fd2).* |
| Account and group activity | `ADD_USER` <br> `ADD_GROUP` |

More record types are listed in the
[RedHat documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/security_guide/sec-audit_record_types).

#### Notable fields

  - The [user and group identifiers](./user_and_group_identifiers.md)
    (`uid` / `gid`, `euid` / `egid`, `ruid` / `rgid`, `suid` / `sgid`, and
    `fsuid` / `fsgidof`) of the process.

  - `auid`: `audit identifier` that, if present, identify the initial `uid` of
    the user (from the initial process). The `auid` remains the same even if
    the user's identity changes (for instance through `sudo` or `su`, as the
    utilities start a new process with updated `uid`, `euid`, `ruid`, and
    `suid`).

    **An `auid` not equal to the event's `uid` / `ruid` can thus be an
    indication of privilege escalation through `sudo` or `su`.**

  - `tty` and `ses`: respectively the terminal and session from which the
    audited process was invoked.

  - Some fields, such as the `PROCTITLE` command (`proctitle` field), may be
    stored in hexadecimal format. Keyword search on parameter values can thus
    be more complex and require adapted tooling.

  - For `SYSCALL` records, the `aX` field(s) define the arguments / parameters
    of the syscall, represented by unsigned long long integers and as such
    cannot be used to determine the values taken by the arguments.

### Tool(s)

#### aureport and ausearch utilities

The `aureport` and `ausearch` utilities are packaged with the `Linux Audit`
framework and can be used to analyze the `auditd` log files. `aureport`
produces a synthesis of events, while `ausearch` can be used to search and
filter the events.

```bash
# -i / --interpret: decodes some non-human readable values, such as epoch timestamps and hex encoded PROCTITLE commands, to plain-text.
# However, the uid are resolved based on the current system local accounts, potentially resulting in incoherent usernames.

# The -ts / --start and -te / --end: can be used to filter events based on their timestamp.
# The timestamp format depends on the local LC_TIME environment variable. For en_US.utf8 locale, the timestamp format is: MM/DD/YYYY hh:mm:ss.

# aureport.

# Displays the time range covered by each input file.
aureport -if <INPUT_FILE | INPUT_DIRECTORY> -t

# Generates a global synthesis: number of logins / authentications, failed logins / authentications, process and commands executed, tty keystrokes entered, etc.
aureport -i [-ts <START_TIME>] [-te <END_TIME>] -if <INPUT_FILE | INPUT_DIRECTORY>

# Generates a report on user logins (type=USER_LOGIN) with -l or authentications (type=USER_AUTH) with -au.
aureport -i [-ts <START_TIME>] [-te <END_TIME>] -if <INPUT_FILE | INPUT_DIRECTORY> -l
aureport -i [-ts <START_TIME>] [-te <END_TIME>] -if <INPUT_FILE | INPUT_DIRECTORY> -au

# Generates a report on the local accounts activity (creation or modification).
aureport -i [-ts <START_TIME>] [-te <END_TIME>] -if <INPUT_FILE | INPUT_DIRECTORY> -m

# Generates a report on program executions, which includes the session and auid of each process.
aureport -i [-ts <START_TIME>] [-te <END_TIME>] -if <INPUT_FILE | INPUT_DIRECTORY> -x

# Generates a report on TTY keystrokes (type=TTY) entered with TTY or commands (type=USER_CMD) with --comm.
aureport -i [-ts <START_TIME>] [-te <END_TIME>] -if <INPUT_FILE | INPUT_DIRECTORY> --tty
aureport -i [-ts <START_TIME>] [-te <END_TIME>] -if <INPUT_FILE | INPUT_DIRECTORY> --comm

# Generates a report on file access.
aureport -i [-ts <START_TIME>] [-te <END_TIME>] -if <INPUT_FILE | INPUT_DIRECTORY> -f

# ausearch.

# Filters on the specified event / message type (TTY, USER_CMD, EXECVE, PATH, USER_AUTH, etc.).
ausearch -i [-ts <START_TIME>] [-te <END_TIME>] -if <INPUT_FILE | INPUT_DIRECTORY>

# Filters on the specified user identifier (across uid, euid, and auid).
ausearch -i [-ts <START_TIME>] [-te <END_TIME>] -if <INPUT_FILE | INPUT_DIRECTORY> -ua "<UID>"

# Filters on the specified session or terminal identifier.
ausearch -i [-ts <START_TIME>] [-te <END_TIME>] -if <INPUT_FILE | INPUT_DIRECTORY> --session <SESSION_ID>
ausearch -i [-ts <START_TIME>] [-te <END_TIME>] -if <INPUT_FILE | INPUT_DIRECTORY> --terminal <TERMINAL_ID>

# Filters on the specified process identifier (PID) or parent process identifier (PPID).
ausearch -i [-ts <START_TIME>] [-te <END_TIME>] -if <INPUT_FILE | INPUT_DIRECTORY> -p <PID>
ausearch -i [-ts <START_TIME>] [-te <END_TIME>] -if <INPUT_FILE | INPUT_DIRECTORY> -pp <PID>
```

#### dissect target-query

The `target-query` tool, of the
[`dissect` Python framework](https://docs.dissect.tools/en/latest/index.html),
can be used to parse `audit` logs in `CSV` or `JSON` outputs. The fields
will be partially decoded, as hex-encoded `PROCTITLE` commands are for instance
not decoded to ascii.

```bash
# If processing an input folder directly, the folder may need to be placed in a tar archive.

target-query -f audit <TARGET> | rdump <--csv | --json | --jsonlines>
```

#### Sigma-based analysis with Zircolite and ChopChopGo

The [`Zircolite`](https://github.com/wagga40/Zircolite) Python script (or
standalone compiled binaries) and / or the
[`ChopChopGo`](https://github.com/M00NLIG7/ChopChopGo) Go tool can be used to
parse and process `audit` logs using
[`Sigma`](https://github.com/SigmaHQ/sigma) rules, to generate a detection
timeline of notable and potentially suspicious activity.

It should be noted however that `Sigma` rules for Linux are nowhere near the
comprehensiveness level of `Sigma` rules for Windows, and only support basic
detection.

```bash
# Prints the summary of the detections and outputs detailed results to a JSON file (by default).
python3 zircolite.py --auditd --ruleset rules/rules_linux.json [--csv] -e <AUDIT_LOG_FILE | AUDIT_LOG_FOLDER>

# Prints the detections in an ascii array by default, output format can be changed to JSON or CSV.
ChopChopGo -target auditd -rules ./rules/linux/auditd/ [-out <csv | json>] -file <AUDIT_LOG_FILE>
```

### References

  - [RedHat - Chapter 7. System Auditing](https://access.redhat.com/documentation/fr-fr/red_hat_enterprise_linux/6/html/security_guide/chap-system_auditing)

  - [RedHat - Understanding Audit Log Files](https://access.redhat.com/documentation/fr-fr/red_hat_enterprise_linux/7/html/security_guide/sec-understanding_audit_log_files)

  - [Debian - MAN AUDIT.RULES(7)](https://manpages.debian.org/unstable/auditd/audit.rules.7.en.html)

  - [Neo23x0 - auditd](https://github.com/Neo23x0/auditd/blob/master/audit.rules)

  - [serverfault - What does auditd log by default (i.e. when no rules are defined?)](https://serverfault.com/questions/774862/what-does-auditd-log-by-default-i-e-when-no-rules-are-defined)

  - [Linux man - aureport(8)](https://linux.die.net/man/8/aureport)

  - [Linux man - ausearch(8)](https://man7.org/linux/man-pages/man8/ausearch.8.html)
