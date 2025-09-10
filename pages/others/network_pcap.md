---
title: Procedure - Network PCAP
summary: 'Wireshark & tshark, Zeek, and ngrep usage.'
keywords: PCAP, PCAP-NG, Wireshark, tshark, Zeek, Bro, ngrep
tags:
  - network
  - procedures
last_updated: 2024-09-19
sidebar: sidebar
permalink: network_pcap.html
folder: others
---

### Wireshark & tshark

[`Wireshark`](https://www.wireshark.org/) is a multi-platform network protocol
analyzer that supports deep packets inspection for hundreds of network
protocol. `tshark` is the command-line counterpart of `Wireshark`.

`Wireshark` and `tshark` can capture network traffic from a live network
interface or can read (and write) packets from a previously saved capture
file, supporting many file formats (such as `tcpdump` and `Pcap NG`).

Both utilities support the same
["display filter" engine](https://www.wireshark.org/docs/man-pages/wireshark-filter.html),
allowing advanced filtering on network protocol fields. Additionally, `tshark`
supports "read filters" that can be used to perform two-pass analysis. The
packets are first filtered using the read filter and the results can be
processed as if working on an intermediate `pcap` export. This can be useful to
combine `tshark` filters and statistic computation.   

#### `Wireshark` and `tshark` tips

  - To easily identify a field name (for later filtering for instance), it is
    possible to select/hover over the field in `Wireshark`, and the field
    name will be displayed in the
    [status bar](https://www.wireshark.org/docs/wsug_html_chunked/ChUseStatusbarSection.html)
    left side.

  - A field can be added as a column in `Wireshark` by right clicking the field
    in the packet details panel and selecting "Apply as Column", or by
    selecting the field and using the "Ctrl + Shift + I" shortcut.

  - The "Find Packet" menu ("Ctrl + F" shortcut) can be used to search string /
    regex in packet (list and content). This feature can be combined with
    display filters to limit the numbers of packets searched. A similar search
    can be conducted with `tshark` using the display filter
    `'frame contains "<STRING>"'` (for string) or `'frame matches "<REGEX>"'`
    (for case-insensitive regular expression).

#### `tshark` basic usage

```bash
# -n: Disables all name resolutions, to avoid DNS queries to external resolvers.
# -t ud: Displays the timestamp associated with the packet in UTC YYYY-MM-DD hh:mm:ss.SSSSSSSSS.
# -Y <DISPLAY_FILTER>: Filters the displayed results using <DISPLAY_FILTER> ("display filter").
# -2 -R <READ_FILTER>: Filters the results with <READ_FILTER>, allowing for 2 pass analysis.
# -z <STATISTIC_EXPRESSION>: Computes statistic based on the specified expression.
# -q: Limits stdout output, notably to avoid printing packets metadata for statistics (-z).
# -T <json | PRINT_FORMAT>: Specifies the output print format. Defaults to "text".
# -T fields -e <FIELD_1> ... -e <FIELD_N>: Extracts the specified fields.

tshark -n -t ud [-i <NETWORK_INTERFACE> | -r <INPUT_FILE>] [-Y '<DISPLAY_FILTER>'] [-T <json | PRINT_FORMAT>]

tshark -n [-i <NETWORK_INTERFACE> | -r <INPUT_FILE>] [-Y '<DISPLAY_FILTER>'] -w <OUTPUT_PCAP_FILE>

tshark -n -t ud [-i <NETWORK_INTERFACE> | -r <INPUT_FILE>] [-Y '<DISPLAY_FILTER>'] -T fields -e <FIELD_1> -e <FIELD_N>

tshark -n [-i <NETWORK_INTERFACE> | -r <INPUT_FILE>] [-2 -R '<READ_FILTER>'] -q -z <STATISTIC_EXPRESSION>
```

#### `tshark` command examples

Example of useful `tshark` commands with display/read filters (that may also
be used in `Wireshark`) and statistic expressions:

| tshark command | Description |
|----------------|-------------|
| `tshark -z help` | Lists the statistic expressions supported by `tshark`. |
| `tshark -nr <INPUT_FILE> -Y 'frame contains "<STRING>"'` <br><br> `tshark -nr <INPUT_FILE> -Y 'frame matches "<REGEX>"'` | Searches the packets content for the specified string or case-insensitive regular expression. |
| `tshark -qnr <INPUT_FILE> -z follow,<tcp | http>,ascii,<TCP_STREAM_NUMBER>` | Follows the specified `TCP` or `HTTP` stream, printing its content. |
| `tshark -nr <INPUT_FILE> -2 -R 'ip.src.addr == <SRC_IP> && ip.dst.addr in {10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16}' -q -z io,phs` <br><br> `tshark -nr <INPUT_FILE> -2 -R 'ip.src.addr == <IP> && ip.dst.addr in {10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16}' -q -z endpoints,ip` | Displays basic metrics (frames and bytes exchanged) per protocol (`io,phs`) or per remote hosts (`endpoints,ip`) for the given `IP` source `SRC_IP` to all private hosts. |
| `tshark -nr <INPUT_FILE> --export-objects smb,<OUTPUT_FOLDER>` <br><br> `tshark -nr <INPUT_FILE> --export-objects http,<OUTPUT_FOLDER>` | Extracts the files found in the `SMB` or `HTTP` streams to disk. |
| `tshark -nr <INPUT_FILE> -Y 'dns || udp.dstport == 53 || tcp.dstport == 53' -T fields -e ip.src -e dns.qry.name | sort -u` | Extracts `DNS` queries made per host. |
| `tshark -nr <INPUT_FILE> -Y 'tls.handshake.type eq 1'` <br><br> `tshark -nr <INPUT_FILE> -Y 'tls.handshake.type eq 1' -T fields -e ip.src -e tls.handshake.extensions_server_name | sort -u` | Lists/extracts the `TLS` `Client Hello` requests, that may contain [`Server Name Indication` (`SNI`)](https://www.cloudflare.com/learning/ssl/what-is-sni/) referencing the requested domain. <br><br> `Client Hello` requests can be listed or the `SNI` specified extracted per host. |
| `tshark -nr <INPUT_FILE> -Y 'http && http.request.method in {"GET", "POST"}'` | Extracts `GET` and `POST` `HTTP` requests. |
| `tshark -nr <INPUT_FILE> -Y 'http' -T fields -e http.host -e http.user_agent | sort -u` | Extracts the `User-Agents` (uniquely) used in `HTTP` requests per host. |

  - Extracts the `HTTP` request or response body content:

    ```bash
    file=<INPUT_FILE>

    # Filter HTTP requests.
    FILTER="http.request.method"

    # Filter HTTP responses.
    FILTER="http.response.code"

    for p in $(tshark -nr $file -Y "$FILTER" -T fields -e http.file_data); do
      printf $p | xxd -r -p
      printf "\n"
    done;
    ```

### Zeek

[`Zeek`](https://github.com/zeek/zeek), formerly `Bro`, is a network traffic
analysis (and security monitoring) framework. The `zeek` command-line utility
can be used to parse `PCAP` files and will produce
[several `log` files](https://docs.zeek.org/en/current/log-formats.html#working-with-log-files)
related to the network activity occurring in the `PCAP`.

By default `zeek` will output `TSV` files, but also support `JSON` outputs.

```bash
# TSV formatted output files.
zeek -r <PCAP_FILE>

# JSON formatted output files.
zeek -C -r <PCAP_FILE> LogAscii::use_json=T
```

### ngrep

`ngrep` is an utility that can be used to combine `grep`-like expression
matching and
[`BPF` filtering](https://www.tcpdump.org/manpages/pcap-filter.7.html)
against network packets and `PCAP` files.

`ngrep` can be used to filter packets into an output `pcap` file, for further
analysis with tools supporting deep packets inspection (such as `Wireshark`).

```bash
# -W Specifies  an  alternate  manner  for displaying output packets matching the expression.
#    The "byline" mode honors embedded line feeds (useful for observing HTTP transactions, for instance).
#    The "none" displays the IP and source/destination header on one line, the payload on a second line.
#    The "single" displays everything (header and payload) on a single line.

ngrep [-i] [-W <normal | single | byline> | -O <OUTPUT_PCAP_FILE] -I <INPUT_PCAP_FILE> '<GREP_EXPRESSION>' '<BPF_FILTER>'
```

Example query to extract `GET` and `POST` `HTTP` requests from `10.0.0.5` to
`10.0.0.0/16`:

```bash
ngrep -W byline -I traffic.pcap "^GET|^POST" "src host 10.0.0.5 and dst host 10.0"
```

### References

  - [Hacker Target - tshark tutorial and filter examples](https://hackertarget.com/tshark-tutorial-and-filter-examples/)
