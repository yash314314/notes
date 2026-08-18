# File Transfer Protocols - FTP, SFTP, TFTP

---

title: "File Transfer Protocols - FTP, SFTP, TFTP"
subject: "Computer Networks"
module: "Application Layer"
difficulty: "Intermediate"
prerequisites: "[[Transmission Control Protocol - TCP Header, Features, and Invariants]], [[User Datagram Protocol - UDP Architecture and Checksum]]"
related: "[[Secure Shell (SSH)], [[Firewalls - Packet Filtering, Stateful Inspection, NGFW]], [[Virtual File System - VFS]]"
aliases: ["FTP", "SFTP", "TFTP"]
tags: ["protocol", "file-transfer", "network"]
status: "Complete"
---

## Mental Model
Imagine a courier service that moves files between locations. **FTP** is like a traditional postal service with a **control connection** (the order forms) on port 21 and a **data connection** (the actual parcels) on a separate port (20 for active, random for passive). **SFTP** upgrades this courier to a secure, encrypted van that travels over **SSH** (port 22), ensuring the contents cannot be read or tampered with en route. **TFTP** is a lightweight, “drop‑off‑only” courier that runs over **UDP** (port 69) with a simple stop‑and‑wait handshake—ideal for burning firmware onto devices.

## Protocol Architecture
| Aspect | FTP | SFTP | TFTP |
|--------|-----|------|------|
| Transport | TCP (reliable) | TCP (via SSH) | UDP (unreliable) |
| Port (default) | 21 (control), 20 (data) | 22 (SSH) | 69 |
| Mode | Active / Passive | Always encrypted, no active/passive distinction | Simple request/response |
| Authentication | Plain text user/password (often insecure) | Public‑key or password over SSH | No auth (often used in trusted LAN) |
| Encryption | None (unless FTPS) | Full session encryption (AES, ChaCha20) | None |
| Use Cases | Bulk file transfers, legacy systems | Secure file management, automation scripts | Network boot (PXE), firmware updates |

## Control vs Data Channels
- **FTP**: Separate **control channel** for commands (`USER`, `PASS`, `RETR`, `STOR`) and **data channel** for the file payload. In *active* mode the server connects back to the client; in *passive* mode the client initiates both connections.
- **SFTP**: Single encrypted channel (SSH) multiplexes both command and data streams, simplifying firewall traversal.
- **TFTP**: No distinct control channel; each request (RRQ/WRQ) includes the filename and mode, and data blocks are sent sequentially.

## Security Differences
- Plain FTP transmits credentials and data in cleartext – vulnerable to sniffing and man‑in‑the‑middle attacks.
- SFTP inherits SSH’s strong authentication (public‑key, host‑key verification) and confidentiality.
- TFTP lacks authentication; security relies on network isolation (e.g., VLANs) and sometimes uses MAC address filtering.

## Common CLI Diagnostics
```bash
# FTP (Linux netkit)
ftp -p -n localhost         # show passive mode, no auto-login
# SFTP (OpenSSH)
sftp -v user@host          # verbose connection, key exchange details
# TFTP (tftp-hpa)
tftp 192.168.1.10          # interactive mode, use `get`/`put`
```

## Comparison Table
| Feature | FTP | SFTP | TFTP |
|---------|-----|------|------|
| Reliability | ✅ (TCP) | ✅ (TCP) | ❌ (UDP – may lose packets) |
| Encryption | ❌ (unless FTPS) | ✅ (SSH) | ❌ |
| Authentication | ❌ (plain) | ✅ (SSH) | ❌ |
| Firewall friendliness | ❌ (multiple ports) | ✅ (single port) | ✅ (single UDP port) |
| Typical max file size | Limited by OS/file‑system | Limited by OS/file‑system | Small (default 512 B blocks) |

## Active‑Recall Prompts
1. **Why does FTP need two ports, and how does passive mode mitigate firewall issues?**
2. **Explain how SFTP achieves end‑to‑end confidentiality without a separate data channel.**
3. **What are the risks of using TFTP in a production environment, and how can they be mitigated?**
4. **Compare the handshake flow of FTP (active) vs SFTP (SSH).**

## Related Notes
- [[Privilege Rings and CPU Modes]]
- [[Firewalls - Packet Filtering, Stateful Inspection, NGFW]]
- [[Dynamic Host Configuration Protocol - DHCP]]
- [[Domain Name System - DNS Hierarchy, Recursive Resolution, EDNS, DNSSEC]]

> **Interview style question:** *“Design a secure file transfer solution for a legacy system that only supports FTP. What upgrades or wrappers would you propose, and why?”*

---

## Visual Diagram
```mermaid
flowchart TD
    subgraph ControlChannel[FTP Control Channel (Port 21)]
        C1[USER] --> C2[PASS]
        C2 --> C3[STOR / RETR]
    end
    subgraph DataChannel[FTP Data Channel]
        D1[Active Mode: Server -> Client (Port 20)]
        D2[Passive Mode: Client -> Server (Random Port)]
    end
    C3 -->|establishes| D1
    C3 -->|establishes| D2
```

## Production Example
```bash
# FTP (active mode)
ftp -p -n localhost <<EOF
user anonymous
pass anonymous@
binary
cd /var/www
put localfile.txt
quit
EOF

# SFTP (secure)
sftp -C user@host <<EOF
put localfile.txt /remote/path/
quit
EOF

# TFTP (network boot firmware)
# Server side (tftpd-hpa):
sudo systemctl start tftpd-hpa
# Client side:
tftp 192.168.1.10 -c get firmware.bin
```

## Failure Modes / Trade‑offs
- **FTP**: Requires two ports; active mode often blocked by firewalls. Data integrity relies on TCP but credentials are sent in cleartext.
- **SFTP**: Single port (22) simplifies firewall traversal, provides confidentiality and integrity via SSH. Overhead of SSH encryption can impact throughput on low‑power devices.
- **TFTP**: No authentication, uses UDP – susceptible to packet loss and spoofing. Suitable only for trusted LAN environments (e.g., PXE boot).
- **Performance**: TCP (FTP/SFTP) guarantees ordered delivery, higher latency on high‑latency links; UDP (TFTP) lower latency but may need application‑level retries.
- **Security**: Deploy FTPS (FTP over TLS) to secure legacy FTP, or wrap FTP in VPN/tunnel if encryption is required.

---
