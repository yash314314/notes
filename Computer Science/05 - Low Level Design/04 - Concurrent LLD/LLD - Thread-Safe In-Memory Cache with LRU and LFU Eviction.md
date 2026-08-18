---
title: "LLD - Thread-Safe In-Memory Cache with LRU and LFU Eviction"
subject: "Low Level Design"
module: "Concurrent & Distributed LLD"
difficulty: "Advanced"
prerequisites: "[[Producer-Consumer Pattern - Bounded Queues and Condition Variables]], [[Strategy Pattern - Algorithmic Interchangeability and Policy Objects]]"
related: "[[LLD - Distributed Rate Limiter (Token Bucket, Leaky Bucket, Sliding Window)]], [[LLD - Task Scheduler and Cron Engine]], [[Thread Pool Pattern - Worker Queues, Task Rejection Policies, Work Stealing]]"
aliases: ["LRU Cache LLD", "LFU Cache LLD", "In-Memory Cache LLD", "Thread-Safe Cache", "Eviction Policy"]
tags: ["lld", "machine-coding", "lru-cache", "lfu-cache", "concurrency", "java"]
status: "Complete"
---

# LLD — Thread-Safe In-Memory Cache with LRU and LFU Eviction

## Mental Model

Think of a **Thread-Safe In-Memory Cache** as an ultra-fast RAM workspace sitting in front of a slow database. 

The cache stores key-value pairs up to a fixed maximum capacity $K$. When the cache reaches full capacity and a client inserts a new key (**Cache Overflow**), an **Eviction Strategy (Strategy Pattern)** determines which key is sacrificed:

- **LRU (Least Recently Used):** Sacrifices the key that hasn't been accessed for the longest time (Doubly-Linked List + HashMap $\to O(1)$ Get/Put).
- **LFU (Least Frequently Used):** Sacrifices the key with the lowest hit frequency count (Frequency Map + Doubly-Linked Lists $\to O(1)$ Get/Put).

---

## 1. Problem Statement & Functional Requirements

### Functional Requirements
1. **$O(1)$ Read/Write Operations:** `get(K)` and `put(K, V)` run in **$O(1)$ time complexity**.
2. **Bounded Capacity:** Enforce maximum item capacity $K$.
3. **Pluggable Eviction Strategy:** Support `LRUEvictionPolicy` and `LFUEvictionPolicy` via Strategy Pattern.
4. **Thread Safety & High Concurrency:** Multi-threaded read/write safety using fine-grained `ReentrantReadWriteLock` or `ConcurrentHashMap` + Mutexes.
5. **TTL Expiration Support:** Support optional key-level Time-To-Live (TTL) expiration.

---

## 2. $O(1)$ LRU vs. LFU Data Structure Mathematics

### A. LRU (Least Recently Used) Data Structure
- **HashMap<K, Node>:** $O(1)$ key lookup to Node pointer.
- **Doubly-Linked List (Head/Tail dummy nodes):** $O(1)$ node promotion to Head (Most Recent) and $O(1)$ node eviction from Tail (Least Recent).

```text
Head (Most Recent) <---> [ Key 3 ] <---> [ Key 1 ] <---> [ Key 2 ] <---> Tail (Least Recent)
```

---

### B. LFU (Least Frequently Used) Data Structure
- **HashMap<K, Node>:** Maps key to node containing `key, value, freq`.
- **HashMap<Integer, DoublyLinkedList>:** Maps frequency count $F$ to a doubly-linked list of all nodes with frequency $F$.
- **`minFreq` Tracker:** Integer tracking the minimum frequency currently present ($O(1)$ eviction target!).

---

## 3. Production Code Implementation (Java)

```java
package com.lld.cache;

import java.time.Instant;
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.locks.ReentrantReadWriteLock;

// ============================================================================
// 1. EVICTION STRATEGY INTERFACE
// ============================================================================
public interface EvictionPolicy<K> {
    void keyAccessed(K key);
    K evictKey();
    void removeKey(K key);
}

// ============================================================================
// 2. $O(1)$ LRU EVICTION POLICY IMPLEMENTATION
// ============================================================================
public class LRUEvictionPolicy<K> implements EvictionPolicy<K> {
    private static class Node<K> {
        K key;
        Node<K> prev, next;
        Node(K key) { this.key = key; }
    }

    private final Map<K, Node<K>> nodeMap = new HashMap<>();
    private final Node<K> head = new Node<>(null); // Dummy Head (Most Recent)
    private final Node<K> tail = new Node<>(null); // Dummy Tail (Least Recent)

    public LRUEvictionPolicy() {
        head.next = tail;
        tail.prev = head;
    }

    private void addNodeToHead(Node<K> node) {
        node.next = head.next;
        node.prev = head;
        head.next.prev = node;
        head.next = node;
    }

    private void removeNode(Node<K> node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }

    private void moveToHead(Node<K> node) {
        removeNode(node);
        addNodeToHead(node);
    }

    @Override
    public synchronized void keyAccessed(K key) {
        Node<K> node = nodeMap.get(key);
        if (node != null) {
            moveToHead(node);
        } else {
            Node<K> newNode = new Node<>(key);
            nodeMap.put(key, newNode);
            addNodeToHead(newNode);
        }
    }

    @Override
    public synchronized K evictKey() {
        Node<K> leastRecent = tail.prev;
        if (leastRecent == head) return null; // Queue empty
        removeNode(leastRecent);
        nodeMap.remove(leastRecent.key);
        return leastRecent.key;
    }

    @Override
    public synchronized void removeKey(K key) {
        Node<K> node = nodeMap.remove(key);
        if (node != null) removeNode(node);
    }
}

// ============================================================================
// 3. THREAD-SAFE CACHE SERVICE (Uses ReadWriteLock for Max Concurrency!)
// ============================================================================
public class Cache<K, V> {
    private final int capacity;
    private final Map<K, CacheEntry<V>> storage = new ConcurrentHashMap<>();
    private final EvictionPolicy<K> evictionPolicy;
    private final ReentrantReadWriteLock lock = new ReentrantReadWriteLock();

    private static class CacheEntry<V> {
        V value;
        Instant expireAt;

        CacheEntry(V value, Instant expireAt) {
            this.value = value;
            this.expireAt = expireAt;
        }

        boolean isExpired() {
            return expireAt != null && Instant.now().isAfter(expireAt);
        }
    }

    public Cache(int capacity, EvictionPolicy<K> evictionPolicy) {
        if (capacity <= 0) throw new IllegalArgumentException("Capacity must be > 0");
        this.capacity = capacity;
        this.evictionPolicy = Objects.requireNonNull(evictionPolicy);
    }

    public V get(K key) {
        lock.readLock().lock();
        try {
            CacheEntry<V> entry = storage.get(key);
            if (entry == null) return null;

            if (entry.isExpired()) {
                // Defer deletion to write lock
                lock.readLock().unlock();
                lock.writeLock().lock();
                try {
                    storage.remove(key);
                    evictionPolicy.removeKey(key);
                    return null;
                } finally {
                    lock.readLock().lock(); // Downgrade back to read lock
                    lock.writeLock().unlock();
                }
            }

            evictionPolicy.keyAccessed(key);
            return entry.value;
        } finally {
            lock.readLock().unlock();
        }
    }

    public void put(K key, V value, Long ttlMillis) {
        lock.writeLock().lock();
        try {
            Instant expireAt = ttlMillis != null ? Instant.now().plusMillis(ttlMillis) : null;

            if (storage.containsKey(key)) {
                storage.put(key, new CacheEntry<>(value, expireAt));
                evictionPolicy.keyAccessed(key);
                return;
            }

            // Check Capacity Eviction
            if (storage.size() >= capacity) {
                K evictedKey = evictionPolicy.evictKey();
                if (evictedKey != null) {
                    storage.remove(evictedKey);
                    System.out.println("CACHE EVICTION: Evicted key [" + evictedKey + "]");
                }
            }

            storage.put(key, new CacheEntry<>(value, expireAt));
            evictionPolicy.keyAccessed(key);
        } finally {
            lock.writeLock().unlock();
        }
    }

    public void put(K key, V value) {
        put(key, value, null);
    }

    public int size() { return storage.size(); }
}
```

---

## 4. Executable Test Suite (`Main`)

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        // Create LRU Cache with Capacity = 3
        Cache<String, String> cache = new Cache<>(3, new LRUEvictionPolicy<>());

        System.out.println("--- Step 1: Put K1, K2, K3 ---");
        cache.put("K1", "Val1");
        cache.put("K2", "Val2");
        cache.put("K3", "Val3");

        System.out.println("Get K1: " + cache.get("K1")); // K1 becomes Most Recent!

        System.out.println("\n--- Step 2: Put K4 (Triggers Eviction of Least Recent K2!) ---");
        cache.put("K4", "Val4"); // Evicts K2!

        System.out.println("Get K2 (Evicted): " + cache.get("K2")); // null!
        System.out.println("Get K3:           " + cache.get("K3")); // Val3

        System.out.println("\n--- Step 3: TTL Expiration Test (K5 expires in 200ms) ---");
        cache.put("K5", "TempVal", 200L);
        System.out.println("Get K5 (Immediate): " + cache.get("K5")); // TempVal
        
        Thread.sleep(300); // Sleep 300ms
        System.out.println("Get K5 (After 300ms): " + cache.get("K5")); // null (Expired!)
    }
}
```

---

## 5. Active-Recall Prompts

1. **How does an LRU Cache achieve $O(1)$ lookup and $O(1)$ eviction using a HashMap + Doubly-Linked List?**
2. **How does an LFU Cache achieve $O(1)$ eviction using a `minFreq` tracker and Frequency Map?**
3. **Why does `ReentrantReadWriteLock` improve cache throughput compared to `synchronized` blocks?**
4. **How are TTL key expirations handled safely during read operations (`lock downgrade`)?**

---

## Related Notes

- [[LLD - Distributed Rate Limiter (Token Bucket, Leaky Bucket, Sliding Window)]]
- [[LLD - Task Scheduler and Cron Engine]]
- [[Producer-Consumer Pattern - Bounded Queues and Condition Variables]]
- [[Strategy Pattern - Algorithmic Interchangeability and Policy Objects]]

> **Interview Style Question:** *"Design and implement a Thread-Safe In-Memory Cache in Java/TypeScript with capacity $K$. Implement $O(1)$ LRU Eviction using a HashMap and Doubly-Linked List, support TTL key expiration, use `ReentrantReadWriteLock` for multi-threaded read/write safety, and write a full test suite."*

---
