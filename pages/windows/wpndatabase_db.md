---
title: Windows - Windows Push Notifications (wpndatabase.db)
summary: 'Introduced in Windows 10, the Windows Push Notification service allows applications to deliver/push notifications, notably in the form of "Toast" notifications (pop-up boxes that can appear at the bottom right of the screen).\n\nEach notification is associated with a dedicated entry in the Notification table of the wpndatabase.db database, with a system-wide database instance (for global notifications) and per-user database instances (for per-user notifications).\n\nInformation of interest: the arrival and expiry times of the notification, as well as a "payload" associated with the notification. For Toast notifications, the payload contains the content of the notification (such as message content for instant messaging applications).\n\nThe notifications are short-lived and deleted from the database after their expiry time or following a user acknowledgment, thus providing very limited historical data.'
keywords: notification, Windows Push Notifications, WPN, wpndatabase, instant messaging, instant message, Teams, Slack, Discord, Badge, Tile, Toast
tags:
  - windows_misc
location: 'Per-user database and Write-Ahead Logging (WAL) files:\n <SYSTEMDRIVE>:\Users\<USERNAME>\AppData\Local\Microsoft\Windows\Notifications\wpndatabase.db\n <SYSTEMDRIVE>:\Users\<USERNAME>\AppData\Local\Microsoft\Windows\Notifications\wpndatabase.db-wal\n\n System-wide database and Write-Ahead Logging (WAL) files:\n <SYSTEMDRIVE>:\Windows\System32\config\systemprofile\AppData\Local\Microsoft\Windows\Notifications\wpndatabase.db\n <SYSTEMDRIVE>:\Windows\System32\config\systemprofile\AppData\Local\Microsoft\Windows\Notifications\wpndatabase.db-wal'
last_updated: 2024-01-16
sidebar: sidebar
permalink: windows_wpndatabase_db.html
folder: windows
---

### Overview

Introduced in Windows 10, the Windows Push Notification service allows
applications to deliver/push notifications, in three different forms:

  - `Badge`: a tiny symbol that appears in the corner of an application's
    taskbar / hidden icon.

    Examples: the number of unread messages on `Teams`, `Slack`, `Discord`, or
    other instant messaging applications.

  - `Tile`: a rectangular shape that is displayed on the screen and linked to
    an application.

  - `Toast`: a rectangular pop-up box that can appear for a limited time
    (5 seconds by default) at the bottom right of the screen or be sent
    directly to the `Windows Action Center`.

    Examples: instant messaging applications (such as `Teams`) notifying the
    user of a new message.

More information on `Windows Push Notifications` can be found in the article
["A Digital Forensic View of Windows 10 Notifications"](https://www.mdpi.com/2673-6756/2/1/7).

### Information of interest

Each notification is associated with a dedicated entry in the `Notification`
table of the `wpndatabase.db` database. There are system-wide notifications and
per-user notifications, stored in different databases (with one database per
user).

Each entry notably contains the arrival and expiry times of the notification, as
well as a "payload" associated with the notification. For `toast notifications`,
the payload contains the content of the notification. In the case of
`toast notifications` from instant messaging applications, or from social media
/ instant messaging web applications accessed through a web browser, the
payload may contain the message received.

The notifications are short-lived and deleted from the database after their
expiry time or following a user acknowledgment (closing of the pop-up or
clearing from the Windows Action Center). The `wpndatabase.db` database thus
provides very limited historical data.

More information might be retrievable from the `Write-Ahead Logging (WAL)` file
`wpndatabase.db-wal` and/or carved from the database (using tools such as
[`bring2lite`](https://github.com/bring2lite/bring2lite) or
[`fqlite`](https://github.com/pawlaszczyk/fqlite)).

### References

  - [Polytechnic of Leiria - Patricio Domingues, Luis Andrade, and Miguel Frade - "A Digital Forensic View of Windows 10 Notifications"](https://www.mdpi.com/2673-6756/2/1/7)
