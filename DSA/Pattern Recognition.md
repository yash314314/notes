---
title: Pattern Recognition & Decision Framework
aliases:
  - Pattern Recognition
  - DSA Patterns
tags:
  - dsa
  - patterns
status: complete
---

# Pattern Recognition & Algorithmic Decision Framework

## TL;DR

When faced with an unfamiliar algorithmic problem, do not attempt to recall specific problem statements. Instead, decompose the **input data structure**, **constraints**, and **required outputs** to match the underlying mathematical and structural properties of known algorithmic patterns.

---

## 🗺️ Algorithmic Decision Tree

```text
                                [Problem Statement]
                                         │
                 ┌───────────────────────┴───────────────────────┐
         Is data linear?                                 Is data non-linear?
                 │                                               │
    ┌────────────┴────────────┐                     ┌────────────┴────────────┐
Array / String           List / Matrix            Trees                     Graphs
    │                         │                     │                         │
    ├─ Sorted?                ├─ Linked List        ├─ Hierarchical Search    ├─ Shortest Path
    │   ├─ Yes ──► [[Binary Search]] │  ├─ Fast/Slow Pointer  │   ├─ Level ──► [[BFS]] │  ├─ Unweighted ──► [[BFS]]
    │   │           [[Two Pointers]] │  └─ Reverse Pointer    │   └─ Path ──► [[DFS]]  │  ├─ Non-Negative ─► [[Dijkstra]]
    │   └─ No               └─ Grid / Matrix        ├─ Ancestor ──► [[LCA]]  │  └─ Negative ───► [[Bellman-Ford]]
    │       ├─ Subarray/Window  ├─ [[BFS]] (Shortest)  └─ Dynamic ──► [[Tree DP]]├─ Dependencies ──► [[Topological Sort]]
    │       │   ├─ Contiguous ─► [[Sliding Window]]                              └─ Connectivity ──► [[Disjoint Set Union]]
    │       │   └─ Sum Target ─► [[Prefix Sum]]
    │       ├─ Pair/Triplet ───► [[Hashing]] / [[Two Pointers]]
    │       └─ Order/Subsequence
    │           ├─ Nesting/Matching ──► [[Stack]] / [[Monotonic Stack]]
    │           └─ Optimal Substructure ──► [[Dynamic Programming]]
```

---

## 🔍 Signal-to-Pattern Mapping Matrix

| Input Clue / Constraint | Key Signal | Recommended Pattern | Primary Note |
|---|---|---|---|
| **Sorted Array** | Search target, find pair, range check | Two Pointers, Binary Search | [[Two Pointers]], [[Binary Search]] |
| **Contiguous Subarray / Substring** | Fixed or variable length range satisfying condition | Sliding Window, Prefix Sum | [[Sliding Window]], [[Prefix Sum]] |
| **Fast Frequency Lookup / Pair Counting** | $O(1)$ lookup needed, $A + B = K$ | Hash Map / Hash Set | [[Hashing]] |
| **Nested Matching / Monotonic Sequence** | Parentheses, next greater element, histogram | Stack, Monotonic Stack | [[Stacks]] |
| **Level-by-Level Traversal / Shortest Unweighted Path** | Minimum moves, nearest cell | Breadth-First Search (BFS) | [[BFS]] |
| **Tree Path / Component Exploration** | All paths, subtree properties, backtracking | Depth-First Search (DFS) | [[DFS]] |
| **Top $K$ Elements / Dynamic Median** | Frequently updating min/max | Priority Queue / Heap | [[Heap]] |
| **Overlapping Subproblems & Optimal Substructure** | Count total ways, min/max cost over choices | Dynamic Programming (DP) | [[Dynamic Programming]] |
| **Dynamic Connected Components** | Incremental connectivity, cycle detection | Disjoint Set Union (DSU) | [[Disjoint Set Union]] |
| **Prerequisites / Dependency DAG** | Directed graph with strict ordering | Topological Sort (Kahn's / DFS) | [[Topological Sort]] |
| **Range Queries with Point/Range Updates** | Prefix aggregate, range sum/min/max updates | Fenwick Tree / Segment Tree | [[Fenwick Tree]], [[Segment Tree]] |

---

## 💡 The Core Optimization Formula

When transforming a brute force approach into an optimal solution, evaluate these fundamental shifts:

```text
Repeated Scanning O(N)       ──►  Hash Lookup / Precomputation O(1)
Repeated Min/Max Search O(N) ──►  Heap / Monotonic Queue O(1) or O(log N)
Unsorted Search O(N)         ──►  Sorting + Binary Search O(log N)
Recomputed Subproblems O(2ⁿ) ──►  Memoization / Tabulation O(N · State)
Repeated Range Sum O(N)      ──►  Prefix Sum Array O(1)
```

---

## 🔗 Related References
- [[00 - DSA MOC|Master DSA MOC]]
- [[Complexity Analysis]]
