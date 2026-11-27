---
title: Security review - Licensing plans
summary: 'The licensing plans in use will define the level of logs available.\n\nFor instance, MailItemsAccessed mail access events will only be available for users associated with an E5 license.'
keywords: Licenses, Get-MsolAccountSku, Get-MsolUser
tags:
  - azure_security_review
  - guides
last_updated: 2024-01-28
sidebar: sidebar
permalink: azure_m365_security_review_licensing_plans.html
folder: azure
---

The licensing plans in use will define the level of logs available.
For instance, `MailItemsAccessed` mail access events will only be available
for users associated with an `Office 365 E5` or `Microsoft 365 E5` license.

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
