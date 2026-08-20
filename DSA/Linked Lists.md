---
title: Linked Lists, Pointer Manipulation & Cycle Detection
aliases:
  - Linked Lists
  - Singly Linked List
  - Doubly Linked List
  - Floyd's Cycle Detection
  - Dummy Node
tags:
  - dsa
  - linear-data-structures
  - linked-list
status: complete
---

# Linked Lists, Pointer Manipulation & Cycle Detection

## TL;DR

A **Linked List** is a linear data structure composed of distinct **Node** objects allocated dynamically across the heap. Each node stores a data payload and one or more explicit pointer references (`next`, `prev`) to neighboring nodes. Unlike arrays, linked lists do not require contiguous memory blocks, allowing $O(1)$ insertions and deletions given a pointer to the location, but sacrificing $O(1)$ random indexing ($O(N)$ traversal required).

---

## 1. Node Structure & Memory Representation

```text
Array (Contiguous Memory):
[ 0x1000: Val 10 | 0x1004: Val 20 | 0x1008: Val 30 ]

Singly Linked List (Heap Nodes Scattered in Memory):
[ Address: 0x4080 ]      [ Address: 0x1020 ]      [ Address: 0x9040 ]
┌───────┬────────┐       ┌───────┬────────┐       ┌───────┬────────┐
│Val: 10│Next:───┼──────►│Val: 20│Next:───┼──────►│Val: 30│Next:Null│
└───────┴────────┘       └───────┴────────┘       └───────┴────────┘
```

---

## 2. Comparison: Linked List vs Array

| Operation / Feature | Array | Singly Linked List | Doubly Linked List |
|---|---|---|---|
| **Access by Index `i`** | $O(1)$ | $O(N)$ | $O(N)$ |
| **Search Element** | $O(N)$ ($O(\log N)$ if sorted) | $O(N)$ | $O(N)$ |
| **Insert / Delete at Head** | $O(N)$ (Shifting required) | **$O(1)$** | **$O(1)$** |
| **Insert / Delete at Tail** | $O(1)$ amortized | $O(N)$ ($O(1)$ with tail pointer) | **$O(1)$** (with tail pointer) |
| **Insert / Delete Given Node Pointer** | $O(N)$ | $O(1)$ (requires prev node) | **$O(1)$** |
| **Memory Overhead** | Low (Data only) | Medium (1 pointer per node) | High (2 pointers per node) |
| **Cache Locality** | **High** (Cache line hits) | **Low** (Cache misses via pointer chasing) | **Low** |

---

## 3. The Dummy Head Node Pattern

When modifying a linked list's head node (e.g., merging lists, removing elements, reversing segments), operations often require special-casing `if head is None` or `if head == target`.

The **Dummy Head Node** creates a persistent placeholder node before `head`. This unifies all node manipulations so that **every real node (including the head) has a valid preceding pointer**.

```text
Without Dummy Node (Special case needed for head):
if head.val == target:
    head = head.next  # Special branch

With Dummy Node (Unified logic):
dummy = ListNode(0)
dummy.next = head
curr = dummy
while curr.next:
    if curr.next.val == target:
        curr.next = curr.next.next  # Works identically for head node!
return dummy.next
```

---

## 4. Fundamental Algorithms & Pointer Techniques

### Pattern A: In-Place Linked List Reversal (Iterative & Recursive)

Given the `head` of a singly linked list, reverse the list in-place.

#### Pointer Snapshot Tracing

Maintain 3 pointers: `prev = None`, `curr = head`, `next_node = None`.

```text
Initial State:
prev = None,  curr = [1] -> [2] -> [3] -> NULL

Step 1:
- Save next_node = curr.next  ([2])
- Reverse pointer: curr.next = prev  (None <- [1])
- Advance prev = curr  ([1])
- Advance curr = next_node  ([2])

State after Step 1:
None <- [1] (prev)     [2] (curr) -> [3] -> NULL

Step 2:
None <- [1] <- [2] (prev)     [3] (curr) -> NULL

Step 3:
None <- [1] <- [2] <- [3] (prev)     NULL (curr)

Loop terminates when curr is NULL. New head is `prev`!
```

#### Iterative & Recursive Python Implementation

```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

def reverse_list_iterative(head: ListNode | None) -> ListNode | None:
    """
    Iterative Linked List Reversal.
    Time Complexity: O(N)
    Auxiliary Space: O(1)
    """
    prev = None
    curr = head
    
    while curr:
        next_node = curr.next  # 1. Save next
        curr.next = prev       # 2. Reverse pointer
        prev = curr            # 3. Advance prev
        curr = next_node       # 4. Advance curr
        
    return prev

def reverse_list_recursive(head: ListNode | None) -> ListNode | None:
    """
    Recursive Linked List Reversal.
    Time Complexity: O(N)
    Auxiliary Space: O(N) call stack
    """
    if not head or not head.next:
        return head
        
    new_head = reverse_list_recursive(head.next)
    head.next.next = head  # Make next node point back to head
    head.next = None       # Break forward link
    
    return new_head
```

---

### Pattern B: Fast & Slow Pointers (Floyd's Cycle Detection)

Given `head`, determine if the linked list has a cycle.

#### Mathematical Convergence Proof

Let `slow` move 1 step per iteration, and `fast` move 2 steps per iteration.
- Suppose the list has a non-cyclic prefix of length $K$, followed by a cycle of length $C$.
- After $K$ steps, `slow` enters the cycle.
- Inside a cycle of length $C$, the distance between `fast` and `slow` increases by 1 step per iteration.
- Equivalently, `fast` catches up to `slow` by closing the gap by 1 node per iteration.
- Since the gap shrinks by 1 modulo $C$ each step, `fast` is **guaranteed to meet `slow` in at most $C$ steps inside the cycle**.

```text
Cycle Detection Visual:
[1] -> [2] -> [3] -> [4]
               ▲      │
               └─ [5] ◄┘

Step 0: S=1, F=1
Step 1: S=2, F=3
Step 2: S=3, F=5
Step 3: S=4, F=4 (S == F -> Cycle Detected!)
```

#### Cycle Start Node Derivation

When `slow` and `fast` meet at node $M$:
- Distance traveled by `slow` = $K + d$
- Distance traveled by `fast` = $K + d + nC = 2(K + d)$
- Simplifying: $K + d = nC \implies K = nC - d$
- **Conclusion**: Placing one pointer at `head` and keeping `slow` at meeting point $M$, moving both 1 step at a time, they will **meet at the exact start of the cycle after $K$ steps**!

#### Python Implementation

```python
def has_cycle(head: ListNode | None) -> bool:
    """
    Floyd's Cycle Detection Algorithm.
    Time Complexity: O(N)
    Auxiliary Space: O(1)
    """
    slow = head
    fast = head
    
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            return True
            
    return False

def detect_cycle_start(head: ListNode | None) -> ListNode | None:
    """
    Find starting node of linked list cycle.
    Time Complexity: O(N)
    Auxiliary Space: O(1)
    """
    slow = fast = head
    has_cycle_flag = False
    
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            has_cycle_flag = True
            break
            
    if not has_cycle_flag:
        return None
        
    # Reset one pointer to head and move both at speed 1
    ptr1 = head
    ptr2 = slow
    while ptr1 != ptr2:
        ptr1 = ptr1.next
        ptr2 = ptr2.next
        
    return ptr1
```

---

## 5. Common Pitfalls & Edge Cases

1. **`NullPointerException` / `AttributeError`**: Attempting to access `curr.next` when `curr` is `None`. Always check `while curr:` or `while fast and fast.next:`.
2. **Losing Pointer References**: Reassigning `node.next` without first saving `next_node = node.next` breaks list connectivity permanently.
3. **Cycle Inversion in Reversal**: Forgetting to set `head.next = None` during recursive reversal creates infinite loops.

---

## 6. Practice Problems

### Foundation
- [LeetCode 206: Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/)
- [LeetCode 21: Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/)
- [LeetCode 876: Middle of the Linked List](https://leetcode.com/problems/middle-of-the-linked-list/)

### Intermediate
- [LeetCode 141: Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/)
- [LeetCode 142: Linked List Cycle II](https://leetcode.com/problems/linked-list-cycle-ii/)
- [LeetCode 19: Remove Nth Node From End of List](https://leetcode.com/problems/remove-nth-node-from-end-of-list/)
- [LeetCode 143: Reorder List](https://leetcode.com/problems/reorder-list/)

### Advanced
- [LeetCode 23: Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/) (Connects Linked Lists + Heap)
- [LeetCode 25: Reverse Nodes in k-Group](https://leetcode.com/problems/reverse-nodes-in-k-group/)

---

## Related Concepts
- [[00 - DSA MOC|Master DSA MOC]]
- [[Arrays]]
- [[Two Pointers]]
- [[Stacks]]
- [[Heap]]
