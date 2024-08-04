---
title: Shell histories
summary: 'The shell history log files are linked to the shell history feature, that tracks a user command line history for a given shell (Bash, Zsh, etc.). While easily bypassed or deleted, the history log files can be a precious source of information on actions performed by a given user.\n\n The shell history log files contain commands entered in an (interactive) shell, with no
additional meta information, such as timestamp (by default). By default, the shell history of a shell session is stored in memory until the shell session is closed.\n\n The behavior of the history feature can be modified by setting a number of environment variables (HISTFILE, HISTCONTROL, HISTTIMEFORMAT, HISTSIZE, HISTFILESIZE, etc.).'
keywords: shell, history, bash, zsh, bash_history, .bash_history, zsh_history, .zsh_history, HISTFILE, HISTCONTROL, HISTTIMEFORMAT, HISTSIZE, HISTFILESIZE
tags:
  - linux_program_execution
  - linux_files_and_folders_access
default_location: 'Bash history:\n ~/.bash_history\n\n Zsh history:\n ~/.zsh_history\n\n Non default history settings may be defined through environment variables set in Shell configuration files:\n .profile, .bash_login / .zlogin, .bashrc / .zshrc, .bash_profile / .zprofile'
last_updated: 2024-08-04
sidebar: sidebar
permalink: linux_shell_histories.html
folder: linux
---

### Overview

The shell history log files are linked to the shell `history` feature. The
shell `history` feature tracks a user command line history for a given shell
(`Bash`, `Zsh`, etc.). While easily bypassed or deleted, the history log files
can be a precious source of information on actions performed by a given user.

Two built-in commands can be used to interact with the shell history:

  - `history`: listing of shell history commands.

  - `fc`: accessing commands from the history to edit and / or execute a past
    command.

The behavior of the `history` feature can be modified by setting a number of
environment variables: `HISTFILE`, `HISTCONTROL`, `HISTTIMEFORMAT`, `HISTSIZE`,
`HISTFILESIZE` (among others).

By default, the shell history log files are stored in their associated user's
home directory. For instance, `Bash`'s history file is stored as
`~/.bash_history` and `Zsh`'s history file as `~/.zsh_history`. The location
of the shell history file can be set using the `HISTFILE` environnement
variable.

#### Shell history bypass

On many Linux distributions and Unix shells, commands with a leading space are
ignored and not added to the history (in memory or in the history log file on
disk).

This behavior can be controlled using the `HISTCONTROL` environnement variable.
For instance, for `Bash`, to ignore command containing `-pass` or `-key`,
`HISTCONTROL` can be set to `HISTIGNORE='*-pass*':'*-key*'`.

Additionally, Shell history files are not protected against tampering, with
no guarantee to their integrity. Specific or all commands can be trivially
deleted from a Shell history file.

### Information of interest

The shell history log files contain commands entered in a shell, with no
additional meta information (by default).

Timestamps can be added to track the execution time of commands if the
`HISTTIMEFORMAT` environnement variable is set. The `%F` parameter adds the
date in `YYYY-MM-DD` format and `%T` adds a `HH:MM:SS` timestamp
(`HISTTIMEFORMAT="%F %T "`).

By default, the shell history of a shell session is stored in memory until the
shell session is closed. While commands for a given shell session are stored in
order, the overall commands may not be stored in chronological if concurrent
shell sessions were used.

#### HISTSIZE & HISTFILESIZE

The number of commands stored in memory can be specified through the `HISTSIZE`
environnement variable. For instance, if `HISTSIZE` is set to 20, only the last
20 commands will be stored in the in memory history and will written to disk
once the shell session terminate. By default, `Bash` limits the history in
memory to 1,000 entries (equivalent of `HISTSIZE` being set to 1,000).

The number of commands stored on disk in the history file is specified through
the `HISTFILESIZE` environnement variable. When the shell session exits, the
commands in memory (up to `HISTSIZE`) are written to the history file and the
history file is truncated to be limited to `HISTFILESIZE` lines. By default,
`Bash` limits the number of commands in the history file to 2,000 entries
(equivalent of `HISTFILESIZE` being set to 2,000).

### References

  - [cherry servers - Mantas Levinas - A Complete Guide to Linux Bash History](https://www.cherryservers.com/blog/a-complete-guide-to-linux-bash-history)

  - [StackOverflow - arturomp - bash (or zsh) HISTSIZE vs. HISTFILESIZE?](https://stackoverflow.com/questions/19454837/bash-or-zsh-histsize-vs-histfilesize/19454838#19454838)
