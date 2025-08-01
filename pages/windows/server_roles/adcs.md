---
title: Active Directory Certificate Services
summary: 'Active Directory Certificate Services (AD CS) is a Windows Server role for issuing and managing Public Key Infrastructure (PKI) certificates used in secure communication and authentication protocols.\n\n In the last few years, a number of possible AD CS misconfigurations, leading to privilege escalation and persistence in an Active Directory environment, have been published by security researchers and exploited by threat actors.\n\n'
keywords: Active Directory, Active Directory Certificate Services, ADCS, AD CS, FarsightAD, ESC1, ESC8, ntml relay, certificates, certificate templates, templates, X.509, X509, X.509v3, X509Certificate2, SubjectAltName, EnhancedKeyUsageList, SerialNumber, userCertificate, Get-X509CertificateStringFromUserCertificate, Export-ADHuntingPrincipalsCertificates
tags:
  - windows_active_directory
location: ''
last_updated: 2024-06-29
sidebar: sidebar
permalink: adcs.html
folder: windows
---

`Active Directory Certificate Services` (`AD CS`) is a Windows Server role for
issuing and managing `Public Key Infrastructure` (`PKI`) certificates used in
secure communication and authentication protocols.

In the last few years, a number of possible AD CS misconfigurations, leading to
privilege escalation and persistence in an Active Directory environment, have
been published by security researchers and exploited by threat actors.

### Active Directory objects linked to AD CS

A number of Active Directory objects, stored in the `Configuration` naming
context, are related to `Active Directory Certificate Services` (and
potentially third-party `Certification Authority`). As any objects stored in
the `Configuration` naming context, the objects are replicated on all the
Domain Controllers forest-wide.

| Name | Path | Description |
|------|------|-------------|
| `NTAuthCertificates` | `CN=NTAuthCertificates,CN=Public Key Services,CN=Services,CN=Configuration,<DOMAIN>` | The `NTAuthCertificates` store, also known as the `Enterprise NTAuth store` store, hold the certificate of the trusted `Certificate Authorities` (in the `cACertificate` attribute). |
| `Enrollment Services` | `CN=Enrollment Services,CN=Public Key Services,CN=Services,CN=Configuration,<DOMAIN>` |  |
| `Certificate Authority` | `CN=Certification Authorities,CN=Public Key Services,CN=Services,CN=Configuration,<DOMAIN>` | |
| `Certificate Templates` | `CN=Certificate Templates,CN=Public Key Services,CN=Services,CN=Configuration,<DOMAIN>` | Container holding the `certificate templates` defined in the domain, whether they are enabled in a `Certificate Authority` or not. |
| `CDP` | `CN=<CA_NAME>,CN=<ADCS_SERVER>,CN=CDP,CN=Public Key Services,CN=Services,CN=Configuration,<DOMAIN>` | Container storing the `Certificate Revocation Lists (CRL)`, with one separate container per `CA` and each `CA` thus having its own `CRL`. |

### Certificate templates

`Certificate templates` are domain objects of type `pKICertificateTemplate`,
stored under the `CN=Certificate Templates,CN=Public Key
Services,CN=Services,CN=Configuration,DC=<DOMAIN>,DC=<TLD>` container, that
govern the certificates that can be requested to and delivered by the
`Active Directory Certificate Services (AD CS)`.

A `certificate template` notably defines a number of parameters for the
certificates issued through the template:

  - The way the `Subject Name` and `Subject Alternative Name (Subject Alternative Name)`
    of the certificates will be constructed. The subject information can be
    either build:
      - automatically based on `Active Directory` information of the principal
        making the request (`User Principal Name (UPN)`,
        `Service Principal Name (SPN)`, `DNS` name, etc.).

      - using user-supplied data provided in the certificate request. In such
        case, the `CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT` (`0x1`) in the
        `msPKI-Certificate-Name-Flag` attribute is set (and the attribute thus
        has an odd value).

  - The issued certificates validity period.

  - The cryptographic parameters of the certificates (the
    `Cryptographic Services Provider (CSP)` and the minimum key size used for
    instance).

  - The `X509v3` extensions added to the certificates, including the
    `Extended / Enhanced Key Usage (EKU)` extension (introduced in more details
    below). The extensions define the purpose of the certificates.

  - Eventual issuance requirements:
      - Approval of a certificate manager to validate (or deny) the
        certificate request. This setting will set the
        `CT_FLAG_PEND_ALL_REQUESTS` (`0x02)` flag in the certificate template
        `msPKI-Enrollment-Flag` attribute.  Certificate requests will be keep
        in a pending state, awaiting for action of the certificate manager.

Additionally, `certificate templates` are `securable objects`, and the access
control rights defined in a `certificate template`'s
`Access Control List (ACL)` govern the operations that can be conducted on the
template itself and the principals that can enroll to the template. Refer to
the `[Active Directory] ACL exploiting - Active Directory Certificate Services`
note for more information on the `certificate templates` `ACL`.

#### Extended / Enhanced Key Usage extension

The `Extended / Enhanced Key Usage (EKU)` extension is a certificate extension
(i.e an additional attribute) that defines the purposes of the certificate,
effectively restricting what the certificate can be used for in an Active
Directory environment. This extension is implemented by the
`pKIExtendedKeyUsage` attribute on Active Directory certificate template
object.

The `EKU` extension is composed of 0 or more `Object Identifier (OID)`, each
`OID` corresponding to a specific purpose. The following notable `OIDs` are
supported in Active Directory:

| OID | Name / Description | Usage | Allow AD authentication |
|-----|--------------------|-------|-------------------------|
| `2.5.29.37.0` | `anyExtendedKeyUsage` | Certificate that can be used for any usage. | Yes. |
| `1.3.6.1.5.5.7.3.2` | `clientAuth` | Certificate used for client authentication (be it  `SSL` / `TLS` authentication for web client or to remote servers in Active Directory). | Yes. |
| `1.3.6.1.5.5.7.3.3` | `codeSigning` | Code signing certificate used to digitally sign executables (such as `PE` binaries or PowerShell scripts). | No. |
| `1.3.6.1.5.5.7.3.4` | `emailProtection` | Certificate used to encrypt or digitally sign emails through the `S/MIME` standard. | No. |
| `1.3.6.1.5.5.7.3.5` <br> `1.3.6.1.5.5.7.3.6` <br> `1.3.6.1.5.5.7.3.7` | `ipsecEndSystem` <br> `ipsecTunnel` <br> `ipsecUser` | Certificates used in an Internet Protocol SECurity (IPSEC) infrastructure. | No. |
| `1.3.6.1.5.2.3.4` | `keyPurposeClientAuth` | Certificate used in Active Directory for PKINIT client authentication (not present by default and requires to be manually added in the certificate template). | Yes. |
| `1.3.6.1.4.1.311.10.3.4` | `msEFS` | Certificate used to encrypt / decrypt `Encrypting File System (EFS)` `NTFS` filesystems on Windows. | No. |
| `1.3.6.1.5.5.7.3.1` | `serverAuth` | Certificate used for server authentication (for instance using the `SSL` / `TLS` protocol). | No. |
| `1.3.6.1.4.1.311.20.2.2` | `Smartcard logon` | Certificate used for smart card logon. | Yes. |

**If no `OID` is specified in the `EKU` extension, the certificate will by
default be valid for all usages in Windows, including client authentication**.
Applications may however rely on the `Constrained` `EKU` validation mode,
as implemented by Microsoft, which determine the valid usage of the certificate
using only the explicitly specified purposes (in all the certificate of the
chain).

#### Arbitrary / user-controlled Subject Alternative Name (ESC1)

As specified in the certificate processing logic in the
[Microsoft documentation](https://docs.microsoft.com/en-us/windows/security/identity-protection/smart-cards/smart-card-certificate-requirements-and-enumeration#client-certificate-requirements-and-mappings),
if an `User Principal Name (UPN)` is specified in a certificate's
`subjectAltName` field, the `UPN` is used to map the certificate to an user
account in Active Directory and conduct the `PKINIT` authentication as that
user. Having control on the `Subject Alternative Name` for which the
certificate will be emitted can thus be leveraged for privilege escalation,
notably if the `certificate template` supports client authentication (and can
be requested under the current privileges). For example, in such circumstances,
an unprivileged user could request a certificate specifying a more privileged
principal, such as the domain `Administrator` account or a member of the
`Domain Admins` group, and later use the certificate to authenticate under the
impersonated principal.

The `Subject Alternative Name` for which the certificate will be emitted is
user-controlled if either:

  - The specific `certificate template` is configured to use user-supplied data
    provided in the certificate request to define the `subjectAltName` (i.e, as
    noted above, the template `msPKI-Certificate-Name-Flag` attribute's
    `CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT` (`0x1`) flag is set).

  - A `Certificate Authority` server has, in its `HKEY_LOCAL_MACHINE` registry
    hive, the `EDITF_ATTRIBUTESUBJECTALTNAME2` (`0x00040000`) flag set. As
    stated in the
    [Microsoft documentation](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn786426\(v=ws.11\)#controlling-user-added-subject-alternative-names),
    if this flag is set, user-supplied alternative names are allowed for any
    `certificate template` published by the given `Certificate Authority`
    server.

    ```
    HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration\<CA_NAME>\PolicyModules\CertificateAuthority_MicrosoftDefault.Policy\EditFlags
    ```

#### Enrollment rights

To enroll for a `certificate template`, the following conditions must be meet:

  - The `certificate template` must be published by at least one `Certificate
    Authority`. The `certificate templates` published by a given `CA` are
    defined in the `certificateTemplates` attribute of the `CA`'s `Enrollment
    Service` (`pKIEnrollmentService`) object.

  - The given principal must be able to enroll to a `CA` publishing the
    `certificate template` (`Certificate-Enrollment` or
    `Certificate-AutoEnrollment` extended rights).

  - The given principal must have enrollment rights for the `certificate
    template`. The enrollment rights are defined on the `certificate
    template` object's `ACL` (notably the `Certificate-Enrollment` or
    `Certificate-AutoEnrollment` rights).

#### Certificate templates enumeration

Multiple tools and utilities can be used to enumerate the
`certificate templates`:
  - Windows built-in `certutil`
  - [`Certify`](https://github.com/GhostPack/Certify)
  - [`Invoke-Leghorn`](https://github.com/RemiEscourrou/Invoke-Leghorn/)
  - [`Certipy`](https://github.com/ly4k/Certipy)
  - `PingCastle`'s `healthcheck`

Additionally, `FarsightAD` can be used to enumerate certificate templates and
retrieve timestamps of last modification for critical attributes
(`msPKI-Certificate-Name-Flag`, `msPKI-Enrollment-Flag`,
`nTSecurityDescriptor`).

```bash
# Requires PowerShell 7+.

. .\FarsightAD.ps1

Export-ADHuntingADCSCertificateTemplates [-Server <DC_IP | DC_HOSTNAME>] [-Credential <PS_CREDENTIAL>] [-ADDriveName <AD_DRIVE_NAME>] [-OutputFolder <OUTPUT_FOLDER>] [-ExportType <CSV | JSON>]
```

### AD CS operations artefacts

#### ESEB database

#### Windows ETW

A number of non-default Windows events can be generated by
`Active Directory Certificate Services` on certificate operations, such as
certificate template updates, certificate requests, etc.

For events to be logged, the [following conditions](https://web.archive.org/web/20250321025912/https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/dn786432(v=ws.11)) must be meet:
  - Advanced auditing policy is enforced and `Audit Certification Services` is
    enabled.
  - The audit filter associated with the event(s) are enabled on the CA(s)
    (for example through: Certificate Authority Properties - in Certificate
    Services MMC snap-in -> Auditing -> <AUDIT_FILTER>).
  - For the certificate template change events to be recorded (events `4898`,
    `4898`, and `4900`), the specific audit configuration flag 
    `EDITF_AUDITCERTTEMPLATELOAD` must be enabled ofr the CA: 
    `certutil –setreg policy\EditFlags +EDITF_AUDITCERTTEMPLATELOAD`.


| Channel | Conditions | Events |
|---------|------------|--------|
| Security | Requires `Audit Certification Services` to be enabled (advanced audit policy) and CA `Issue and manage certificate requests` auditing filter. | Event `4886: Certificate Services received a certificate request`. <br><br> Logged upon a certificate request attempt, whether the request succeed or not. <br><br> Information of interest: <br> - Requester identity (`Requester`). <br> - client machine name (within the `Attributes` undocumented field, `ccm` - for `Cert Client Machine` - section). <br> - DOES NOT include certificate subject information. |
| Security | Requires `Audit Certification Services` to be enabled (advanced audit policy) and CA `Issue and manage certificate requests` auditing filter. | Event `4887: Certificate Services approved a certificate request and issued a certificate`. <br><br> Logged upon a successful certificate request (that lead to a certificate being emitted). <br><br> Information of interest: <br> - Requester identity (`Requester`). <br> - Client machine name (within the `Attributes` undocumented field, field `ccm`). <br> - Subject information: subject of the certificate (whether automatically deduced or manually specified). <br> - Depending on the `AD CS` version, the certificate template and eventual Subject Alternative Name specified may be tracked in the `Attributes` field. |
| Security | Requires `Audit Certification Services` to be enabled (advanced audit policy) and CA `Issue and manage certificate requests` auditing filter. | Event `4889: Certificate Services set the status of a certificate request to pending`. <br><br> Logged upon a certificate request attempt if the associated certificate template requires `CA certificate manager` approval. <br><br> Similar level of information as event `4886`. |   
| Security | Requires `Audit Certification Services` to be enabled (advanced audit policy) and `Revoke certificates and publish CRLs`. | Event `4870: Certificate Services revoked a certificate`. <br><br> Logged whenever an issued certificate is revoked. <br><br> Information of interest: <br> - Account that performed the revocation and associated `LogonID`. <br> - Serial number of the revoked certificate (`CertificateSerialNumber`). |
| Security | Requires `Audit Certification Services` to be enabled (advanced audit policy) and `Change CA configuration`. | Event `4892: A property of Certificate Services changed`. <br><br> Logged whenever their is a change to the `AD CS` CA and notably when a new certificate template is published to the CA. <br><br> Information of interest: <br> - Account that performed the operation and associated `LogonID`. <br> - For certificate templates publication (`PropertyName 29`), the list of all published certificate templates with their respective EKU. |
| Security | Requires `Audit Certification Services` to be enabled (advanced audit policy) and `Change CA security settings`. | Event `4885: The audit filter for Certificate Services changed`. <br><br> Logged upon changes of the `AD CS` CA audit filter policy. <br><br> Information of interest: <br> - Account that performed the update and associated `LogonID`. <br> - New audit filter configured (`AuditFilter`). |
| Security | Requires `Audit Certification Services` to be enabled (advanced audit policy) and `Change CA security settings`. | Event `4882: The security permissions for Certificate Services changed`. <br><br> Logged whenever the access rights on the `AD CS` CA itself are changed (and not on certificate template access rights update, which are defined `AD DS` side). <br><br> Information of interest: <br> - Account that performed the update and associated `LogonID`. <br> - The `AD CS` CA access rights in full (`SecuritySettings`). |
| Security | Requires `Audit Certification Services` to be enabled (advanced audit policy) and `Change CA security settings`. | Event `4890: The certificate manager settings for Certificate Services changed`. <br><br> Logged whenever the certificate manager restrictions of the `AD CS` are updated (and not the certificate manager rights, which is defined as an access right in the CA security descriptor). <br><br> Information of interest: <br> - Account that performed the update and associated `LogonID`. <br> - Whether restricted permissions are enabled or disabled (`EnableRestrictedPermissions`). <br> - If restricted permissions are enabled, the restricted permissions enforced (`RestrictedPermissions`). |
| Security | Requires `Audit Certification Services` to be enabled (advanced audit policy) and `EDITF_AUDITCERTTEMPLATELOAD` flag. | Event `4898: Certificate Services loaded a template`. <br><br> Logged whenever a certificate template is loaded by the `AD CS` CA. Certificate templates appear to be loaded periodically or after a certificate template update but only upon certificate requests. <br><br> Information of interest: <br> - All information of the certificate template: name and distinguished name, `ExtendedKeyUsage`, security descriptor, [`msPKI-Certificate-Name-Flag` attribute](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-crtd/1192823c-d839-4bc3-9b6b-fa8c53507ae1), etc. |
| Security | Requires `Audit Certification Services` to be enabled (advanced audit policy) and `EDITF_AUDITCERTTEMPLATELOAD` flag. | Event `4899: A Certificate Services template was updated`. <br><br> Logged whenever a certificate template published by the `AD CS` CA is modified. The event appears to be logged upon loading of the modified template (with similar loading circumstances as event `4898`). <br><br> Information of interest: <br> - Certificate template name and distinguished name. <br> - Settings / properties updated, with the old and new value(s) logged. |
| Security | Requires `Audit Certification Services` to be enabled (advanced audit policy) and `EDITF_AUDITCERTTEMPLATELOAD` flag. | Event `4899: A Certificate Services template was updated`. <br><br> Logged whenever a certificate template, published and loaded by the `AD CS` CA, is updated. The event appears to be logged upon loading of the modified template (with similar loading circumstances as event `4898`). If the certificate template is not already loaded, the event does not appear to be logged (a certificate template loading event `4898` is logged instead). <br><br> Information of interest: <br> - Certificate template name and distinguished name. <br> - Settings / properties updated, with the old and new value(s) logged. |
| Security | Requires `Audit Certification Services` to be enabled (advanced audit policy) and `EDITF_AUDITCERTTEMPLATELOAD` flag. | Event `4900: Certificate Services template security was updated`. <br><br> Logged whenever a certificate template, published and loaded by the `AD CS` CA, security descriptor is updated. The event appears to be logged upon loading of the modified template (with similar loading circumstances as event `4898`). If the certificate template is not already loaded, the event does not appear to be logged (a certificate template loading event `4898` is logged instead). <br><br> Information of interest: <br> - Certificate template name and distinguished name. <br> - Old and new security descriptors, in `Security Descriptor Definition Language` (`SDDL)` notation and human-readable format. |

The full list of events generated by `AD CS` CA is available in
[Microsoft official documentation](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/dn786423(v=ws.11)).

### Active Directory Domain Services artefacts

#### AD userCertificate attribute

The `userCertificate` attribute of a user / computer object is a multi-value
attribute that contains the public key of X.509 certificates issued to the
principal through `AD CS`. The `userCertificate` attribute is composed of an
array of nested byte arrays, containing the certificate objects.

The public key of the certificate issued to a principal is stored in this
attribute only if the `Publish to Active Directory` is set in the
`Certificate Template` associated with the certificate. This setting is
configured by default for some certificate templates.

The following PowerShell cmdlet, from
[`FarsightAD`](https://github.com/Qazeer/FarsightAD), can be used to parse
a (single) `userCertificate` attribute:

```
function Get-X509CertificateStringFromUserCertificate {
<#
.SYNOPSIS

Return a formatted string constructed from an object's userCertificate attribute.

.PARAMETER userCertificate

Specifies the Certificates attribute.

.OUTPUTS

[string]

#>

    Param(
        [Parameter(Mandatory=$True)] $userCertificate
    )
    
    $CertificatesString = ""
    
    for ($i = 0; $i -lt $userCertificate.Count; $i++) {
        # Requires PowerShell >= v5.
        # New-Object System.Security.Cryptography.X509Certificates.X509Certificate2($userCertificate[$i]) bugs in PowerShell v7+.
        $X509Certificate = [System.Security.Cryptography.X509Certificates.X509Certificate2]::new([byte[]] $userCertificate[$i])
        $EnhancedKeyUsageListString = If ($X509Certificate.EnhancedKeyUsageList) { [string]::join("-", $X509Certificate.EnhancedKeyUsageList) } Else { "None" }
        $X509CertificateAsString = [string]::Format("SerialNumber={0}|Subject={1}|NotBefore={2}|NotAfter={3}|EnhancedKeyUsageList={4}", $X509Certificate.SerialNumber, $X509Certificate.Subject, $X509Certificate.NotBefore, $X509Certificate.NotAfter, $EnhancedKeyUsageListString)
        $CertificatesString += "$X509CertificateAsString;"
    }

    return $CertificatesString
}

$account = Get-ADObject -Properties userCertificate [...]
Get-X509CertificateStringFromUserCertificate -userCertificate $account.userCertificate
```

Additionally, `FarsightAD`'s `Export-ADHuntingPrincipalsCertificates` can be
used to enumerate and parse users / computers `userCertificate` attribute,
identifying certificates valid for client authentication. A number of
parameters are retrieved for each certificate: certificate validity timestamps,
certificate purpose, certificate subject and eventual `SubjectAltName(s)`.

The cmdlet highlights `SubjectAltName` that do not match the associated account
`UPN` and will attempt to determine if the `SubjectAltName` is linked to a
privileged account. The last modification timestamp, from AD replication data,
is also retrieved for all `userCertificate` attributes.

```bash
Export-ADHuntingPrincipalsCertificates [-Server <DC_IP | DC_HOSTNAME>] [-Credential <PS_CREDENTIAL>] [-OutputFolder <OUTPUT_FOLDER>] [-ExportType <CSV | JSON>]
```

### References

  - [Wavestone - Jean Marsault - Microsoft ADCS – Abusing PKI in Active Directory Environment](https://www.riskinsight-wavestone.com/en/2021/06/microsoft-adcs-abusing-pki-in-active-directory-environment/)

  - [SpecterOps - Will Schroeder & Lee Christensen - Certified Pre-Owned (synthesis)](https://posts.specterops.io/certified-pre-owned-d95910965cd2)

  - [SpecterOps - Will Schroeder & Lee Christensen - Certified Pre-Owned](https://www.specterops.io/assets/resources/Certified_Pre-Owned.pdf)

  - [Sysadmins LV - Vadims Podans - Understanding Active Directory Certificate Services containers in Active Directory](https://www.sysadmins.lv/blog-en/understanding-active-directory-certificate-services-containers-in-active-directory.aspx)

  - [ldapwiki - ExtendedKeyUsage](https://ldapwiki.com/wiki/Wiki.jsp?page=ExtendedKeyUsage)

  - [Sysadmins LV - Vadims Podans - Constraining Extended Key Usages in Microsoft Windows](https://www.sysadmins.lv/blog-en/constraining-extended-key-usages-in-microsoft-windows.aspx)

  - [KEYFACTOR Hidden Dangers: Certificate Subject Alternative Names (SANs)](https://www.keyfactor.com/blog/hidden-dangers-certificate-subject-alternative-names-sans/)

  - [decoder - A "deep dive" in Cert Publishers Group](https://decoder.cloud/2023/11/20/a-deep-dive-in-cert-publishers-group/)

  - [Uwe Gradenegger - Overview of audit events generated by the Certification Authority](https://www.gradenegger.eu/en/overview-of-the-audit-events-generated-by-the-certification-body/)

  - [Microsoft - Securing PKI: Monitoring Public Key Infrastructure](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/dn786432(v=ws.11))
