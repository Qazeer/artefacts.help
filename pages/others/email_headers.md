---
title: SMTP email headers
summary: 'An email object contains an envelope and a content. The envelope is the information that the email client and server use to send the email to the correct recipient(s). The email content is composed of the header section and the email body.\n\n A number of email envelope and content headers are common / mandatory for the email lifecycle, and some headers can be of precious forensics value. Additionally, some headers are linked to optional security mechanisms (SPF, DKIM, and DMARC) that can help detect illegitimate / spoofed emails.'
keywords: SMTP, mail, email, envelope, content, headers, Message Transfer Agent, MTA, MAIL FROM, RCPT TO, Return-Path, From, Reply-To, SPF, DKIM, DMARC
tags:
  - network
  - mailservers
location: 'Email sender: Return-Path, From, and Reply-To headers. The Return-Path header can be protected against spoofing with SPF. The From header can be protected using DMARC (with SPF and / or DKIM).\n\n Originating server and MTA: Received header(s).\n\n Email legitimacy and anti-spoofing mechanisms, with associated headers:\n\n SPF: validate that the originating server is authorized to send emails for the sender domain.\n\n DKIM header: digitally sign (part of) the email using a public key associated with the sender domain.\n\n DMARC: extends SPF and DKIM by indicating to the receiver the actions to follow (block the email and notify an alerting address for example) if receiving an email with no or a failed SPK / DKIM authentication. Additionally, DMARC check the consistency of the domains from the "From" header, the DKIM signature, and the SMTP "MAIL FROM" command.'
last_updated: 2024-08-09
sidebar: sidebar
permalink: email_headers.html
folder: others
---

### Email envelope versus content

An email object contains an envelope and a content.

The envelope is the information that the email client and server use to
send the email to the correct recipient(s). It notably include the sender email
and the recipient email(s) specified with the `MAIL FROM` and `RCPT TO` `SMTP`
commands respectively. The envelope also includes the `Received` headers added
by the `Message Transfer Agent (MTA)` that relayed the email. The envelope
headers are not displayed to end-users.

The email content (sent through the `SMTP DATA` command) is composed of the
header section and the email body. The header section consists of a collection
of header fields, which can be displayed to end-users in their email client.
For instance, the email content `From` header is used to display the email
sender in email clients. The email body is the actual content of the email.

A number of email envelope and content headers are common / mandatory for the
email lifecycle, and some headers can be of precious forensics value.
Additionally, some headers are linked to optional security mechanisms (`SPF`,
`DKIM`, and `DMARC`) that can help detect illegitimate / spoofed emails.

### Email original sender

#### From, Return-Path, and Reply-To headers

The email address of the sender is positioned in three headers:

  - The `Return-Path` envelope header, whose value is based on the email
    specified in the `MAIL FROM` `SMTP` command. This header is verified by the
    `SPF` mechanism and is thus a more reliable source of information for
    determining the sender of an email. The `Return-Path` header is used to
    process the "bounces" that may occur with an email.

  - The `From` content header, that is displayed to the end-user as the sender
    of the email. The `From` header is not verified by the `SPF` mechanism and
    can thus be spoofed.

  - The `Reply-To` header, which simply specify the email to which human
    replies should be sent to (as the recipient of the new email). An
    arbitrary email can be specified with no incidence on email security
    mechanisms.

If the `From` and `Return-Path` headers differ, the `From` header may have been
spoofed for social engineering purpose. If `SPF` verification (detailed below)
fails, the `Return-Path` header may have been spoofed as well.

Note that the
`Domain-based Message Authentication Reporting and Conformance (DMARC)`
mechanism can be used to detect / prevent spoofing of the `From` header.

#### MTA Received headers

A `Received` header is added to the email envelope headers by each
`Message Transfer Agent (MTA)` that relayed the email. `Received` headers are
ordered in reverse chronological order, with the last `Received` header
corresponding to the one added first by the `MTA` closer to the email sender
(and the first appearing `Received` header corresponding to the `MTA` closer to
destination). The last `Received` header (placed the closest from the
`From` / `To` headers and the message body) can thus be used to identify the
`MTA` from which the email originated. The reputation and legitimacy of the
sender `MTA`, in the email context, can be analysed to determine the legitimacy
of the email.

Each `Received` header logs the sending and receiving `MTA` hostname and IP
address as well as the time of reception. Example of the first `Received`
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
detect and / or block spoofed emails by detecting illegitimate sender servers.
More specifically, the `SPF` mechanism will validate the domains a mail server
can send emails for (through the `MAIL FROM` envelope header of an email).

`SPF` can be used by organizations to define servers authorized to send emails
for their domain names. `SPF` relies on specific `DNS` `TXT` records, that
identify authorized servers and the comportment the receiver should follow in
case of an email reception from a non authorized server.

`SPK` `DNS` records follow the format below, with mechanisms / rules evaluated
from left-to-right and stopping on the first match (except for the `INCLUDE`
mechanism).

```
# Only the version 1 of SPF is supported, so the version tag will always be set to v=spf1.

v=<spf1 | SPF_VERSION> <QUALIFIER><MECHANISM_1> ... <QUALIFIER><MECHANISM_N>
```

The following `mechanisms` are supported:

| Mechanism | Description |
|-----------|-------------|
| `all` | Always matches. |
| `include:<DOMAIN>` | Evaluate the `SPF` policy of the specified domain, returning a `PASS` / `Neutral` / `Fail` / `Softfail` result (or an error). <br><br> Only `PASS` result will however be processed, effectively stopping the following mechanisms evaluation. Non-matched results will resume processing of the other further mechanisms. |
| `a[:<DOMAIN>]` | Check if the sender email server `IP` address is included in the `A` or `AAAA` `DNS` records of the `MAIL FROM` / `HELO` domain or the domain specified in the mechanism. |
| `mx[:<DOMAIN>]` | Check if the sender email server `IP` address is included in the `MX` `DNS` records of the `MAIL FROM` / `HELO` domain or the domain specified in the mechanism. |
| `ip4:<IPV4 | IPV4_CIDR>` | Check if the sender email server `IP` address is the specified IPv4 address or in the specified IPv4 address range. |
| `ip6:<IPV6 | IPV6_CIDR>` | Check if the sender email server `IP` address is the specified IPv6 address or in the specified IPv6 address range. |

The `qualifiers` determine the comportment the receiving email server should
follow if the `mechanism` match. The following `qualifiers` are supported:

| Qualifier keyword | Qualifier description | Description |
|-------------------|-----------------------|-------------|
| `+` | `PASS` | Allow the message. <br><br> I.e if the associated `mechanism` match, the message should be accepted by the receiving email server. <br><br> Default if the `qualifier` is not specified. |
| `-` | `FAIL` | Reject the message. <br><br> I.e if the associated `mechanism` match, the message should be rejected by the receiving email server. |
| `?` | `NEUTRAL` | The authoritative domain explicitly state that it is not asserting whether the sender email server `IP` address is authorized. <br><br> Can be processed as if the `SPF` record did not exist. I.e if the associated `mechanism` match, the message could be process as if no `SPF` record was configured by the receiving email server. |
| `~` | `SOFTFAIL` | The authoritative domain explicitly state that it is not asserting whether the sender email server `IP` address is authorized. <br><br> Same comportment as `NEUTRAL`, with difference in processing left to the receiving email server. |

#### Spoofed email SPF headers example

The following email headers correspond to a spoofed email headers (assuming
that `SPF` records are correctly configured):

```
Authentication-Results: spf=fail (sender IP is <SENDING_SERVER_IP>)
[...]

Received-SPF: Fail (protection.outlook.com: domain of <MAIL_FROM_OR_HELO_DOMAIN>
 does not designate <SENDING_SERVER_IP> as permitted sender)
 receiver=protection.outlook.com; client-ip=<SENDING_SERVER_IP>;
 helo=<SENDING_SERVER_FQDN>;
```

### References

  - [IETF - RFC5321 - Simple Mail Transfer Protocol](https://datatracker.ietf.org/doc/html/rfc5321)

  - [IETF - RFC7208 - Sender Policy Framework (SPF)](https://datatracker.ietf.org/doc/html/rfc7208)

  - [TrustedSec - Chris Camejo - Real or Fake? Spoof-Proofing Email With SPF, DKIM, and DMARC](https://www.trustedsec.com/blog/real-or-fake-spoof-proofing-email-with-spf-dkim-and-dmarc/)

  - [Peter Matkovski - Email Forensics; 2. Headers and Body](https://medium.com/@p.matkovski/email-forensics-2-headers-and-body-3e6280820983)
