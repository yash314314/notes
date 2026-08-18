---
title: Copy-on-Write - CoW
subject: Operating Systems
module: Memory Management & Virtual Memory
difficulty: Advanced
prerequisites:
  - "[[Operating System]]"
  - "[[Kernel]]"
  - "[[Process Creation and Termination - fork, exec, wait, exit]]"
  - "[[Paging Architecture]]"
  - "[[Demand Paging and Page Faults]]"
  - "[[Virtual Memory Architecture]]"
related:
  - "[[Page Tables and Multi-Level Page Tables]]"
  - "[[Page Replacement Algorithms - FIFO, LRU, Clock, Optimal]]"
  - "[[Working Set Model and Thrashing]]"
aliases:
  - Copy-on-Write
  - CoW
  - Copy on Write
  - do_wp_page
  - KSM
  - Kernel Samepage Merging
  - Fork Optimization
tags:
  - os
  - memory
  - linux-kernel
  - optimization
  - algorithms
status: complete
---

# Copy-on-Write (CoW)

> [!abstract] Mental Model
> Copy-on-Write is a **shared read-only Google Doc that only creates a private branch when you type a modification**:
> - In early UNIX, `fork()` physically cloned every byte of the parent process's memory into new DRAM chips. For a 16 GB database, `fork()` would stall the server for 200 ms and demand an extra 16 GB of RAM, only for the child to call `execve()` 5 microseconds later and discard the copied memory entirely.
> - With **Copy-on-Write (CoW)**, `fork()` takes **$< 10\ \mu\text{s}$**: the child simply receives a duplicate page table pointing to the **exact same physical DRAM frames as the parent**, with all shared pages marked **Read-Only**. Physical duplication only occurs if and when a process writes to a specific 4 KB page.

---

## The CoW Execution Lifecycle

```mermaid
sequenceDiagram
    autonumber
    participant Parent as Parent Process
    participant Child as Child Process (Spawned by fork)
    participant PTE as Page Table Entries (PTE)
    participant DRAM as Physical 4KB Page Frame
    participant Kernel as Linux Kernel (do_wp_page)

    Note over Parent,Child: 1. fork() executed: Child shares Parent DRAM frames!
    Parent->>PTE: 2. Kernel marks shared PTEs as READ-ONLY (R/W = 0)
    Note over DRAM: 3. atomic_inc(&page->_refcount) -> refcount = 2
    
    Note over Child: 4. Child attempts to write to variable: *ptr = 42;
    Child->>PTE: 5. MMU intercepts write on Read-Only page!
    PTE-->>Kernel: 6. Hardware Page Fault (#PF with Write bit set)
    
    Note over Kernel: 7. do_wp_page() checks refcount > 1
    Kernel->>DRAM: 8. Allocate NEW 4KB Physical Frame & Copy 4096 bytes
    Kernel->>PTE: 9. Point Child PTE to new frame, mark READ-WRITE (R/W = 1)
    Note over DRAM: 10. atomic_dec(&old_page->_refcount) -> refcount = 1
    
    Kernel->>Child: 11. IRET (Restart write instruction)
    Note over Child: 12. Child writes 42 to private frame without affecting Parent!
```

---

## Kernel Internal Mechanics: `do_wp_page()`

When a write-protection fault occurs on a CoW page, the Linux kernel executes `do_wp_page()` (`mm/memory.c`):

```mermaid
flowchart TD
    Fault["Hardware #PF (Protection Violation on Write)"] --> CheckRef{"page_count(page) == 1?"}
    
    CheckRef -->|YES (Sole Remaining Owner)| Reuse["Fast Path: No Memory Copy Needed!<br/>• Mark PTE as Read-Write (R/W = 1)<br/>• Execute INVLPG to flush TLB entry"]
    
    CheckRef -->|NO (Multiple References)| Clone["Slow Path: Allocate & Duplicate<br/>1. alloc_page(GFP_HIGHUSER_MOVABLE)<br/>2. cow_user_page(new_page, old_page) -> 4KB memcpy<br/>3. Set Child PTE = new_pfn (R/W = 1)<br/>4. atomic_dec(&old_page->_refcount)"]
    
    Reuse --> Resume["Resume CPU Execution"]
    Clone --> Resume
```

---

## Production Impact: The Redis Fork-Snapshot Architecture

Redis relies fundamentally on Copy-on-Write for non-blocking persistence (**RDB Background Save - `BGSAVE`** and **AOF Rewrite**):

```mermaid
flowchart LR
    subgraph ParentRedis ["Main Redis Thread (Port 6379)"]
        P_PTE["Parent Page Table"]
        ActiveWrites["Handles 100k writes/sec<br/>(Modifies < 2% of keys)"]
    end

    subgraph ChildRedis ["Background BGSAVE Child Process"]
        C_PTE["Child Page Table (Pointed to snapshot DRAM)"]
        DiskDump["Sequentially streams frozen memory to disk /var/lib/redis/dump.rdb"]
    end

    subgraph DRAM_Pool ["Physical DRAM Silicon Frames"]
        SharedClean["98% Shared Read-Only Frames (Zero memory overhead!)"]
        PrivateDirty["2% CoW Duplicated Frames (Created dynamically on write)"]
    end

    ParentRedis --- DRAM_Pool
    ChildRedis --- DRAM_Pool
```

> [!important] The Redis CoW Memory Trap
> If a workload performs high-frequency random writes across 100% of its database keys during a `BGSAVE`, Copy-on-Write will gradually duplicate *every single page frame*, causing memory usage to balloon by $2\times$ and risking immediate termination by the Linux **OOM Killer**.

---

## The Inverse of CoW: Kernel Samepage Merging (KSM)

While CoW splits pages on write, **Kernel Samepage Merging (KSM - `mm/ksm.c`)** performs the reverse operation:

```mermaid
flowchart TD
    KSM_Daemon["ksmd Kernel Daemon Scans Physical DRAM"] --> FindDuplicates["Detects identical 4 KB pages across unrelated VMs / Containers"]
    
    FindDuplicates --> Merge["Merges duplicate frames into ONE single Read-Only physical frame"]
    
    Merge --> MarkCoW["Marks Page Table Entries of both VMs as Read-Only Copy-on-Write!"]
    
    MarkCoW --> SavedRAM["Reclaims duplicate physical frames, saving 20-40% hypervisor RAM!"]
```

---

## Production Commands & CoW Monitoring

```bash
# 1. Inspect Copy-on-Write memory footprint for a process in /proc/PID/smaps
cat /proc/<PID>/smaps | grep -i "cow"
# Output:
# Shared_Clean:      102400 kB (Shared CoW memory untouched)
# Shared_Dirty:           0 kB
# Private_Dirty:       4096 kB (Memory written and duplicated via CoW)

# 2. Monitor Kernel Samepage Merging (KSM) deduplication metrics
cat /sys/kernel/mm/ksm/pages_sharing
# (Number of physical page frames saved by KSM deduplication)
```

---

## Active Recall & Interview Perspective

### Reasoning Prompts
1. *Why does Linux mark shared pages as Read-Only in BOTH the Parent and Child page tables during `fork()`?*
   - **Answer**: The kernel cannot predict whether the parent or the child will be the first process to perform a write operation. By marking the Page Table Entries of **both** processes as Read-Only (`R/W = 0`), the hardware MMU guarantees that whichever process attempts to write first will trigger a write-protection Page Fault (`#PF`). This trap gives the kernel control to allocate a new physical frame, duplicate the 4KB contents, update only the writing process's PTE to point to the new frame with write permissions (`R/W = 1`), and leave the other process referencing the original frame untouched.
2. *What is the difference between `fork()` and `vfork()` in systems programming?*
   - **Answer**: `fork()` creates a new process with an independent copy of the parent's page tables protected by Copy-on-Write; parent and child run concurrently in separate address spaces. `vfork()` was an early optimization created before hardware CoW was efficient: `vfork()` completely suspends the parent process while the child borrows the parent's exact address space and page tables without copying anything. The child is strictly forbidden from modifying memory or returning from the function, and must immediately call `execve()` or `_exit()`, after which the parent process resumes. Today, standard `fork()` with CoW or `posix_spawn()` is preferred.
3. *Why does a memory-heavy process experience a sudden spike in Minor Page Faults immediately following a `fork()` call?*
   - **Answer**: After a `fork()`, all writable heap and stack pages are marked Read-Only in both parent and child page tables. As the parent and child resume executing and write to their local variables or data structures, every single initial write to a distinct $4\text{ KB}$ page triggers a write-protection Page Fault. Because the page is already in physical DRAM, the kernel resolves these traps in microseconds via `do_wp_page()` without disk I/O, recording them as a large burst of **Minor Page Faults**.

---

## Key Takeaways
- **Copy-on-Write (CoW)** enables near-instantaneous `fork()` by sharing physical frames and deferring duplication until an actual write occurs.
- The kernel traps writes via **Read-Only (`R/W = 0`) PTEs**, handling duplication in **`do_wp_page()`** and tracking owners via `struct page._refcount`.
- CoW powers **Redis background snapshots** and the **Fork-Exec** pattern, while **KSM** performs reverse CoW memory deduplication.

---

## Related Notes
- [[Operating System]] — Memory subsystem fundamentals.
- [[Process Creation and Termination - fork, exec, wait, exit]] — `fork()` internals.
- [[Paging Architecture]] — PTE bitfield permissions.
- [[Demand Paging and Page Faults]] — Faulting mechanisms.
- [[Virtual Memory Architecture]] — Virtual memory structures.
- [[Page Replacement Algorithms - FIFO, LRU, Clock, Optimal]] — Page eviction policies.
- [[Operating Systems MOC]] — Master table of contents for Operating Systems.
