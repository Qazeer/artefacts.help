---
title: wget HSTS history
summary: 'wget utility HTTP Strict Transport Security (HSTS) history. This history references websites implementing HSTS that were accessed in HTTPS using wget.\n\n wget history is implemented as a plaintext file, with one entry per line. For each entry, the domain and timestamp of last access are notably referenced.'
keywords: wget, HSTS, HTTP, Strict Transport Security, wget-hsts
tags:
  - linux_misc
default_location: '~/.wget-hsts'
last_updated: 2024-08-09
sidebar: sidebar
permalink: wget_hsts_history.html
folder: linux
---

### Overview

`wget` utility's `HTTP Strict Transport Security (HSTS)` history. This history
references websites, implementing `HSTS`, that were accessed in `HTTPS` using
`wget`.

`HTTP Strict Transport Security` is a mechanism to limit access to a
particular website in `HTTPS` only if that website defines an `HSTS` policy and
was already accessed in `HTTPS` once. The `HSTS` policy that the web client
should follow is defined by the web server through the
`Strict-Transport-Security` `HTTP` response header. The web browser or utility
has to store the websites accessed in `HTTPS` (with `HSTS` implemented) for the
duration specified in the header to support `HSTS` and only allow subsequent
requests to that particular website in `HTTPS`.

### Information of interest

`wget`'s `HSTS` history is implemented as a plaintext file, with one entry per
line.

For each entry, the following notable information are available:

  - Domain name of the website accessed.

  - Created timestamp in `UTC` (in `epoch` format) that defines when the entry
    was created. As the entry is overwritten upon new access to a website
    defining an `HSTS` policy, the created timestamp matches the last access to
    the website.

  - The maximum age retention timestamp in `UTC` (in `epoch` format) for the
    entry, as specified by the `Strict-Transport-Security` `HTTP` response
    header from the server.

Example of a `wget`'s `HSTS` history:

```
# HSTS 1.0 Known Hosts database for GNU Wget.
# Edit at your own risk.
# <hostname>    <port>  <incl. subdomains>      <created>       <max-age>
www.wikipedia.org       0       1       1723158309      106384710
```
