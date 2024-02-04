---
title: Linux Audit framework (audit logs)
summary: 'The Linux Audit system is an non-default auditing and logging framework that can be configured to log multiple type of operations, such as authentication successes or failures, process executions, file accesses, user commands executed in a TTY, etc.\n\nThe Linux Audit framework implements kernel-mode hooks to monitor user-mode processes and generate audit telemetry. The auditd daemon is the main user-mode component of the Linux Audit framework, that receives audit messages sent by the kernel and other components (such as pam).\n\nThe Linux Audit system operates on rules, that define what records will be captured in the audit logs. If no rules are defined, which is the case by default, only distribution specific records and telemetry from other components may be logged to audit log file by the auditd daemon.\n\nDepending on the rule configured, multiple events can be generated for the same operation. An event can be split in multiple records, with each record of the same event sharing the same timestamp and same unique identifier.\n\nEach record is associated with a given type: USER_LOGIN_SUCCESS, EXECVE, SYSCALL, OPENAT, USER_CMD, TTY, SOCKADDR, etc.'
keywords:
tags:
  - linux_logging_frameworks
  - linux_program_execution
  - linux_files_and_folders_access
  - linux_file_knowledge
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
| User authentication and access | `USER_LOGIN_SUCCESS` <br> `USER_LOGIN_FAILED` <br> `USER_AUTH_SUCCESS` <br> `USER_AUTH_FAILED` <br> `USER_START_SUCCESS` <br> `USER_START_FAILED` <br> `SESSION_TERMINATED` |
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

### References

  - [RedHat - Chapter 7. System Auditing](https://access.redhat.com/documentation/fr-fr/red_hat_enterprise_linux/6/html/security_guide/chap-system_auditing)

  - [RedHat - Understanding Audit Log Files](https://access.redhat.com/documentation/fr-fr/red_hat_enterprise_linux/7/html/security_guide/sec-understanding_audit_log_files)

  - [Debian - MAN AUDIT.RULES(7)](https://manpages.debian.org/unstable/auditd/audit.rules.7.en.html)

  - [Neo23x0 - auditd](https://github.com/Neo23x0/auditd/blob/master/audit.rules)

  - [serverfault - What does auditd log by default (i.e. when no rules are defined?)](https://serverfault.com/questions/774862/what-does-auditd-log-by-default-i-e-when-no-rules-are-defined)
