---
title: Office365 - Microsoft Flow workload
summary: 'Microsoft Flow workload logs record CreateFlow events. Flows can be used to forward emails, automatically copy or download files, etc.'
keywords: Flow, Power Automate
tags:
  - azure_logs
last_updated: 2024-01-28
sidebar: sidebar
permalink: azure_o365_microsoft_flow_logs.html
folder: azure
---

| Operation | Description |
|-----------|-------------|
| `CreateFlow` | Creation of a new flow. Flows can be used to forward emails, automatically copy or download files, etc. <br><br> Emails forwarded through `Microsoft Flow` can be identified in "Send" operation of the `UAL` logs' "Exchange" workload, as the user agent associated with the event will be "Microsoft Power Automate" (and the client IP will be an IP belonging to Microsoft). |
