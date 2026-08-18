---
title: "Distributed Locks - Redis Redlock vs ZooKeeper or Etcd Consensus"
subject: "High Level Design"
module: "Messaging & Event-Driven Systems"
difficulty: "Advanced"
prerequisites: "[[Concurrency Control - Two-Phase Locking, Deadlock Detection, Lock Granularity]], [[Distributed Transactions - 2PC, Saga Pattern, and Outbox Pattern]]"
related: "[[LLD - Movie Ticket Booking System (BookMyShow)]], [[LLD - Hotel Room Booking System]], [[LLD - Thread-Safe In-Memory Cache with LRU and LFU Eviction]]"
aliases: ["Distributed Locks", "Distributed Locking", "Redlock", "Redis Redlock", "ZooKeeper Locks", "Etcd Locks", "Fencing Tokens", "Martin Kleppmann Redlock"]
tags: ["hld", "system-design", "distributed-locks", "redlock", "zookeeper", "etcd", "concurrency", "fencing-tokens"]
status: "Complete"
---

# Distributed Locks — Redis Redlock vs. ZooKeeper or Etcd Consensus

## Mental Model

Think of a **Distributed Lock** as a single physical golden key to an executive conference room shared across 50 international branch offices. 

Only one worker in the entire global enterprise can hold the golden key at any single instant (**Mutual Exclusion**). 

If the worker holding the key falls asleep inside the room (**Process GC Pause / Crash**), the lock must automatically expire after a timeout (**Lock Lease TTL**) so the enterprise doesn't deadlock. 

To guarantee safety, the key provider must ensure that two workers in different cities NEVER receive duplicate copies of the golden key simultaneously (**Fencing Tokens / Consensus Validation**).

---

## 1. Why Single-Machine Locks Fail in Distributed Systems

In a single monolithic process, language primitives like `ReentrantLock` (Java), `pthread_mutex` (C++), or `asyncio.Lock` (Python) enforce mutual exclusion across threads sharing the **same memory space**. 

In a distributed microservices environment spanning 100 Kubernetes pods across multiple data centers, threads do **NOT** share memory! A centralized **Distributed Lock Manager (DLM)** is required.

```mermaid
flowchart TD
    subgraph DistributedLockUseCases["Core Use Cases for Distributed Locks"]
        Efficiency["1. Efficiency / Performance Locks\n- Avoid duplicate expensive work.\n- e.g., Ensuring only 1 worker node calculates a heavy nightly report.\n- Failure Impact: Minor duplicate work."]
        
        Correctness["2. Absolute Correctness Locks\n- Prevent data corruption or financial loss.\n- e.g., Ensuring only 1 user books a specific seat or charges a bank card.\n- Failure Impact: Financial loss / Data corruption!"]
    end
```

---

## 2. Redis Distributed Locking: Single Node vs. Redlock Algorithm

### A. Single-Node Redis Lock (`SETNX` with Lease TTL)
Acquire lock:
```bash
SET resource_lock "random_client_id_12345" NX PX 30000
```
- `NX`: Set key ONLY if it does not exist (**Mutual Exclusion**).
- `PX 30000`: Expire key automatically after 30,000 ms (**Prevents Deadlocks on Client Crash**).

Release lock (Lua Script to verify ownership):
```lua
if redis.call("get", KEYS[1]) == ARGV[1] then
    return redis.call("del", KEYS[1])
else
    return 0
end
```

---

### B. Redis Redlock Algorithm (Multi-Node Fault Tolerance)
Designed by Salvatore Sanfilippo (antirez) for distributed fault tolerance across $N=5$ independent Redis masters:

```mermaid
flowchart TD
    Client["Client App"] -->|1. Gets current timestamp T1| Client
    
    Client -->|2. Attempts SETNX on all 5 independent Redis Masters sequentially| R1 & R2 & R3 & R4 & R5
    
    R1 & R2 & R3 -->|3. Responds OK (3 out of 5 ACKs)| Client
    
    Client -->|4. Checks T2 - T1 < Lock TTL AND Acquired Quorum (>=3 ACKs)| Success["LOCK ACQUIRED SUCCESSFULLY!"]
```

---

## 3. ZooKeeper / Etcd Consensus-Based Distributed Locking

While Redis Redlock relies on wall-clock time expiration, **ZooKeeper** and **Etcd** use strong consensus protocols (ZAB / Raft) and ephemeral sequential nodes.

```mermaid
flowchart TD
    subgraph ZooKeeperLock["ZooKeeper Ephemeral Sequential Node Locking"]
        ClientA["Client A"] -->|Creates Node `/locks/lock-00001`| ZK1["ZK Master Node"]
        ClientB["Client B"] -->|Creates Node `/locks/lock-00002`| ZK1
        
        ZK1 -->|1. Client A is LOWEST sequence number| GrantA["Client A GRANTED LOCK!"]
        
        ClientB -->|2. Client B sets Watcher on `/locks/lock-00001`| Watcher["Watcher Listener"]
        
        ClientA -.->|3. Disconnects / Crashes -> Ephemeral node auto-deleted!| Watcher
        Watcher -->|4. Triggers Event| GrantB["Client B GRANTED LOCK!"]
    end
```

---

## 4. Martin Kleppmann's Redlock Critique & Fencing Tokens

In 2016, distributed systems researcher Martin Kleppmann published a famous critique demonstrating that **Redlock is unsafe for correctness** due to process pauses (Stop-the-World Garbage Collection), network delays, and clock drift!

### The GC Pause Lock Violation Scenario:

```mermaid
flowchart TD
    ClientA["Client A"] -->|1. Acquires Redlock (TTL = 10s)| Redis
    ClientA -->|2. Enters 15-second Stop-The-World GC Pause!| GCPause["STW GC Pause (15s)"]
    
    Redis -->|3. Lock TTL Expires after 10s!| Expired["Lock Released Automatically"]
    
    ClientB["Client B"] -->|4. Acquires Redlock!| Redis
    ClientB -->|5. Writes to Shared Storage| Storage["Shared Storage File"]
    
    GCPause -->|6. Client A Wakes up (Thinks it still holds lock!)| ClientA
    ClientA -->|7. Writes to Shared Storage -> DATA CORRUPTION!| Storage
```

### The Solution: Fencing Tokens
To guarantee safety against GC pauses, the Distributed Lock Manager MUST return a monotonically increasing **Fencing Token** (e.g. `token = 83`) with every lock acquisition:

```mermaid
flowchart TD
    LockManager["Lock Manager"] -->|Grants Lock + Fencing Token = 83| ClientA["Client A (Token 83)"]
    Storage["Storage Engine (Tracks Max Token Seen = 83)"]
    
    ClientA -->|1. Writes with Token 83| Storage
    
    LockManager -->|Grants Lock + Fencing Token = 84| ClientB["Client B (Token 84)"]
    ClientB -->|2. Writes with Token 84| Storage
    
    ClientA -->|3. Delayed write with Token 83 arrives!| Storage
    Storage -->|4. REJECTS Write! Token 83 < Current Max 84| Reject["REJECTED!"]
```

---

## 5. Architectural Comparison Matrix

| Property | Single-Node Redis | Redis Redlock | ZooKeeper / Etcd |
|---|---|---|---|
| **Consensus Protocol** | None (Single Instance). | Majority Quorum ($3/5$). | **ZAB / Raft (Strong Consensus)**. |
| **Dependency Requirement** | Lightweight. | Requires 5 independent Redis nodes. | Requires ZooKeeper / Etcd cluster. |
| **Lock Expiration Mechanism** | TTL Wall-Clock Timeout. | TTL Wall-Clock Timeout. | **Ephemeral Session Heartbeats**. |
| **Fencing Token Generator** | Manual implementation. | Manual implementation. | **Built-in (zxid / revision number)**. |
| **Best Use Case** | **Efficiency & Performance** (Avoiding duplicate work). | High-speed transient locking. | **Absolute Correctness** (Financial & inventory locks). |

---

## 6. Active-Recall Prompts

1. **Why do language-level mutex locks (`ReentrantLock`) fail in multi-pod distributed systems?**
2. **Explain the single-node Redis lock pattern (`SETNX` with TTL and Lua release script).**
3. **How did Martin Kleppmann prove that Redlock can be unsafe during Stop-the-World GC pauses?**
4. **What is a Fencing Token, and how does a storage engine use it to reject stale write operations?**

---

## Related Notes

- [[LLD - Movie Ticket Booking System (BookMyShow)]]
- [[LLD - Hotel Room Booking System]]
- [[LLD - Thread-Safe In-Memory Cache with LRU and LFU Eviction]]
- [[Distributed Transactions - 2PC, Saga Pattern, and Outbox Pattern]]
- [[Concurrency Control - Two-Phase Locking, Deadlock Detection, Lock Granularity]]

> **Interview Style Question:** *"Design a Distributed Lock Manager for a financial settlement system processing 50,000 concurrent transfers. Compare Redis Redlock vs. ZooKeeper vs. Etcd, analyze Martin Kleppmann's GC pause attack vector, implement Fencing Tokens to guarantee strict mutual exclusion on storage writes, and write a complete Java/TypeScript client."*

---
