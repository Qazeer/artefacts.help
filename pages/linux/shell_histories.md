---
title: Shell histories
summary: 'The shell history log files are linked to the shell history feature, which tracks a user''s command-line history for a given shell (Bash, Zsh, etc.). While easily bypassed or deleted, the history log files can be a valuable source of information on actions performed by a given user.\n\n The shell history log files contain commands entered in an (interactive) shell, with no additional meta information, such as timestamps (by default). By default, the shell history of a shell session is stored in memory until the shell session is closed.\n\n The behavior of the history feature can be modified by setting a number of environment variables (HISTFILE, HISTCONTROL, HISTIGNORE, HISTTIMEFORMAT, HISTSIZE, HISTFILESIZE, etc.).'
keywords: shell, history, bash, zsh, bash_history, .bash_history, zsh_history, .zsh_history, HISTFILE, HISTCONTROL, HISTIGNORE, HISTTIMEFORMAT, HISTSIZE, HISTFILESIZE, HIST_IGNORE_SPACE
tags:
  - linux_program_execution
  - linux_files_and_folders_access
default_location: 'Bash history:\n ~/.bash_history\n\n Zsh history:\n ~/.zsh_history\n\n Non-default history settings may be defined through environment variables set in shell configuration files:\n .profile, .bash_login / .zlogin, .bashrc / .zshrc, .bash_profile / .zprofile'
last_updated: 2024-08-04
sidebar: sidebar
permalink: linux_shell_histories.html
folder: linux
---

### Overview

The shell history log files are linked to the shell `history` feature. The
shell `history` feature tracks a user's command-line history for a given shell
(`Bash`, `Zsh`, etc.). While easily bypassed or deleted, the history log files
can be a valuable source of information on actions performed by a given user.

Two built-in commands can be used to interact with the shell history:

  - `history`: listing of shell history commands.

  - `fc`: accessing commands from the history to edit and/or execute a past
    command.

The behavior of the `history` feature can be modified by setting a number of
environment variables: `HISTFILE`, `HISTCONTROL`, `HISTIGNORE`,
`HISTTIMEFORMAT`, `HISTSIZE`, `HISTFILESIZE` (among others).

By default, the shell history log files are stored in their associated user's
home directory. For instance, `Bash`'s history file is stored as
`~/.bash_history` and `Zsh`'s history file as `~/.zsh_history`. The location
of the shell history file can be set using the `HISTFILE` environment
variable.

#### Shell history bypass

Depending on the shell configuration, commands with a leading space may be
ignored and not added to the history (in memory or in the history log file on
disk):

  - For `Bash`, this behavior is controlled by the `HISTCONTROL` environment
    variable (`ignorespace` or `ignoreboth` values), and is not enabled by
    default. `HISTCONTROL` is however set to `ignoreboth` in the default user
    `~/.bashrc` file of `Debian`-based distributions (such as `Ubuntu`). On
    `Red Hat`-based distributions, `HISTCONTROL` is set to `ignoredups` in
    `/etc/profile` (commands with a leading space are thus recorded).

  - For `Zsh`, this behavior is controlled by the `HIST_IGNORE_SPACE` option,
    which is not enabled by default (but is enabled by frameworks such as
    `Oh My Zsh`).

Specific commands can also be excluded from the `Bash` history using the
`HISTIGNORE` environment variable (colon-separated list of patterns). For
instance, to ignore commands containing `-pass` or `-key`, `HISTIGNORE` can be
set to `'*-pass*:*-key*'`.

The history can also be disabled for a given shell session, for example by
unsetting the `HISTFILE` environment variable (`unset HISTFILE`), in which case
the history is not saved when the shell exits.

Additionally, shell history files are not protected against tampering, with
no guarantee of their integrity. Specific or all commands can be trivially
deleted from a shell history file.

### Information of interest

The shell history log files contain commands entered in a shell, with no
additional meta information (by default).

Timestamps can be added to track the execution time of commands if the
`HISTTIMEFORMAT` environment variable is set. The `%F` parameter adds the
date in `YYYY-MM-DD` format, and `%T` adds a `HH:MM:SS` timestamp
(`HISTTIMEFORMAT="%F %T "`).

By default, the shell history of a shell session is stored in memory until the
shell session is closed. While commands for a given shell session are stored in
order, the overall commands may not be stored in chronological order if concurrent
shell sessions were used.

#### HISTSIZE & HISTFILESIZE

The number of commands stored in memory can be specified through the `HISTSIZE`
environment variable. For instance, if `HISTSIZE` is set to 20, only the last
20 commands will be stored in the in-memory history and will be written to disk
once the shell session terminates. By default, `Bash` limits the history in
memory to 1,000 entries (equivalent to `HISTSIZE` being set to 1,000).

The number of commands stored on disk in the history file is specified through
the `HISTFILESIZE` environment variable. When the shell session exits, the
commands in memory (up to `HISTSIZE`) are written to the history file and the
history file is truncated to be limited to `HISTFILESIZE` lines. By default,
`Bash` limits the number of commands in the history file to 2,000 entries
(equivalent of `HISTFILESIZE` being set to 2,000).

### References

  - [cherry servers - Mantas Levinas - A Complete Guide to Linux Bash History](https://www.cherryservers.com/blog/a-complete-guide-to-linux-bash-history)

  - [StackOverflow - arturomp - bash (or zsh) HISTSIZE vs. HISTFILESIZE?](https://stackoverflow.com/questions/19454837/bash-or-zsh-histsize-vs-histfilesize/19454838#19454838)
