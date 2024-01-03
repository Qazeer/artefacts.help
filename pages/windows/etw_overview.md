---
title: ETW - Overview
summary: 'Event Tracing is broken into three distinct components: controllers, providers, and consumers.\n\nControllers start and stop an event tracing session and enable providers.\n\nProviders: provide the events, consumed by Consumers in real time.\n\nProviders can also write events to (new or existing) channels, with each event only being writable to a single channel.'
keywords: 'Event Tracing for Windows, ETW, EVTX, controllers, providers, consumers, trace sessions, channel'
tags:
  - windows_etw
location: 'EVTX files on disk: <SYSTEMROOT>\System32\winevt\Logs\*'
last_updated: 2024-01-02
sidebar: sidebar
permalink: windows_etw_overview.html
folder: windows
---

Overall system usage: Accounts authentication successes and failures, local
accounts and groups management, Windows Services or scheduled tasks operations,
PowerShell activity, etc.

`Event Tracing` is broken into three distinct components:

  - `Controllers`: start and stop an event `tracing session` and enable `providers`.

  - `Providers`: provide the events.

  - `Consumers`: consume the events in real time.

Events can eventually be written to event log `channels` (assimilable to the
log file names), `event tracing` log files, or both. The provider itself
defines the event log `channel(s)` to which events should be written (trough
its ["instrumentation manifest" for manifested-based providers](https://learn.microsoft.com/en-us/windows/win32/wes/defining-channels)).
Providers can define new `channels` or import existing `channels`. While the
provider may use different `channels` for different events, each event can only
be written to a single `channel` (as specified in the event's `event element`
in the instrumentation manifest). If no `channel` is defined for a given event,
the event will not be written to an event log channel, but can still be
consumed (in memory) by a consumer through a `trace session`.

Event `trace sessions` record events by subscribing to one or more `providers`
and may write to a log file. Events can only be written to one `channel` at a
time, but can also be collected by up to 7 `trace sessions`.

`Security`, `System`, and `Application` are legacy `channels`. Only the
`LSASS` process can write to the `Security` channel.

Four types of channels are supported: `Admin`, `Operational`, `Analytic`, and
`Debug`.

`Provider` example: `Microsoft-Windows-TerminalServices-RemoteConnectionManager`.
Associated `channel` example: `Microsoft-Windows-TerminalServices-RemoteConnectionManager/Operational`.
