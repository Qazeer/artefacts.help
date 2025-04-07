---
title: ETW - Active Directory Domain Services (Domain Controllers) RBCD detections
summary: 'The resource-based constrained delegation, introduced in Windows Server 2012, deports the trust in Kerberos delegations to the final resources. Instead of trusting a service account to impersonate other users to destination services, service accounts can specifically authorize other services to authenticate using delegated service tickets. The authorized services are identified, by their SPN, in the msDS-AllowedToActOnBehalfOfOtherIdentity attribute of the final services.\n\n The first step of the RBCD attack is usually the modification of an objects "msDS-AllowedToActOnBehalfOfOtherIdentity" attribute, which generates an associated 5136 event.\n\n The exploitation of the configured delegation relies on a first S4U2Self request and a subsequent S4U2Proxy request, each request generating a specific 4769 event.'
keywords: Active Directory, Domain Controllers, Kerberos, RBCD, Delegations, Resource-Based Constrained Delegations, msDS-AllowedToActOnBehalfOfOtherIdentity, Service-for-User-to-Self, S4U2self, 5136, 4769, 0x40800018, Service for User to Proxy, S4U2Proxy, 0x40820010, service ticket, Transited Services
tags:
  - windows_etw
  - windows_active_directory
location: 'Channel: Security.\nEvents: 5136 ("Attribute" field equal to "msDS-AllowedToActOnBehalfOfOtherIdentity"), 4769 (two separate events, for S4U2Self and S4U2Proxy requests, with the second event with a non-null "Transited Services" field).'
last_updated: 2025-03-27
sidebar: sidebar
permalink: adds_etw_rbcd_detection.html
folder: windows
---

### Overview

`Kerberos` is an authentication protocol used within Active Directory that
uses tickets to identify users and to grant access to domain resources.
To do so, `Kerberos` implements two types of tickets, issued by two distinct
services of the `Key Distribution Center (KDC)`:
  - `Ticket-Granting Ticket (TGT)`, obtained from the `Authentication Service
  (AS)`.
  - `service tickets`, obtained from the `Ticket-Granting Service (TGS)`.

A valid `TGT` is necessary to request `service tickets`, which in turn grant
access to service accounts (user or machine domain accounts that have a
`ServicePrincipalName (SPN)`).

###### Unconstrained, constrained and resource-based constrained delegations

Three types of delegation are implemented in the (Microsoft Active Directory)
`Kerberos` protocol:
  - `unconstrained delegation`,
  - `constrained delegation`,
  - `resource-based constrained delegation (RBCD)`.

Service accounts that are trusted for `unconstrained delegation`, i.e service
accounts with the `ADS_UF_TRUSTED_TO_AUTHENTICATE_FOR_DELEGATION` flag set in
their `User-Account-Control` attribute, can fully act on behalf of other domain
users. `Service tickets` received by such services will contain a copy of the
`TGT` of the user accessing the service. The received `TGTs` can be extracted
from the `LSASS` process of the machine running the service / receiving the
authentication, and further used to authenticate to any domain resources.

Service accounts that are trusted for `constrained delegation`, i.e service
accounts with a non-empty `msDS-AllowedToDelegateTo` attribute, can impersonate
other domain users to configured services (in the service account 's
`msDS-AllowedToDelegateTo` attribute). `Constrained delegations` are
implemented in the `Kerberos` protocol by Microsoft through the
`Service-for-User-to-Proxy (S4U2proxy)` extension. `S4U2proxy` allows service
accounts to request `service tickets` to the `KDC` (`TGS-REQ`) on behalf of
other users by arbitrarily specifying the name of the user the `service ticket`
should be emitted to. In order to do so, the service account must join, in the
`req-body.additional-tickets` field of the `TGS-REQ` request, a
`service ticket` marked as `forwardable` from the specified user. Such
`service ticket` may be received from a delegable user as part of the
`Kerberos` authentication process to the "proxifying" service or directly
requested by the "proxifying" service through a `S4U2self` request (if the
service is allowed to do so). More details on the `Kerberos` `S4U2self`
extension are provided below.

The `resource-based constrained delegation`, introduced in Windows
Server 2012, shifts the trust to the final resources. Instead of trusting a
service account to impersonate other users to destination services, service
accounts can specifically authorize other services to authenticate using
delegated `service tickets`. The authorized services are identified, by their
`SPN`, in the `msDS-AllowedToActOnBehalfOfOtherIdentity` attribute of the
final services. Similarly to `constrained delegation`, the service accessing
the final service will request a `service ticket` to the `KDC` on behalf of a
domain user through a `S4U2proxy` request. However, in a `resource-based
constrained delegation` scenario, the "proxifying" service can provide a
standard `service ticket` to the `KDC` and thus does not have to be in
possession of a `service ticket` marked as `forwardable` from the specified
user.

In summary, the services a service account can request `S4U2proxy`
`service tickets` for are restricted by the `KDC` to the ones either:
  - *`[constrained delegation]`* configured, using their `SPNs`, in the
  requesting service account's `msDS-AllowedToDelegateTo` attribute.
  - *`[resource-based constrained delegation]`* having in their
  `msDS-AllowedToActOnBehalfOfOtherIdentity` attribute the `SPN` of the
  requesting service.

###### Service-for-User-to-Self (S4U2self)

In order to provide protocol transition, Microsoft implemented the `S4U2self`
extension into the `Kerberos` protocol. This extension makes `Kerberos`
delegation possible for domain users accessing a service through other Windows
authentication protocols, such as the `NT Lan Manager (NTLM)` protocol. As a
`service ticket` is required for `Kerberos` `constrained` and `resource-based
constrained delegation` (both leveraging the `Kerberos` `S4U2proxy` extension),
the `S4U2self` allows a service account to obtain a `service ticket` for itself
on behalf of an arbitrarily specified user. The user is specified in the of the
`PA-FOR-USER` field in the `preauthentication data` of the `KRB_TGS_REQ`
request to the `KDC`.

Any service account can request `service tickets` for itself through the
`S4U2self` extension. However, only service accounts configured as being able
to `"use any authentication protocol"`, which corresponds to the
`ADS_UF_TRUSTED_TO_AUTHENTICATE_FOR_DELEGATION` /
`TRUSTED_TO_AUTH_FOR_DELEGATION` flag being set in the service accounts'
`User-Account-Control` attribute, will receive `service tickets` marked as
`forwardable`.

The `S4U2self` `service tickets` contain, in its
`Privilege Attribute Certificate (PAC)`, the `authorization data` of the
requested user and can be used:

  - to impersonate the user locally on the system executing the service if the
    service account has the `SeTcbPrivilege` privilege on the local system.

  - through a `constrained delegation` to impersonate the user on remote
    services, using subsequent `S4U2proxy` requests, if the `S4U2self`
    `service ticket` is marked as `forwardable`. The `S4U2self` `service ticket`
    can only be used to make `S4U2proxy` requests for the services defined in
    the receiving service account's `msDS-AllowedToDelegateTo` attribute. The
    `S4U2proxy` request will result in the retrieval of a `service ticket` to
    the remote service impersonating another user identity.

  - through a `resource-based constrained delegation` to impersonate the user
    on remote services, using subsequent `S4U2proxy` requests, even if the
    `S4U2self` `service ticket` is **not** marked as `forwardable`. Indeed,
    in a `resource-based constrained delegation` scenario, `S4U2Proxy` requests
    will grant `service tickets` for services even if the `service ticket`
    provided to the `KDC` is non-`forwardable`. The `S4U2self` `service ticket`
    can only be used to make `S4U2proxy` requests for the services that define
    in their `msDS-AllowedToActOnBehalfOfOtherIdentity` attribute the service
    emitting the `S4U2self` `service ticket`.

###### Accounts that cannot be delegated

Domain users can be protected from `Kerberos` delegation by either being:
  - configured as non-delegable (`"Account is sensitive and cannot be
    delegated"`), which corresponds to the `ADS_UF_NOT_DELEGATED` flag being
    positioned in a user' `User-Account-Control` attribute.
  - members of the `Protected Users` domain group.

### RBCD attack Windows events

###### Modification of a `msDS-AllowedToActOnBehalfOfOtherIdentity` attribute

The first step of the `resource-based constrained delegation (RBCD)` attack is
usually the modification of an object's
`msDS-AllowedToActOnBehalfOfOtherIdentity` attribute, which generates an
associated `5136` ETW event:

| Channel | Conditions | Events |
|---------|------------|--------|
| `Security` | Default configuration. | Upon modification of the `msDS-AllowedToActOnBehalfOfOtherIdentity`, the following event will be generated: <br><br> - Event `5136`: `A directory service object was modified` with the `Attribute` field equal to `msDS-AllowedToActOnBehalfOfOtherIdentity`. |

###### S4U2self and S4U2Proxy service ticket requests

The exploitation of the configured delegation relies on a first
`Service for User to Self` (`S4U2Self`) request and a subsequent
`Service for User to Proxy` (`S4U2Proxy`) request. Each request generates a
distinct `4769` event:

| Channel | Conditions | Events |
|---------|------------|--------|
| `Security` | Default configuration. | Upon exploitation of the configured delegation, two `4769`: `A Kerberos service ticket was requested` events are generated in close succession: <br><br> - For the initial `S4U2Self` request, a `4769` event for the service account emitting the request itself (configured in the delegation, such as the machine account controlled by the attacker) and on behalf of the user that will be impersonated. Additionally, the event `Ticket Options` field is usually `0x40800018`. <br><br> - For the subsequent `S4U2Proxy` request, a `4769` event for the targeted service (such as the targeted machine account `HOST` / `CIFS` service in case of machine takeover) under the identity of the impersonated user. The `Transited Services` field, only populated ["if the logon was a result of a `S4U` (`Service For User`) logon process"](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4624) references the service account that made the initial `S4U2Self` request. Additionally, the event `Ticket Options` field is usually `0x40820010`. |

### References

  - [Elad Shamir - Wagging the Dog: Abusing Resource-Based Constrained Delegation to Attack Active Directory](https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html)

  - [Sean Metcalf - Active Directory Security Risk #101: Kerberos Unconstrained Delegation (or How Compromise of a Single Server Can Compromise the Domain)](https://adsecurity.org/?p=1667)

  - [Microsoft - S4U2proxy](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-sfu/bde93b0e-f3c9-4ddf-9f44-e1453be7af5a)

  - [SpecterOps - Roberto Rodriguez - Hunting in Active Directory: Unconstrained Delegation & Forests Trusts](https://posts.specterops.io/hunting-in-active-directory-unconstrained-delegation-forests-trusts-71f2b33688e1)

  - [Stephan Wolfert - Detecting Resource-Based Constrained Delegation Abuse](https://swolfsec.github.io/2023-11-29-Detecting-Resource-Based-Constrained-Delegation/)

  - [Microsoft - 4624(S): An account was successfully logged on](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4624)
