---
title: Required privileges
summary: 'The "Global Reader" role on the Azure AD tenant is required to access the Azure AD sign-ins and audit logs.\n\nThe "View-Only Audit Logs" role in Exchange Online is required to access the Office 365 Unified and Mailbox Audit Logs.\n\nThe "Log Analytics Reader" role on the Azure subscription is required to access the Azure Activity logs.\n\nThe "Auditing\View audit log" permission is required in the Azure DevOps organization to access the Azure DevOps Activity logs.'
keywords: Global Reader, View-Only Audit Logs, Compliance Management, Organization Management, Log Analytics Reader, View audit log
tags:
  - azure_logs
  - azure_security_review
last_updated: 2024-01-28
sidebar: sidebar
permalink: azure_required_privileges.html
folder: azure
---

The following privileges / roles are required in the Azure AD tenant and
Exchange Online instance:

  - Azure AD tenant: `Global Reader` ("Lecteur Général") role.

  - Exchange Online environment: `View-Only Audit Logs` role
    ("Journaux d’audit en affichage seul") role. This role is by default
    granted to the `Compliance Management` and `Organization Management` role
    groups (for which members can be assigned). Members can be assigned to the
    aforementioned groups through the
    [Exchange administration portal](https://admin.exchange.microsoft.com/).

    If the required rights are not correctly granted to the user conducting the log collection, the following error will arise:
    
    ```
    Search-UnifiedAuditLog : The term 'Search-UnifiedAuditLog' is not recognized as the name of a cmdlet, function, script file, or operable program. Check the spelling of the name, or if a path was included, verify that the path is correct and try again.
    ```

  - Azure subscription (for retrieving `Azure Activity logs` for the given
    subscription): `Log Analytics Reader` role.

  - Azure DevOps organization (for retrieving `Azure DevOps Activity logs` for
    the given Azure DevOps organization): `Auditing\View audit log` permission.

Note that accessing Azure AD logs through the `MS Graph API` requires at least
**one user with an Azure `AD Premium P1` or `AD Premium P2` license**. These
license can be included in other license plans, such as
`Microsoft 365 E3 / E5 / F3`. The other to which is associated the license does
not matter.
