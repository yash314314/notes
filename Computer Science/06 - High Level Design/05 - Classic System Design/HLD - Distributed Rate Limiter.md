---
title: "HLD - Distributed Rate Limiter"
subject: "High Level Design"
module: "Classic System Design Core Problems"
difficulty: "Advanced"
prerequisites: "[[LLD - Distributed Rate Limiter (Token Bucket, Leaky Bucket, Sliding Window)]], [[API Gateway Pattern - Rate Limiting, Authentication, TLS Termination, Request Routing]]"
related: "[[Consistent Hashing - Hash Rings, Virtual Nodes, and Data Rebalancing]], [[Distributed Locks - Redis Redlock vs ZooKeeper or Etcd Consensus]], [[High Availability Architecture - SLAs, SLOs, Fault Tolerance, and High-9s Math]]"
aliases: ["Distributed Rate Limiter HLD", "Rate Limiter HLD", "HLD Rate Limiter", "Redis Rate Limiter", "Sliding Window Counter HLD"]
tags: ["hld", "system-design", "rate-limiter", "redis", "lua-script", "sliding-window", "architecture"]
status: "Complete"
---

# HLD — Distributed Rate Limiter

## Mental Model

Think of a **Distributed Rate Limiter** as an enterprise-wide perimeter traffic control system protecting a multi-region cloud cluster. 

While a local in-memory rate limiter protects a single application instance, a **Distributed Rate Limiter** coordinates rate limit quotas across 500 stateless API Gateway nodes handling 1,000,000 requests/sec. 

Every client request passing through any API gateway node is checked against a shared, low-latency distributed counter store (**Redis Cluster**). Using high-speed atomic Lua scripts, the system evaluates quotas in sub-millisecond time ($< 1\text{ms}$), approving allowed requests (**HTTP 200**) and shedding abusive DDoS traffic (**HTTP 429 Too Many Requests**).

---

## 1. Requirement & Capacity Estimation

### A. Functional & Non-Functional Requirements
1. **Accurate Quota Enforcement:** Limit requests per Client IP, User ID, or API Key (e.g. max 100 requests/min).
2. **Sub-Millisecond Overhead:** Rate limiter check latency must be $< 2\text{ms}$ (cannot add noticeable latency to client requests).
3. **High Throughput & Distributed Scale:** Support 500,000 QPS across distributed API gateway nodes.
4. **Fault Tolerance & Graceful Degradation:** If the rate limiter cluster fails, the system should default to "Open" (allow requests) rather than blocking all global traffic.

### B. Capacity Estimation Math
- **Traffic Assumptions:**
  - Total System QPS = 500,000 Requests/sec
  - Active Users = 50 Million Users
- **Memory Estimation for Redis:**
  - User Key Payload Size = 64 Bytes (`rate_limit:user_123456` $\to$ Counter + Window Timestamp)
  - 50 Million Active Users = $50 \times 10^6 \times 64 \text{ Bytes} = \mathbf{3.2 \text{ Gigabytes (GB) RAM}}$
  - Result: Extremely compact memory footprint! Fits comfortably inside a small 3-node Redis cluster!

---

## 2. High-Level Distributed System Architecture

```mermaid
flowchart TD
    Client["Clients (Web / Mobile)"] -->|HTTPS Requests| LB["Layer 4 Load Balancer (AWS NLB)"]
    
    subgraph GatewayTier["Stateless API Gateway Tier (Envoy / Kong)"]
        LB --> GW1["API Gateway Node 1"]
        LB --> GW2["API Gateway Node 2"]
        LB --> GW3["API Gateway Node 3"]
    end
    
    subgraph RateLimiterTier["Distributed Rate Limiter Tier"]
        GW1 & GW2 & GW3 -->|Atomic Lua Eval (<1ms)| RedisCluster["Redis Cluster (Consistent Hashing Shards)\nStores Sliding Window Counters"]
    end

    RedisCluster -->|Quota Exceeded| HTTP429["Return HTTP 429 Too Many Requests"]
    RedisCluster -->|Quota Allowed| Microservices["Route to Internal Microservices"]
```

---

## 3. Distributed Redis Rate Limiting Algorithms

### A. Redis Lua Script Implementation: Sliding Window Counter
To avoid race conditions between checking a key's count and incrementing it (**Check-Then-Act Race Condition**), rate limiting logic MUST execute as an **Atomic Redis Lua Script**:

```lua
-- Redis Atomic Sliding Window Counter Lua Script
-- KEYS[1]: User Rate Limit Key (e.g. "rate_limit:user_99")
-- ARGV[1]: Current Timestamp in Milliseconds
-- ARGV[2]: Window Size in Milliseconds (e.g. 60000 for 1 min)
-- ARGV[3]: Max Allowed Requests (e.g. 100)

local key = KEYS[1]
local now = tonumber(ARGV[1])
local window = tonumber(ARGV[2])
local limit = tonumber(ARGV[3])
local clearBefore = now - window

-- 1. Remove expired timestamps outside sliding window
redis.call('ZREMRANGEBYSCORE', key, 0, clearBefore)

-- 2. Count remaining requests in window
local currentRequests = redis.call('ZCARD', key)

-- 3. Check quota limit
if currentRequests < limit then
    -- Add current timestamp to Sorted Set (ZSET)
    redis.call('ZADD', key, now, now)
    -- Set TTL on key to auto-expire after window duration
    redis.call('EXPIRE', key, math.ceil(window / 1000))
    return 1 -- ALLOWED (HTTP 200)
else
    return 0 -- LIMITED (HTTP 429)
end
```

---

## 4. Local In-Memory Batching vs. Centralized Sync (Optimization)

Calling Redis over a network connection for every single incoming request adds network hop latency ($1\text{ms} - 2\text{ms}$). High-throughput platforms optimize this using **Local Token Batching**:

```mermaid
flowchart TD
    subgraph LocalBatching["Local Gateway Token Batching Optimization"]
        GWNode["API Gateway Node"] -->|1. Requests 100 Tokens at once| CentralRedis["Central Redis Rate Limiter"]
        CentralRedis -->|2. Grants 100 Tokens Batch| GWNode
        
        GWNode -->|3. Consumes tokens LOCALLY in RAM (0ms latency!)| LocalClient["Local Client Requests"]
        NoteBatch["Reduces Redis QPS by 100x! Trade-off: Slightly looser rate enforcement."]
    end
```

---

## 5. Failure Modes and Trade-offs

1. **Redis Cluster Outage (Fail-Open vs. Fail-Closed)** — The entire Redis rate-limiter cluster crashes or suffers a network partition. Should API Gateways block all requests (Fail-Closed) or allow all requests (Fail-Open)? *Mitigation*: Implement **Fail-Open Policy** (log warning and allow traffic to preserve availability) combined with local fallback rate limiting.
2. **Race Conditions in Multi-Threaded Gateways** — Executing non-atomic `GET` then `INCR` commands across separate network calls. Two concurrent gateway threads read `count = 99` and both increment to `100`, bypassing the limit. *Mitigation*: **Must use Atomic Redis Lua Scripts** (`EVALSHA`).
3. **Hotspot Shard Key Explosion** — A malicious DDoS botnet attacks using a single IP, generating 200,000 QPS to 1 Redis shard. *Mitigation*: Use **Consistent Hashing with Virtual Nodes** across Redis cluster nodes.

---

## 6. Active-Recall Prompts

1. **Why must distributed rate limiting logic be executed via atomic Redis Lua scripts (`EVALSHA`)?**
2. **How does the Sliding Window Counter algorithm using Redis Sorted Sets (`ZSET`) eliminate boundary burst vulnerabilities?**
3. **What is Local Token Batching, and how does it reduce central Redis QPS by 100x?**
4. **Compare Fail-Open vs. Fail-Closed behavior when the distributed rate limiter cluster fails.**

---

## Related Notes

- [[LLD - Distributed Rate Limiter (Token Bucket, Leaky Bucket, Sliding Window)]]
- [[API Gateway Pattern - Rate Limiting, Authentication, TLS Termination, Request Routing]]
- [[Consistent Hashing - Hash Rings, Virtual Nodes, and Data Rebalancing]]
- [[Distributed Locks - Redis Redlock vs ZooKeeper or Etcd Consensus]]

> **Interview Style Question:** *"Design a Distributed Rate Limiter for an enterprise API processing 500,000 QPS across 3 global regions. Estimate memory for 50M active users (3.2 GB RAM), write the exact Redis Sorted Set Lua script for atomic sliding window rate limiting, evaluate Local Token Batching optimizations, and design a Fail-Open disaster recovery strategy."*

---
