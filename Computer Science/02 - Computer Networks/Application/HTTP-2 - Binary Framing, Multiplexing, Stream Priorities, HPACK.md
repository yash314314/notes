---
title: HTTP-2 - Binary Framing, Multiplexing, Stream Priorities, HPACK
subject: Computer Networks
module: Application Layer Protocols
difficulty: Advanced
prerequisites:
  - "[[Computer Networks MOC]]"
  - "[[Transmission Control Protocol - TCP Header, Features, and Invariants]]"
  - "[[TCP Reliability - Sequence Numbers, ACKs, Retransmission, SACK]]"
  - "[[Hypertext Transfer Protocol - HTTP 1.0, 1.1, Pipelining, Persistent Connections]]"
related:
  - "[[HTTP-3 and QUIC - UDP-Based Transport, Head-of-Line Blocking Elimination]]"
  - "[[gRPC and Protocol Buffers]]"
  - "[[WebSockets and Server-Sent Events - SSE]]"
  - "[[Transport Layer Security - TLS 1.2 vs TLS 1.3 Handshake]]"
aliases:
  - HTTP-2 - Binary Framing, Multiplexing, Stream Priorities, HPACK
  - HTTP 2
  - HTTP/2
  - Binary Framing Layer
  - Stream Multiplexing
  - HPACK
  - Server Push
  - Stream Priorities
tags:
  - networking
  - application-layer
  - http2
  - web
  - protocols
  - backend
status: complete
---

# HTTP/2: Binary Framing Layer, Stream Multiplexing, HPACK, and Transport Bottlenecks

> [!abstract] Mental Model
> - **The Multi-Track High-Speed Railway**: HTTP/1.1 was a single-lane road plagued by ASCII text parsing overhead, domain sharding, and FIFO Head-of-Line blocking.
> - **HTTP/2 (RFC 7540 / RFC 9113)** introduces a foundational **Binary Framing Layer** that breaks requests and responses into discrete binary frames, multiplexing thousands of concurrent bidirectional streams over a **single persistent TCP connection**.

---

## 1. The Core Hierarchy: Connection, Stream, Message, and Frame

```mermaid
graph TD
    Conn["Single TCP Connection (4-Tuple Socket)"]
    
    Stream1["Stream 1 (Client Initiated: GET /index.html)"]
    Stream3["Stream 3 (Client Initiated: GET /style.css)"]
    Stream2["Stream 2 (Server Push: PUSH_PROMISE script.js)"]

    Msg1["Message: HTTP Request / Response"]
    
    F_Head["HEADERS Frame (Index / Metadata)"]
    F_Data1["DATA Frame 1 (Payload Chunk)"]
    F_Data2["DATA Frame 2 (Payload Chunk + END_STREAM)"]

    Conn --> Stream1 & Stream3 & Stream2
    Stream1 --> Msg1
    Msg1 --> F_Head & F_Data1 & F_Data2
```

---

## 2. The 9-Byte Binary Frame Header Format

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                 Length (24 Bits)              |  Type (8b)    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|   Flags (8b)  |R|              Stream Identifier (31 Bits)    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                     Frame Payload (Variable)                  |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

### Field Specifications & Standard Frame Types

| Field / Frame Type | Bit Width | Technical Mechanics & Protocol Invariants |
| :--- | :---: | :--- |
| **Length** | $24\text{ Bits}$ | Payload size in bytes (default max $16,384\text{B} = 16\text{ KB}$, configurable up to $16\text{ MB}$). |
| **Type** | $8\text{ Bits}$ | `DATA (0x0)`, `HEADERS (0x1)`, `PRIORITY (0x2)`, `RST_STREAM (0x3)`, `SETTINGS (0x4)`, `PUSH_PROMISE (0x5)`, `PING (0x6)`, `GOAWAY (0x7)`, `WINDOW_UPDATE (0x8)`, `CONTINUATION (0x9)`. |
| **Flags** | $8\text{ Bits}$ | Modifiers: `END_STREAM (0x1)` (last chunk), `END_HEADERS (0x4)`, `ACK (0x1)`. |
| **Stream ID** | $31\text{ Bits}$ | `0`: Connection-level control.<br/>**Odd IDs**: Client-initiated.<br/>**Even IDs**: Server-initiated (Push). |

---

## 3. Full Multiplexing & Frame Interleaving

HTTP/2 interleaves binary frames from different streams across the wire concurrently, eliminating HTTP/1.1 application HoL blocking:

```mermaid
sequenceDiagram
    autonumber
    participant Client as Web Browser
    participant Srv as Server

    Note over Client,Srv: ALL FRAMES INTERLEAVED OVER A SINGLE TCP CONNECTION!
    
    Client->>Srv: Stream 1: HEADERS Frame [GET /api/slow-report]
    Client->>Srv: Stream 3: HEADERS Frame [GET /logo.png]
    
    Srv->>Client: Stream 3: DATA Frame (logo.png Chunk 1)
    Srv->>Client: Stream 3: DATA Frame (logo.png Chunk 2 + END_STREAM)
    Note over Client: Stream 3 (logo.png) COMPLETE in 2ms! (No waiting for slow report!)
    
    Srv->>Client: Stream 1: DATA Frame (slow-report Chunk 1)
    Srv->>Client: Stream 1: DATA Frame (slow-report Chunk 2 + END_STREAM)
    Note over Client: Stream 1 (slow-report) finishes after 5,000ms.
```

---

## 4. HPACK: High-Performance Header Compression (RFC 7541)

In HTTP/1.1, repetitive headers (`User-Agent`, `Cookie`, `Authorization`) wasted 1KB+ per request. **HPACK** reduces header overhead by $85\% - 95\%$:

```mermaid
flowchart TD
    subgraph HPACK_Engine ["HPACK Compression Architecture"]
        Static["1. Static Table (61 Pre-defined Entries)<br/>• Index 2 = 'GET'<br/>• Index 7 = ':status 200'<br/>• Index 14 = ':scheme https'"]
        
        Dynamic["2. Dynamic Table (In-Memory FIFO Buffer)<br/>• Stores newly observed custom headers (Authorization, Cookies).<br/>• Subsequent requests reference single-byte integer indexes!"]
        
        Huffman["3. Canonical Static Huffman Coding<br/>• Custom Huffman tree optimized for web string distributions."]

        Static --- Dynamic --- Huffman
    end
```

---

## 5. Stream Prioritization & Flow Control

1. **Stream Priority Dependency Trees**:
   - Clients define a dependency tree where streams have a parent dependency and an integer weight ($1 - 256$).
   - Servers allocate CPU and bandwidth proportionally according to the weight tree (e.g. Critical CSS receives $80\%$ bandwidth; background telemetry receives $5\%$).
2. **Dual-Level Credit-Based Flow Control**:
   - Both the **Connection** and each **Individual Stream** have separate credit windows regulated via `WINDOW_UPDATE` frames. Prevents a single fast stream from exhausting server receive memory.

---

## 6. The Remaining Bottleneck: TCP Transport-Layer Head-of-Line Blocking

```mermaid
flowchart TD
    subgraph Bottleneck ["The HTTP/2 Transport-Layer Head-of-Line Bottleneck"]
        AllStreams["Streams 1, 3, 5, 7 multiplexed over Single TCP Socket"]
        --> WireDrop["1 TCP Packet (Carrying Stream 1 DATA Frame) Dropped on Physical Wire!"]
        --> KernelHold["Receiver OS Kernel TCP Stack detects missing byte sequence.<br/>HOLDS ALL SUBSEQUENT INCOMING BYTES IN SK_BUFF BUFFER!"]
        --> TotalStall["CRITICAL FAILURE: Streams 3, 5, and 7 are 100% healthy,<br/>but are COMPLETELY BLOCKED in kernel buffer until TCP retransmits Stream 1!"]
    end
```

> [!important] The Transition to HTTP/3
> While HTTP/2 solved **Application-Layer** HoL blocking, its reliance on a single TCP connection made it vulnerable to **Transport-Layer** HoL blocking over lossy networks (Wi-Fi/Cellular). This single flaw drove the development of **[[HTTP-3 and QUIC - UDP-Based Transport, Head-of-Line Blocking Elimination|HTTP/3 on top of QUIC / UDP]]**.

---

## Production Diagnostics & Protocol Inspection

```bash
# 1. Test HTTP/2 Handshake & Negotiation via ALPN with curl:
curl -I --http2 -v https://example.com/

# Output:
# * ALPN: offers h2, http/1.1
# * ALPN: server accepted h2
# * Using HTTP2, server supports multiplexing
# < HTTP/2 200
# < content-type: text/html; charset=UTF-8
# < server: ECS (phd/FD69)

# 2. Inspect Live HTTP/2 Binary Frames via nghttp CLI:
nghttp -nv https://nghttp2.org/

# Output:
# [  0.012] send SETTINGS frame <length=12, flags=0x00, stream_id=0>
# [  0.015] send HEADERS frame <length=39, flags=0x05, stream_id=1>
#           ; END_STREAM | END_HEADERS
#           (padlen=0) :method: GET
#           :path: /
# [  0.028] recv SETTINGS frame <length=0, flags=0x01, stream_id=0> ; ACK
# [  0.030] recv HEADERS frame <length=142, flags=0x04, stream_id=1>
#           :status: 200
```

---

## Active Recall & Interview Perspective

### Reasoning Prompts
1. *How does the Binary Framing Layer in HTTP/2 eliminate the Application-Layer Head-of-Line (HoL) blocking of HTTP/1.1?*
   - **Answer**: In HTTP/1.1, messages were transmitted as uninterrupted monolithic streams of ASCII text, requiring the entire response for Request #1 to finish before Response #2 could begin on that connection. HTTP/2 inserts a **Binary Framing Layer** that slices every request and response message into small, self-contained binary frames tagged with a **31-bit Stream Identifier**. Because each frame is an independent unit, the sender can interleave frames from hundreds of concurrent streams over a single TCP socket. A large or slow response (Stream 1) emits frames intermittently without blocking frames of a fast static asset (Stream 3), allowing Stream 3 to complete immediately without waiting for Stream 1.
2. *How does HPACK header compression differ from generic gzip/deflate compression, and why was a specialized algorithm created?*
   - **Answer**: Generic compression algorithms like gzip/deflate use LZ77 and Huffman coding, which compress data by referencing repeated byte patterns across the entire stream. In 2012, researchers discovered the **CRIME vulnerability (Compression Ratio Info-leak Made Easy)**: an attacker could inject plaintext into an HTTPS request and observe subtle changes in the compressed message length to deduce secret headers like session cookies byte-by-byte. **HPACK (RFC 7541)** was designed specifically to eliminate CRIME side-channel vulnerabilities while maximizing efficiency: it isolates header names and values, maintains strict **Static and Dynamic indexing tables** across request lifetimes, and applies **Canonical Static Huffman Coding** without cross-field LZ77 string matching.
3. *Why does packet loss on a high-latency network degrade HTTP/2 performance worse than HTTP/1.1 with domain sharding?*
   - **Answer**: In HTTP/1.1 with domain sharding, the browser opens 6 independent TCP connections. If a packet is dropped on Connection #1, only Connection #1 stalls while the remaining 5 TCP connections continue transmitting data unaffected. In HTTP/2, all streams are multiplexed over a **single TCP socket**. Because TCP enforces a strict, ordered byte stream at Layer 4, a single dropped packet forces the receiver's kernel TCP stack to buffer all subsequent incoming packets until the missing packet is retransmitted (Transport-Layer HoL blocking). Consequently, **all multiplexed HTTP/2 streams on that connection are completely frozen simultaneously**, causing severe latency degradation on lossy networks ($1\% - 2\%$ packet loss).

---

## Key Takeaways
- HTTP/2 uses a **9-Byte Binary Frame Header** (`Length`, `Type`, `Flags`, `Stream ID`).
- **Stream Multiplexing** interleaves frames from hundreds of requests over a single TCP socket.
- **HPACK (RFC 7541)** compresses headers using **Static/Dynamic tables** and **Huffman coding**.
- **Transport HoL Blocking**: A single TCP packet drop halts *all* multiplexed streams, necessitating HTTP/3.

---

## Related Notes
- [[Computer Networks MOC]] — Master table of contents for Computer Networks.
- [[Transmission Control Protocol - TCP Header, Features, and Invariants]] — Underlying transport foundation.
- [[TCP Flow Control - Sliding Window and Silly Window Syndrome]] — Window updates vs HTTP/2 flow control.
- [[Hypertext Transfer Protocol - HTTP 1.0, 1.1, Pipelining, Persistent Connections]] — Predecessor protocol comparison.
- [[HTTP-3 and QUIC - UDP-Based Transport, Head-of-Line Blocking Elimination]] — Eliminates TCP transport HoL blocking.
- [[gRPC and Protocol Buffers]] — Built natively on HTTP/2 binary framing and multiplexing.
- [[Transport Layer Security - TLS 1.2 vs TLS 1.3 Handshake]] — ALPN negotiation for HTTP/2.
