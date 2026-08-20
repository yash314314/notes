---
title: Topological Sort, Dependency Resolution & Kahn's Algorithm
aliases:
  - Topological Sort
  - Topological Ordering
  - Kahn's Algorithm
  - DAG
tags:
  - dsa
  - graphs
  - topological-sort
status: complete
---

# Topological Sort, Dependency Resolution & Kahn's Algorithm

## TL;DR

**Topological Sorting** produces a linear ordering of vertices in a **Directed Acyclic Graph (DAG)** such that for every directed edge $u \to v$, vertex $u$ appears **before** vertex $v$ in the ordering. If the graph contains even a single cycle, no valid topological ordering exists.

$$\forall \, (u \to v) \in E \implies \text{Position}(u) < \text{Position}(v)$$

---

## 1. What is Topological Sorting? Real-World Analogy

### Real-World Analogy: Course Prerequisites & Build Systems

```text
Course Prerequisites Graph (DAG):
  [CS101: Intro to CS] ──► [CS102: Data Structures] ──► [CS201: Algorithms]
           │                                                  ▲
           └──────────────► [CS105: Discrete Math] ───────────┘

Valid Topological Execution Order:
1. CS101 ──► 2. CS105 ──► 3. CS102 ──► 4. CS201
```

### Why Topological Sort FAILS on Cyclic Graphs
If CS101 requires CS201, and CS201 requires CS101 ($101 \rightleftarrows 201$), neither course can be taken first. Topological sorting serves as a fundamental **Cycle Detector for Directed Graphs**.

---

## 2. Two Classical Algorithms Compared

| Feature | Kahn's Algorithm (BFS In-Degree) | DFS Postorder Reversal |
|---|---|---|
| **Mechanism** | BFS with In-Degree array | Recursive DFS with Postorder Stack |
| **Data Structures**| Array `in_degree[V]` + FIFO `Queue` | `visited` array + `Stack` |
| **Cycle Detection**| Automatic (`len(topo_order) != V`) | Requires 3-color state tracking |
| **Time Complexity**| **$O(V + E)$** | **$O(V + E)$** |
| **Space Complexity**| **$O(V)$** | **$O(V)$** |

---

## 3. Algorithm Breakdown 1: Kahn's Algorithm (BFS In-Degree)

### Algorithm Steps
1. Calculate the **In-Degree** (number of incoming edges) for every vertex.
2. Enqueue all vertices with `in_degree == 0` into a FIFO Queue (these nodes have no prerequisites!).
3. While Queue is non-empty:
   - Dequeue vertex $u$ and append it to `topo_order`.
   - For each outgoing neighbor $v$ of $u$, decrement `in_degree[v]` by 1 (simulating prerequisite completion).
   - If `in_degree[v]` becomes `0`, enqueue $v$.
4. If `len(topo_order) == V`, return `topo_order`. Otherwise, the graph contains a **Cycle**!

```text
Kahn's Execution Trace:

Graph: (5) ──► (0) ◄── (4)
        │               │
        ▼               ▼
       (2) ──► (3) ──► (1)

Initial In-Degrees:
Node:      0   1   2   3   4   5
In-Degree: 2   1   1   1   0   0

Queue = [4, 5] (Nodes with in-degree 0)

Step 1: Pop 4 -> Order: [4]. Decrement in-degree of 0 (2->1), 1 (1->0). Queue = [5, 1]
Step 2: Pop 5 -> Order: [4, 5]. Decrement in-degree of 0 (1->0), 2 (1->0). Queue = [1, 0, 2]
Step 3: Pop 1 -> Order: [4, 5, 1]. Queue = [0, 2]
Step 4: Pop 0 -> Order: [4, 5, 1, 0]. Queue = [2]
Step 5: Pop 2 -> Order: [4, 5, 1, 0, 2]. Decrement in-degree of 3 (1->0). Queue = [3]
Step 6: Pop 3 -> Order: [4, 5, 1, 0, 2, 3].

Final Valid Topological Order: [4, 5, 1, 0, 2, 3]
```

#### Python Implementation (Kahn's Algorithm)

```python
from collections import deque

def topological_sort_kahn(num_nodes: int, edges: list[list[int]]) -> list[int]:
    """
    Kahn's Algorithm for Topological Sort & Cycle Detection.
    Time Complexity: O(V + E)
    Auxiliary Space: O(V)
    """
    # 1. Build Adjacency List and Calculate In-Degrees
    graph = {i: [] for i in range(num_nodes)}
    in_degree = [0] * num_nodes
    
    for u, v in edges:
        graph[u].append(v)
        in_degree[v] += 1
        
    # 2. Enqueue nodes with in-degree 0
    queue = deque([i for i in range(num_nodes) if in_degree[i] == 0])
    topo_order = []
    
    # 3. Process Queue
    while queue:
        u = queue.popleft()
        topo_order.append(u)
        
        for v in graph[u]:
            in_degree[v] -= 1
            if in_degree[v] == 0:
                queue.append(v)
                
    # 4. Cycle Detection Check
    if len(topo_order) != num_nodes:
        return [] # Cycle detected! No valid topological ordering.
        
    return topo_order
```

---

## 4. Algorithm Breakdown 2: DFS-Based Topological Sort

In a DFS traversal, a node $u$ finishes processing after all of its descendants have finished processing.
Therefore, popping nodes in **Reverse Postorder** (or pushing to a stack upon finishing) guarantees that $u$ appears before $v$ for all edges $u \to v$.

```python
def topological_sort_dfs(num_nodes: int, edges: list[list[int]]) -> list[int]:
    """
    DFS-based Topological Sort.
    Time Complexity: O(V + E)
    Auxiliary Space: O(V)
    """
    graph = {i: [] for i in range(num_nodes)}
    for u, v in edges:
        graph[u].append(v)
        
    visited = [0] * num_nodes # 0: Unvisited, 1: Visiting, 2: Visited
    stack = []
    has_cycle = False
    
    def dfs(u: int):
        nonlocal has_cycle
        if has_cycle:
            return
            
        visited[u] = 1 # Mark Visiting
        for v in graph[u]:
            if visited[v] == 1:
                has_cycle = True # Cycle detected!
                return
            if visited[v] == 0:
                dfs(v)
                
        visited[u] = 2 # Mark Visited
        stack.append(u) # Push to postorder stack

    for i in range(num_nodes):
        if visited[i] == 0:
            dfs(i)
            
    if has_cycle:
        return []
        
    return stack[::-1] # Reverse postorder stack
```

---

## 5. Common Pitfalls & Edge Cases

1. **Cycle Detection Failure**: Returning a partial list when a cycle exists causes broken orderings. Always check `if len(topo_order) != V`.
2. **Multiple Valid Topological Orders**: A DAG can have **multiple valid topological orderings**. Tests checking for an exact list representation should sort or validate dependencies!
3. **Graph with Isolated Nodes**: Nodes with `in_degree == 0` and no edges must still be enqueued initially, otherwise they are omitted.

---

## 6. Practice Problems

### Foundation
- [LeetCode 207: Course Schedule](https://leetcode.com/problems/course-schedule/) (Cycle Detection)
- [LeetCode 210: Course Schedule II](https://leetcode.com/problems/course-schedule-ii/) (Full Topo Sort)

### Intermediate
- [LeetCode 310: Minimum Height Trees](https://leetcode.com/problems/minimum-height-trees/) (Topological Pruning from Leaves)
- [LeetCode 802: Find Eventual Safe States](https://leetcode.com/problems/find-eventual-safe-states/)
- [LeetCode 269: Alien Dictionary](https://leetcode.com/problems/alien-dictionary/) (Graph Construction + Topo Sort)

### Advanced
- [LeetCode 1203: Sort Items by Groups Respecting Dependencies](https://leetcode.com/problems/sort-items-by-groups-respecting-dependencies/) (2-Level Dual Topo Sort)

---

## Related Concepts
- [[00 - DSA MOC|Master DSA MOC]]
- [[Graph Fundamentals]]
- [[BFS]]
- [[DFS]]
- [[Dynamic Programming]]
