---
title: "[How-To] Microsoft 365 security review - Licensing plans"
summary: 'The licensing plans in use will define the level of logs available.\n\nFor instance, Microsoft Purview Audit (Premium) licenses (such as E5 licenses) extend the default retention of the Exchange, SharePoint, OneDrive, and Entra ID audit records from 180 days to one year.'
keywords: Licenses, Get-MgSubscribedSku, Get-MgUserLicenseDetail, Get-MsolAccountSku, Get-MsolUser
tags:
  - azure_security_review
  - guides
last_updated: 2024-01-28
sidebar: sidebar
permalink: azure_m365_security_review_licensing_plans.html
folder: azure
---

The licensing plans in use will define the level of logs available.
For instance, the audit records of users associated with a
`Microsoft Purview Audit (Premium)` license (included in the `Microsoft 365 E5`
and `Microsoft 365 E5` licenses) are retained for one year by default (for the
Exchange, SharePoint, OneDrive, and Entra ID workloads), versus 180 days for
`Audit (Standard)`.

The `MailItemsAccessed` mail access and `Send` events were historically only
available for users associated with an `E5` license. Since mid-2024, these
events are also generated for `Audit (Standard)` users (unless the default
mailbox audit configuration of the mailbox was modified).

The `MSOnline` (`Get-Msol*` cmdlets) and `AzureAD` PowerShell modules are
retired (`MSOnline` stopped working between April and May 2025), and are
replaced by the `Microsoft Graph PowerShell` SDK.

```bash
# Connects to Microsoft Graph with the required permissions.
Connect-MgGraph -Scopes "Organization.Read.All","User.Read.All"

# Lists the license plans (SKUs) subscribed by the tenant.
Get-MgSubscribedSku | Select-Object SkuPartNumber, SkuId, ConsumedUnits, @{Name="EnabledUnits"; Expression={$_.PrepaidUnits.Enabled}}

# Retrieves the license plans associated with the specified user.
Get-MgUserLicenseDetail -UserId <EMAIL> | Select-Object SkuPartNumber, SkuId
```

The license plans in use, and their associated users, can also be visualized
from the
[Microsoft 365 admin center](https://portal.office.com/Adminportal/Home/?#/licenses).
