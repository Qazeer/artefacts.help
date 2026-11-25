---
title: SMTP email headers
summary: 'An email object contains an envelope and content. The envelope is the information that the email client and server use to send the email to the correct recipient(s). The email content is composed of the header section and the email body.\n\n A number of email headers are common/mandatory for the email lifecycle, and some headers can be of great forensic value. Additionally, some headers are linked to optional security mechanisms (SPF, DKIM, and DMARC) that can help detect illegitimate/spoofed emails.'
keywords: SMTP, mail, email, envelope, content, headers, Message Transfer Agent, MTA, MAIL FROM, RCPT TO, Return-Path, From, Reply-To, Received, SPF, DKIM, DMARC, DKIM-Signature, Authentication-Results
tags:
  - network
  - mailservers
location: 'Email sender: Return-Path, From, and Reply-To headers. The Return-Path header can be protected against spoofing with SPF. The From header can be protected using DMARC (with SPF and/or DKIM).\n\n Originating server and MTA: Received header(s), with only the Received headers added by trusted MTAs being reliable.\n\n Email legitimacy and anti-spoofing mechanisms, with associated headers (Authentication-Results, Received-SPF, and DKIM-Signature headers):\n\n SPF: validates that the originating server is authorized to send emails for the sender domain.\n\n DKIM: digitally signs (part of) the email using a private key of the signing domain, with the associated public key published in the DNS records of the signing domain for verification.\n\n DMARC: extends SPF and DKIM by checking that the domain of the "From" header is aligned with the domain validated by SPF (domain of the SMTP "MAIL FROM" command) and/or DKIM (signing domain). Additionally, DMARC indicates to the receiver the policy to follow (none, quarantine, or reject) for emails failing the DMARC validation and the addresses to which reports should be sent.'
last_updated: 2024-08-09
sidebar: sidebar
permalink: email_headers.html
folder: others
---

### Email envelope versus content

An email object contains an envelope and content.

The envelope is the information that the email client and server use to
send the email to the correct recipient(s). It notably includes the sender
email address and the recipient email address(es) specified with the
`MAIL FROM` and `RCPT TO` `SMTP` commands, respectively. The envelope is not
part of the email content and is thus not displayed to end users (the envelope
sender email address is, however, stored in the email content's `Return-Path`
header upon final delivery of the email).

The email content (sent through the `SMTP DATA` command) is composed of the
header section and the email body. The header section consists of a collection
of header fields, which can be displayed to end users in their email client.
For instance, the email content `From` header is used to display the email
sender in email clients. The email body is the actual content of the email.

The header section notably includes trace headers, added by each
`Message Transfer Agent (MTA)` that received the email (`Received` headers)
and upon final delivery of the email (`Return-Path` header). The trace headers
are usually not displayed by email clients, but can be reviewed by viewing the
email source / headers.

A number of email headers are common/mandatory for the email lifecycle, and
some headers can be of great forensic value.
Additionally, some headers are linked to optional security mechanisms (`SPF`,
`DKIM`, and `DMARC`) that can help detect illegitimate/spoofed emails.

### Email original sender

#### From, Return-Path, and Reply-To headers

The email address of the sender is present in three headers:

  - The `Return-Path` trace header, added upon final delivery of the email and
    whose value is based on the email address specified in the `MAIL FROM`
    `SMTP` command (envelope sender). The domain of this email address is
    verified by the `SPF` mechanism, and the header is thus a more reliable
    source of information for determining the sender of an email. The
    `Return-Path` header is used to process the "bounces" that may occur with
    an email.

  - The `From` content header, which is displayed to the end user as the sender
    of the email. The `From` header is not verified by the `SPF` mechanism and
    can thus be spoofed.

  - The `Reply-To` header, which simply specifies the email address to which
    human replies should be sent (as the recipient of the new email). An
    arbitrary email address can be specified with no impact on email security
    mechanisms.

If the `From` and `Return-Path` headers differ, the `From` header may have been
spoofed for social engineering purposes. If `SPF` verification (detailed below)
fails, the `Return-Path` header may have been spoofed as well.

Note that the
`Domain-based Message Authentication, Reporting, and Conformance (DMARC)`
mechanism (detailed below) can be used to detect/prevent spoofing of the
`From` header.

#### MTA Received headers

A `Received` header is added at the top of the email header section by every
`Message Transfer Agent (MTA)` that received the email. `Received` headers are
ordered in reverse chronological order, with the last `Received` header
corresponding to the one added first by the `MTA` closest to the email sender
(and the first appearing `Received` header corresponding to the `MTA` closest
to the destination). The last `Received` header (placed closest to the
`From`/`To` headers and the message body) can thus be used to identify the
`MTA` from which the email originated. The reputation and legitimacy of the
sender `MTA`, in the email context, can be analyzed to determine the legitimacy
of the email.

However, the `Received` headers can be forged, as the sender (or an `MTA`
under its control) can add arbitrary `Received` headers to the email before
sending it. Only the `Received` headers added by trusted `MTAs` (such as the
`MTAs` of the recipient organization) are reliable. The `Received` header
added by the first trusted `MTA` (i.e., the lowest `Received` header added by
a trusted `MTA`) records the `IP` address, determined from the network
connection, of the `MTA` from which the email was received, which can thus be
considered reliable. The hostname of the sending `MTA` (as presented by the
sending `MTA` itself in the `EHLO` `SMTP` command) and the `Received` headers
below this header can, however, be arbitrary.

Each `Received` header logs the sending and receiving `MTA` hostnames and IP
addresses, as well as the time of reception. Example of the first `Received`
header of an email sent through `O365`:

```
Received: from XXX.PROD.OUTLOOK.COM
 ([<IP>]) by YYY.PROD.OUTLOOK.COM
 ([<IP>]) with mapi id 15.20.5250.018; <DATE>
```

### SPF

#### Overview

`Sender Policy Framework (SPF)` is an email authentication mechanism, defined
in [RFC 7208](https://datatracker.ietf.org/doc/html/rfc7208), designed to
detect and/or block spoofed emails by detecting illegitimate sender servers.
More specifically, the `SPF` mechanism will validate the domains a mail server
can send emails for (through the domain of the envelope sender email address
specified with the `MAIL FROM` `SMTP` command, and optionally the domain
specified with the `HELO` / `EHLO` `SMTP` command).

`SPF` can be used by organizations to define servers authorized to send emails
for their domain names. `SPF` relies on specific `DNS` `TXT` records that
identify authorized servers and the behavior the receiver should follow in
case an email is received from an unauthorized server.

`SPF` `DNS` records follow the format below, with mechanisms/rules evaluated
from left to right and stopping on the first match (except for the `INCLUDE`
mechanism).

```
# Only version 1 of SPF is supported, so the version tag will always be set to v=spf1.

v=<spf1 | SPF_VERSION> <QUALIFIER><MECHANISM_1> ... <QUALIFIER><MECHANISM_N>
```

The following `mechanisms` are supported:

| Mechanism | Description |
|-----------|-------------|
| `all` | Always matches. |
| `include:<DOMAIN>` | Evaluate the `SPF` policy of the specified domain, returning a `PASS`/`Neutral`/`Fail`/`Softfail` result (or an error). <br><br> Only a `PASS` result will, however, be processed, effectively stopping the evaluation of the following mechanisms. Non-matching results will resume the processing of the remaining mechanisms. |
| `a[:<DOMAIN>]` | Check if the sender email server `IP` address is included in the `A` or `AAAA` `DNS` records of the `MAIL FROM`/`HELO` domain or the domain specified in the mechanism. |
| `mx[:<DOMAIN>]` | Check if the sender email server `IP` address is one of the `IP` addresses of the email servers referenced in the `MX` `DNS` records of the `MAIL FROM`/`HELO` domain or the domain specified in the mechanism. |
| `ip4:<IPV4 | IPV4_CIDR>` | Check if the sender email server `IP` address is the specified IPv4 address or in the specified IPv4 address range. |
| `ip6:<IPV6 | IPV6_CIDR>` | Check if the sender email server `IP` address is the specified IPv6 address or in the specified IPv6 address range. |

The `qualifiers` determine the behavior the receiving email server should
follow if the `mechanism` matches. The following `qualifiers` are supported:

| Qualifier keyword | Qualifier description | Description |
|-------------------|-----------------------|-------------|
| `+` | `PASS` | Allow the message. <br><br> I.e. if the associated `mechanism` matches, the message should be accepted by the receiving email server. <br><br> Default if the `qualifier` is not specified. |
| `-` | `FAIL` | Reject the message. <br><br> I.e. if the associated `mechanism` matches, the message should be rejected by the receiving email server (the final handling of the message being, however, a matter of local policy of the receiving email server). |
| `?` | `NEUTRAL` | The authoritative domain explicitly states that it is not asserting whether the sender email server `IP` address is authorized. <br><br> Can be processed as if the `SPF` record did not exist. I.e. if the associated `mechanism` matches, the message could be processed by the receiving email server as if no `SPF` record was configured. |
| `~` | `SOFTFAIL` | The authoritative domain states that the sender email server `IP` address is probably not authorized, without publishing a stronger, more definitive, `FAIL` policy. <br><br> I.e. if the associated `mechanism` matches, the message should be accepted but marked as suspicious (or subjected to closer scrutiny) by the receiving email server. The message should not be rejected based solely on this result. |

#### Spoofed email SPF headers example

The following email headers correspond to the headers of a spoofed email (assuming
that `SPF` records are correctly configured):

```
Authentication-Results: spf=fail (sender IP is <SENDING_SERVER_IP>)
[...]

Received-SPF: Fail (protection.outlook.com: domain of <MAIL_FROM_OR_HELO_DOMAIN>
 does not designate <SENDING_SERVER_IP> as permitted sender)
 receiver=protection.outlook.com; client-ip=<SENDING_SERVER_IP>;
 helo=<SENDING_SERVER_FQDN>;
```

Similarly to the `Received` headers, the `Authentication-Results` headers can
be forged by the sender. Only the `Authentication-Results` headers added by
the email servers of the recipient organization should be trusted (with
conformant email servers removing, upon reception, the
`Authentication-Results` headers claiming to have been added by their
organization).

### DKIM

#### Overview

`DomainKeys Identified Mail (DKIM)` is an email authentication mechanism,
defined in [RFC 6376](https://datatracker.ietf.org/doc/html/rfc6376), that
allows a domain (the signing domain) to claim responsibility for an email by
digitally signing (part of) the email. The signature is computed using a
private key of the signing domain and is stored in a `DKIM-Signature` header
added to the email. The associated public key is published in a `DNS` `TXT`
record of the signing domain (`<SELECTOR>._domainkey.<SIGNING_DOMAIN>`), which
is retrieved by the receiving email server to verify the signature.

A valid `DKIM` signature indicates that the signed headers and the body of the
email were not modified since the email was signed with the private key of the
signing domain. However, the signing domain is not necessarily the domain of
the `From` header, as any domain (including a domain controlled by an
attacker) can sign an email. The `DMARC` mechanism (detailed below) verifies
the alignment between the signing domain and the domain of the `From` header.
An email can also contain multiple `DKIM-Signature` headers, from the same or
different organizations involved with the email.

#### DKIM-Signature header

The `DKIM-Signature` header notably includes the following tags:

| Tag | Description |
|-----|-------------|
| `d` | Signing domain, used (with the selector) to retrieve the public key from `DNS`. |
| `s` | Selector, allowing multiple public keys to be published for the signing domain. |
| `h` | List of the signed headers. |
| `bh` | Hash of the (canonicalized) body of the email. |
| `b` | Signature of the signed headers and of the `DKIM-Signature` header itself (which includes the body hash). |
| `t` | Timestamp, in `epoch` format, of the signature creation (recommended but optional tag). |
| `x` | Timestamp, in `epoch` format, of the signature expiration (recommended but optional tag). |

Example of a `DKIM-Signature` header:

```
DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed; d=<SIGNING_DOMAIN>;
 s=<SELECTOR>; t=<EPOCH_TIMESTAMP>;
 h=From:To:Subject:Date:Message-ID:Content-Type:MIME-Version;
 bh=<BASE64_BODY_HASH>;
 b=<BASE64_SIGNATURE>
```

The result of the `DKIM` signature verification is recorded by the receiving
email server in the `Authentication-Results` header
(`dkim=<pass | fail | none>`, with `header.d` referencing the signing domain).
For example, for an email received in `Microsoft 365` whose body was modified
after signing:

```
Authentication-Results: [...] dkim=fail (body hash did not verify)
 header.d=<SIGNING_DOMAIN>;[...]
```

As the signing domains may rotate their keys (and remove or revoke the
previous public keys from `DNS`), the `DKIM` signature of an old email may not
be verifiable anymore.

### DMARC

#### Overview

`Domain-based Message Authentication, Reporting, and Conformance (DMARC)` is an
email authentication mechanism, defined in
[RFC 9989](https://datatracker.ietf.org/doc/html/rfc9989) (which obsoleted
[RFC 7489](https://datatracker.ietf.org/doc/html/rfc7489) in May 2026), that
extends `SPF` and `DKIM` to detect and/or block emails spoofing the domain of
the `From` header (displayed to end users as the email sender).

The `DMARC` validation passes if the domain of the `From` header is aligned
with at least one of the following domains:

  - The domain of the `MAIL FROM` `SMTP` command (envelope sender), if the
    `SPF` validation passed.

  - The signing domain (`d` tag) of a valid `DKIM` signature.

The alignment can be relaxed (the domains have the same organizational domain,
for example `mail.example.com` and `example.com`), which is the default mode,
or strict (the domains are identical).

`DMARC` relies on a specific `DNS` `TXT` record (`_dmarc.<DOMAIN>`) that
defines the policy the receiving email server should follow if the `DMARC`
validation fails, as well as the addresses to which reports should be sent.
`DMARC` `DNS` records follow the format below:

```
v=DMARC1; p=<none | quarantine | reject>; [<TAG>=<VALUE>; ...]

# Example
v=DMARC1; p=reject; rua=mailto:<EMAIL_ADDRESS>
```

The following tags are notably supported:

| Tag | Description |
|-----|-------------|
| `p` | Policy for the emails failing the `DMARC` validation: <br><br> - `none`: no specific action requested (monitoring of the domain usage through reports). <br><br> - `quarantine`: the emails should be treated as suspicious (for example, placed into the spam folder). <br><br> - `reject`: the emails should be rejected. <br><br> The final handling of the emails is, however, a matter of local policy of the receiving email server. |
| `sp` / `np` | Policy for the existing / non-existent subdomains (same values as the `p` tag). |
| `adkim` / `aspf` | Alignment mode for `DKIM` / `SPF`: `r` for relaxed (default) or `s` for strict. |
| `rua` | Addresses to which aggregate reports should be sent. |
| `ruf` | Addresses to which per-message failure reports should be sent. |
| `t` | Test mode: if set to `y`, the receiving email server is requested to apply a policy one level below the specified policy (`n` by default). <br><br> Replaces the `pct` tag (percentage of emails to which the policy should be applied) of `RFC 7489`. |

#### Spoofed email DMARC headers example

The result of the `DMARC` validation is recorded by the receiving email server
in the `Authentication-Results` header (`dmarc=<pass | fail | none>`, with
`header.from` referencing the domain of the `From` header).

The following email headers correspond to the headers of a spoofed email
received in `Microsoft 365`, sent from a domain controlled by the attacker
(with valid `SPF` and `DKIM` results for this domain) but with a spoofed
domain in the `From` header (assuming that a `DMARC` record is configured for
the spoofed domain):

```
Authentication-Results: spf=pass (sender IP is <SENDING_SERVER_IP>)
 smtp.mailfrom=<ATTACKER_DOMAIN>; dkim=pass (signature was verified)
 header.d=<ATTACKER_DOMAIN>;dmarc=fail action=<ACTION>
 header.from=<SPOOFED_DOMAIN>;[...]
```

### References

  - [IETF - RFC5321 - Simple Mail Transfer Protocol](https://datatracker.ietf.org/doc/html/rfc5321)

  - [IETF - RFC5322 - Internet Message Format](https://datatracker.ietf.org/doc/html/rfc5322)

  - [IETF - RFC7208 - Sender Policy Framework (SPF)](https://datatracker.ietf.org/doc/html/rfc7208)

  - [IETF - RFC6376 - DomainKeys Identified Mail (DKIM) Signatures](https://datatracker.ietf.org/doc/html/rfc6376)

  - [IETF - RFC9989 - Domain-Based Message Authentication, Reporting, and Conformance (DMARC)](https://datatracker.ietf.org/doc/html/rfc9989)

  - [IETF - RFC8601 - Message Header Field for Indicating Message Authentication Status](https://datatracker.ietf.org/doc/html/rfc8601)

  - [Microsoft Learn - Anti-spam message headers in Microsoft 365](https://learn.microsoft.com/en-us/defender-office-365/message-headers-eop-mdo)

  - [Apache SpamAssassin - Trusted relays, and how header trust works](https://cwiki.apache.org/confluence/display/SPAMASSASSIN/TrustedRelays)

  - [TrustedSec - Chris Camejo - Real or Fake? Spoof-Proofing Email With SPF, DKIM, and DMARC](https://www.trustedsec.com/blog/real-or-fake-spoof-proofing-email-with-spf-dkim-and-dmarc/)

  - [Peter Matkovski - Email Forensics; 2. Headers and Body](https://medium.com/@p.matkovski/email-forensics-2-headers-and-body-3e6280820983)
