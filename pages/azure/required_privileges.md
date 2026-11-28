---
title: "[How-To] Azure / Microsoft 365 - Required privileges for collection"
summary: 'The "Global Reader" role on the Entra ID tenant is required to access the Entra ID sign-ins and audit logs.\n\nThe "View-Only Audit Logs" role in Exchange Online is required to access the Microsoft 365 Unified and Mailbox Audit Logs.\n\nThe "Log Analytics Reader" role on the Azure subscription is required to access the Azure Activity logs.\n\nThe "Auditing\View audit log" permission is required in the Azure DevOps organization to access the Azure DevOps Activity logs.'
keywords: Global Reader, View-Only Audit Logs, Compliance Management, Organization Management, Log Analytics Reader, View audit log
tags:
  - azure_logs
  - azure_security_review
  - guides
last_updated: 2024-01-28
sidebar: sidebar
permalink: azure_required_privileges.html
folder: azure
---

The following privileges/roles are required in the Entra ID tenant and
Exchange Online instance:

  - Entra ID tenant: `Global Reader` role.

  - Exchange Online environment: `View-Only Audit Logs` role. This role is
    granted by default to the `Compliance Management` and
    `Organization Management` role groups (to which members can be assigned).
    Members can be assigned to the aforementioned groups through the
    [Exchange administration portal](https://admin.exchange.microsoft.com/).

    If the required rights are not correctly granted to the user conducting the log collection, the following error will arise:
    
    ```
    Search-UnifiedAuditLog : The term 'Search-UnifiedAuditLog' is not recognized as the name of a cmdlet, function, script file, or operable program. Check the spelling of the name, or if a path was included, verify that the path is correct and try again.
    ```

  - Azure subscription (for retrieving `Azure Activity logs` for the given
    subscription): `Log Analytics Reader` role.

  - Azure DevOps organization (for retrieving `Azure DevOps Activity logs` for
    the given Azure DevOps organization): `Auditing\View audit log` permission.

Note that accessing Entra ID logs through the `MS Graph API` requires at least
**one user with a `Microsoft Entra ID P1` or `Microsoft Entra ID P2` license**
(formerly `Azure AD Premium P1` / `P2`). These licenses can be included in
other license plans, such as `Microsoft 365 E3/E5/F3`. The user to which the
license is assigned does not matter.
