---
title: ETW - Users and security groups operations
summary: 'For user accounts and security groups operations, such as a user object creation or modification, and a security group membership update.\n\nMain events:\n\nChannel: Security.\nEvent ID 4720: "A user account was created".\nEvent ID 4724: "An attempt was made to reset an accounts password".\nEvent ID 4738: "A user account was changed".\nEvent ID 4732: "A member was added to a security-enabled local group".'
keywords: ''
tags:
  - windows_etw
  - windows_system_information
  - windows_local_persistence
location: 'Channel: Security.\nEvents: 4720, 4722, 4723, 4724, 4731, 4732, 4733, 4738.'
last_updated: 2024-01-21
sidebar: sidebar
permalink: windows_etw_users_and_groups.html
folder: windows
---

### User operations Windows events

| Channel | Conditions | Events |
|---------|------------|--------|
| `Security` | Default configuration. | Event `4720: A user account was created`. <br><br> Logged whenever a local account is created. <br><br> Information of interest: <br> - Domain, username, and `Logon ID` of the account that created the user. <br> - Domain, `SamAccountName`, `SID`, `UserPrincipalName`, `PrimaryGroupId`, and other user object properties of the created user. <br><br> Legacy: <br> Event `624: User Account Created`. |
| `Security` | Default configuration. | Event `4722: A user account enabled`. <br><br> Always logged after an event `4720`. <br><br> Information of interest: <br> - Domain, username, and `Logon ID` of the account that created the user. <br> - Domain, `SamAccountName`, and `SID` of the created user. <br><br> Legacy: <br> Event `626: User Account Enabled`. |
| `Security` | Default configuration for success. <br><br> Failures logged only if `Audit User Account Management` is set to `(Success), Failure`. | Event `4723: An attempt was made to change an account's password`. <br><br> Logged as a success (`Audit Success`) if the user did change the target user password (which requires to enter the target user current password). <br><br> Otherwise reported as a failure (`Audit Failure`) if failures are logged and an error occurred (wrong current password given, new password fails to meet the password policy, etc.). <br><br> Information of interest: <br> - Domain, username and `Logon ID` of the user that performed the password change. <br> - Domain, username, and `SID` of the target user whose password was changed. <br><br> Legacy: <br> Event `627: Change Password Attempt`. |
| `Security` | Default configuration for success. <br><br> Failures logged only if `Audit User Account Management` is set to `(Success), Failure`. | Event `4724: An attempt was made to reset an accounts password`. <br><br> Logged as a success (`Audit Success`) if the user did reset the target user password (which requires elevated rights for local accounts). <br><br> Otherwise reported as a failure (`Audit Failure`) if failures are logged and the new password failed to meet the password policy. A Failure event is NOT generated if the user gets an `Access Denied` error while attempting the password reset. <br><br> Information of interest: <br> - Domain, username, and `Logon ID` of the user that performed the password change. <br> - Domain, username, and `SID` of the target user whose password was reset. <br><br> Legacy: <br> Event `628: User Account password set`. |
| `Security` | Default configuration. | Event `4738: A user account was changed`. <br><br> Logged whenever an user object attribute is modified. For each change, a separate `4738` event will be generated. <br><br> Only a subset of attributes are displayed / logged in the event. If a change is made to an attribute that is not listed in the event, an event `4738` will be generated with all listed field set to `-`. <br><br> The `security descriptor`, and thus the `Discretionary Access Control List` (`DACL`), of an user is not listed in the `4738` event. An update to an account's `DACL` will thus generate an event `4738` with all listed attributes set to `-`, making it impossible to detect the `DACL` update through this event alone. <br><br> In case of a password change, the timestamp update of the `PasswordLastSet` field will be logged in the event. <br><br> Information of interest: <br> - Domain, username, and `Logon ID` of the user that performed the property change. <br> - Domain, username, and `SID` of the target user whose attribute was updated. <br><br> Legacy: <br> Event `642: User Account Changed`. |

### Security group operations Windows events

| Channel | Conditions | Events |
|---------|------------|--------|
| `Security` | Default configuration. | Event `4731: A security-enabled local group was created`. <br><br> Logged whenever a new security local group is created. <br><br> Information of interest: <br> - Domain, username, and `Logon ID` of the user that created the group. <br> - Domain, name, `SamAccountName`, and `SID` of the created group. <br><br> Legacy: <br> Event `636: Security Enabled Local Group Member Added`. |
| `Security` | Default configuration. | Event `4732: A member was added to a security-enabled local group`. <br><br> Logged whenever an account is added to a local security group. <br><br> Information of interest: <br> - Domain, username, and `Logon ID` of the user that performed the action. <br> - Target group and added user's domain and username. <br><br> Legacy: <br> Event `636: Security Enabled Local Group Member Added`. |
| `Security` | Default configuration. | Event `4733: A member was removed from a security-enabled local group`. <br><br> Logged whenever an account is removed from a local security group. <br><br> Information of interest: <br> - Domain, username, and `Logon ID` of the user that performed the action. <br> - Target group and added user's domain and username. <br><br> Legacy: <br> Event `636: Security Enabled Local Group Member Added`. |


### References

  - [4720(S): A user account was created](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4720)

  - [4722(S): A user account was enabled](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4722)

  - [4723(S, F): An attempt was made to change an account's password](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4723)

  - [4724(S, F): An attempt was made to reset an account's password](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4724)

  - [4738(S): A user account was changed](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4738)

  - [4731(S): A security-enabled local group was created](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4731)

  - [4732(S): A member was added to a security-enabled local group](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4732)

  - [4733(S): A member was removed from a security-enabled local group](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4733)