---
title: ETW - Tools
summary: 'Tools for processing ETW and EVTX files, including: LogParser, Winlogbeat, EvtxECmd, Chainsaw, Hayabusa, and Velociraptor.'
keywords: 'Event Tracing for Windows tools, ETW tools, EVTX tools, EVTX parsing, Event Viewer, Event Log Explorer, LogParser, LogParser_LogonLogoffEvents, Winlogbeat, EvtxECmd, Chainsaw, Hayabusa, Velociraptor'
tags:
  - windows_etw
last_updated: 2024-01-02
sidebar: sidebar
permalink: windows_etw_tools.html
folder: windows
---

### Live system ETW providers and channels review

```bash
# Lists the system's provider
logman.exe query providers

# Retrieves information about a provider, including its channel and the process sending events to it.
logman.exe query providers "<PROVIDER_NAME>"

# Lists the providers the specified process emit events to.
logman query providers -pid <PID>

# List the available `channels` and their associated event counts.
Get-WinEvent -ListLog *
```

### EVTX files

#### Graphical applications

 - `Event Viewer`: Windows built-in `GUI` events viewer utility.

 - `Event Log Explorer`: Proprietary `GUI` events viewer utility.

#### [CLI] LogParser

`LogParser` can be used to conduct `SQL`-like queries on `EVTX files`.

Notable `KAPE` modules available that leverage `LogParser`:
`LogParser_LogonLogoffEvents`, `LogParser_RDPUsageEvents`, and
`LogParser_DetailedNetworkShareAccess`.

The following `LogParser.exe` query, as implemented in `KAPE` as the
`Logon-Logoff-events` module, extract and parse multiple `Security` events
related to Windows logon into an output `CSV` file. The following `events ID`
are processed: 4624, 4625, 4634, 4647, 4648, 4772, 4778, 4779, 4800, 4801,
4802, and 4803.

This query can prove useful for analysis of events from both Domain Controllers
and Windows servers or workstations.

```bash
# Author: Brian Maloney (idea by @0x47617279).

LogParser.exe -stats:OFF -i:EVT -o CSV "SELECT TO_UTCTIME(TimeGenerated) AS Date, EventID, CASE EventID WHEN 4624 THEN 'An account was successfully logged on' WHEN 4625 THEN 'An account failed to log on' WHEN 4634 THEN 'An account was logged off' WHEN 4647 THEN 'User initiated logoff' WHEN 4648 THEN 'A logon was attempted using explicit credentials' WHEN 4672 THEN 'Special privileges assigned to new logon' WHEN 4778 THEN 'A session was reconnected to a Window Station' WHEN 4779 THEN 'A session was disconnected from a Window Station' WHEN 4800 THEN 'The workstation was locked' WHEN 4801 THEN 'The workstation was unlocked' WHEN 4802 THEN 'The screen saver was invoked' WHEN 4803 THEN 'The screen saver was dismissed' END as Description, CASE EventID WHEN 4624 THEN EXTRACT_TOKEN(Strings, 5, '|') WHEN 4625 THEN EXTRACT_TOKEN(Strings, 5, '|') WHEN 4634 THEN EXTRACT_TOKEN(Strings, 1, '|') WHEN 4647 THEN EXTRACT_TOKEN(Strings, 1, '|') WHEN 4648 THEN EXTRACT_TOKEN(Strings, 1, '|') WHEN 4672 THEN EXTRACT_TOKEN(Strings, 1, '|') WHEN 4778 THEN EXTRACT_TOKEN(Strings, 0, '|') WHEN 4779 THEN EXTRACT_TOKEN(Strings, 0, '|') WHEN 4800 THEN EXTRACT_TOKEN(Strings, 1, '|') WHEN 4801 THEN EXTRACT_TOKEN(Strings, 1, '|') WHEN 4802 THEN EXTRACT_TOKEN(Strings, 1, '|') WHEN 4803 THEN EXTRACT_TOKEN(Strings, 1, '|') END as Username, CASE EventID WHEN 4624 THEN EXTRACT_TOKEN(Strings, 6, '|') WHEN 4625 THEN EXTRACT_TOKEN(Strings, 6, '|') WHEN 4634 THEN EXTRACT_TOKEN(Strings, 2, '|') WHEN 4647 THEN EXTRACT_TOKEN(Strings, 2, '|') WHEN 4648 THEN EXTRACT_TOKEN(Strings, 2, '|') WHEN 4672 THEN EXTRACT_TOKEN(Strings, 2, '|') WHEN 4778 THEN EXTRACT_TOKEN(Strings, 1, '|') WHEN 4779 THEN EXTRACT_TOKEN(Strings, 1, '|') WHEN 4800 THEN EXTRACT_TOKEN(Strings, 2, '|') WHEN 4801 THEN EXTRACT_TOKEN(Strings, 2, '|') WHEN 4802 THEN EXTRACT_TOKEN(Strings, 2, '|') WHEN 4803 THEN EXTRACT_TOKEN(Strings, 2, '|') END as Domain, CASE EventID WHEN 4648 THEN STRCAT(EXTRACT_TOKEN(Strings, 6, '|'),STRCAT('\\',EXTRACT_TOKEN(Strings, 5, '|'))) END AS CredentialsUsed, CASE EventID WHEN 4624 THEN EXTRACT_TOKEN(Strings, 7, '|') WHEN 4624 THEN EXTRACT_TOKEN(Strings, 7, '|') WHEN 4634 THEN EXTRACT_TOKEN(Strings, 3, '|') WHEN 4647 THEN EXTRACT_TOKEN(Strings, 3, '|') WHEN 4648 THEN EXTRACT_TOKEN(Strings, 3, '|') WHEN 4672 THEN EXTRACT_TOKEN(Strings, 3, '|') WHEN 4778 THEN EXTRACT_TOKEN(Strings, 2, '|') WHEN 4779 THEN EXTRACT_TOKEN(Strings, 2, '|') WHEN 4800 THEN EXTRACT_TOKEN(Strings, 3, '|') WHEN 4801 THEN EXTRACT_TOKEN(Strings, 3, '|') WHEN 4802 THEN EXTRACT_TOKEN(Strings, 3, '|') WHEN 4803 THEN EXTRACT_TOKEN(Strings, 3, '|') END AS LogonID, CASE EventID WHEN 4778 THEN EXTRACT_TOKEN(Strings, 3, '|') WHEN 4779 THEN EXTRACT_TOKEN(Strings, 3, '|') WHEN 4800 THEN EXTRACT_TOKEN(Strings, 4, '|') WHEN 4801 THEN EXTRACT_TOKEN(Strings, 4, '|') WHEN 4802 THEN EXTRACT_TOKEN(Strings, 4, '|') WHEN 4803 THEN EXTRACT_TOKEN(Strings, 4, '|') END AS SessionName, REPLACE_STR(REPLACE_STR(REPLACE_STR(REPLACE_STR(REPLACE_STR(REPLACE_STR(REPLACE_STR(REPLACE_STR(REPLACE_STR(REPLACE_STR(REPLACE_STR(CASE EventID WHEN 4624 THEN EXTRACT_TOKEN(Strings, 8, '|') WHEN 4625 THEN EXTRACT_TOKEN(Strings, 10, '|') WHEN 4634 THEN EXTRACT_TOKEN(Strings, 4, '|') END,'2','Logon via console'),'3','Network Logon'),'4','Batch Logon'),'5','Windows Service Logon'),'7','Credentials used to unlock screen'),'8','Network logon sending credentials (cleartext)'),'9','Different credentials used than logged on user'),'10','Remote interactive logon (RDP)'),'11','Cached credentials used to logon'),'12','Cached remote interactive (similar to Type 10)'),'13','Cached unlock (similar to Type 7)') AS LogonType, CASE EventID WHEN 4625 THEN CASE EXTRACT_TOKEN(strings, 7, '|') WHEN '0xc000005e' THEN 'There are currently no logon servers available to service the logon request' WHEN '0xc0000064' THEN 'user name does not exist' WHEN '0xc000006a' THEN 'user name is correct but the password is wrong' WHEN '0xc000006d' THEN 'user logon with misspelled or bad password' WHEN '0xc000006e' THEN 'unknown user name or bad password' WHEN '0xc000006f' THEN 'user tried to logon outside his day of week or time of day restrictions' WHEN '0xc0000070' THEN 'workstation restriction, or Authentication Policy Silo violation (look for event ID 4820 on domain controller)' WHEN '0xc0000071' THEN 'expired password' WHEN '0xc0000072' THEN 'account is currently disabled' WHEN '0xc00000dc' THEN 'Indicates the Sam Server was in the wrong state to perform the desired operation.' WHEN '0xc0000133' THEN 'clocks between DC and other computer too far out of sync' WHEN '0xc000015b' THEN 'The user has not been granted the requested logon type (aka logon right) at this machine' WHEN '0xc000018c' THEN 'The logon request failed because the trust relationship between the primary domain and the trusted domain failed' WHEN '0xc0000192' THEN 'An attempt was made to logon, but the netlogon service was not started' WHEN '0xc0000193' THEN 'account expiration' WHEN '0xc0000224' THEN 'user is required to change password at next logon' WHEN '0xc0000225' THEN 'evidently a bug in Windows and not a risk' WHEN '0xc0000234' THEN 'user is currently locked out' WHEN '0xc00002ee' THEN 'Failure Reason. An Error occurred during Logon' WHEN '0xc0000413' THEN 'Logon Failure. The machine you are logging onto is protected by an authentication firewall. The specified account is not allowed to authenticate to the machine' ELSE EXTRACT_TOKEN(strings, 7, '|') END END AS Status, CASE EventID WHEN 4625 THEN CASE EXTRACT_TOKEN(strings, 9, '|') WHEN '0xc000005e' THEN 'There are currently no logon servers available to service the logon request' WHEN '0xc0000064' THEN 'user name does not exist' WHEN '0xc000006a' THEN 'user name is correct but the password is wrong' WHEN '0xc000006d' THEN 'user logon with misspelled or bad password' WHEN '0xc000006e' THEN 'unknown user name or bad password' WHEN '0xc000006f' THEN 'user tried to logon outside his day of week or time of day restrictions' WHEN '0xc0000070' THEN 'workstation restriction, or Authentication Policy Silo violation (look for event ID 4820 on domain controller)' WHEN '0xc0000071' THEN 'expired password' WHEN '0xc0000072' THEN 'account is currently disabled' WHEN '0xc00000dc' THEN 'Indicates the Sam Server was in the wrong state to perform the desired operation.' WHEN '0xc0000133' THEN 'clocks between DC and other computer too far out of sync' WHEN '0xc000015b' THEN 'The user has not been granted the requested logon type (aka logon right) at this machine' WHEN '0xc000018c' THEN 'The logon request failed because the trust relationship between the primary domain and the trusted domain failed' WHEN '0xc0000192' THEN 'An attempt was made to logon, but the netlogon service was not started' WHEN '0xc0000193' THEN 'account expiration' WHEN '0xc0000224' THEN 'user is required to change password at next logon' WHEN '0xc0000225' THEN 'evidently a bug in Windows and not a risk' WHEN '0xc0000234' THEN 'user is currently locked out' WHEN '0xc00002ee' THEN 'Failure Reason. An Error occurred during Logon' WHEN '0xc0000413' THEN 'Logon Failure. The machine you are logging onto is protected by an authentication firewall. The specified account is not allowed to authenticate to the machine' ELSE EXTRACT_TOKEN(strings, 9, '|') END END AS SubStatus, CASE EventID WHEN 4624 THEN EXTRACT_TOKEN(strings, 9, '|') WHEN 4625 THEN EXTRACT_TOKEN(strings, 11, '|') END AS AuthPackage, CASE EventID WHEN 4624 THEN EXTRACT_TOKEN(Strings, 11, '|') WHEN 4625 THEN EXTRACT_TOKEN(Strings, 13, '|') WHEN 4648 THEN EXTRACT_TOKEN(Strings, 8, '|') WHEN 4778 THEN EXTRACT_TOKEN(Strings, 4, '|') WHEN 4779 THEN EXTRACT_TOKEN(Strings, 4, '|') END AS Workstation, CASE EventID WHEN 4624 THEN EXTRACT_TOKEN(Strings, 18, '|') WHEN 4625 THEN EXTRACT_TOKEN(Strings, 19, '|') WHEN 4648 THEN EXTRACT_TOKEN(Strings, 12, '|') WHEN 4778 THEN EXTRACT_TOKEN(Strings, 5, '|') WHEN 4779 THEN EXTRACT_TOKEN(Strings, 5, '|') END AS SourceIP INTO <DESTINATION_FOLDER>\logparser-Logon-Logoff-events.csv' FROM '<SECURITY_EVTX_FILE>' WHERE EventID IN (4624;4625;4634;4647;4648;4672;4778;4779;4800;4801;4802;4803) AND Username NOT IN ('SYSTEM'; 'ANONYMOUS LOGON'; 'LOCAL SERVICE'; 'NETWORK SERVICE') AND Domain NOT IN ('NT AUTHORITY')" -filemode:0
```

#### [CLI] Winlogbeat

`Winlogbeat` can be used to parse `EVTX` into JSON or to ship them to `ELK`.

The `Winlogbeat_ALL` `KAPE` module can be used to parse every `EVTX`
recursively found in the specified directory to JSON. As `winlogbeat` is
executed once per `EVTX` file, the module however suffers from poor
performance.

The [`Execute-Winlogbeat.ps1`](https://gist.github.com/Qazeer/4936ec6c9fa511500f9496d0ceacab22)
PowerShell script (and associated `KAPE` module
`PowerShell_Execute-Winlogbeat`) can be used to recursively process the
specified directory to execute `Winlogbeat` once on all EVTX file(s) found,
improving performance.

#### [CLI] EvtxECmd

`EvtxECmd` can be used to parse `EVTX` into CSV, JSON, or XML outputs, without
doing a full per fields extract however.

Associated `KAPE` modules: `EvtxECmd` and `EvtxECmd_RDP`.

#### [CLI] Chainsaw & Hayabusa

[`Chainsaw`](https://github.com/WithSecureLabs/chainsaw) (`KAPE` module
`Chainsaw`) and [`Hayabusa`](https://github.com/Yamato-Security/hayabusa)
(`KAPE` module `hayabusa_OfflineEventLogs` notably) are Rust utilities to
parse and extract key information from `EVTX` files. The utilities support
`Sigma` rules to generate a detection timeline of notable and potentially
suspicious activity.

#### Velociraptor

`Velociraptor` has built-in `EVTX` parsing capabilities and dedicated modules
to event logs analysis, such as `Windows.EventLogs.CondensedAccountUsage`,
`Windows.EventLogs.Chainsaw`, `Windows.EventLogs.Hayabusa`, etc.

#### LogonTracer for logon-related events

[`LogonTracer`](https://github.com/JPCERTCC/LogonTracer) is a tool to display
Active Directory and host logon-related events as a graph. Logon events are
represented as two nodes, the host (hostname or IP address) and the account
name, linked by the event information (`event ID`, number of occurrences,
etc.).

The following `events ID` are processed: 4624, 4625, 4768, 4769, 4776, and
4672.

Events can be filtered on a number of criteria:
  - The host(s) (hostname or IP) or user(s) concerned by the logon.
  - If the authentication provider is `NTLM` (AuthName: NTLM).
  - The logon type: `RDP` (Logon type 10), `Network` (Logon type 3), `Batch`
    (Logon type 4), and `Service` (Logon type 5).
  - If the logon was associated to special privileges (`event ID` 4672).
  - etc.

```bash
# LogonTracer default username: neo4j
# LogonTracer default password: password

# Pulls and installs the Docker container.
docker pull jpcertcc/docker-logontracer

# Runs the LogonTracer container.
docker run --detach --publish=7474:7474 --publish=7687:7687 --publish=8080:8080 -e LTHOSTNAME=<IP> jpcertcc/docker-logontracer

# Deletes the example data present by default in the container.
docker exec <CONTAINER_ID> python /usr/local/src/LogonTracer/logontracer.py --delete -u '<USERNAME>' -p '<PASSWORD>' -s <IP>

# It is advised to add Security.evtx hives through the web interface, exposed by default on the TCP port 8080.
Upload Event Log (bottom left) -> Browse -> One or multiple files can be selected -> Upload

# Alternatively, the Security.evtx hives can be upload using the logontracer.py Python script.
python3 logontracer.py [-e <EVTX> | -x <EVTX_XML>] -z <TIME_ZONE> -u '<USERNAME>' -p '<PASSWORD>' -s <IP>
```
