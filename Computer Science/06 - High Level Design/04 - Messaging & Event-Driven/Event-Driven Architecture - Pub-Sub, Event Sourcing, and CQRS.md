---
title: "Event-Driven Architecture - Pub-Sub, Event Sourcing, and CQRS"
subject: "High Level Design"
module: "Messaging & Event-Driven Systems"
difficulty: "Advanced"
prerequisites: "[[Message Queues vs Event Streams - RabbitMQ vs Apache Kafka]], [[Observer Pattern - Event Buses, Publish-Subscribe, and Reactive Streams]]"
related: "[[Distributed Transactions - 2PC, Saga Pattern, and Outbox Pattern]], [[Command Pattern - Encapsulating Requests, Undo-Redo, and Macro Commands]], [[Memento Pattern - Capturing and Restoring State without Violating Encapsulation]]"
aliases: ["Event-Driven Architecture", "EDA", "Pub-Sub", "Event Sourcing", "CQRS", "Command Query Responsibility Segregation"]
tags: ["hld", "system-design", "event-driven", "pub-sub", "event-sourcing", "cqrs", "architecture"]
status: "Complete"
---

# Event-Driven Architecture — Pub-Sub, Event Sourcing, and CQRS

## Mental Model

Think of **Event-Driven Architecture (EDA)** as an automated stock exchange trading floor. 

When a trader submits a buy order (**Event Published**), the stock exchange does not call 50 individual bank, tax, compliance, and notification databases synchronously (**Coupled Request-Response**). 

Instead, the exchange broadcasts the event to an **Event Bus (Pub-Sub)**. Interested subscribers (**Compliance Service, Billing Service, Push Notification Service, Risk Engine**) consume the event asynchronously at their own speed. 

Furthermore, instead of storing only the final account balance ($10,000), **Event Sourcing** records every single historical transaction event (`+ $5,000`, `- $2,000`, `+ $7,000`) in an immutable log. Replaying this log reconstructs exact account state at any point in history. **CQRS** splits the heavy Write path (**Commands**) from the fast Read path (**Queries**).

---

## 1. Request-Driven vs. Event-Driven Architecture

```mermaid
flowchart TD
    subgraph RequestDriven["1. Request-Driven Architecture (Coupled REST)"]
        OrderS1["Order Service"] -->|Sync REST Call| Inventory1["Inventory Service"]
        OrderS1 -->|Sync REST Call| Payment1["Payment Service"]
        OrderS1 -->|Sync REST Call| Email1["Email Service"]
        NoteRD["If Email Service is down or slow, the entire Order Checkout FAILS!"]
    end

    subgraph EventDriven["2. Event-Driven Architecture (Decoupled Pub-Sub)"]
        OrderS2["Order Service"] -->|Publish 'OrderPlaced' Event| EventBus["Event Bus (Kafka / NATS)"]
        EventBus -.->|Async Consume| Inventory2["Inventory Service"]
        EventBus -.->|Async Consume| Payment2["Payment Service"]
        EventBus -.->|Async Consume| Email2["Email Service"]
        NoteED["Zero coupling! Adding a new Fraud Service requires ZERO changes to Order Service."]
    end
```

---

## 2. Event Sourcing Pattern

Instead of storing current state in a relational database table using `UPDATE`, **Event Sourcing** records all state changes as an immutable, append-only sequence of domain events.

```mermaid
flowchart TD
    subgraph TraditionalState["Traditional State Storage (CRUD)"]
        Table["Bank Account Table\n`account_id: 101 | balance: $8,000`\n(Old values OVERWRITTEN and lost forever!)"]
    end

    subgraph EventSourcedState["Event Sourced State (Immutable Event Log)"]
        Log["Event Store Log:\n1. AccountOpened (id=101, initial=$0)\n2. MoneyDeposited (amount=$10,000)\n3. MoneyWithdrawn (amount=$3,000)\n4. InterestCredited (amount=$1,000)"] -->|Replay Events| State["Current Reconstructed State = $8,000"]
    end
```

### Advantages of Event Sourcing:
1. **Complete Audit Trail & Time-Travel Debugging:** Full historical audit log built-in by design. You can reconstruct system state at any exact past timestamp ($T$).
2. **Zero Update Locks:** High-throughput append-only writes ($O(1)$ sequential log inserts) with zero database row locking.
3. **Event Replay:** Easily spin up new analytical microservices and replay historical events from day 1!

---

## 3. CQRS (Command Query Responsibility Segregation)

CQRS strictly separates the data model for **Commands** (state-mutating operations: Create, Update, Delete) from **Queries** (read-only views).

```mermaid
flowchart TD
    Client["Client / User Interface"] -->|1. Write Command: `PlaceOrder`| CommandAPI["Command API"]
    Client -->|4. Read Query: `GetOrderDetails`| QueryAPI["Query API"]

    CommandAPI -->|2. Writes to Write DB| WriteDB[("Write Database\n(Normalized SQL / Event Store)\nOptimized for ACID Transactions")]

    WriteDB -.->|3. Async Event Replication / CDC| SyncEngine["Sync Engine / Kafka"]
    SyncEngine -.->|Updates Read View| ReadDB[("Read Database\n(Elasticsearch / MongoDB / Redis)\nOptimized for Fast Denormalized Queries")]

    QueryAPI -->|5. Fast Read Lookup| ReadDB
```

---

## 4. Architectural Comparison Matrix

| Architectural Pattern | Core Focus | Primary Benefit | Trade-off / Complexity |
|---|---|---|---|
| **Pub-Sub (Publish-Subscribe)** | Decoupled Async Messaging. | High extensibility, zero service coupling. | Eventual consistency, asynchronous debugging. |
| **Event Sourcing** | Immutable Event Log State. | Complete audit trail, time-travel state replay. | Complex schema evolution, requires Snapshots for long event streams. |
| **CQRS** | Segregated Read vs. Write Models. | Independent scaling of Reads vs Writes. | Eventual consistency lag between Write DB and Read DB. |

---

## 5. Failure Modes and Trade-offs

1. **Event Sourcing Infinite Event Replay Latency** — An account has 10,000,000 historical transaction events over 5 years. Reconstructing current account state takes 45 seconds of reading disk events! *Mitigation*: Periodically generate **State Snapshots** (e.g. every 1,000 events); load nearest Snapshot and replay only remaining recent events.
2. **CQRS Read-Side Eventual Consistency Lag** — A user submits an order via Command API. The browser immediately queries Query API, but the async sync worker hasn't updated the Read DB yet. The user sees an empty order history! *Mitigation*: Return newly created event payload directly in the Command response or use WebSocket pushes.
3. **Out-of-Order Event Processing in Event-Driven Systems** — `OrderCancelled` event arrives at Inventory Service BEFORE `OrderPlaced` event due to partition misconfiguration. *Mitigation*: Enforce Partition Keys (`order_id`) in Kafka or include Event Sequence Numbers.

---

## 6. Active-Recall Prompts

1. **How does Event-Driven Architecture (EDA) decouple microservices compared to synchronous REST API chains?**
2. **What is Event Sourcing, and how do State Snapshots prevent long event replay delays?**
3. **Explain CQRS (Command Query Responsibility Segregation) and why it separates Write DBs from Read DBs.**
4. **How do you handle eventual consistency lag when a user reads immediately after issuing a CQRS command?**

---

## Related Notes

- [[Message Queues vs Event Streams - RabbitMQ vs Apache Kafka]]
- [[Distributed Transactions - 2PC, Saga Pattern, and Outbox Pattern]]
- [[Observer Pattern - Event Buses, Publish-Subscribe, and Reactive Streams]]
- [[Command Pattern - Encapsulating Requests, Undo-Redo, and Macro Commands]]

> **Interview Style Question:** *"Design a scalable banking ledger system (like Stripe) using Event-Driven Architecture, Event Sourcing, and CQRS. Show how events are published to Kafka, design the Event Store schema, explain how State Snapshots optimize account balance recalculations, and solve CQRS eventual consistency read lag."*

---
