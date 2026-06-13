# 📡 Computer Networks — Complete Study Guide for Placement Interviews

> **How to use this guide:** This is written for *learning*, not just revision. Every concept is explained with *why it works* and *how to answer it in an interview*. Read section by section, understand the reasoning, then use the quick-reference tables at the end of each section to drill yourself.

---

## Table of Contents

1. [OSI Model — The 7-Layer Framework](#1-osi-model--the-7-layer-framework)
2. [TCP/IP Model — The Real-World Stack](#2-tcpip-model--the-real-world-stack)
3. [TCP vs UDP — The Big Transport Decision](#3-tcp-vs-udp--the-big-transport-decision)
4. [TCP Deep Dive — Handshake, Headers & Congestion](#4-tcp-deep-dive--handshake-headers--congestion)
5. [IP Addressing & Subnetting](#5-ip-addressing--subnetting)
6. [IPv4 vs IPv6](#6-ipv4-vs-ipv6)
7. [DNS — Domain Name System](#7-dns--domain-name-system)
8. [HTTP & HTTPS](#8-http--https)
9. [Well-Known Ports & Protocols](#9-well-known-ports--protocols)
10. [Routing Protocols](#10-routing-protocols)
11. [ARP, DHCP & NAT](#11-arp-dhcp--nat)
12. [Network Devices](#12-network-devices)
13. [Network Security](#13-network-security)
14. [Network Topologies & Types](#14-network-topologies--types)
15. [Error Detection & Flow Control](#15-error-detection--flow-control)
16. [Mnemonics & Memory Tricks](#16-mnemonics--memory-tricks)
17. [Common Interview Traps](#17-common-interview-traps)
18. [Scenario Questions — "What Happens When..."](#18-scenario-questions--what-happens-when)
19. [Must-Know Essentials — Final Checklist](#19-must-know-essentials--final-checklist)

---

## 1. OSI Model — The 7-Layer Framework

### Why Does the OSI Model Exist?

When you send a message from one computer to another, dozens of things happen — the message gets broken down, addressed, converted to signals, sent over a wire, received, rebuilt, and displayed. Without a standard framework, different companies' hardware and software would never communicate.

The **OSI (Open Systems Interconnection)** model, created by ISO in 1984, divides this complex process into **7 distinct layers**, each with a specific responsibility. Think of it as a conveyor belt — each layer does one job and hands off to the next.

> 🎯 **Interview tip:** Interviewers rarely ask you to memorize OSI blindly. They want to know *what happens at each layer* and *which protocols/devices belong there*.

---

### The 7 Layers — Top to Bottom

| Layer | Number | Name | What It Does | Protocols / Examples | PDU Name |
|---|---|---|---|---|---|
| **Application** | 7 | Application | User-facing services | HTTP, FTP, SMTP, DNS, SNMP | Data |
| **Presentation** | 6 | Presentation | Encoding, encryption, compression | SSL/TLS, JPEG, MPEG, ASCII | Data |
| **Session** | 5 | Session | Opens, manages, closes sessions | NetBIOS, RPC, PPTP | Data |
| **Transport** | 4 | Transport | End-to-end delivery, error recovery | TCP, UDP, SCTP | Segment |
| **Network** | 3 | Network | Logical addressing, routing | IP, ICMP, ARP, OSPF, BGP | Packet |
| **Data Link** | 2 | Data Link | Physical addressing, framing, error detection | Ethernet, PPP, MAC, ARP | Frame |
| **Physical** | 1 | Physical | Raw bit transmission over medium | Cables, Hubs, NIC, Fiber | Bits |

---

### Layer-by-Layer Explanation

#### Layer 7 — Application Layer
This is what you *see*. When you open a browser and visit `google.com`, your browser is an **application-layer process** using **HTTP**. This layer doesn't refer to apps like Chrome itself — it refers to the *network services* those apps use.

- **HTTP/HTTPS** — Web browsing
- **FTP** — File transfer
- **SMTP/IMAP/POP3** — Email
- **DNS** — Name resolution
- **SNMP** — Network device monitoring

#### Layer 6 — Presentation Layer
Think of this layer as a **translator**. It handles:
- **Encoding** (ASCII, Unicode)
- **Encryption/Decryption** (SSL/TLS operates conceptually here)
- **Compression** (JPEG, MP4)

> If Layer 7 is the message, Layer 6 is how the message is formatted and sealed.

#### Layer 5 — Session Layer
This layer **opens and maintains conversations** between computers. Like a phone call — someone dials, the call is active, then it's hung up. It manages:
- Session establishment, maintenance, and termination
- Synchronization (restart from checkpoints if connection drops)

#### Layer 4 — Transport Layer
This is one of the most interview-critical layers. It handles **end-to-end communication** between processes (not machines — processes, identified by *ports*).

- **TCP** — Reliable, ordered, connection-based
- **UDP** — Fast, connectionless, no reliability guarantee
- Handles **segmentation** (breaking data into segments), **flow control**, and **error recovery**

> 💡 **Port numbers live here.** HTTP uses port 80 because the Transport layer multiplexes many conversations using ports.

#### Layer 3 — Network Layer
Handles **logical addressing (IP addresses)** and **routing** — deciding the best path from source to destination across multiple networks.

- **IP** — Internet Protocol (addressing)
- **ICMP** — Internet Control Message Protocol (ping, error reporting)
- **OSPF, BGP** — Routing protocols
- **Routers** operate here

#### Layer 2 — Data Link Layer
Handles **physical addressing (MAC addresses)** and ensures reliable transfer across a *single* network link. It breaks data into **frames** and attaches MAC addresses.

- **Ethernet, Wi-Fi (802.11)** — LAN protocols
- **MAC addresses** — Hardware addresses burned into NICs
- **Switches** operate here
- Error detection via **CRC**

#### Layer 1 — Physical Layer
The raw transmission of **bits** over a physical medium — cables, fiber optic, radio waves. No addressing, no structure — just 0s and 1s.

- **Hubs, Repeaters** operate here
- Deals with voltage levels, bit rates, connectors

---

### Data Flow: Encapsulation vs Decapsulation

**Sending (Encapsulation):** Data travels DOWN the layers. Each layer **wraps** the data with its own header (and sometimes trailer).

```
Application Data
    ↓ + TCP Header  → Segment (Layer 4)
    ↓ + IP Header   → Packet  (Layer 3)
    ↓ + MAC Header  → Frame   (Layer 2)
    ↓               → Bits    (Layer 1) → travels over wire
```

**Receiving (Decapsulation):** Data travels UP the layers. Each layer **unwraps** its header and passes the rest up.

---

### Mnemonic

> **Top to Bottom:** "**A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing"
> Application → Presentation → Session → Transport → Network → Data Link → Physical

> **Bottom to Top:** "**P**lease **D**o **N**ot **T**hrow **S**ausage **P**izza **A**way"
> Physical → Data Link → Network → Transport → Session → Presentation → Application

---

## 2. TCP/IP Model — The Real-World Stack

### OSI vs TCP/IP — What's the Difference?

The OSI model is **theoretical** — a reference model. The **TCP/IP model** (also called the Internet model) is what the internet *actually* uses. It collapses OSI's 7 layers into 4.

| TCP/IP Layer | Equivalent OSI Layers | Key Protocols |
|---|---|---|
| **Application** | Layer 7 + 6 + 5 | HTTP, DNS, SMTP, FTP, SSH |
| **Transport** | Layer 4 | TCP, UDP |
| **Internet** | Layer 3 | IP, ICMP, ARP |
| **Network Access** | Layer 2 + 1 | Ethernet, Wi-Fi, MAC |

> 🎯 **Interview tip:** If asked "which model does the internet use?", say **TCP/IP model**, not OSI. OSI is a reference framework; TCP/IP is the actual implementation.

### PDU (Protocol Data Unit) Names

Each layer calls its unit of data something different:

```
Layer 7-5  →  Data (message)
Layer 4    →  Segment (TCP) or Datagram (UDP)
Layer 3    →  Packet
Layer 2    →  Frame
Layer 1    →  Bits
```

---

## 3. TCP vs UDP — The Big Transport Decision

### The Core Trade-off

Imagine you're sending a letter vs making a phone call. A letter (TCP) ensures delivery — if it's lost, you resend it. A phone call (UDP) is real-time — if words get cut off, you don't replay them; you keep going.

That's the **TCP vs UDP trade-off**: **reliability vs speed**.

---

### Side-by-Side Comparison

| Feature | TCP | UDP |
|---|---|---|
| **Full Name** | Transmission Control Protocol | User Datagram Protocol |
| **Connection** | Connection-oriented (3-way handshake required) | Connectionless (no handshake) |
| **Reliability** | ✅ Guaranteed delivery with ACKs and retransmission | ❌ No guarantee — fire and forget |
| **Ordering** | ✅ Data delivered in order (sequence numbers) | ❌ Packets may arrive out of order |
| **Speed** | Slower (due to handshake + ACK overhead) | Faster (no setup, no waiting for ACK) |
| **Flow Control** | ✅ Yes — Sliding Window mechanism | ❌ No |
| **Congestion Control** | ✅ Yes — Slow Start, AIMD | ❌ No |
| **Header Size** | 20–60 bytes | Fixed 8 bytes |
| **Error Recovery** | ✅ Yes — retransmits lost segments | ❌ No — only basic checksum |
| **Broadcasting** | ❌ Not supported | ✅ Supported |
| **Use Cases** | HTTP, FTP, SMTP, SSH, Banking, File Transfer | DNS, DHCP, Video Streaming, Gaming, VoIP, Live broadcasts |

---

### When to Choose TCP vs UDP?

**Choose TCP when:**
- Data correctness matters more than speed
- You cannot afford missing pieces (file downloads, web pages, emails)

**Choose UDP when:**
- Speed matters more than perfection
- A dropped packet is better than a delayed one (live video, gaming, VoIP)
- The application handles reliability itself (e.g., QUIC protocol built over UDP)

> 💡 **Real-world example:** YouTube uses UDP for streaming (a few dropped frames are fine) but TCP for uploading your video (every byte must arrive correctly).

---

## 4. TCP Deep Dive — Handshake, Headers & Congestion

### The 3-Way Handshake

Before any data is exchanged, TCP **establishes a connection** using three steps. This ensures both sides are ready and agree on starting sequence numbers.

```
Client                          Server
  |  ── SYN (seq=x) ────────→  |   Step 1: Client says "I want to connect, my seq starts at x"
  |  ←── SYN-ACK (seq=y, ack=x+1) ──  |   Step 2: Server says "OK! My seq is y, I received up to x"
  |  ── ACK (ack=y+1) ───────→  |   Step 3: Client says "Got it. Ready."
  |                              |
  |  ←── DATA EXCHANGE ──────→  |   Connection is now ESTABLISHED
```

**Why sequence numbers?** TCP breaks data into segments. Sequence numbers let the receiver reassemble them in the correct order even if they arrive out of sequence.

---

### The 4-Way Termination (Connection Close)

Closing a TCP connection takes **four steps** because each side closes *independently* (half-close).

```
Client                          Server
  |  ── FIN ──────────────────→  |   Client: "I'm done sending."
  |  ←── ACK ──────────────────  |   Server: "Acknowledged."
  |  ←── FIN ──────────────────  |   Server: "I'm done too."
  |  ── ACK ──────────────────→  |   Client: "Acknowledged. Bye."
  |                              |
  [Client waits in TIME_WAIT = 2×MSL before fully closing]
```

> **Why TIME_WAIT?** The final ACK might be lost. TIME_WAIT lets the server retransmit the FIN if needed. MSL = Maximum Segment Lifetime (~30s to 2 min).

---

### Key TCP Header Fields

| Field | Size | Purpose |
|---|---|---|
| **Source Port** | 16 bits | Identifies sending process |
| **Destination Port** | 16 bits | Identifies receiving process |
| **Sequence Number** | 32 bits | Byte offset of first byte in this segment |
| **Acknowledgment Number** | 32 bits | Next byte expected from the other side |
| **Header Length** | 4 bits | Size of TCP header in 32-bit words |
| **Flags** | 6 bits | SYN, ACK, FIN, RST, PSH, URG |
| **Window Size** | 16 bits | Flow control buffer advertised by receiver |
| **Checksum** | 16 bits | Error detection |
| **Urgent Pointer** | 16 bits | Points to urgent data (if URG flag set) |

**Flag meanings:**
- `SYN` — Synchronize sequence numbers (connection setup)
- `ACK` — Acknowledge received data
- `FIN` — Finish, no more data from sender
- `RST` — Reset — abort connection immediately
- `PSH` — Push — deliver data immediately to application
- `URG` — Urgent data present

---

### TCP Congestion Control

The internet is a shared resource. If everyone sends at full speed, routers get overwhelmed and drop packets. TCP uses **congestion control** to regulate its sending rate.

#### Four Phases:

**1. Slow Start**
- Begin with `cwnd = 1 MSS` (Maximum Segment Size, typically 1460 bytes)
- Double cwnd every RTT (exponential growth)
- Continue until cwnd reaches `ssthresh` (slow start threshold)

**2. Congestion Avoidance**
- After hitting ssthresh, increase cwnd by 1 MSS per RTT (linear growth)
- This is the "cruise control" phase

**3. Fast Retransmit**
- If 3 duplicate ACKs are received → assume a segment is lost → retransmit *immediately* (don't wait for timeout)
- Set `ssthresh = cwnd / 2`, `cwnd = ssthresh + 3`

**4. Fast Recovery**
- After fast retransmit, don't go back to Slow Start
- Stay in congestion avoidance mode at the new (lower) ssthresh

> If a **timeout** occurs (no ACK at all), that's worse than 3 dup-ACKs → reset `cwnd = 1`, restart Slow Start.

```
cwnd
 ↑
 |        /
 |       /
 |  ssth|    ___________
 |      |   /           \
 |      |  /             \___
 |      | /                  → time
 |______|
 Slow   Cong.
 Start  Avoid.
```

---

### Flow Control vs Congestion Control

| Aspect | Flow Control | Congestion Control |
|---|---|---|
| **Goal** | Prevent *receiver* from being overwhelmed | Prevent *network* from being overwhelmed |
| **Mechanism** | Receiver advertises Window Size | TCP adjusts cwnd based on loss signals |
| **Who controls it** | Receiver | Sender |
| **Where** | Between two endpoints | In the network |

---

## 5. IP Addressing & Subnetting

### What is an IP Address?

An IP address is a **32-bit logical address** (in IPv4) that uniquely identifies a device on a network. Written as 4 octets in dotted decimal: `192.168.1.100`.

Unlike MAC addresses (permanent hardware addresses), IP addresses are **logical** — they can change and are assigned based on the network you join.

---

### IPv4 Address Classes

IP addresses are grouped into **classes** based on their first octet. This historically determined how many networks and hosts each class could support.

| Class | First Octet Range | Default Subnet Mask | Max Networks | Max Hosts/Network | Typical Use |
|---|---|---|---|---|---|
| **A** | 1 – 126 | 255.0.0.0 (/8) | 126 | 16,777,214 | Large orgs, ISPs |
| **B** | 128 – 191 | 255.255.0.0 (/16) | 16,384 | 65,534 | Medium enterprises |
| **C** | 192 – 223 | 255.255.255.0 (/24) | 2,097,152 | 254 | Small networks |
| **D** | 224 – 239 | N/A | N/A | N/A | Multicast |
| **E** | 240 – 255 | N/A | N/A | N/A | Reserved/Experimental |

> ⚠️ 127.x.x.x is reserved for **loopback** (your own machine). That's why Class A goes 1–126, then skips to 128.

---

### Private IP Ranges (RFC 1918)

These ranges are **not routable on the internet** — they're used inside private networks. NAT translates them to public IPs at the router.

| Range | CIDR | Class |
|---|---|---|
| 10.0.0.0 – 10.255.255.255 | 10.0.0.0/8 | A |
| 172.16.0.0 – 172.31.255.255 | 172.16.0.0/12 | B |
| 192.168.0.0 – 192.168.255.255 | 192.168.0.0/16 | C |

---

### Subnetting — How it Works

**Why subnet?** A company with 200 devices doesn't need a Class A network with 16 million hosts. Subnetting lets you divide a large address block into smaller, manageable chunks.

#### The Key Formula

```
Given: IP address + Prefix (e.g., 192.168.1.0/26)

Host bits     = 32 - prefix length
               = 32 - 26 = 6

Number of usable hosts = 2^(host bits) - 2
                        = 2^6 - 2 = 64 - 2 = 62

(Subtract 2: one for network address, one for broadcast address)
```

#### Why subtract 2?
- **Network address** (all host bits = 0): Identifies the subnet itself. E.g., `192.168.1.0`
- **Broadcast address** (all host bits = 1): Sends to all hosts in subnet. E.g., `192.168.1.63`

---

### CIDR Quick Reference Table

| CIDR | Subnet Mask | Hosts | Usable Hosts |
|---|---|---|---|
| /24 | 255.255.255.0 | 256 | **254** |
| /25 | 255.255.255.128 | 128 | **126** |
| /26 | 255.255.255.192 | 64 | **62** |
| /27 | 255.255.255.224 | 32 | **30** |
| /28 | 255.255.255.240 | 16 | **14** |
| /29 | 255.255.255.248 | 8 | **6** |
| /30 | 255.255.255.252 | 4 | **2** ← point-to-point links |

> 💡 **Interview trick:** For any /N prefix, usable hosts = `2^(32-N) - 2`. Memorize /24 (254) and /30 (2) at minimum.

---

### Subnetting Example

**Problem:** You have network `192.168.10.0/24`. Divide into 4 equal subnets.

**Solution:**
- Need 4 subnets → need 2 subnet bits → /24 + 2 = **/26**
- Each subnet has 6 host bits → `2^6 - 2 = 62` usable hosts

| Subnet | Network Address | Broadcast | Usable Range |
|---|---|---|---|
| 1 | 192.168.10.0 | 192.168.10.63 | .1 – .62 |
| 2 | 192.168.10.64 | 192.168.10.127 | .65 – .126 |
| 3 | 192.168.10.128 | 192.168.10.191 | .129 – .190 |
| 4 | 192.168.10.192 | 192.168.10.255 | .193 – .254 |

---

## 6. IPv4 vs IPv6

### Why IPv6?

IPv4 has ~4.3 billion addresses. With billions of smartphones, IoT devices, and always-on connections, we ran out. **IPv6** provides 3.4 × 10³⁸ addresses — enough for every grain of sand on Earth to have trillions of addresses.

| Feature | IPv4 | IPv6 |
|---|---|---|
| **Address Size** | 32-bit | 128-bit |
| **Notation** | Dotted decimal (192.168.1.1) | Hexadecimal colon (2001:0db8::1) |
| **Total Addresses** | ~4.3 billion | ~3.4 × 10³⁸ |
| **Header Size** | Variable (20–60 bytes) | Fixed 40 bytes (simpler) |
| **NAT Required?** | Yes (to conserve IPs) | No (enough addresses) |
| **Checksum** | Yes (in header) | Removed (delegated to L4) |
| **Broadcast** | Yes | No (uses multicast instead) |
| **Fragmentation** | Done by routers | Done only by source host |
| **Auto-configuration** | DHCP required | SLAAC (Stateless Address Autoconfiguration) |
| **IPSec** | Optional | Built-in (mandatory originally, now optional) |
| **Loopback** | 127.0.0.1 | ::1 |

---

### IPv6 Address Shortening Rules

1. **Omit leading zeros** in each group: `0042` → `42`
2. **Replace one consecutive group of all-zero fields** with `::` (only once)

Example:
```
Full:     2001:0db8:0000:0000:0000:0000:0000:0001
Short:    2001:db8::1
```

---

## 7. DNS — Domain Name System

### Why DNS Exists

Computers communicate using IP addresses. Humans remember names. DNS is the **phone book of the internet** — it translates `www.google.com` to `142.250.77.46`.

### DNS Resolution — Step by Step

When you type `www.google.com` in your browser:

```
1. Browser cache          → Is the answer already cached? If yes, use it.
2. OS/hosts file          → Check /etc/hosts (local override)
3. Recursive Resolver     → Your ISP's DNS server takes over
4. Root Name Server       → "I don't know google.com, but I know .com TLD servers"
5. TLD Name Server (.com) → "I don't know google.com, but here's Google's authoritative NS"
6. Authoritative NS       → "google.com = 142.250.77.46" ✅
7. Cache + Return         → Resolver caches result and returns IP to you
```

> 🎯 **Interview insight:** This process is called **recursive resolution** (from client's perspective) and **iterative resolution** (from resolver's perspective). The recursive resolver does the heavy lifting so your browser doesn't have to.

---

### DNS Record Types

| Record | Full Name | Purpose | Example |
|---|---|---|---|
| **A** | Address | Domain → IPv4 | `google.com → 142.250.77.46` |
| **AAAA** | IPv6 Address | Domain → IPv6 | `google.com → 2607:f8b0::200e` |
| **CNAME** | Canonical Name | Alias → another domain | `www.google.com → google.com` |
| **MX** | Mail Exchange | Where to send email | `google.com → gmail-smtp-in.l.google.com` |
| **NS** | Name Server | Authoritative NS for domain | `google.com → ns1.google.com` |
| **PTR** | Pointer | Reverse DNS: IP → domain | `142.250.77.46 → google.com` |
| **SOA** | Start of Authority | Zone admin info, serial number | — |
| **TXT** | Text | Verification, SPF, DKIM | `"v=spf1 include:_spf.google.com"` |
| **SRV** | Service | Service location | `_http._tcp.example.com` |

---

### DNS Key Facts

- **Port:** 53 (UDP for queries, TCP for zone transfers & large responses)
- **TTL (Time-to-Live):** How long a record is cached. Short TTL = faster propagation, more queries.
- **Zone Transfer:** Replication of DNS records from primary to secondary server. Uses TCP port 53.
- **DNS Cache Poisoning:** Attacker injects fake DNS records into cache → users sent to malicious IP.
- **DNSSEC:** Adds digital signatures to DNS records to prevent poisoning.
- **DoH (DNS over HTTPS):** Encrypts DNS queries using HTTPS. Prevents snooping on DNS traffic.

---

## 8. HTTP & HTTPS

### How HTTP Works

HTTP (HyperText Transfer Protocol) is the **language of the web**. It's a **request-response protocol** — client sends a request, server sends a response.

Key characteristics:
- **Stateless:** Each request is independent. The server remembers nothing from previous requests. Cookies and sessions are used *on top* of HTTP to add state.
- **Text-based:** Human-readable headers
- **Port 80** (HTTP), **Port 443** (HTTPS)

---

### HTTP Methods

| Method | Purpose | Safe? | Idempotent? | Has Body? |
|---|---|---|---|---|
| **GET** | Retrieve a resource | ✅ Yes | ✅ Yes | Optional (not recommended) |
| **POST** | Create a resource | ❌ No | ❌ No | ✅ Yes |
| **PUT** | Replace a resource entirely | ❌ No | ✅ Yes | ✅ Yes |
| **PATCH** | Partially update a resource | ❌ No | ❌ No | ✅ Yes |
| **DELETE** | Remove a resource | ❌ No | ✅ Yes | Optional |
| **HEAD** | Same as GET but no body in response | ✅ Yes | ✅ Yes | ❌ No |
| **OPTIONS** | What methods are allowed? | ✅ Yes | ✅ Yes | Optional |

> **Safe** = Does not modify server state (read-only)
> **Idempotent** = Calling it multiple times has the same effect as calling it once

---

### HTTP Status Codes

| Range | Category | Common Codes |
|---|---|---|
| **1xx** | Informational | 100 Continue, 101 Switching Protocols |
| **2xx** | Success | 200 OK, 201 Created, 204 No Content, 206 Partial Content |
| **3xx** | Redirection | 301 Moved Permanently, 302 Found, 304 Not Modified, 307 Temp Redirect |
| **4xx** | Client Error | 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 429 Too Many Requests |
| **5xx** | Server Error | 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable, 504 Gateway Timeout |

> 💡 **Interview trick:** 401 = "Who are you?" (authentication needed). 403 = "I know who you are, but you can't" (authorization denied).

---

### HTTP Versions

| Version | Key Feature |
|---|---|
| **HTTP/1.0** | One request per connection |
| **HTTP/1.1** | Persistent connections (keep-alive), pipelining, Host header |
| **HTTP/2** | Multiplexing (multiple requests over one connection), header compression, binary protocol |
| **HTTP/3** | Built on **QUIC** (UDP-based), faster handshake, better for mobile |

---

### HTTPS = HTTP + TLS

HTTPS is not a different protocol — it's HTTP running over **TLS (Transport Layer Security)**.

TLS provides:
1. **Confidentiality** — Data is encrypted (eavesdroppers see gibberish)
2. **Integrity** — Data can't be tampered with (MAC ensures this)
3. **Authentication** — Server's identity verified via digital certificate

#### TLS Handshake (Simplified)

```
Client                           Server
  │── ClientHello (TLS version, cipher suites) ──────→│
  │←── ServerHello + Certificate + ServerHelloDone ───│
  │── ClientKeyExchange (pre-master secret) ─────────→│
  │── ChangeCipherSpec + Finished ───────────────────→│
  │←── ChangeCipherSpec + Finished ───────────────────│
  │                                                    │
  │←──────── Encrypted HTTP begins ──────────────────→│
```

> **TLS 1.3** (current standard) uses fewer round trips and supports **Perfect Forward Secrecy** — even if the server's private key is compromised later, past sessions remain secure.

---

## 9. Well-Known Ports & Protocols

### Understanding Ports

A **port** is a 16-bit number (0–65535) that identifies a specific *process or service* on a machine. When a packet arrives, the OS uses the port number to know which application should receive it.

- **Well-known ports:** 0–1023 (assigned by IANA, require root/admin)
- **Registered ports:** 1024–49151 (used by software, registered with IANA)
- **Dynamic/Ephemeral ports:** 49152–65535 (used temporarily by clients)

---

### Must-Memorize Port Table

| Protocol | Port(s) | Transport | Purpose |
|---|---|---|---|
| **FTP** | 20 (data), 21 (control) | TCP | File transfer |
| **SSH** | 22 | TCP | Secure remote login |
| **Telnet** | 23 | TCP | Remote login (unencrypted — avoid!) |
| **SMTP** | 25, 587 (TLS) | TCP | Sending email |
| **DNS** | 53 | UDP (queries), TCP (zone transfer) | Name resolution |
| **DHCP** | 67 (server), 68 (client) | UDP | IP address assignment |
| **TFTP** | 69 | UDP | Simple file transfer (no auth) |
| **HTTP** | 80 | TCP | Web browsing |
| **POP3** | 110, 995 (TLS) | TCP | Download email from server |
| **NTP** | 123 | UDP | Time synchronization |
| **IMAP** | 143, 993 (TLS) | TCP | Read email on server |
| **SNMP** | 161, 162 (traps) | UDP | Network device management |
| **HTTPS** | 443 | TCP | Encrypted web browsing |
| **RDP** | 3389 | TCP | Windows Remote Desktop |
| **MySQL** | 3306 | TCP | MySQL database |
| **PostgreSQL** | 5432 | TCP | PostgreSQL database |
| **MongoDB** | 27017 | TCP | MongoDB NoSQL database |
| **Redis** | 6379 | TCP | In-memory data store |
| **Kafka** | 9092 | TCP | Message streaming |

---

### POP3 vs IMAP — Email Protocols Compared

| Feature | POP3 (Port 110/995) | IMAP (Port 143/993) |
|---|---|---|
| **Storage** | Downloads email, deletes from server | Keeps email on server |
| **Multi-device** | ❌ Poor (email on one device) | ✅ Great (syncs across devices) |
| **Offline access** | ✅ After download | Partial |
| **Use case** | Single-device, limited bandwidth | Modern multi-device use |

---

## 10. Routing Protocols

### What is Routing?

A router receives packets and decides **where to forward them next** to get them closer to the destination. This decision is based on a **routing table**, built and maintained by routing protocols.

### Two Types of Routing

| Type | Description | Example |
|---|---|---|
| **Static Routing** | Admin manually configures routes | Small, stable networks |
| **Dynamic Routing** | Routers automatically discover routes via protocols | Internet, large enterprise |

---

### Dynamic Routing Protocols

| Protocol | Full Name | Type | Algorithm | Metric | Max Hops |
|---|---|---|---|---|---|
| **RIP** | Routing Information Protocol | Distance-Vector (IGP) | Bellman-Ford | Hop count | 15 |
| **OSPF** | Open Shortest Path First | Link-State (IGP) | Dijkstra's | Cost (bandwidth-based) | None |
| **EIGRP** | Enhanced Interior Gateway Routing Protocol | Hybrid (IGP) | DUAL | Bandwidth + Delay | None |
| **BGP** | Border Gateway Protocol | Path-Vector (EGP) | Best-path | AS-path, policies | None |
| **IS-IS** | Intermediate System to Intermediate System | Link-State (IGP) | Dijkstra's | Cost | None |

---

### Key Concepts

**IGP vs EGP:**
- **IGP (Interior Gateway Protocol):** Routes *within* a single Autonomous System (AS). Think within a company or ISP. Examples: OSPF, RIP, EIGRP.
- **EGP (Exterior Gateway Protocol):** Routes *between* different ASes. BGP is the only EGP in use today.

**Autonomous System (AS):** A collection of IP networks under a single administrative policy (e.g., Google, Comcast). Each AS has a unique **AS Number (ASN)**.

---

### Routing Algorithms Explained

**Distance-Vector (RIP):**
- Each router only knows about its *neighbors* and shares its routing table with them
- Slow to converge, simple to configure
- Prone to "count to infinity" problem (max 15 hops prevents loops)

**Link-State (OSPF, IS-IS):**
- Each router has a *complete map* of the network (topology database)
- Runs Dijkstra's algorithm locally to find shortest paths
- Faster convergence, more scalable
- Routers exchange **LSAs (Link State Advertisements)**

**Path-Vector (BGP):**
- Tracks the full path (sequence of ASes) to each destination
- Policy-based: ISPs can prefer certain paths for business reasons
- The protocol that makes the internet work at a global scale

---

### OSPF Areas

OSPF divides large networks into **areas** to reduce overhead:
- **Area 0 (Backbone):** All other areas must connect to this
- **ABR (Area Border Router):** Connects areas to backbone
- **ASBR (Autonomous System Boundary Router):** Connects OSPF to external routing (e.g., BGP)

---

## 11. ARP, DHCP & NAT

### ARP — Address Resolution Protocol

**Problem:** You know a device's IP address, but you need its **MAC address** to actually send a frame on the local network. ARP solves this.

**How it works:**
```
1. Device A wants to reach 192.168.1.20 on the same LAN
2. A broadcasts: "Who has 192.168.1.20? Tell 192.168.1.10!"
   (Destination MAC = FF:FF:FF:FF:FF:FF — broadcast)
3. Device with 192.168.1.20 responds: "That's me! My MAC is AA:BB:CC:DD:EE:FF"
4. A stores this in its ARP cache for future use
```

**ARP Cache:** Temporarily stores IP→MAC mappings (typically 15–20 minutes).

**Variants:**
| Type | Purpose |
|---|---|
| **Gratuitous ARP** | Device announces its own IP+MAC (used in failover/IP conflict detection) |
| **Proxy ARP** | Router answers ARP requests on behalf of remote hosts |
| **RARP** | Reverse ARP: MAC → IP (obsolete, replaced by DHCP) |

**ARP Spoofing Attack:** Attacker sends fake ARP replies associating their MAC with a victim's IP → traffic redirected to attacker (Man-in-the-Middle attack).

---

### DHCP — Dynamic Host Configuration Protocol

**Purpose:** Automatically assign IP addresses (and other network config) to devices when they join a network. Without DHCP, every device would need to be manually configured.

**What DHCP provides:**
- IP address
- Subnet mask
- Default gateway
- DNS server address
- Lease duration

**DORA Process:**

```
Client                          DHCP Server
  │── DHCP Discover ──────────→ │   "Anyone there? I need an IP!" (Broadcast)
  │←── DHCP Offer ─────────────  │   "Here, take 192.168.1.50 (valid for 24h)"
  │── DHCP Request ────────────→ │   "I'd like to use 192.168.1.50 please!"
  │←── DHCP ACK ───────────────  │   "Confirmed. It's yours for 24 hours."
```

> 💡 **Mnemonic:** **DORA** — Discover, Offer, Request, Acknowledge

**DHCP Lease Renewal:** Client tries to renew at 50% of lease time. If server is unavailable, retries at 87.5%. If still no response, acquires a new IP.

---

### NAT — Network Address Translation

**Problem:** You have 50 devices in your home/office, all with private IPs. But you only have one public IP from your ISP. How do they all access the internet?

**Solution:** NAT — the router translates private IP:port pairs to the public IP with different port numbers.

```
Private: 192.168.1.10:5000  →  NAT Table  →  Public: 203.0.113.5:10001
Private: 192.168.1.20:6000  →  NAT Table  →  Public: 203.0.113.5:10002

Response comes back to 203.0.113.5:10001 → NAT maps back → 192.168.1.10:5000
```

**Types of NAT:**

| Type | Description |
|---|---|
| **Static NAT** | One-to-one mapping. One private IP always maps to same public IP. |
| **Dynamic NAT** | Pool of public IPs. Private IPs mapped dynamically (no port reuse). |
| **PAT / Masquerade** | Many private IPs share ONE public IP using different ports. What your home router does. |
| **SNAT** | Source NAT — modifies source IP (outgoing traffic) |
| **DNAT** | Destination NAT — modifies destination IP (incoming traffic, port forwarding) |

> 🎯 **Interview insight:** NAT breaks end-to-end connectivity (servers can't directly initiate connections to private IPs). Solutions: Port forwarding (DNAT), UPnP, or IPv6 (no NAT needed).

---

## 12. Network Devices

### Understanding Which Device Does What

A common interview question: "What's the difference between a hub, switch, and router?" Understanding *which layer* each device operates at is key.

| Device | OSI Layer | Address Type Used | Intelligence | Connects |
|---|---|---|---|---|
| **Hub** | Layer 1 (Physical) | None | None — broadcasts to all ports | Devices in one collision domain |
| **Switch** | Layer 2 (Data Link) | MAC address | Learns MAC table, unicast forwarding | Devices in a LAN |
| **Router** | Layer 3 (Network) | IP address | Routing table, chooses best path | Different networks |
| **Bridge** | Layer 2 (Data Link) | MAC address | Divides collision domains | Two LAN segments |
| **Gateway** | Layer 4–7 | IP + higher | Protocol translation | Incompatible networks |
| **Repeater** | Layer 1 (Physical) | None | Signal amplification | Extended cable segments |
| **Firewall** | Layer 3–7 | IP, ports, content | Filters based on rules | LAN ↔ WAN |
| **Load Balancer** | Layer 4 / Layer 7 | IP, ports, URLs | Distributes traffic | Clients ↔ Server pool |
| **Proxy Server** | Layer 7 | URLs, IPs | Caches, filters, anonymizes | Client ↔ Internet |

---

### Deep Dives on Key Devices

#### Hub vs Switch — The Critical Distinction
- **Hub:** Receives a frame and sends it to **all ports**. Every device sees every frame. Creates one big **collision domain**. Half-duplex.
- **Switch:** Learns which MAC address is on which port (MAC address table). Sends frames **only to the correct port**. Each port is its own collision domain. Full-duplex.

> In modern networks, hubs are obsolete. Switches replaced them entirely.

#### Layer 3 Switch
A switch that can also do routing. Has the MAC intelligence of a switch AND the IP routing of a router. Used within large enterprise LANs for fast inter-VLAN routing.

#### Firewall Types
| Type | How it works |
|---|---|
| **Packet Filter** | Inspects IP/port headers. Stateless. Simple rules. |
| **Stateful Firewall** | Tracks connection state. Knows if a packet belongs to an established connection. |
| **Application Firewall (WAF)** | Inspects application-layer content. Blocks SQL injection, XSS. |

---

## 13. Network Security

### Common Attacks — How They Work

| Attack | Mechanism | Defense |
|---|---|---|
| **DoS** | Flood target with traffic to exhaust resources | Rate limiting, firewalls |
| **DDoS** | Same as DoS but from thousands of compromised machines (botnets) | Traffic scrubbing, CDN, Anycast |
| **Man-in-the-Middle (MITM)** | Intercepts communication between two parties | TLS/HTTPS, certificate pinning |
| **ARP Spoofing** | Sends fake ARP replies to redirect traffic | Dynamic ARP Inspection, static ARP |
| **DNS Spoofing / Cache Poisoning** | Poisons DNS cache with fake records | DNSSEC, short TTLs |
| **SYN Flood** | Sends thousands of SYN packets, never completes handshake → exhausts connection table | SYN cookies, rate limiting |
| **Smurf Attack** | Spoofs victim's IP, sends broadcast ICMP ping → all hosts reply to victim | Disable directed broadcasts |
| **Packet Sniffing** | Captures raw network traffic | Encryption (TLS), switch instead of hub |
| **IP Spoofing** | Forges source IP address | Ingress filtering, RFC 2827 |
| **Replay Attack** | Captures valid packet and re-sends it | Timestamps, nonces, sequence numbers |

---

### Security Protocols

#### SSL vs TLS
- **SSL** (Secure Sockets Layer) — older, deprecated. SSL 2.0 and 3.0 have known vulnerabilities.
- **TLS** (Transport Layer Security) — the modern replacement. Use TLS 1.2 minimum; TLS 1.3 preferred.
- Colloquially, people still say "SSL" when they mean TLS.

#### IPSec
Provides security at the **IP layer** (Layer 3). Used primarily for **VPNs**.

Two modes:
- **Transport Mode:** Encrypts only the payload (data). Header unchanged. Used between two endpoints.
- **Tunnel Mode:** Encrypts the entire original packet, adds new IP header. Used for VPNs (gateway-to-gateway).

Two protocols:
- **AH (Authentication Header):** Provides integrity and authentication. No encryption.
- **ESP (Encapsulating Security Payload):** Provides encryption + integrity + authentication.

#### VPN Types
| Type | Protocol | Use Case |
|---|---|---|
| **Site-to-Site VPN** | IPSec, GRE | Connect two office networks |
| **Remote Access VPN** | SSL/TLS, IPSec | Work from home |
| **PPTP** | GRE + Point-to-Point | Older, weak encryption |
| **L2TP/IPSec** | L2TP + IPSec | Stronger, common on mobile |
| **OpenVPN** | SSL/TLS | Open source, highly flexible |
| **WireGuard** | UDP | Modern, fast, simple |

---

### Key Security Concepts

**PKI (Public Key Infrastructure):**
- System for creating, managing, and distributing digital certificates
- **CA (Certificate Authority)** signs certificates to verify identity
- When your browser connects to a site, it verifies the site's certificate was signed by a trusted CA

**IDS vs IPS:**
| | IDS (Intrusion Detection System) | IPS (Intrusion Prevention System) |
|---|---|---|
| **Action** | Detects and alerts | Detects and **blocks** |
| **Mode** | Passive (monitors traffic copy) | Inline (traffic passes through it) |
| **Response** | Human must act | Automatically blocks |

**Perfect Forward Secrecy (PFS):**
Each session generates unique ephemeral keys. Even if the server's private key is later compromised, past sessions cannot be decrypted.

---

## 14. Network Topologies & Types

### Network Topologies

The **physical or logical arrangement** of devices in a network.

| Topology | Structure | Pros | Cons |
|---|---|---|---|
| **Bus** | All devices on single cable | Simple, cheap | Single point of failure; collisions |
| **Star** | All devices connect to central hub/switch | Easy to manage, isolates failures | Hub/switch failure = whole network down |
| **Ring** | Each device connects to next, forming a ring | Equal access, predictable performance | One break can fail entire ring |
| **Mesh** | Every device connected to every other | Highly fault-tolerant, redundant | Expensive, complex wiring |
| **Tree (Hierarchical)** | Star networks connected to bus backbone | Scalable, structured | Root failure is critical |
| **Hybrid** | Combination of topologies | Flexible, tailored | Complex to manage |

> Most real-world enterprise networks use a **hierarchical (tree) topology** with:
> - **Core layer:** High-speed backbone switches
> - **Distribution layer:** Aggregation and policy enforcement
> - **Access layer:** End devices connect here

---

### Network Types by Geographic Scope

| Type | Full Name | Range | Example |
|---|---|---|---|
| **PAN** | Personal Area Network | < 10 meters | Bluetooth, USB |
| **LAN** | Local Area Network | Building / Campus | Office ethernet, Wi-Fi |
| **MAN** | Metropolitan Area Network | City (~50 km) | City-wide fiber networks |
| **WAN** | Wide Area Network | Country / Global | The Internet |
| **WLAN** | Wireless LAN | Same as LAN | Wi-Fi (802.11) |
| **SAN** | Storage Area Network | Data center | Fiber Channel, iSCSI |
| **VPN** | Virtual Private Network | Any (logical) | Remote work connections |

---

### Wi-Fi Standards (IEEE 802.11)

| Standard | Wi-Fi Name | Band | Max Speed | Key Feature |
|---|---|---|---|---|
| 802.11b | Wi-Fi 1 | 2.4 GHz | 11 Mbps | Early consumer Wi-Fi |
| 802.11a | Wi-Fi 2 | 5 GHz | 54 Mbps | Less interference |
| 802.11g | Wi-Fi 3 | 2.4 GHz | 54 Mbps | Backward compatible with b |
| 802.11n | Wi-Fi 4 | 2.4/5 GHz | 600 Mbps | MIMO antennas |
| 802.11ac | Wi-Fi 5 | 5 GHz | 3.5 Gbps | MU-MIMO, beamforming |
| 802.11ax | Wi-Fi 6 | 2.4/5/6 GHz | 9.6 Gbps | OFDMA, better in dense areas |

> 🎯 **Interview tip:** 2.4 GHz has longer range but more interference (microwaves, Bluetooth share it). 5 GHz has shorter range but faster speeds and less congestion.

---

## 15. Error Detection & Flow Control

### Error Detection Techniques

Errors can corrupt bits during transmission (noise, interference). Detection mechanisms catch these errors.

| Technique | How It Works | Detects | Corrects? | Used In |
|---|---|---|---|---|
| **Parity Bit** | Add extra bit to make count of 1s even or odd | Single-bit errors | ❌ No | Simple hardware |
| **Checksum** | Sum of all data segments, send sum with data | Most errors | ❌ No | TCP, UDP, IP headers |
| **CRC** | Polynomial division; remainder appended to data | Burst errors | ❌ No | Ethernet, Wi-Fi |
| **Hamming Code** | Inserts redundant bits at power-of-2 positions | Single-bit | ✅ Yes | RAM, some comms |
| **Reed-Solomon** | Advanced polynomial coding | Burst errors | ✅ Yes | CDs, DVDs, QR codes |

---

### ARQ — Automatic Repeat Request (Error Control)

When errors are detected, ARQ protocols request retransmission.

| Scheme | How It Works | Efficiency |
|---|---|---|
| **Stop-and-Wait** | Send 1 frame, wait for ACK before next | Low (wastes bandwidth waiting) |
| **Go-Back-N** | Send up to N frames; on error, retransmit from error onward | Medium |
| **Selective Repeat** | Send up to N frames; only retransmit the specific erred frame | High |

---

### Flow Control

Prevents a **fast sender** from overwhelming a **slow receiver**.

**Sliding Window Protocol:**
- Receiver advertises a "window" — how many frames it can buffer
- Sender can send up to that many frames without waiting for ACK
- When ACK received, window "slides" forward

In TCP, this is the **rwnd (receiver window)** field in the header.

---

### Media Access Control (MAC Protocols)

When multiple devices share a medium, they need rules to avoid collisions.

| Protocol | Full Name | Used In | How It Works |
|---|---|---|---|
| **CSMA/CD** | Carrier Sense Multiple Access / Collision Detection | Wired Ethernet | Listen before sending; if collision detected, stop, wait random time, retry |
| **CSMA/CA** | Carrier Sense Multiple Access / Collision Avoidance | Wi-Fi (802.11) | Announce intention to send before sending; avoid collisions proactively |
| **Token Ring** | — | Old ring networks | Pass a "token" — only holder can send |
| **TDMA** | Time Division Multiple Access | Cellular | Divide time into slots, each user gets a slot |

> **Why CSMA/CA instead of CSMA/CD for Wi-Fi?** In wireless, you can't easily detect a collision while you're transmitting (you can't "hear" yourself and others simultaneously). So Wi-Fi avoids collisions instead of detecting them.

---

## 16. Mnemonics & Memory Tricks

### OSI Layers

**Top to Bottom (7→1):**
> **"All People Seem To Need Data Processing"**
> Application, Presentation, Session, Transport, Network, Data Link, Physical

**Bottom to Top (1→7):**
> **"Please Do Not Throw Sausage Pizza Away"**
> Physical, Data Link, Network, Transport, Session, Presentation, Application

---

### DHCP — DORA
> **D**iscover → **O**ffer → **R**equest → **A**cknowledge

---

### IPv4 Classes
> **"A Big Company Does Eat"**
> Class **A** (1-126), **B** (128-191), **C** (192-223), **D** (224-239), **E** (240-255)

---

### PDU Names (Top to Bottom)
> **"Dirty Streets Never Fail Boys"**
> Data → Segment → Packet → Frame → Bits

*(Or just remember: Data, Segment, Packet, Frame, Bits)*

---

### TCP Congestion Control Phases
> **"Slow Cars Are Fast Runners"**
> **S**low Start → **C**ongestion **A**voidance → **F**ast **R**etransmit → Fast **R**ecovery

---

### HTTP Methods Idempotency
> **GET, PUT, DELETE, HEAD** are idempotent. **POST, PATCH** are NOT.
> Memory hook: "GPD-H" — the **safe players**. POST and PATCH **change things**.

---

### Subnet Calculation Quick Check
> `/24 = 254 hosts` (base case — memorize this!)
> Every bit removed from prefix = **double the hosts** approximately
> `/23 ≈ 510 hosts`, `/22 ≈ 1022 hosts`
> Every bit added to prefix = **halve the hosts**
> `/25 = 126`, `/26 = 62`, `/27 = 30`

---

### Key Port Numbers
> **"FTP(21) SSH(22) Tel(23) SMTP(25) DNS(53) HTTP(80) HTTPS(443)"**
> These 7 are the bare minimum to memorize cold.

---

## 17. Common Interview Traps

These are mistakes interviewers commonly use to catch unprepared candidates. Know these cold.

---

### ❌ Trap 1: "ARP works at the Network Layer (Layer 3)"
✅ **Correct:** ARP operates at the **Data Link Layer (Layer 2)**. It resolves IP (Layer 3) to MAC (Layer 2) addresses. It operates within a single broadcast domain.

---

### ❌ Trap 2: "UDP is always faster than TCP"
✅ **Correct:** UDP has *lower overhead* and *no connection setup*, but "faster" depends on context. In lossy networks, TCP's retransmission can cause delays, but in ideal conditions the raw throughput difference is minimal. UDP is preferred when latency matters more than reliability.

---

### ❌ Trap 3: "127.0.0.1 is your router's IP"
✅ **Correct:** `127.0.0.1` is the **loopback address** — it refers to *your own machine*. Your router's internal IP is typically `192.168.1.1` or `192.168.0.1`.

---

### ❌ Trap 4: "Switch and Hub do the same thing"
✅ **Correct:** Completely different. A Hub broadcasts every frame to all ports (creates a single collision domain). A Switch learns MAC addresses and forwards frames only to the correct port (each port is its own collision domain, full-duplex).

---

### ❌ Trap 5: "TCP connection is established in 4 steps"
✅ **Correct:** **Opening** = 3-way handshake (SYN, SYN-ACK, ACK). **Closing** = 4-way handshake (FIN, ACK, FIN, ACK). Don't mix them up.

---

### ❌ Trap 6: "Firewalls and IPS are the same thing"
✅ **Correct:** Firewall filters by rules (allow/deny based on IP, port, protocol). IPS actively inspects traffic patterns for attacks and *blocks* them automatically. IPS is smarter and more active.

---

### ❌ Trap 7: "IP address uniquely identifies a device"
✅ **Correct:** **MAC addresses** uniquely identify a NIC (burned in at manufacturing). IP addresses identify a device *on a network* and can change (DHCP, NAT, reassignment). Behind NAT, multiple devices share one public IP.

---

### ❌ Trap 8: "Routers work at Layer 2"
✅ **Correct:** Routers work at **Layer 3 (Network)**. Switches work at Layer 2. *Layer 3 switches* do both — they have switching hardware but can also route between VLANs.

---

### ❌ Trap 9: "GET requests cannot have a body"
✅ **Correct:** HTTP spec technically allows a body in GET, but it's **undefined behavior** — servers may ignore it. Best practice: never use a body in GET. Use query parameters instead.

---

### ❌ Trap 10: "HTTPS encrypts DNS queries"
✅ **Correct:** By default, **DNS queries are plaintext** even when visiting HTTPS sites. The DNS resolution happens *before* HTTPS is established. **DoH (DNS over HTTPS)** or **DoT (DNS over TLS)** are needed to encrypt DNS traffic.

---

### ❌ Trap 11: "VLAN = Virtual LAN = different physical network"
✅ **Correct:** A VLAN is a *logical* segmentation of a *physical* network. Multiple VLANs can share the same physical switch. Devices in different VLANs need a router (or Layer 3 switch) to communicate, even if they're on the same physical switch.

---

### ❌ Trap 12: "POST is idempotent"
✅ **Correct:** POST is **NOT idempotent**. Calling POST twice creates two resources. PUT is idempotent — calling PUT twice results in the same state. This is critical in API design.

---

## 18. Scenario Questions — "What Happens When..."

### Scenario 1: What happens when you type `https://www.google.com` and press Enter?

This is the most famous interview question in networking. Here's the complete answer:

**Step 1 — URL Parsing**
Browser parses the URL: protocol = `HTTPS`, host = `www.google.com`, path = `/`.

**Step 2 — Browser Cache Check**
Check if the resolved IP for `www.google.com` is already cached. If yes, skip DNS.

**Step 3 — DNS Resolution**
- Check OS hosts file (`/etc/hosts`)
- Query local DNS resolver (ISP's DNS)
- If not cached, resolver queries Root NS → TLD NS (.com) → Google's Authoritative NS
- Returns: `www.google.com → 142.250.77.46`

**Step 4 — TCP 3-Way Handshake**
- Client → Server: `SYN` (seq=x)
- Server → Client: `SYN-ACK` (seq=y, ack=x+1)
- Client → Server: `ACK` (ack=y+1)
- Connection established on port 443

**Step 5 — TLS Handshake**
- ClientHello (supported cipher suites, TLS version)
- ServerHello + Certificate (Google's cert signed by trusted CA)
- Client verifies certificate, generates session keys
- Both sides switch to encrypted communication

**Step 6 — HTTP GET Request**
```
GET / HTTP/1.1
Host: www.google.com
Accept: text/html
```

**Step 7 — Server Processes Request**
Google's load balancers route request to a backend server, which generates the response.

**Step 8 — HTTP Response**
```
HTTP/1.1 200 OK
Content-Type: text/html
...
<html>...</html>
```

**Step 9 — Browser Renders**
Browser parses HTML → discovers CSS, JS, images → fires additional requests for each → renders page.

**Step 10 — Connection Management**
In HTTP/1.1, connection stays alive (keep-alive) for subsequent requests. Eventually closed with TCP FIN.

---

### Scenario 2: How does a packet travel from your laptop to a server in another country?

```
Laptop (192.168.1.100)
  ↓ [Ethernet frame to gateway MAC]
Home Router (192.168.1.1)
  ↓ [NAT: 192.168.1.100:54321 → 203.0.113.5:54321]
ISP Router
  ↓ [BGP routing across multiple ISP networks]
Undersea/Backbone Cables
  ↓ [Multiple hops, each router makes forwarding decision]
Destination Country ISP
  ↓ [Route to destination data center]
Server Load Balancer
  ↓ [Layer 7 routing to correct backend]
Server (e.g., 172.16.0.50 in private datacenter)
  ↓ [Application processes request, sends response back]
```

At every **router hop**, the **IP header** (destination IP) stays the same, but the **MAC addresses** change (Source MAC = router's outgoing port, Destination MAC = next router's incoming port).

---

### Scenario 3: Two devices on the same network can't communicate — how do you troubleshoot?

**Systematic approach:**
```
1. ping 127.0.0.1           → Is TCP/IP stack working on this machine?
2. ping own IP              → Is NIC working?
3. ping default gateway     → Is local network reachable?
4. ping remote IP           → Is routing working?
5. ping 8.8.8.8             → Is internet reachable? (if DNS issue, this works but domain names don't)
6. ping google.com          → Is DNS working?
7. traceroute/tracert       → Where does the path fail?
8. nslookup/dig             → DNS lookup specific debug
9. netstat / ss             → What ports are open/listening?
10. Check firewall rules     → Is a firewall blocking the connection?
```

---

### Scenario 4: Your website is slow. How do you diagnose?

Layer by layer:
- **DNS slow?** → `dig` to check query time, try different DNS resolver
- **TCP handshake slow?** → Check server load, geographic distance, CDN
- **TLS slow?** → Session resumption? TLS 1.3?
- **TTFB (Time to First Byte) high?** → Server-side processing, database queries
- **Large assets?** → Compression (gzip/brotli), image optimization, CDN caching
- **Network path?** → Traceroute to identify slow hops
- **Too many HTTP requests?** → Bundle assets, HTTP/2 multiplexing

---

## 19. Must-Know Essentials — Final Checklist

Before your interview, verify you can answer each of these without hesitation:

---

### Models & Layers

- [ ] Name all 7 OSI layers top-to-bottom and bottom-to-top
- [ ] Name the 4 TCP/IP layers and which OSI layers they map to
- [ ] Know what PDU (Protocol Data Unit) each layer uses
- [ ] Know which protocols and devices belong to each layer

### TCP & UDP

- [ ] Differences between TCP and UDP (at least 5 differences)
- [ ] Three steps of TCP 3-way handshake with sequence numbers
- [ ] Four steps of TCP 4-way termination and why it's 4 steps
- [ ] What TCP flags are (SYN, ACK, FIN, RST, PSH, URG)
- [ ] What TCP Slow Start does and when it triggers
- [ ] Difference between Flow Control and Congestion Control

### IP Addressing

- [ ] IPv4 address classes (A, B, C, D, E) and their ranges
- [ ] Three private IP ranges
- [ ] Calculate usable hosts for any /prefix (formula: 2^(32-n) - 2)
- [ ] Why we subtract 2 (network + broadcast)
- [ ] At least 5 CIDR prefixes and their host counts from memory

### IPv4 vs IPv6

- [ ] Why IPv6 was created
- [ ] 5+ differences between IPv4 and IPv6
- [ ] IPv6 address shortening rules

### DNS

- [ ] What DNS does and why it's needed
- [ ] The full DNS resolution flow (7 steps)
- [ ] 8 DNS record types and what they're used for
- [ ] DNS uses port 53 (UDP for queries, TCP for zone transfers)

### HTTP & HTTPS

- [ ] HTTP is stateless — what that means and how state is added
- [ ] All 6 HTTP methods, which are idempotent, which are safe
- [ ] HTTP status code ranges (1xx-5xx) and 10+ specific codes
- [ ] What HTTPS adds over HTTP (TLS: auth + encryption + integrity)
- [ ] TLS handshake flow (simplified)

### Ports

- [ ] At minimum: FTP(21), SSH(22), SMTP(25), DNS(53), HTTP(80), HTTPS(443)
- [ ] DHCP (67/68), NTP (123), IMAP (143/993), POP3 (110/995)

### Routing

- [ ] Difference between IGP and EGP
- [ ] RIP (distance-vector, Bellman-Ford, max 15 hops)
- [ ] OSPF (link-state, Dijkstra, converges faster)
- [ ] BGP (path-vector, used between ISPs, backbone of internet)

### Network Devices

- [ ] Hub = Layer 1, Switch = Layer 2, Router = Layer 3
- [ ] Hub broadcasts, Switch uses MAC table to unicast
- [ ] What a firewall does vs what an IPS does

### ARP / DHCP / NAT

- [ ] ARP: IP → MAC, operates at Layer 2, uses broadcast
- [ ] DHCP DORA process (Discover, Offer, Request, Acknowledge)
- [ ] NAT: how it allows many private IPs to share one public IP
- [ ] PAT = Port Address Translation (most common NAT)

### Security

- [ ] 6+ types of network attacks and how they work
- [ ] TLS vs SSL (TLS is current, SSL is deprecated)
- [ ] IDS = detects, IPS = detects + blocks
- [ ] IPSec: AH vs ESP, Transport vs Tunnel mode

### Miscellaneous

- [ ] TTL: decremented at each router hop, 0 = drop (prevents loops)
- [ ] MTU: Maximum Transmission Unit, Ethernet default = 1500 bytes
- [ ] VLAN: logical segmentation on physical switch
- [ ] CSMA/CD (wired Ethernet) vs CSMA/CA (Wi-Fi)
- [ ] CDN: edge servers cache content near users for low latency

---

## 📚 Quick Reference: Interview One-Liners

These are crisp, confident answers for rapid-fire interview questions:

| Question | One-liner Answer |
|---|---|
| What is the OSI model? | 7-layer reference framework that standardizes network communication |
| What's the difference between TCP and UDP? | TCP = reliable, ordered, connection-based; UDP = fast, connectionless, no guarantee |
| What is a 3-way handshake? | SYN → SYN-ACK → ACK — establishes TCP connection and syncs sequence numbers |
| What is DNS? | Translates domain names to IP addresses; port 53; resolution: Root → TLD → Auth NS |
| What is subnetting? | Dividing a network into smaller sub-networks; hosts = 2^(host bits) - 2 |
| What is NAT? | Translates private IPs to public IPs; allows many devices to share one public IP |
| What is HTTPS? | HTTP over TLS; adds encryption, authentication, and integrity |
| What does a switch do? | Forwards frames using MAC address table; each port = separate collision domain |
| What is ARP? | Resolves IP address to MAC address within a local network |
| What is VLAN? | Logical network segmentation on a physical switch |
| What is BGP? | Path-vector protocol that routes traffic between autonomous systems (ISPs) |
| What is a firewall? | Filters network traffic based on rules (IP, port, protocol, state) |
| What is TLS? | Cryptographic protocol providing encryption, auth, and integrity over a network |
| What is DHCP? | Automatically assigns IP, subnet mask, gateway, DNS to devices (DORA) |
| What is TTL? | Hop counter decremented at each router; prevents routing loops |

---

*📌 Good luck with your placements! Remember: understanding the **why** behind each concept is what separates good answers from great ones.*
