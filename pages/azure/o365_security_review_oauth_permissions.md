---
title: Security review - OAuth permissions
summary: 'OAuth is a protocol to delegate access and grant third party websites or applications access to users data and perform operations on their behalf.\n\nOAuth applications can be leveraged by threat actors: in illicit consent grant phishing attacks, to maintain persistence, or to automate operations (such as virtual machines creation for cryptomining activity).'
keywords: OAuth, permissions, OAuth2PermissionGrants, consent grant, phishing, 
tags:
  - azure_security_review
last_updated: 2024-01-28
sidebar: sidebar
permalink: azure_o365_security_review_oauth_permissions.html
folder: azure
---

`OAuth` is a protocol to delegate access and grant third party websites or
applications access to users data and perform operations on their behalf. With
`OAuth`, users don't have to reveal their credentials to the third party
service, as access is granted through the `Identity Provider (IdP)`
(`Azure AD` in `Azure` case).

`OAuth` applications can be leveraged by threat actors:

  - In illicit consent grant phishing attacks. A victim authorizes a malicious
    third-party `OAuth ` application to access their account data. However
    Microsoft implemented security measures in November 2020 to limit this kind
    of attack. An administrator's approval is now required for sensitive
    permission requests made by unverified application created outside the
    tenant.

  - To maintain persistence in the tenant.

  - As an automation tool, to automate operations such as virtual machines
    creation for cryptomining activity.

Microsoft Graph supports two access types, delegated permissions and
application permissions. With delegated permissions, the application calls
Microsoft Graph on behalf of a signed-in user. With application permissions,
the application calls Microsoft Graph with its own identity, without a signed
in user.

The delegated and application permissions available are referenced in the
[Microsoft documentation](https://learn.microsoft.com/en-us/graph/permissions-reference).

[`Microsoft-Extractor-Suite`](https://github.com/invictus-ir/Microsoft-Extractor-Suite)'s
`Get-OAuthPermissions` PowerShell cmdlet can be used to enumerate delegated
permissions (`OAuth2PermissionGrants`) and application permissions
(`AppRoleAssignments`) for all accounts:

```bash
Get-OAuthPermissions
```

### References

  - [Wavestone - Raymond CHAN, Younes LAABOUDI - Illicit consent grant attacks targeting Azure and Office 365: still a threat?](https://www.riskinsight-wavestone.com/en/2023/03/illicit-consent-grant-attacks-targeting-azure-and-office-365-still-a-threat/)

  - [Microsoft - Threat actors misuse OAuth applications to automate financially driven attacks](https://www.microsoft.com/en-us/security/blog/2023/12/12/threat-actors-misuse-oauth-applications-to-automate-financially-driven-attacks/)
