# Wireshark — Practical Guide & Command Cheat Sheet

Wireshark is an open-source network protocol analyzer used to capture, inspect, and analyze network traffic. It is useful for network troubleshooting, security analysis, protocol debugging, and computer forensics.

## Table of Contents

* [What is Wireshark?](#what-is-wireshark)
* [Installation](#installation)
* [Starting Wireshark](#starting-wireshark)
* [Capturing Packets](#capturing-packets)
* [Important Display Filters](#important-display-filters)
* [Protocol Filters](#protocol-filters)
* [IP and Port Filters](#ip-and-port-filters)
* [TCP Analysis](#tcp-analysis)
* [HTTP Analysis](#http-analysis)
* [DNS Analysis](#dns-analysis)
* [ARP Analysis](#arp-analysis)
* [ICMP Analysis](#icmp-analysis)
* [Useful tshark Commands](#useful-tshark-commands)
* [Useful Wireshark Tools](#useful-wireshark-tools)
* [Packet Analysis Workflow](#packet-analysis-workflow)
* [Security Analysis Examples](#security-analysis-examples)
* [Useful Resources](#useful-resources)

---

## What is Wireshark?

Wireshark captures network packets and allows you to inspect their contents.

A packet can contain information such as:

* Source IP
* Destination IP
* Source port
* Destination port
* Protocol
* TCP flags
* DNS queries
* HTTP information
* Packet length
* Timestamps

Wireshark can be used to investigate:

```text
Client
   |
   | Network traffic
   v
[ Wireshark ]
   |
   +-- TCP
   +-- UDP
   +-- DNS
   +-- HTTP
   +-- TLS
   +-- ICMP
   +-- ARP
```

---

# Installation

## Windows

Download Wireshark from the official website:

https://www.wireshark.org/download.html

Install Wireshark and, when prompted, install Npcap if you need to capture traffic.

## Linux

Ubuntu/Debian:

```bash
sudo apt update
sudo apt install wireshark
```

Start Wireshark:

```bash
wireshark
```

## macOS

Install using Homebrew:

```bash
brew install --cask wireshark
```

---

# Starting Wireshark

Start the graphical application:

```bash
wireshark
```

Alternatively, use `tshark`, Wireshark's command-line packet analyzer:

```bash
tshark
```

List available interfaces:

```bash
tshark -D
```

Example:

```text
1. eth0
2. wlan0
3. lo
```

Capture packets on an interface:

```bash
sudo tshark -i wlan0
```

Replace `wlan0` with the interface you want to monitor.

---

# Capturing Packets

In the Wireshark GUI:

1. Open Wireshark.
2. Select a network interface.
3. Click **Start Capturing Packets**.
4. Generate some network traffic.
5. Stop the capture.
6. Save the capture as `.pcapng`.

Example filename:

```text
network-analysis.pcapng
```

---

# Capture Filters vs Display Filters

Wireshark has two different types of filters.

## Capture Filter

A capture filter determines which packets are captured.

Example:

```text
host 192.168.1.10
```

Only traffic involving that host is captured.

Other examples:

```text
tcp
udp
icmp
port 80
port 443
host 192.168.1.10
net 192.168.1.0/24
```

## Display Filter

A display filter is applied after packets have been captured.

Examples:

```text
tcp
```

```text
dns
```

```text
http
```

```text
ip.addr == 192.168.1.10
```

Display filters are generally more powerful for investigating an existing capture.

---

# Important Display Filters

## Filter by IP address

```text
ip.addr == 192.168.1.10
```

Source IP:

```text
ip.src == 192.168.1.10
```

Destination IP:

```text
ip.dst == 192.168.1.10
```

---

## Filter by protocol

TCP:

```text
tcp
```

UDP:

```text
udp
```

ICMP:

```text
icmp
```

DNS:

```text
dns
```

HTTP:

```text
http
```

TLS:

```text
tls
```

ARP:

```text
arp
```

---

# Port Filters

TCP port 80:

```text
tcp.port == 80
```

TCP port 443:

```text
tcp.port == 443
```

UDP port 53:

```text
udp.port == 53
```

Any traffic using port 53:

```text
tcp.port == 53 || udp.port == 53
```

---

# Combining Filters

Use `&&` for AND:

```text
ip.addr == 192.168.1.10 && tcp
```

Use `||` for OR:

```text
dns || http
```

Exclude traffic using `!`:

```text
!arp
```

Example:

```text
ip.src == 192.168.1.10 && tcp.dstport == 443
```

---

# TCP Analysis

Show TCP packets:

```text
tcp
```

TCP SYN packets:

```text
tcp.flags.syn == 1
```

SYN packets without ACK:

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

TCP RST packets:

```text
tcp.flags.reset == 1
```

TCP FIN packets:

```text
tcp.flags.fin == 1
```

TCP retransmissions:

```text
tcp.analysis.retransmission
```

TCP duplicate ACKs:

```text
tcp.analysis.duplicate_ack
```

TCP out-of-order packets:

```text
tcp.analysis.out_of_order
```

### Important

A filter such as:

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

only identifies SYN packets without ACK. It does **not by itself prove that a SYN flood attack is occurring**. You need to examine packet volume, sources, destinations, timing, and other evidence.

---

# HTTP Analysis

Show HTTP traffic:

```text
http
```

HTTP GET requests:

```text
http.request.method == "GET"
```

HTTP POST requests:

```text
http.request.method == "POST"
```

Find a specific host:

```text
http.host contains "example"
```

Find a specific URI:

```text
http.request.uri contains "login"
```

---

# DNS Analysis

Show DNS packets:

```text
dns
```

DNS queries:

```text
dns.flags.response == 0
```

DNS responses:

```text
dns.flags.response == 1
```

DNS query name:

```text
dns.qry.name
```

Search for a particular domain:

```text
dns.qry.name contains "example"
```

---

# ARP Analysis

Show ARP packets:

```text
arp
```

ARP requests:

```text
arp.opcode == 1
```

ARP replies:

```text
arp.opcode == 2
```

To investigate possible ARP spoofing, examine whether multiple MAC addresses appear associated with the same IP address.

Do not conclude that an attack exists from a single packet alone.

---

# ICMP Analysis

Show ICMP:

```text
icmp
```

ICMP echo requests:

```text
icmp.type == 8
```

ICMP echo replies:

```text
icmp.type == 0
```

This can be useful when investigating `ping` traffic.

---

# Finding Conversations

Wireshark provides:

**Statistics → Conversations**

This can help identify:

* Top communicating hosts
* TCP conversations
* UDP conversations
* Number of packets
* Number of bytes
* Duration

This is useful when investigating unusual network activity.

---

# Finding Endpoints

Go to:

**Statistics → Endpoints**

You can inspect:

* IPv4 endpoints
* IPv6 endpoints
* TCP endpoints
* UDP endpoints

This helps identify which systems are communicating.

---

# Follow a TCP Stream

Right-click a TCP packet and select:

**Follow → TCP Stream**

This reconstructs the conversation belonging to that TCP stream.

It can be useful for understanding application-level communication.

Only analyze traffic that you are authorized to inspect.

---

# Inspecting a Packet

Select a packet and examine the three main panes:

```text
Packet List
     |
     v
Packet Details
     |
     v
Packet Bytes
```

The packet details usually contain layers such as:

```text
Frame
Ethernet
Internet Protocol
Transmission Control Protocol
Application Protocol
```

Expand each layer to inspect its fields.

---

# Useful tshark Commands

`tshark` is the command-line version of Wireshark.

## Read a capture

```bash
tshark -r capture.pcapng
```

## Read only the first 20 packets

```bash
tshark -r capture.pcapng -c 20
```

## Apply a display filter

```bash
tshark -r capture.pcapng -Y "dns"
```

## Filter HTTP traffic

```bash
tshark -r capture.pcapng -Y "http"
```

## Filter an IP

```bash
tshark -r capture.pcapng -Y "ip.addr == 192.168.1.10"
```

## Show packet fields

```bash
tshark -r capture.pcapng -T fields \
-e ip.src \
-e ip.dst \
-e tcp.srcport \
-e tcp.dstport
```

## List interfaces

```bash
tshark -D
```

## Capture from an interface

```bash
sudo tshark -i wlan0
```

## Capture a limited number of packets

```bash
sudo tshark -i wlan0 -c 100
```

---

# Useful Wireshark Features

## Statistics

The **Statistics** menu contains useful analysis tools:

* Protocol Hierarchy
* Conversations
* Endpoints
* I/O Graphs
* Flow Graph
* HTTP
* DNS
* TCP Stream Graphs

## Protocol Hierarchy

Go to:

**Statistics → Protocol Hierarchy**

This shows which protocols appear in the capture.

Example:

```text
Ethernet
 └── IPv4
      ├── TCP
      │    ├── HTTP
      │    └── TLS
      └── UDP
           └── DNS
```

---

# I/O Graph

Go to:

**Statistics → I/O Graph**

This allows you to visualize packet/traffic volume over time.

It can be useful when investigating:

* Traffic spikes
* Bursts of traffic
* Possible flooding
* Network performance problems

---

# Basic Network Attack Analysis

Wireshark can help investigate network attacks, but a packet filter alone does not automatically prove that an attack occurred.

## Possible SYN Flood Investigation

Start with:

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

Then investigate:

* Number of SYN packets
* Source IP addresses
* Destination IP
* Time distribution
* Whether SYN-ACK responses are returned
* Whether connections complete

---

## Possible ARP Spoofing Investigation

Start with:

```text
arp
```

Then examine:

* Sender IP
* Sender MAC
* Target IP
* Target MAC

Look for suspicious changes or conflicting IP/MAC associations.

---

## Possible DNS Anomaly Investigation

Start with:

```text
dns
```

Then inspect:

* Query names
* Query frequency
* Response codes
* Unusual domains
* Unexpected DNS servers

---

# Recommended Analysis Workflow

When analyzing a PCAP file, use this workflow:

```text
1. Open the PCAP
       |
       v
2. Check Protocol Hierarchy
       |
       v
3. Check Conversations
       |
       v
4. Check Endpoints
       |
       v
5. Identify suspicious traffic
       |
       v
6. Apply display filters
       |
       v
7. Follow relevant streams
       |
       v
8. Inspect packet details
       |
       v
9. Correlate multiple packets
       |
       v
10. Document your findings
```

---

# Common Filters Cheat Sheet

| Purpose         | Display Filter                             |
| --------------- | ------------------------------------------ |
| TCP             | `tcp`                                      |
| UDP             | `udp`                                      |
| DNS             | `dns`                                      |
| HTTP            | `http`                                     |
| TLS             | `tls`                                      |
| ARP             | `arp`                                      |
| ICMP            | `icmp`                                     |
| Source IP       | `ip.src == 192.168.1.10`                   |
| Destination IP  | `ip.dst == 192.168.1.10`                   |
| IP address      | `ip.addr == 192.168.1.10`                  |
| TCP port        | `tcp.port == 443`                          |
| UDP port        | `udp.port == 53`                           |
| SYN             | `tcp.flags.syn == 1`                       |
| SYN without ACK | `tcp.flags.syn == 1 && tcp.flags.ack == 0` |
| TCP reset       | `tcp.flags.reset == 1`                     |
| Retransmission  | `tcp.analysis.retransmission`              |
| HTTP GET        | `http.request.method == "GET"`             |
| HTTP POST       | `http.request.method == "POST"`            |
| DNS query       | `dns.flags.response == 0`                  |

---

# Useful Keyboard Shortcuts

| Shortcut           | Action             |
| ------------------ | ------------------ |
| `Ctrl + E`         | Start/stop capture |
| `Ctrl + K`         | Capture options    |
| `Ctrl + O`         | Open capture file  |
| `Ctrl + S`         | Save capture       |
| `Ctrl + F`         | Find packet        |
| `Ctrl + G`         | Go to packet       |
| `Ctrl + Shift + P` | Preferences        |

Shortcuts can vary slightly by operating system/version.

---

# Capture File Formats

Common Wireshark capture files include:

```text
.pcap
.pcapng
```

`pcapng` is the modern default format used by Wireshark.

---

# Safety and Legal Use

Only capture or analyze network traffic that you own or have explicit permission to inspect.

Do not use Wireshark to intercept private traffic on networks without authorization.

For learning, use:

* Your own computer
* Your own lab network
* Intentionally vulnerable training environments
* Public PCAP files provided for analysis

---

# Quick Start

For a beginner, remember these commands/filters first:

```text
tcp
udp
dns
http
icmp
arp
```

Then learn:

```text
ip.addr == x.x.x.x
tcp.port == 443
tcp.flags.syn == 1
tcp.analysis.retransmission
```

And learn these GUI features:

```text
Statistics → Protocol Hierarchy
Statistics → Conversations
Statistics → Endpoints
Statistics → I/O Graph
Follow → TCP Stream
```

---

# Goal

The main goal of Wireshark analysis is not simply to find a packet that looks suspicious.

A good investigation should answer:

1. **Who** communicated?
2. **Who** received the traffic?
3. **What protocol** was used?
4. **When** did the communication occur?
5. **How much traffic** was generated?
6. **What pattern** does the traffic show?
7. **What evidence** supports the conclusion?

Use multiple packets and multiple indicators before concluding that traffic represents an attack.

---

## Official Resources

Wireshark Documentation:

https://www.wireshark.org/docs/

Wireshark User's Guide:

https://www.wireshark.org/docs/wsug_html/

Wireshark Display Filter Reference:

https://www.wireshark.org/docs/dfref/

Wireshark Downloads:

https://www.wireshark.org/download.html

---

## License

This README is intended as a learning reference for Wireshark and network traffic analysis.
