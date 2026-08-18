---
title: "Design Patterns in LLD - Practical Application Guide"
subject: "Low Level Design"
module: "LLD Foundations & OOAD"
difficulty: "Advanced"
prerequisites: "[[Object-Oriented Analysis and Design - Requirement Gathering, Use Cases, and Class Diagrams]], [[Machine Coding Interview Framework - 45-Minute Execution Blueprint]]"
related: "[[LLD - Parking Lot System]], [[LLD - Elevator Control System]], [[LLD - Vending Machine System]]"
aliases: ["Design Patterns in LLD", "LLD Pattern Mapping", "Practical Patterns", "Design Pattern Guide"]
tags: ["lld", "design-patterns", "machine-coding", "system-design", "clean-architecture"]
status: "Complete"
---

# Design Patterns in LLD — Practical Application Guide

## Mental Model

Think of **Design Patterns in LLD** as a professional master craftsman's specialized toolbelt (a hammer, torque wrench, electrical tester, and soldering iron). 

A novice builder attempts to solve every construction problem using a single sledgehammer (**Monolithic Procedural Code**). They use `if/else` checks for pricing, hardcode thread creation, and tightly couple database queries directly inside UI code. 

A master software engineer selects the exact pattern for the job: applying **Strategy** for dynamic pricing algorithms, **State** for order lifecycles, **Factory** for object creation, **Observer** for event notifications, and **Decorator** for stream processing. Knowing *when*, *why*, and *how* to combine patterns under interview pressure is what separates junior coders from Staff Engineers.

---

## 1. Top 10 Design Patterns for LLD Machine Coding

```mermaid
flowchart TD
    subgraph PatternToolbelt["The 10 High-Frequency Machine Coding Patterns"]
        Strategy["1. Strategy Pattern\n(Pluggable Algorithms: Pricing, Matching, Eviction, Sorting)"]
        State["2. State Pattern\n(Lifecycle FSM: Order, Vending Machine, ATM, Elevator)"]
        Factory["3. Factory Method / Abstract Factory\n(Object Creation: Payment Gateways, Notification Handlers)"]
        Observer["4. Observer Pattern\n(Event Notifications: Order Status Updates, Stock Ticker)"]
        Decorator["5. Decorator Pattern\n(Dynamic Wrapping: Pizza Toppings, Custom Coffee, Middleware)"]
        Command["6. Command Pattern\n(Undo/Redo & Task Queues: Text Editor, Job Queue)"]
        Singleton["7. Singleton / Enum Singleton\n(Global Registry: Connection Pool, Configuration Engine)"]
        Builder["8. Builder Pattern\n(Complex Objects: HTTP Requests, User Profiles, Game Entities)"]
        Composite["9. Composite Pattern\n(Tree Hierarchies: File System, Org Chart, DOM Tree)"]
        Adapter["10. Adapter Pattern\n(Third-Party Integration: Legacy Payment SDK to System Gateway)"]
    end
```

---

## 2. Comprehensive LLD Problem to Pattern Mapping Matrix

When presented with a specific LLD problem, immediately identify the governing design patterns:

| LLD System Design Problem | Primary Pattern(s) Required | Secondary Pattern(s) | Architectural Purpose |
|---|---|---|---|
| **Parking Lot System** | **Strategy Pattern** | Factory Method, Singleton | Dynamic Spot Allocation (`NearestSpot`, `FCFS`) & Hourly Pricing. |
| **Elevator Control System** | **State Pattern** & **Strategy** | Observer | Elevator States (`IDLE`, `MOVING_UP`, `MOVING_DOWN`) & Dispatching Algorithms (`SCAN`, `LOOK`). |
| **Vending Machine System** | **State Pattern** | Chain of Responsibility | Vending States (`NoCoin`, `HasCoin`, `Sold`, `Dispensing`) & Money Validation. |
| **Automated Teller Machine (ATM)** | **State Pattern** & **Chain of Resp.** | Command | ATM States (`NoCard`, `HasPin`) & Cash Dispensing Chain ($100 \to $50 \to $20 notes). |
| **In-Memory Cache (LRU/LFU)** | **Strategy Pattern** | Proxy, Factory | Pluggable Eviction Policies (`LRUEviction`, `LFUEviction`) & Cache Decorators. |
| **Rate Limiter System** | **Strategy Pattern** | Singleton | Rate Limiting Algorithms (`TokenBucket`, `LeakyBucket`, `SlidingWindow`). |
| **Movie Ticket Booking (BookMyShow)** | **State Pattern** & **Observer** | Strategy, Lock Manager | Seat States (`AVAILABLE`, `BLOCKED`, `BOOKED`) & Event Notifications. |
| **Pizza / Coffee Ordering System** | **Decorator Pattern** | Builder, Factory | Dynamic Topping Wrapping (`BasePizza + CheeseDecorator + MushroomDecorator`). |
| **Notification Gateway** | **Strategy Pattern** & **Factory** | Observer | Multi-channel Routing (`Email`, `SMS`, `Push`) & Vendor Fallback. |
| **Text Editor / Graphics Canvas** | **Command Pattern** & **Memento** | Composite | Undo/Redo Engine, State Snapshots, and Layer Component Trees. |

---

## 3. Practical Code Synergy: Combining Patterns Cleanly

In real-world LLD problems, design patterns do not exist in isolation; they **collaborate synergy-wise**.

### Example: Combining Factory + Strategy + Observer in a Notification Engine

```mermaid
flowchart TD
    App["Application Order Event"] -->|Publishes Event| EventBus["NotificationObserver (Observer Pattern)"]
    EventBus -->|Requests Channel| Factory["NotificationFactory (Factory Method Pattern)"]
    Factory -->|Instantiates| Strategy["EmailNotificationStrategy (Strategy Pattern)"]
    Strategy -->|Sends Payload| Vendor["SendGrid API"]
```

```java
// 1. STRATEGY INTERFACE
public interface NotificationStrategy {
    void sendNotification(String recipient, String message);
}

// 2. CONCRETE STRATEGIES
public class EmailStrategy implements NotificationStrategy {
    @Override public void sendNotification(String recipient, String msg) {
        System.out.println("Email Sent to " + recipient + ": " + msg);
    }
}

public class SmsStrategy implements NotificationStrategy {
    @Override public void sendNotification(String recipient, String msg) {
        System.out.println("SMS Sent to " + recipient + ": " + msg);
    }
}

// 3. FACTORY METHOD (Creates Strategies)
public class NotificationFactory {
    public static NotificationStrategy getStrategy(String channel) {
        if ("EMAIL".equalsIgnoreCase(channel)) return new EmailStrategy();
        if ("SMS".equalsIgnoreCase(channel)) return new SmsStrategy();
        throw new IllegalArgumentException("Unknown channel: " + channel);
    }
}

// 4. OBSERVER (Triggers Notification on Order Event)
public class OrderNotificationObserver {
    public void onOrderPlaced(String email, String phone, double amount) {
        // Factory + Strategy Synergy!
        NotificationStrategy emailSender = NotificationFactory.getStrategy("EMAIL");
        emailSender.sendNotification(email, "Your order of $" + amount + " is confirmed!");

        NotificationStrategy smsSender = NotificationFactory.getStrategy("SMS");
        smsSender.sendNotification(phone, "Order placed successfully!");
    }
}
```

---

## 4. Pattern Anti-Patterns: When NOT to Use Design Patterns

Applying design patterns unnecessarily is a major failure mode in machine coding interviews (**Patternitis / Over-Engineering**).

```mermaid
flowchart TD
    subgraph PatternitisWarnings["Anti-Patterns & Over-Engineering Hazards"]
        Warn1["1. Abstract Factory for a Single Product\nCreating an Abstract Factory interface when only ONE single product family will ever exist."]
        Warn2["2. Strategy Pattern for a Constant Formula\nCreating 5 Strategy classes for a simple calculation that never changes or varies."]
        Warn3["3. Singleton Overuse for Global Variables\nUsing Singleton as a dumping ground for mutable global state, destroying unit testability."]
        Warn4["4. Visitor Pattern for Dynamic Classes\nUsing Visitor when concrete element classes change daily, forcing constant interface updates."]
    end
```

---

## 5. Architectural Decision Flowchart: Selecting the Right Pattern

```mermaid
flowchart TD
    Start["What is the primary technical problem you need to solve?"] --> Q1{"Is it about Object Creation?"}
    
    Q1 -->|YES| Q1A{"Creating complex immutable object or family?"}
    Q1A -->|Complex Immutable| BuilderP["Use BUILDER Pattern"]
    Q1A -->|Product Family| AbstractFactoryP["Use ABSTRACT FACTORY Pattern"]
    Q1A -->|Subclass Decision| FactoryMethodP["Use FACTORY METHOD Pattern"]
    
    Q1 -->|NO| Q2{"Is it about Object Relationships / Wrappers?"}
    Q2 -->|YES| Q2A{"Interface mismatch, dynamic behavior, or access control?"}
    Q2A -->|Translate Incompatible Interface| AdapterP["Use ADAPTER Pattern"]
    Q2A -->|Dynamic Wrapper Chaining| DecoratorP["Use DECORATOR Pattern"]
    Q2A -->|Access Control / Lazy Load| ProxyP["Use PROXY Pattern"]
    Q2A -->|Tree Hierarchy| CompositeP["Use COMPOSITE Pattern"]
    
    Q2 -->|NO| Q3{"Is it about Behavior & State?"}
    Q3 -->|Swap Algorithm| StrategyP["Use STRATEGY Pattern"]
    Q3 -->|Lifecycle State Machine| StateP["Use STATE Pattern"]
    Q3 -->|Event Notification| ObserverP["Use OBSERVER Pattern"]
    Q3 -->|Undo/Redo & Queuing| CommandP["Use COMMAND Pattern"]
```

---

## 6. Active-Recall Prompts

1. **Which 3 design patterns are most frequently applied when building an Elevator System or Vending Machine?**
2. **What is the difference between how the Strategy Pattern and the State Pattern model class relationships?**
3. **Demonstrate how Factory Method, Strategy, and Observer collaborate cleanly in an enterprise Notification Engine.**
4. **What is "Patternitis", and how do you avoid over-engineering simple machine coding problems?**

---

## Related Notes

- [[Object-Oriented Analysis and Design - Requirement Gathering, Use Cases, and Class Diagrams]]
- [[Machine Coding Interview Framework - 45-Minute Execution Blueprint]]
- [[LLD - Parking Lot System]]
- [[LLD - Elevator Control System]]

> **Interview Style Question:** *"You are designing a high-throughput Ride-Sharing Platform (Uber/Lyft). Map the top 5 design patterns required across Driver Matching, Surge Pricing, Trip Lifecycle State Machine, Location Tracking Event Broadcasting, and Multi-Payment Gateway Integration. Write Java/TypeScript interface skeletons proving how these patterns collaborate cleanly."*

---
