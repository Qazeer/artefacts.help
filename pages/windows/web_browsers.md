---
title: Web browsers
summary: 'The web browsers related artefacts can be split in the following categories:\n\n- User profile: web browsers, such as Chronium-based browsers (Chrome, Edge) and Firefox, implement a profile feature to store users settings, history, favorites, etc. The databases and files that store these information are usually stored under a user specific profile folder.\n\n- History: web browsing history and download history.\n\n- Cookies: web browsing cookies (session tokens).\n\n- Cache: cache of resources downloaded from accessed websites.\n\n- Sessions: tabs and windows from a browsing session.\n\n- Settings: configuration settings.'
keywords:
tags:
  - windows_browsing_history
location: 'Web browsers artefacts are often located under:\n - %LocalAppData%: <SYSTEMDRIVE>:\Users\<USERNAME>\AppData\Local.\n - %AppData%: <SYSTEMDRIVE>:\Users\<USERNAME>\AppData\Roaming.\n\n Internet Explorer\n----------------------\nBrowsing history, downloads, cache, cookies metadata: %LocalAppData%\Microsoft\Windows\WebCache\WebCacheV01.dat.\nCookies: %AppData%\Microsoft\Windows\Cookies.\nSessions: %LocalAppData%\Microsoft\Internet Explorer\Recovery\*.dat.\n\n Edge (Legacy)\n-------------------\nBrowsing history, downloads, cache, cookies metadata: %LocalAppData%\Microsoft\Windows\WebCache\WebCacheV01.dat;\nUser profiles: %LocalAppData%\Packages\Microsoft.MicrosoftEdge_XXX\AC.\nCache: <USER_PROFILE>\MicrosoftEdge\Cache.\nSessions: <USER_PROFILE>\MicrosoftEdge\User\Default\Recovery\Active.\nSettings: <USER_PROFILE>\User\Default\DataStore\Data\nouser1\XXX\DBStore\spartan.edb.\n\nEdge (Chronium-based)\n--------------------------------\nUser profiles: %LocalAppData%\Microsoft\Edge\User Data\<Default | Profile X>.\nBrowsing history: <USER_PROFILE>\History.\nCookies: <USER_PROFILE>\Cookies.\nCache: <USER_PROFILE>\Cache.\nSessions: <USER_PROFILE>\Sessions.\nSettings: <USER_PROFILE>\Preferences.\n\nGoogle Chrome\n----------------------\nUser profiles: %LocalAppData%\Google\Chrome\User Data\<Default | Profile X>.\nBrowsing history: <USER_PROFILE>\History.\nCookies: <USER_PROFILE>\Cookies.\nCache: <USER_PROFILE>\Cache.\nSessions: <USER_PROFILE>\Sessions.\nSettings: <USER_PROFILE>\Preferences.\n\nMozilla Firefox\n--------------------\nUser profiles: %AppData%\Mozilla\Firefox\Profiles\<ID>.default-release.\nBrowsing history, downloads, bookmarks: <USER_PROFILE>\places.sqlite.\nCookies: <USER_PROFILE>\cookies.sqlite.\nCache: <USER_PROFILE>\cache2\*.\nSessions: <USER_PROFILE>\sessionstorebackups\*.\nSettings: <USER_PROFILE>\prefs.js.'
last_updated: 2024-01-17
sidebar: sidebar
permalink: windows_web_browsers.html
folder: windows
---

### Overview

The web browsers related artefacts can be split in the following categories:

  - User profile: web browsers, such as `Chronium`-based browsers and
    `Firefox`, implement a profile feature to store user's settings, history,
    favorites, etc. The databases and files that store these information are
    usually stored under a user specific profile folder.

  - History: web browsing history and download history.

  - Cookies: web browsing cookies (session tokens).

  - Cache: cache of resources downloaded from accessed websites (images, text
    content, `HTML`, `CSS`, `Javascript` files, etc.).

  - Sessions: tabs and windows from a browsing session.

  - Settings: configuration settings.

These files are often located under `%LocalAppData%`
(`%SystemDrive%:\Users\<USERNAME>\AppData\Local\`) and
`%AppData%` (`%SystemDrive%:\Users\<USERNAME>\AppData\Roaming\`).

### Artefacts

#### Internet Explorer

| Type | Description | Location |
|------|-------------|----------|
| Browsing history <br><br> Downloads <br><br> Cache <br><br> Cookies metadata | The `WebCacheV01.dat` is an `ESE` database, with information about `Internet Explorer` browsing activity split across multiple tables: <br><br> - Browsing history: `History` table. <br><br> - Downloads: `iedownload` table. <br><br> - Cache: `content` table. <br><br> - Cookies metadata: `Cookies` table. <br><br> [Local files accessed, not necessarily through the web browser, may also appear in the `WebCacheV01.dat` database](./webcachev01.md). | `%LocalAppData%\Microsoft\Windows\WebCache\WebCacheV01.dat` |
| Cookies | - | `%AppData%\Microsoft\Windows\Cookies` |
| Sessions | - | `%LocalAppData%\Microsoft\Internet Explorer\Recovery\*.dat` |

#### Edge (Legacy)

| Type | Description | Location |
|------|-------------|----------|
| User profile(s) | - | `%LocalAppData%\Packages\Microsoft.MicrosoftEdge_XXX\AC`
| Browsing history <br><br> Downloads <br><br> Cache <br><br> Cookies metadata | Shared with `Microsoft Internet Explorer`. | `%LocalAppData%\Microsoft\Windows\WebCache\WebCacheV01.dat` |
| Cache | - | `%LocalAppData%\Packages\Microsoft.MicrosoftEdge_XXX\AC#!XXX\MicrosoftEdge\Cache`
| Sessions | - | `%LocalAppData%\Packages\Microsoft.MicrosoftEdge_XXX\AC\MicrosoftEdge\User\Default\Recovery\Active`
| Settings | - | `%LocalAppData%\Packages\Microsoft.MicrosoftEdge_XXX\AC\MicrosoftEdge\User\Default\DataStore\Data\nouser1\XXX\DBStore\spartan.edb` |

#### Edge (Chronium-based)

Since Edge version `v79` (January 2020), `Microsoft Edge` uses a `Chronium`
backend and shares similar artefacts to `Google Chrome`.

| Type | Description | Location |
|------|-------------|----------|
| User profile(s) | - | `%LocalAppData%\Microsoft\Edge\User Data\<Default | Profile X>\*` <br><br> *With `X` ranging from one to n.* |
| Browsing history | - |`%LocalAppData%\Microsoft\Edge\User Data\<Default | Profile X>\History` |
| Cookies | - | `%LocalAppData%\Microsoft\Edge\User Data\<Default | Profile X>\Network\Cookies` |
| Cache | - | `%LocalAppData%\Microsoft\Edge\User Data\<Default | Profile X>\Cache` |
| Sessions | - | `%LocalAppData%\Microsoft\Edge\User Data\<Default | Profile X>\Sessions` |
| Settings | - | `%LocalAppData%\Microsoft\Edge\User Data\<Default | Profile X>\Preferences` |

#### Google Chrome

Browsing history: <USER_PROFILE>\History.\nCookies: <USER_PROFILE>\Cookies.\nCache: <USER_PROFILE>\Cache.\nSessions: <USER_PROFILE>\Sessions.\nSettings: <USER_PROFILE>\Preferences.\n

| Type | Description | Location |
|------|-------------|----------|
| User profile(s) | - | `%LocalAppData%\Google\Chrome\User Data\<Default | Profile X>\*` <br><br> *With `X` ranging from one to n.* |
| History | - | `%LocalAppData%\Google\Chrome\User Data\<Default | Profile X>\History` |
| Cookies | - | `%LocalAppData%\Google\Chrome\User Data\<Default | Profile X>\Network\Cookies` |
| Cache | - | `%LocalAppData%\Google\Chrome\User Data\<Default | Profile X>\Cache` |
| Sessions | - | `%LocalAppData%\Google\Chrome\User Data\<Default | Profile X>\Sessions` |
| Settings | - | `%LocalAppData%\Google\Chrome\User Data\<Default | Profile X>\Preferences` |

#### Mozilla Firefox

| Type | Description | Location |
|------|-------------|----------|
| User profile(s) | - | `%AppData%\Mozilla\Firefox\Profiles\<ID>.default-release\*` |
| Browsing history <br><br> Downloads <br><br> Bookmarks | - | `%AppData%\Mozilla\Firefox\Profiles\<ID>.default-release\places.sqlite` |
| Cookies | - | `%AppData%\Mozilla\Firefox\Profiles\<ID>.default-release\cookies.sqlite` |
| Cache | - | `%LocalAppData%\Mozilla\Firefox\Profiles\<ID>.default-release\cache2\*` |
| Sessions | - | `%AppData%\Mozilla\Firefox\Profiles\<ID>.default-release\sessionstorebackups\*` |
| Settings | - | `%AppData%\Mozilla\Firefox\Profiles\<ID>.default-release\prefs.js` |

### Tool(s)

The [`NirSoft's BrowsingHistoryView`](https://www.nirsoft.net/utils/browsing_history_view.html)
utility (`NirSoft_BrowsingHistoryView` KAPE module) can be used to parse a
number of browsers artefacts to extract browsing history information.

`BrowsingHistoryView` can be used either as a graphical application or as a
command-line utility to export the parsing result (for instance in the CSV
format).

```bash
# /HistorySource 3: Load history from the specified profiles folder (specified using /HistorySourceFolder).
# /HistorySourceFolder <USER_PROFILES_FOLDER> example: "C:\Users" or "\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Users" (for shadow copy).
# /VisitTimeFilterType 1: Load history dating back to any time.
# /ShowTimeInGMT 1: Converts timestamps to UTC-0 (default to the local timezone).

browsinghistoryview.exe /HistorySource 3 /HistorySourceFolder "<USER_PROFILES_FOLDER>" /VisitTimeFilterType 1 /ShowTimeInGMT 1 /scomma <OUTPUT_CSV>
```

### References

  - [13Cubed - Windows Browser Artifacts Cheat Sheet](https://www.13cubed.com/downloads/windows_browser_artifacts_cheat_sheet.pdf)

  - [hacktricks - Browser Artifacts](https://book.hacktricks.xyz/forensics/basic-forensic-methodology/specific-software-file-type-tricks/browser-artifacts)

  - [Nirsoft - BrowsingHistoryView](https://www.nirsoft.net/utils/browsing_history_view.html)
