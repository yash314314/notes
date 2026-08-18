---
title: HTTP-3 and QUIC - UDP-Based Transport, Head-of-Line Blocking Elimination
subject: Computer Networks
module: Application Layer Protocols
difficulty: Advanced
prerequisites:
  - "[[Computer Networks MOC]]"
  - "[[User Datagram Protocol - UDP Architecture and Checksum]]"
  - "[[Transmission Control Protocol - TCP Header, Features, and Invariants]]"
  - "[[TCP Reliability - Sequence Numbers, ACKs, Retransmission, SACK]]"
  - "[[HTTP-2 - Binary Framing, Multiplexing, Stream Priorities, HPACK]]"
related:
  - "[[Transport Layer Security - TLS 1.2 vs TLS 1.3 Handshake]]"
  - "[[Dynamic Routing of Web Traffic - Anycast, CDN Architecture, Edge Computing]]"
aliases:
  - HTTP-3 and QUIC - UDP-Based Transport, Head-of-Line Blocking Elimination
  - HTTP 3
  - HTTP/3
  - QUIC
  - RFC 9000
  - RFC 9114
  - Connection Migration
  - 0-RTT Handshake
  - QPACK
tags:
  - networking
  - application-layer
  - http3
  - quic
  - udp
  - security
  - protocols
status: complete
---

# HTTP/3 and QUIC: UDP-Based Transport, 0-RTT, Connection Migration, and QPACK

> [!abstract] Mental Model
> - **The User-Space Cryptographic Transport Paradigm**: Decades of middlebox ossification trapped TCP in kernel space, rendering protocol upgrades impossible and saddling HTTP/2 with Transport-Layer Head-of-Line blocking.
> - **QUIC (RFC 9000)** and **HTTP/3 (RFC 9114)** abandon TCP entirely, constructing a high-speed, multiplexed, encrypted transport layer directly on top of **raw UDP**. It provides **True Independent Stream Isolation**, **0-RTT Handshakes**, **Seamless Connection Migration** across IP/Wi-Fi transitions, and out-of-order **QPACK compression**.

---

## 1. The Protocol Stack Architectural Inversion

```mermaid
flowchart TD
    subgraph H2_Stack ["HTTP/2 Protocol Stack"]
        H2_App["HTTP/2 Application Layer (Streams & Framing)"]
        H2_TLS["TLS 1.2 / 1.3 (Encryption)"]
        H2_TCP["TCP (Kernel Space: Ordering, Congestion, Loss Recovery)"]
        H2_IP["IP Substrate"]

        H2_App --> H2_TLS --> H2_TCP --> H2_IP
    end

    subgraph H3_Stack ["HTTP/3 + QUIC Protocol Stack"]
        H3_App["HTTP/3 Application Layer (QPACK Encoding)"]
        H3_QUIC["QUIC Transport (User Space: Streams, Loss, Congestion + TLS 1.3 Encrypted)"]
        H3_UDP["UDP (Stateless Datagram Delivery)"]
        H3_IP["IP Substrate"]

        H3_App --> H3_QUIC --> H3_UDP --> H3_IP
    end
```

---

## 2. Elimination of Transport-Layer Head-of-Line (HoL) Blocking

In HTTP/2, stream multiplexing was managed at Layer 7 while Layer 4 TCP treated all traffic as a single byte stream. In **QUIC**, streams are native first-class Layer 4 transport primitives:

```mermaid
sequenceDiagram
    autonumber
    participant Client as Client Host
    participant Srv as Edge Server

    Note over Client,Srv: HTTP/3: Streams 1, 3, and 5 transmitted concurrently via UDP!
    
    Client->>Srv: UDP Datagram 1 [Stream 1: Data Chunk A] -> DROPPED ON WI-FI!
    Client->>Srv: UDP Datagram 2 [Stream 3: Data Chunk A] -> DELIVERED!
    Client->>Srv: UDP Datagram 3 [Stream 5: Data Chunk A] -> DELIVERED!

    Note over Srv: ZERO HEAD-OF-LINE BLOCKING!<br/>Server immediately delivers Stream 3 & 5 to Application Layer!<br/>Only Stream 1 waits for retransmission!
    
    Client->>Srv: Retransmits UDP Datagram 1 [Stream 1: Data Chunk A] -> DELIVERED!
```

---

## 3. 0-RTT Connection Establishment

By integrating the transport handshake and TLS 1.3 cryptographic key negotiation into a single protocol pass, QUIC reduces connection establishment to zero overhead on reconnects:

```mermaid
sequenceDiagram
    autonumber
    participant Client as Client Browser
    participant Srv as QUIC Edge Server

    Note over Client,Srv: 1-RTT Initial Handshake (First Connection)
    Client->>Srv: QUIC Initial [Client Hello + Transport Parameters]
    Srv-->>Client: QUIC Handshake [Server Hello + Encrypted Extensions + Cert]
    Note over Client,Srv: Connection 100% Established & Encrypted in 1 Round-Trip!

    Note over Client,Srv: 0-RTT Resumption Handshake (Subsequent Connections)
    Client->>Srv: QUIC Initial [0-RTT Encrypted Data: GET /api/v1/user]
    Note over Srv: Server processes and responds immediately in the VERY FIRST PACKET!
    Srv-->>Client: QUIC 1-RTT [HTTP/3 200 OK + Response Payload]
```

---

## 4. Connection Migration via 64-Bit Connection IDs (CIDs)

### The TCP Invariant Failure:
TCP binds socket identity to the 4-tuple (`Src IP, Src Port, Dst IP, Dst Port`). When a user steps out of their home, transitioning from Wi-Fi to 5G cellular, their local IP changes, causing all active TCP connections to crash with `ECONNRESET`.

```mermaid
flowchart LR
    subgraph Migration ["QUIC Connection Migration (RFC 9000)"]
        WiFi["Client on Home Wi-Fi<br/>IP: 192.168.1.50:54321<br/>CID: 0x8a92f01c4e7b3a91"]
        --> Cell["User walks outside -> Switches to 5G Cellular!<br/>IP changes to: 172.56.21.9:61204<br/>★ CID REMAINS: 0x8a92f01c4e7b3a91 ★"]
        --> Server["Server receives packet from new 5G IP.<br/>Matches CID 0x8a92f01c4e7b3a91.<br/>Seamlessly continues active video call with ZERO drop!"]
    end
```

---

## 5. QPACK: Header Compression for Out-of-Order Transports (RFC 9204)

HPACK (HTTP/2) required strict sequential order to update its dynamic compression table. Because QUIC streams arrive out-of-order over UDP, HPACK would cause transport stalls.

```mermaid
flowchart TD
    subgraph QPACK_Architecture ["QPACK Asynchronous Stream Compression"]
        ReqStream["Request Stream (Carries QPACK Encoded Headers)"]
        
        Encoder["Encoder Stream (Unidirectional Control Stream)<br/>• Transmits dynamic table updates ahead of time."]
        
        Decoder["Decoder Stream (Unidirectional Control Stream)<br/>• Acknowledges dynamic table insertions to sender."]

        Encoder --> ReqStream --> Decoder
    end
```

---

## 6. Monotonically Increasing Packet Numbers & Total Wire Encryption

1. **Elimination of Retransmission Ambiguity**:
   - In TCP, a retransmitted packet carries the identical Sequence Number.
   - In QUIC, **every UDP packet receives a strictly increasing Packet Number**, even when retransmitting identical payload data. Eliminates Karn's ambiguity completely!
2. **Total Wire Encryption**:
   - Unlike TCP headers (which are plaintext and inspected/modified by middleboxes), QUIC headers, packet numbers, flags, and payload are **100% encrypted and authenticated** via TLS 1.3 AEAD ciphers. Middleboxes cannot tamper with or ossify QUIC.

---

## Production Diagnostics & Protocol Verification

```bash
# 1. Inspect HTTP/3 Negotiation via Alt-Svc Header with curl:
curl -I -v https://cloudflare.com/

# Output:
# < HTTP/2 200
# < alt-svc: h3=":443"; ma=86400, h3-29=":443"; ma=86400

# 2. Execute Direct Native HTTP/3 Request over UDP Port 443:
curl --http3 -I -v https://cloudflare-quic.com/

# Output:
# * Connect to cloudflare-quic.com port 443 via UDP
# * Using HTTP3, server supports multiplexing
# * QUIC connection established (TLS 1.3 / AES-128-GCM)
# < HTTP/3 200
# < content-type: text/html; charset=UTF-8

# 3. Capture QUIC UDP Traffic via tcpdump:
sudo tcpdump -i eth0 -nn "udp port 443"
```

---

## Active Recall & Interview Perspective

### Reasoning Prompts
1. *Why was UDP selected as the underlying transport protocol for QUIC and HTTP/3 instead of designing a new Layer 4 protocol (like SCTP)?*
   - **Answer**: The global internet infrastructure suffers from severe **middlebox ossification**: millions of residential NAT routers, corporate firewalls, and ISP deep-packet inspection (DPI) appliances hardcode support strictly for IP protocols 6 (TCP) and 17 (UDP). Any attempt to deploy a novel IP protocol number (such as SCTP / Protocol 132) results in $>90\%$ packet drop rates across transit networks. By encapsulating QUIC inside standard **UDP datagrams (Port 443)**, QUIC traverses all existing internet middleboxes without modification, while user-space client and server applications retain full autonomy to innovate and evolve transport algorithms.
2. *How does QUIC Connection Migration eliminate the connection drops that plague TCP when mobile devices switch between Wi-Fi and Cellular?*
   - **Answer**: In TCP, a connection's kernel state is strictly anchored to the physical network 4-tuple (`Src IP, Src Port, Dst IP, Dst Port`). When a mobile device switches from Wi-Fi to Cellular, its source IP address changes instantly, rendering the old TCP socket invalid and causing active streams to crash with `ECONNRESET`. QUIC decouples transport identity from the network IP address by embedding an opaque **64-bit Connection ID (CID)** inside every QUIC packet header. When the client's network interface switches, it transmits UDP packets from its new cellular IP carrying the existing CID. The server inspects the CID, verifies path ownership via a lightweight cryptographic `PATH_CHALLENGE`/`PATH_RESPONSE` frame exchange, and transparently migrates the active session without restarting handshakes or dropping application state.
3. *Why was QPACK designed to replace HPACK for HTTP/3, and what catastrophic failure would occur if HPACK were used over QUIC?*
   - **Answer**: HPACK relies on the invariant that all HTTP/2 binary frames are delivered in strict, reliable FIFO order over a single TCP connection. When a sender inserts a new header into its dynamic table, it immediately references that table entry in subsequent frames. Over QUIC, however, UDP datagrams may be reordered or lost independently across different streams. If HPACK were used, a packet on Stream 3 referencing a dynamic table entry created on Stream 1 would be forced to block and stall if Stream 1's packet were dropped or delayed on the network. This would re-introduce **Transport-Layer Head-of-Line Blocking** into HTTP/3. **QPACK (RFC 9204)** solves this by utilizing dedicated, unidirectional **Encoder and Decoder control streams** to synchronize table modifications asynchronously and allow streams to decode headers without blocking on unrelated streams.

---

## Key Takeaways
- **HTTP/3 + QUIC** runs over **UDP Port 443**, moving transport and congestion control to user space.
- **Stream Isolation**: A dropped packet on Stream 1 causes **zero latency stalls** on Streams 3, 5, or 7.
- **0-RTT Handshakes** combine transport and TLS 1.3 security into a single pass.
- **Connection Migration (CID)** maintains sessions across Wi-Fi $\leftrightarrow$ 5G mobile IP changes.
- **QPACK (RFC 9204)** delivers asynchronous out-of-order header compression.

---

## Related Notes
- [[Computer Networks MOC]] — Master table of contents for Computer Networks.
- [[User Datagram Protocol - UDP Architecture and Checksum]] — Foundation for QUIC datagrams.
- [[Transmission Control Protocol - TCP Header, Features, and Invariants]] — TCP comparison.
- [[HTTP-2 - Binary Framing, Multiplexing, Stream Priorities, HPACK]] — Predecessor protocol comparison.
- [[Transport Layer Security - TLS 1.2 vs TLS 1.3 Handshake]] — Integrated cryptographic engine in QUIC.
- [[Dynamic Routing of Web Traffic - Anycast, CDN Architecture, Edge Computing]] — CDN Anycast deployment of HTTP/3.
