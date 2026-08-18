# Computer Science Curriculum Registry

## Overview
This document tracks the canonical curriculum of the Computer Science knowledge base, mapping subjects, modules, core concepts, and their completion status.

---

## 01 - Operating Systems (100% COMPLETE — 93 Canonical Notes)

### Foundations (Complete: 10/10)
- [x] [[Operating System]]
- [x] [[Kernel]]
- [x] [[Privilege Rings and CPU Modes]]
- [x] [[User Mode vs Kernel Mode]]
- [x] [[System Calls]]
- [x] [[Interrupts and Interrupt Handling]]
- [x] [[Traps and Exceptions]]
- [x] [[OS Boot Process]]
- [x] [[Kernel Architecture - Monolithic vs Microkernel vs Hybrid]]
- [x] [[Kernel Modules and Device Drivers]]

### Processes (Complete: 9/9)
- [x] [[Program vs Process]]
- [x] [[Process Address Space]]
- [x] [[Process States and Lifecycle]]
- [x] [[Process Control Block]]
- [x] [[Process Creation and Termination - fork, exec, wait, exit]]
- [x] [[Zombie and Orphan Processes]]
- [x] [[Daemons and Background Services]]
- [x] [[Context Switching]]
- [x] [[Inter-Process Communication - IPC]]

### Threads & Concurrency Models (Complete: 6/6)
- [x] [[Thread]]
- [x] [[Process vs Thread]]
- [x] [[User-Level Threads vs Kernel Threads]]
- [x] [[Multithreading Models - 1-1, N-1, M-N|Multithreading Models - 1:1, N:1, M:N]]
- [x] [[Thread Pools and Worker Queues]]
- [x] [[Thread Safety and Reentrancy]]

### CPU Scheduling (Complete: 10/10)
- [x] [[CPU Scheduler and Dispatcher]]
- [x] [[Preemptive vs Non-Preemptive Scheduling]]
- [x] [[Scheduling Metrics - Turnaround, Response, Waiting Time]]
- [x] [[First-Come First-Served - FCFS]]
- [x] [[Shortest Job First - SJF and SRTF]]
- [x] [[Round Robin Scheduling]]
- [x] [[Priority Scheduling and Aging]]
- [x] [[Multilevel Queue and MLFQ]]
- [x] [[Linux CFS - Completely Fair Scheduler]]
- [x] [[Real-Time Scheduling - Rate Monotonic and EDF]]

### Synchronization & Concurrency (Complete: 13/13)
- [x] [[Race Conditions and Data Races]]
- [x] [[Critical Section Problem]]
- [x] [[Hardware Synchronization Primitives - CAS, TAS, LL-SC]]
- [x] [[Memory Ordering and Memory Barriers]]
- [x] [[Spinlocks]]
- [x] [[Producer-Consumer Pattern - Bounded Queues and Condition Variables]]
- [x] [[Binary and Counting Semaphores]]
- [x] [[Producer-Consumer Pattern - Bounded Queues and Condition Variables]]
- [x] [[Monitors]]
- [x] [[Reader-Writer Problem and RWLocks]]
- [x] [[Producer-Consumer Problem]]
- [x] [[Dining Philosophers Problem]]
- [x] [[Lock-Free and Wait-Free Data Structures]]

### Deadlocks (Complete: 6/6)
- [x] [[Deadlock Fundamentals and Coffman Conditions]]
- [x] [[Resource Allocation Graph]]
- [x] [[Deadlock Prevention Strategies]]
- [x] [[Deadlock Avoidance and Banker's Algorithm]]
- [x] [[Deadlock Detection and Recovery]]
- [x] [[Deadlock vs Livelock vs Starvation]]

### Memory Management & Virtual Memory (Complete: 15/15)
- [x] [[Logical vs Physical Address Space]]
- [x] [[Memory Allocation - Contiguous, Fixed, Variable Partitioning]]
- [x] [[Internal vs External Fragmentation]]
- [x] [[Paging Architecture]]
- [x] [[Segmentation]]
- [x] [[Page Tables and Multi-Level Page Tables]]
- [x] [[Inverted Page Tables]]
- [x] [[Translation Lookaside Buffer - TLB]]
- [x] [[Virtual Memory Architecture]]
- [x] [[Demand Paging and Page Faults]]
- [x] [[Copy-on-Write - CoW]]
- [x] [[Page Replacement Algorithms - FIFO, LRU, Clock, Optimal]]
- [x] [[Belady's Anomaly]]
- [x] [[Working Set Model and Thrashing]]
- [x] [[Swapping and Swap Space Management]]

### File Systems (Complete: 11/11)
- [x] [[File Concept and File Attributes]]
- [x] [[File Descriptors and File Tables]]
- [x] [[Directory Structures and Path Resolution]]
- [x] [[File Allocation Methods - Contiguous, Linked, Indexed]]
- [x] [[Inodes and File System Metadata]]
- [x] [[Hard Links vs Symbolic Links]]
- [x] [[Virtual File System - VFS]]
- [x] [[Page Cache and Buffer Cache]]
- [x] [[Journaling File Systems and Crash Consistency]]
- [x] [[ext4 Architecture Overview]]
- [x] [[XFS and ZFS Overview]]

### Storage & I/O Subsystems (Complete: 6/6)
- [x] [[IO Hardware - Port-Mapped vs Memory-Mapped IO]]
- [x] [[Direct Memory Access - DMA]]
- [x] [[Polling vs Interrupt-Driven IO]]
- [x] [[Disk Scheduling Algorithms - SSTF, SCAN, C-SCAN]]
- [x] [[RAID Levels and Reliability]]
- [x] [[Solid State Drives - Flash Memory, Wear Leveling, TRIM]]

### Advanced Systems Concepts (Complete: 7/7)
- [x] [[Virtualization and Hypervisors - Type 1 vs Type 2]]
- [x] [[OS-Level Virtualization - Linux Namespaces and cgroups]]
- [x] [[Containers vs Virtual Machines]]
- [x] [[Memory-Mapped IO and mmap]]
- [x] [[Zero-Copy IO - sendfile, splice, io_uring]]
- [x] [[eBPF Architecture and Observability]]
- [x] [[NUMA Architecture]]

---

## 02 - Computer Networks (Target: ~50 Canonical Notes)

### 1. Network Models & Physical Layer (Complete: 4/4)
- [x] [[Computer Networks MOC|Computer Networks MOC]]
- [x] [[OSI vs TCP-IP Model]]
- [x] [[Encapsulation, De-encapsulation, and Protocol Data Units - PDUs]]
- [x] [[Physical Layer - Transmission Media, Bandwidth, and Latency]]

### 2. Data Link Layer & Local Networks (Complete: 6/6)
- [x] [[Data Link Layer Framing and Error Detection - CRC, Checksum, Parity]]
- [x] [[Flow Control - Stop-and-Wait, Go-Back-N, Selective Repeat]]
- [x] [[Media Access Control - CSMA-CD, CSMA-CA, ALOHA]]
- [x] [[Ethernet Protocol and IEEE 802.3 Frame Format]]
- [x] [[MAC Addressing and Address Resolution Protocol - ARP]]
- [x] [[Switches and VLANs - 802.1Q, Trunking, Spanning Tree Protocol STP]]

### 3. Network Layer & Addressing (Complete: 8/8)
- [x] [[Network Layer - Packet Forwarding vs Routing]]
- [x] [[IPv4 Addressing - Classful, Classless, CIDR, and Subnetting]]
- [x] [[IPv4 Header Format and Packet Fragmentation]]
- [x] [[IPv6 Architecture - 128-bit Addressing, Header Format, Transition]]
- [x] [[Network Address Translation - NAT, PAT, CGNAT]]
- [x] [[Dynamic Host Configuration Protocol - DHCP]]
- [x] [[Internet Control Message Protocol - ICMP and Traceroute]]
- [x] [[Virtual Private Networks - IPsec, WireGuard, GRE]]

### 4. Routing Protocols & Algorithms (Complete: 6/6)
- [x] [[Routing Algorithms - Distance Vector vs Link State vs Path Vector]]
- [x] [[Dijkstra's Shortest Path and Bellman-Ford Algorithms]]
- [x] [[Open Shortest Path First - OSPF and Link State Advertisements]]
- [x] [[Border Gateway Protocol - BGP, AS, BGP Peering, Path Attributes]]
- [x] [[Multiprotocol Label Switching - MPLS]]
- [x] [[Software-Defined Networking - SDN and OpenFlow]]

### 5. Transport Layer Protocols (Complete: 7/7)
- [x] [[Transport Layer Fundamentals - Multiplexing, Demultiplexing, Ports]]
- [x] [[User Datagram Protocol - UDP Architecture and Checksum]]
- [x] [[Transmission Control Protocol - TCP Header, Features, and Invariants]]
- [x] [[TCP Three-Way Handshake and Connection Termination]]
- [x] [[TCP Reliability - Sequence Numbers, ACKs, Retransmission, SACK]]
- [x] [[TCP Flow Control - Sliding Window and Silly Window Syndrome]]
- [x] [[TCP Congestion Control - AIMD, Slow Start, Reno, CUBIC, BBR]]

### 6. Application Layer Protocols (Complete: 9/9)
- [x] [[Domain Name System - DNS Hierarchy, Recursive Resolution, EDNS, DNSSEC]]
- [x] [[Hypertext Transfer Protocol - HTTP 1.0, 1.1, Pipelining, Persistent Connections]]
- [x] [[HTTP-2 - Binary Framing, Multiplexing, Stream Priorities, HPACK]]
- [x] [[HTTP-3 and QUIC - UDP-Based Transport, Head-of-Line Blocking Elimination]]
- [x] [[WebSockets and Server-Sent Events - SSE]]
- [x] [[gRPC and Protocol Buffers]]
- [x] [[Email Protocols - SMTP, IMAP, POP3]]
- [x] [[File Transfer Protocols - FTP, SFTP, TFTP]] (2026-08-18)
- [x] [[Dynamic Routing of Web Traffic - Anycast, CDN Architecture, Edge Computing]] (2026-08-18)

### 7. Network Security & Cryptography (Complete: 6/6)
- [x] [[Transport Layer Security - TLS 1.2 vs TLS 1.3 Handshake]] (2026-08-18)
- [x] [[PKI, X.509 Certificates, and Certificate Authorities]] (2026-08-18)
- [x] [[Firewalls - Packet Filtering, Stateful Inspection, NGFW]] (2026-08-18)
- [x] [[Distributed Denial of Service - DDoS Attack Vectors and Mitigations]] (2026-08-18)
- [x] [[Man-in-the-Middle - MitM, ARP Spoofing, DNS Cache Poisoning]] (2026-08-18)
- [x] [[Zero Trust Network Architecture - Microsegmentation, mTLS]] (2026-08-18)

---

## 03 - Database Management Systems (100% COMPLETE — 34 Canonical Notes)

### 1. Relational Model & SQL Foundations (Complete: 3/3)
- [x] [[Relational Model - Tables, Keys, Functional Dependencies, Normalization]] (2026-08-18)
- [x] [[SQL Foundations - DDL, DML, Joins, Subqueries, Window Functions]] (2026-08-18)
- [x] [[SQL Advanced - CTEs, Recursive Queries, JSON Operations, Full-Text Search]] (2026-08-18)

### 2. Query Processing & Optimization (Complete: 3/3)
- [x] [[Query Processing Pipeline - Parser, Rewriter, Planner, Executor]] (2026-08-18)
- [x] [[Cost-Based Optimizer - Statistics, Cardinality Estimation, Join Ordering]] (2026-08-18)
- [x] [[Query Execution - Join Algorithms, Aggregation, Sorting, Parallel Query]] (2026-08-18)

### 3. Indexing & Storage Structures (Complete: 4/4)
- [x] [[B-Tree and B+ Tree Indexes - Structure, Search, Insert, Split]] (2026-08-18)
- [x] [[Hash Indexes and Adaptive Hash Index in MySQL InnoDB]] (2026-08-18)
- [x] [[LSM Tree - Log-Structured Merge Tree, Compaction, RocksDB]] (2026-08-18)
- [x] [[Index Design Patterns - Composite, Covering, Partial, Expression Indexes]] (2026-08-18)

### 4. Transactions & Concurrency Control (Complete: 3/3)
- [x] [[Transactions and ACID Properties - Atomicity, Consistency, Isolation, Durability]] (2026-08-18)
- [x] [[Concurrency Control - Two-Phase Locking, Deadlock Detection, Lock Granularity]] (2026-08-18)
- [x] [[MVCC - Multi-Version Concurrency Control, Snapshot Isolation, PostgreSQL vs MySQL]] (2026-08-18)

### 5. Crash Recovery & Durability (Complete: 2/2)
- [x] [[Write-Ahead Logging - WAL, Redo Log, Undo Log, fsync Guarantees]] (2026-08-18)
- [x] [[ARIES Recovery Protocol - Analysis, Redo, Undo Phases, Checkpointing]] (2026-08-18)

### 6. Distributed Databases (Complete: 4/4)
- [x] [[Distributed Database Fundamentals - CAP Theorem, PACELC, Consistency Models]] (2026-08-18)
- [x] [[Database Replication - Single-Leader, Multi-Leader, Leaderless, Raft Consensus]] (2026-08-18)
- [x] [[Database Sharding - Horizontal Partitioning, Consistent Hashing, Routing]] (2026-08-18)
- [x] [[Google Spanner and CockroachDB - TrueTime, Serializable Distributed Transactions]] (2026-08-18)

### 7. NoSQL & NewSQL Systems (Complete: 4/4)
- [x] [[Document Databases - MongoDB Architecture, WiredTiger, Aggregation Pipeline]] (2026-08-18)
- [x] [[Key-Value Stores - Redis Architecture, Data Structures, Persistence, Cluster]] (2026-08-18)
- [x] [[Wide-Column Stores - Apache Cassandra, SSTable, Consistent Hashing, Tunable Consistency]] (2026-08-18)
- [x] [[Graph Databases - Neo4j, Property Graph Model, Cypher, Traversal Algorithms]] (2026-08-18)

### 8. Database Internals (Complete: 4/4)
- [x] [[Buffer Pool Manager - Page Cache, Replacement Policies, Dirty Pages]] (2026-08-18)
- [x] [[Disk Storage and Page Format - Heap Files, Page Layout, Tuple Format]] (2026-08-18)
- [x] [[PostgreSQL Internals - MVCC Implementation, Tuple Header, VACUUM, Toast]] (2026-08-18)
- [x] [[MySQL InnoDB Architecture - Clustered Index, Buffer Pool, Doublewrite Buffer]] (2026-08-18)

### 9. Advanced Topics (Complete: 3/3)
- [x] [[Time-Series Databases - InfluxDB, TimescaleDB, Prometheus Storage]] (2026-08-18)
- [x] [[Search Engines - Elasticsearch, Inverted Index, TF-IDF, BM25, Relevance Scoring]] (2026-08-18)
- [x] [[Vector Databases - Embeddings, HNSW, ANN Search, pgvector]] (2026-08-18)

### 10. Interview & Practical Design (Complete: 2/2)
- [x] [[Database Design Interview Patterns - Normalization vs Denormalization Trade-offs]] (2026-08-18)
- [x] [[Database Performance Tuning - EXPLAIN ANALYZE, Slow Query Log, Connection Pooling]] (2026-08-18)

---

## 04 - Object-Oriented Programming & Design Patterns (100% COMPLETE — 37 Canonical Notes)

### 1. Object-Oriented Fundamentals (Complete: 4/4)
- [x] [[Encapsulation, Data Hiding, and Information Hiding]] (2026-08-18)
- [x] [[Inheritance, Subtyping, and Composition vs Inheritance]] (2026-08-18)
- [x] [[Polymorphism - Dynamic Binding, Virtual Method Tables, Method Overloading vs Overriding]] (2026-08-18)
- [x] [[Abstraction and Interface-Driven Design]] (2026-08-18)

### 2. SOLID Principles & Clean Architecture (Complete: 5/5)
- [x] [[Single Responsibility Principle - SRP and Cohesion]] (2026-08-18)
- [x] [[Open-Closed Principle - OCP and Extensibility]] (2026-08-18)
- [x] [[Liskov Substitution Principle - LSP and Subtyping Invariants]] (2026-08-18)
- [x] [[Interface Segregation Principle - ISP and Decoupling]] (2026-08-18)
- [x] [[Dependency Inversion Principle - DIP and Dependency Injection Containers]] (2026-08-18)

### 3. Creational Design Patterns (Complete: 5/5)
- [x] [[Singleton Pattern - Thread-Safe Initialization, Double-Checked Locking, Enum Singleton]] (2026-08-18)
- [x] [[Factory Method Pattern - Virtual Constructors and Extensible Object Creation]] (2026-08-18)
- [x] [[Abstract Factory Pattern - Product Families and Platform Decoupling]] (2026-08-18)
- [x] [[Builder Pattern - Fluent Interfaces, Step Builders, and Immutable Object Construction]] (2026-08-18)
- [x] [[Prototype Pattern - Deep vs Shallow Copying and Prototype Managers]] (2026-08-18)

### 4. Structural Design Patterns (Complete: 7/7)
- [x] [[Adapter Pattern - Class vs Object Adapters and Interface Translation]] (2026-08-18)
- [x] [[Bridge Pattern - Decoupling Abstraction from Implementation]] (2026-08-18)
- [x] [[Composite Pattern - Tree Structures, Uniformity vs Type Safety]] (2026-08-18)
- [x] [[Decorator Pattern - Dynamic Behavior Extension and Wrapper Chaining]] (2026-08-18)
- [x] [[Facade Pattern - Simplifying Subsystem Interfaces and Boundary Layering]] (2026-08-18)
- [x] [[Flyweight Pattern - Intrinsic vs Extrinsic State and Memory Optimization]] (2026-08-18)
- [x] [[Proxy Pattern - Virtual, Remote, Protection, and Smart Proxies]] (2026-08-18)

### 5. Behavioral Design Patterns (Complete: 11/11)
- [x] [[Chain of Responsibility Pattern - Handler Chains and Middleware Pipelines]] (2026-08-18)
- [x] [[Command Pattern - Encapsulating Requests, Undo-Redo, and Macro Commands]] (2026-08-18)
- [x] [[Interpreter Pattern - Domain-Specific Languages and Abstract Syntax Trees]] (2026-08-18)
- [x] [[Iterator Pattern - Custom Collections, External vs Internal Iteration]] (2026-08-18)
- [x] [[Mediator Pattern - Decoupling Peer Communication and Centralized Coordination]] (2026-08-18)
- [x] [[Memento Pattern - Capturing and Restoring State without Violating Encapsulation]] (2026-08-18)
- [x] [[Observer Pattern - Event Buses, Publish-Subscribe, and Reactive Streams]] (2026-08-18)
- [x] [[State Pattern - Finite State Machines and State-Driven Behavior]] (2026-08-18)
- [x] [[Strategy Pattern - Algorithmic Interchangeability and Policy Objects]] (2026-08-18)
- [x] [[Template Method Pattern - Inversion of Control and Hook Methods]] (2026-08-18)
- [x] [[Visitor Pattern - Double Dispatch and Operations over Heterogeneous Trees]] (2026-08-18)

### 6. Concurrency & Async Design Patterns (Complete: 4/4)
- [x] [[Producer-Consumer Pattern - Bounded Queues and Condition Variables]] (2026-08-18)
- [x] [[Active Object Pattern - Decoupling Method Execution from Invocation]] (2026-08-18)
- [x] [[Thread Pool Pattern - Worker Queues, Task Rejection Policies, Work Stealing]] (2026-08-18)
- [x] [[Reactor and Proactor Patterns - Event-Driven Asynchronous IO Multiplexing]] (2026-08-18)

---

## 05 - Low Level Design (100% COMPLETE — 17 Canonical Notes)

### 1. LLD Foundations & OOAD (Complete: 3/3)
- [x] [[Object-Oriented Analysis and Design - Requirement Gathering, Use Cases, and Class Diagrams]] (2026-08-18)
- [x] [[Machine Coding Interview Framework - 45-Minute Execution Blueprint]] (2026-08-18)
- [x] [[Design Patterns in LLD - Practical Application Guide]] (2026-08-18)

### 2. Classic Game & Object LLD (Complete: 4/4)
- [x] [[LLD - Parking Lot System]] (2026-08-18)
- [x] [[LLD - Elevator Control System]] (2026-08-18)
- [x] [[LLD - Tic-Tac-Toe and N-by-N Board Game Engine]] (2026-08-18)
- [x] [[LLD - Snake and Ladders Game Engine]] (2026-08-18)

### 3. Enterprise System LLD (Complete: 5/5)
- [x] [[LLD - Automated Teller Machine (ATM) System]] (2026-08-18)
- [x] [[LLD - Vending Machine System]] (2026-08-18)
- [x] [[LLD - Movie Ticket Booking System (BookMyShow)]] (2026-08-18)
- [x] [[LLD - Library Management System]] (2026-08-18)
- [x] [[LLD - Hotel Room Booking System]] (2026-08-18)

### 4. Concurrent & Distributed LLD (Complete: 4/4)
- [x] [[LLD - Thread-Safe In-Memory Cache with LRU and LFU Eviction]] (2026-08-18)
- [x] [[LLD - Distributed Rate Limiter (Token Bucket, Leaky Bucket, Sliding Window)]] (2026-08-18)
- [x] [[LLD - Task Scheduler and Cron Engine]] (2026-08-18)
- [x] [[LLD - Thread-Safe Key-Value Store with Transactions]] (2026-08-18)

---

## 06 - High Level Design (100% COMPLETE — 28 Canonical Notes)

### 1. HLD Foundations & Capacity Estimation (Complete: 3/3)
- [x] [[System Design Interview Framework - 4-Step Blueprint]] (2026-08-18)
- [x] [[Back-of-the-Envelope Calculations - Throughput, Storage, and Bandwidth Estimation]] (2026-08-18)
- [x] [[High Availability Architecture - SLAs, SLOs, Fault Tolerance, and High-9s Math]] (2026-08-18)

### 2. Scalability & Gateway Architecture (Complete: 4/4)
- [x] [[Load Balancers - Layer 4 vs Layer 7, Algorithms, and Health Checks]] (2026-08-18)
- [x] [[API Gateway Pattern - Rate Limiting, Authentication, TLS Termination, Request Routing]] (2026-08-18)
- [x] [[Content Delivery Networks - CDN Architecture, Edge Caching, and Anycast Routing]] (2026-08-18)
- [x] [[Domain Name System in HLD - GeoDNS, Latency-Based Routing, and Failover]] (2026-08-18)

### 3. Distributed Data Architecture (Complete: 5/5)
- [x] [[Database Scaling - Vertical vs Horizontal, Read Replicas, and Master-Slave]] (2026-08-18)
- [x] [[Database Sharding - Hash-Based, Range-Based, and Directory-Based Partitioning]] (2026-08-18)
- [x] [[Consistent Hashing - Hash Rings, Virtual Nodes, and Data Rebalancing]] (2026-08-18)
- [x] [[Distributed Caching Strategies - Cache-Aside, Write-Through, Write-Around, Write-Back]] (2026-08-18)
- [x] [[Database Indexing & Storage Engine Selection for HLD]] (2026-08-18)

### 4. Messaging & Event-Driven Systems (Complete: 4/4)
- [x] [[Message Queues vs Event Streams - RabbitMQ vs Apache Kafka]] (2026-08-18)
- [x] [[Event-Driven Architecture - Pub-Sub, Event Sourcing, and CQRS]] (2026-08-18)
- [x] [[Distributed Transactions - 2PC, Saga Pattern, and Outbox Pattern]] (2026-08-18)
- [x] [[Distributed Locks - Redis Redlock vs ZooKeeper or Etcd Consensus]] (2026-08-18)

### 5. Classic System Design Core Problems (Complete: 5/5)
- [x] [[HLD - URL Shortener (TinyURL)]] (2026-08-18)
- [x] [[HLD - Distributed Unique ID Generator (Twitter Snowflake)]] (2026-08-18)
- [x] [[HLD - Distributed Web Crawler]] (2026-08-18)
- [x] [[HLD - Distributed Rate Limiter]] (2026-08-18)
- [x] [[HLD - Distributed Notification Service (Email, SMS, Push)]] (2026-08-18)

### 6. Enterprise Scale Case Studies (Complete: 6/6)
- [x] [[HLD - Video Streaming Platform (YouTube or Netflix)]] (2026-08-18)
- [x] [[HLD - Ride-Sharing Platform (Uber or Lyft)]] (2026-08-18)
- [x] [[HLD - Social Network News Feed (Twitter or X)]] (2026-08-18)
- [x] [[HLD - Real-Time Chat Application (WhatsApp or Slack)]] (2026-08-18)
- [x] [[HLD - E-Commerce Flash Sale System (Amazon Flash Sale)]] (2026-08-18)
- [x] [[HLD - Collaborative Document Editor (Google Docs or Figma)]] (2026-08-18)

---

## 07 - Distributed Systems (100% COMPLETE — 15 Canonical Notes)

### 1. Distributed Models, Time & Clocks (Complete: 3/3)
- [x] [[Lamport Timestamps and Happened-Before Relation]] (2026-08-18)
- [x] [[Vector Clocks - Causal Ordering and Conflict Detection]] (2026-08-18)
- [x] [[Google TrueTime and Synchronized Atomic Clocks in Spanner]] (2026-08-18)

### 2. Distributed Consensus & Leader Election (Complete: 4/4)
- [x] [[Raft Consensus Algorithm - Leader Election, Log Replication, and Safety]] (2026-08-18)
- [x] [[Paxos Consensus Protocol - Basic Paxos, Multi-Paxos, and Proposers]] (2026-08-18)
- [x] [[ZooKeeper Atomic Broadcast (Zab) Protocol]] (2026-08-18)
- [x] [[Distributed Leader Election - Bully Algorithm and Ring Algorithm]] (2026-08-18)

### 3. Consistency Models & Formal Isolation (Complete: 4/4)
- [x] [[CAP Theorem vs. PACELC Theorem - Trade-offs and Proofs]] (2026-08-18)
- [x] [[Linearizability vs. Serializability - Formal Definitions and Differences]] (2026-08-18)
- [x] [[Eventual Consistency and Strong Eventual Consistency (CRDTs)]] (2026-08-18)
- [x] [[Causal Consistency and Read-Your-Writes Guarantees]] (2026-08-18)

### 4. Failure Detection & Fault Tolerance (Complete: 3/3)
- [x] [[Gossip Protocol - Epidemic Dissemination and Peer-to-Peer Membership]] (2026-08-18)
- [x] [[SWIM Protocol - Scalable Weakly-Consistent Infection-Style Process Group Membership]] (2026-08-18)
- [x] [[Byzantine Fault Tolerance - BFT, PBFT, and Nakamoto Consensus]] (2026-08-18)

---

## 90 - Interview & Revision (100% COMPLETE — 7 Canonical Notes)

### 1. Computer Science Cross-Domain Synthesis (Complete: 2/2)
- [x] [[Computer Science Master Cheat Sheet - Cross-Domain Engineering Synthesis]] (2026-08-18)
- [x] [[System Design & Architecture Pattern Cheat Sheet]] (2026-08-18)

### 2. Staff/Principal Engineer Interview Frameworks (Complete: 2/2)
- [x] [[Staff and Principal Engineer Interview Playbook - Leadership, Trade-offs, and Communication]] (2026-08-18)
- [x] [[System Design Incident Response and Disaster Recovery Framework]] (2026-08-18)

### 3. Production Incident Walkthroughs & Trade-offs (Complete: 2/2)
- [x] [[Production Outage Case Studies - Post-Mortem Analysis & Lessons Learned]] (2026-08-18)
- [x] [[Computer Science Knowledge Base Master Index and Definition of Done Verification]] (2026-08-18)

---


---


---


---


---


---

