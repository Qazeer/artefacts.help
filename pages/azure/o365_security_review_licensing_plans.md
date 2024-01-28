---
title: Security review - Licensing plans
summary: 'The licensing plans in use will define the level of logs available.\n\nFor instance, MailItemsAccessed mail access events will only be available for users associated to an E5 license.'
keywords: Licenses, Get-MsolAccountSku, Get-MsolUser
tags:
  - azure_security_review
last_updated: 2024-01-28
sidebar: sidebar
permalink: azure_o365_security_review_licensing_plans.html
folder: azure
---

The licensing plans in use will define the level of logs available.
For instance, `MailItemsAccessed` mail access events will only be available
for users associated to an `Office 365 E5` or `Microsoft 365 E5` license.

```bash
# List the license plans in the tenant.
Get-MsolAccountSku

# Retrieves the license plans associated with the specified user.
Get-MsolUser -UserPrincipalName <EMAIL> | Select-Object DisplayName -ExpandProperty Licenses
```

The license plans in use, and their associated users, can also be visualized
from the
[Microsoft 365 admin center](https://portal.office.com/Adminportal/Home/?#/licenses).
