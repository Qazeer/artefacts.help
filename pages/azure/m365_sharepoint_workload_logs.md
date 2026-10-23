---
title: Microsoft 365 - SharePoint/OneDrive workload
summary: 'The SharePoint/OneDrive workload regroup events from the SharePoint and OneDrive activity.\n\nThe audit logs include events on file access, download, creation, and modification activity, such as FileAccessed, FileDownloaded, FileUploaded, and FileModified as well as execution of search query (SearchQueryPerformed).'
keywords: SharePoint, Drive, OneDrive, Unified Audit Logs, UAL, SearchQueryPerformed, FilePreviewed, FileAccessed, FileAccessedExtended, FileDownloaded, FileUploaded, FileModified, FileModifiedExtended, FileMoved, FileRenamed, FileDeleted, FileMalwareDetected
tags:
  - azure_logs
last_updated: 2026-10-21
sidebar: sidebar
permalink: azure_m365_sharepoint_onedrive_workload_logs.html
folder: azure
---

The following operations are notable and common to the `SharePoint` and
`OneDrive` workloads:

| Operation | Description |
|-----------|-------------|
| `SearchQueryPerformed` | The user performed a search query. |
| `FilePreviewed` | The user previewed a file. |
| `FileAccessed` | The user accessed/downloaded a file. While the Microsoft documentation states that the `FileAccessed` event is generated on file accesses, the full file content may be transmitted to the user's client, allowing saving an offline copy of the file. |
| `FileAccessedExtended` | This activity is related to the `FileAccessed` event. A `FileAccessedExtended` event is logged when the same user continually accesses a file for an extended period (up to three hours), in order to reduce the number of events that are logged. |
| `FileDownloaded` | The user downloaded a file. |
| `FileUploaded` | The user uploaded a file. |
| `FileModified` | The user modified a file. |
| `FileModifiedExtended` | This activity is related to the `FileModified` event. A `FileModifiedExtended` event is logged when the same user continually modifies a file for an extended period (up to three hours), in order to reduce the number of events that are logged. |
| `FileMoved` | The user moved a file. |
| `FileRenamed` | The user renamed a file. |
| `FileDeleted` | The user deleted a file. |
| `FileMalwareDetected` | SharePoint anti-virus engine detected malware in a file. |

The full list of actions logged for the `SharePoint` and `OneDrive` workloads
can be found in the
[official Microsoft documentation](https://learn.microsoft.com/en-us/purview/audit-log-activities#file-and-page-activities).
