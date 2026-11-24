---
title: wget HSTS history
summary: 'The wget utility''s HTTP Strict Transport Security (HSTS) history. This history references websites implementing HSTS that were accessed over HTTPS using wget.\n\n wget history is implemented as a plaintext file, with one entry per line. For each entry, the domain and timestamp of last access are notably referenced.'
keywords: wget, HSTS, HTTP, Strict Transport Security, wget-hsts
tags:
  - linux_misc
default_location: '~/.wget-hsts'
last_updated: 2024-08-09
sidebar: sidebar
permalink: linux_wget_hsts_history.html
folder: linux
---

### Overview

The `wget` utility's `HTTP Strict Transport Security (HSTS)` history. This
history references websites implementing `HSTS` that were accessed over `HTTPS`
using `wget`.

`HTTP Strict Transport Security` is a mechanism that restricts access to a
particular website to `HTTPS` only, if that website defines an `HSTS` policy
and was already accessed over `HTTPS` once. The `HSTS` policy that the web
client should follow is defined by the web server through the
`Strict-Transport-Security` `HTTP` response header. The web browser or utility
has to store the websites accessed over `HTTPS` (with `HSTS` implemented) for
the duration specified in the header to support `HSTS` and only allow
subsequent requests to that particular website over `HTTPS`.

### Information of interest

`wget`'s `HSTS` history is implemented as a plaintext file, with one entry per
line.

For each entry, the following notable information is available:

  - Domain name of the website accessed.

  - Creation timestamp in `UTC` (in `epoch` format) that defines when the
    entry was created. As the entry is overwritten upon each new access to a
    website defining an `HSTS` policy, the creation timestamp matches the last
    access to the website.

  - The maximum age (`max-age`) of the entry, as a duration in seconds, as
    specified by the `Strict-Transport-Security` `HTTP` response header from
    the server. The entry expires once the maximum age has elapsed since the
    creation timestamp (`<CREATED>` + `<MAX_AGE>`).

Example of a `wget` `HSTS` history:

```
# HSTS 1.0 Known Hosts database for GNU Wget.
# Edit at your own risk.
# <hostname>    <port>  <incl. subdomains>      <created>       <max-age>
www.wikipedia.org       0       1       1723158309      106384710
```
